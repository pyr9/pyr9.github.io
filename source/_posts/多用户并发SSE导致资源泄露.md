---
title: 多用户并发SSE导致资源泄露解决方案
date: 2025-08-17 21:07:04
tags: 通信
categories: 实战问题复盘
---

# 1 场景

AI对话项目使用SSE通信，如果用户刷新 / 关闭页面，后端不清理连接，就可能导致 **资源泄露**。

# 2. 解决方案

## 1. **非零超时**+ **回调清理**（**必做**）

### 1. **非零超时**

- `SseEmitter` 默认 `0L` 表示永不超时，最容易导致僵尸连接。
- 设置合理的超时时间（比如 **30s~5min**），避免连接永远挂在服务器上。
- 超时会触发 `onTimeout` → 你就能清理 session。

```java
// 每个 SSE 连接最长维持 5 分钟
SseEmitter emitter = new SseEmitter(TimeUnit.MINUTES.toMillis(5));
```

### 2. **回调清理**

- - 注册 `onCompletion`、`onTimeout`、`onError`，确保**一旦连接断开就清理资源**。
  - 这样即使异常关闭，也能释放 `EventSource`。

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.NoArgsConstructor;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;
import okhttp3.sse.EventSource;
import org.jetbrains.annotations.NotNull;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.mvc.method.annotation.SseEmitter;

@Slf4j
@Component
@NoArgsConstructor
public class ChatSSEEventSourceListener extends BaseSSEEventSourceListener {

    private static final ChatSessionManager chatSessionManager = SpringUtils.getBean(ChatSessionManager.class);

    public ChatSSEEventSourceListener(SseEmitter emitter, Long userId, Long sessionId) {
        super(emitter, userId, sessionId);

        // 在构造函数里统一注册回调，确保前端关闭 / 刷新时也能清理
        emitter.onCompletion(() -> {
            log.info("SSE 完成，释放资源，sessionId={}", sessionId);
            chatSessionManager.cancelSession(sessionId);
        });
        emitter.onTimeout(() -> {
            log.warn("SSE 超时，释放资源，sessionId={}", sessionId);
            chatSessionManager.cancelSession(sessionId);
        });
        emitter.onError((ex) -> {
            log.error("SSE 出错，释放资源，sessionId={}", sessionId, ex);
            chatSessionManager.cancelSession(sessionId);
        });
    }
 }
```

### 3. 心跳机制

有些浏览器直接关闭页面时，TCP 连接不会立刻触发 `onError`，所以需要主动检测。因为SSE 的设计初衷是 “**服务端向客户端单向推送数据**”，所以**连接的生命周期由服务端主导**，也就是需要后端去发心跳

**后端：定时发心跳**

```java
ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();
scheduler.scheduleAtFixedRate(() -> {
    try {
        emitter.send(SseEmitter.event().comment("heartbeat"));
    } catch (IOException e) {
        log.warn("SSE 心跳失败，关闭 sessionId={}", sessionId);
        emitter.completeWithError(e);
    }
}, 15, 15, TimeUnit.SECONDS);
```

**前端：监听断开并自动重连**

```tsx
function connectSSE() {
  const evtSource = new EventSource("/sse/chat");
  evtSource.onopen = () => console.log("SSE connected");
  evtSource.onerror = () => {
    console.log("SSE error, reconnecting...");
    evtSource.close();
    setTimeout(connectSSE, 2000);
  };
}
connectSSE();
```

这样前端挂了，后端心跳失败，就能立即释放资源。

### 4. SessionManager 清理机制

你现在有个 `ChatSessionManager`，里面存着 `ConcurrentHashMap<Long, EventSource>`。
要避免泄露，可以：

- 在 `cancelSession` 里一定要 `remove`。
- 增加一个 **后台清理线程**（比如每 5 分钟扫一遍），把长时间没活跃的连接干掉。

示例：

```java
@Scheduled(fixedDelay = 300000) // 每 5 分钟执行一次
public void cleanupStaleSessions() {
    sessions.forEach((sessionId, eventSource) -> {
        // 可以维护一个 lastActiveTime，如果超过 N 分钟就清理
        // 简单起见直接 cancel()
        eventSource.cancel();
        sessions.remove(sessionId);
        log.info("清理过期 SSE 连接 sessionId={}", sessionId);
    });
}
```

### 5. 如果并发量特别大 → 考虑 WebSocket

- SSE 的线程模型是 **长轮询风格**，高并发下对 Tomcat/Jetty 压力较大。
- WebSocket 本身就支持双向心跳，连接状态更可控。
- 如果用户量很大（10w+），可以考虑转成 WebSocket，或者放在 Nginx + SSE 反向代理层做断线管理。
