---
title: Java基于SSE流式返回
date: 2025-07-16 21:15:22
tags:
categories: Java基础
---

# 1. 服务器推送技术价值

传统"拉取"模式需客户端主动发起请求获取更新，而服务器推送采用"推送"机制实现实时信息传输。其核心优势体现在：

1. 实时性提升：系统自动推送最新内容，免去用户手动刷新操作
2. 资源优化：仅在产生有效数据时建立连接，减少无效请求
3. 用户体验升级：消息即时触达，适用于实时通信场景

# 2. 服务器推送主流实现技术对比

## 2.1. 长轮询（Long Polling）

- 客户端维持长连接等待服务器响应
- 兼容性优秀但资源消耗较高

## 2.2. 服务器发送事件（SSE）

- 基于HTTP协议
- 单向通信
- 适合只需要服务器向客户端推送实时更新数据的场景，如实时新闻更新、股票行情推送等

## 2.3. WebSocket

- 全双工TCP通信协议

- 双向通信

- 适合需要客户端和服务器之间进行实时双向通信的场景，如聊天室、在线游戏等

  

# 3. Java 基于okhttp请求SSE接口流式返回

## 3.1. 整体结构简述

整个流程理解为 **两个 SSE 连接的“接力”传输**：

### 3.1.1. 第一个 SSE 连接：

**前端发起， 后端给前端推数据**

- 使用 `SseEmitter` 实现。
- 前端通过 `EventSource` 发起请求。
- 后端接收请求并创建 `SseEmitter`，用于向前端推送数据。
- 这个连接是你主动控制的，用来与用户通信。

### 3.1.2. 第二个 SSE 连接：

**后端发起，AI 接口给后端推数据**

- 使用 OkHttp 的 `EventSource.Factory` 创建。
- 后端向 AI 模型服务发起流式请求。
- 接收模型返回的逐字输出（如 AI 流式回答）。
- 这个连接是代理性质的，用来获取远程 AI 数据。

```java
[前端] 
   ↓ (发起请求)
[后端 Controller]
   ↓ (构造请求体 + 创建 emitter)
[SSEEventSourceListener(listener)]
   ↓ (通过 OkHttp EventSource 发起 SSE 请求)
[调用远程 AI 接口（如 OpenAI / Ollama）]
   ↓ (流式返回数据)
[onEvent() 方法处理数据]
   ↓
[通过 emitter.send(data) 把数据推给前端]
```

## 3.2. 核心代码

1. 我们可以借助okhttp来实现，首先引入okhttp-sse的依赖：

```xml
        <dependency>
            <groupId>com.squareup.okhttp3</groupId>
            <artifactId>okhttp</artifactId>
            <version>4.10.0</version>
        </dependency>

        <dependency>
            <groupId>com.squareup.okhttp3</groupId>
            <artifactId>okhttp-sse</artifactId>
            <version>4.10.0</version>
        </dependency>
```

1. 核心代码

```java
OkHttpClient okHttpClient = new OkHttpClient
            .Builder()
            .addInterceptor(this.authInterceptor)
            .connectTimeout(10, TimeUnit.SECONDS)
            .writeTimeout(50, TimeUnit.SECONDS)
            .readTimeout(50, TimeUnit.SECONDS)
            .build();

 SSEEventSourceListener listener = new SSEEventSourceListener(emitter,chatRequest.getUserId(),chatRequest.getSessionId());

// 创建事件, 使用了OkHttp的EventSource（Server-Sent Events，简称SSE）机制来发送一个 POST 请求
EventSource.Factory factory = EventSources.createFactory(okHttpClient);
            ObjectMapper mapper = new ObjectMapper();
            String requestBody = mapper.writeValueAsString(chatCompletion);
            Request request = new Request.Builder()
                .url(this.apiHost)
                .post(RequestBody.create(MediaType.parse(ContentType.JSON.getValue()), requestBody))
                .build();
            factory.newEventSource(request, listener);
@Slf4j
@RequiredArgsConstructor
@Component
public class SSEEventSourceListener extends EventSourceListener {

    private SseEmitter emitter;

   
    @Override
    public void onOpen(EventSource eventSource, Response response) {
        log.info("OpenAI建立sse连接...");
    }

  
    @SneakyThrows
    @Override
    public void onEvent(@NotNull EventSource eventSource, String id, String type, String data) {
        try {
            if ("[DONE]".equals(data)) {
                //成功响应
                emitter.complete();
                return;
            }

            ObjectMapper mapper = new ObjectMapper();
            ChatCompletionResponse completionResponse = mapper.readValue(data, ChatCompletionResponse.class);
            if(completionResponse == null || CollectionUtil.isEmpty(completionResponse.getChoices())){
                return;
            }
            Object content = completionResponse.getChoices().get(0).getDelta().getContent();

            if(content != null ){
                // 给前端发送请求到的数据
                emitter.send(data);
            }
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
    }

    @Override
    public void onClosed(EventSource eventSource) {
        log.info("OpenAI关闭sse连接...");
    }

    @SneakyThrows
    @Override
    public void onFailure(EventSource eventSource, Throwable t, Response response) {
        if (Objects.isNull(response)) {
            return;
        }
        ResponseBody body = response.body();
        if (Objects.nonNull(body)) {
            log.error("OpenAI  sse连接异常data：{}，异常：{}", body.string(), t);
        } else {
            log.error("OpenAI  sse连接异常data：{}，异常：{}", response, t);
        }
        eventSource.cancel();
    }
}
```

# 4. **当前端取消请求时，通知后端也立即中断 AI 的处理流程**

## 4.1. 流程

前端取消请求时，通知后端取消对应的 AI 请求

- 前端使用 `useFetchHook`（假设是基于 `fetch` 或 `axios`）时，可以使用 `AbortController` 来中断请求。
- 前端中断请求后，主动发送一个取消请求到后端，告诉后端“我不要这个请求了，请取消 AI 推理”。

## 4.2. 前端实现（Vue + useFetchHook）

```typescript
import { ref } from 'vue'
import { useFetch } from '@vueuse/core'

export default {
  setup() {
    const controller = new AbortController()
    const signal = controller.signal

    const { data, error, isFetching } = useFetch('https://your-api.com/chat', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ prompt: 'Tell me a joke' }),
      signal
    }).json()

    const cancelRequest = () => {
      controller.abort() // 中断当前请求

      // 可选：发送一个取消请求给后端，通知它停止处理
      fetch('https://your-api.com/chat/cancel', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          userId: 'user123',
          sessionId: 'session456'
        })
      })
    }

    return {
      data,
      error,
      isFetching,
      cancelRequest
    }
  }
}
```

## 4.3. 后端实现

```java
@RestController
@RequestMapping("/chat")
public class ChatController {

    @Autowired
    private ChatSessionManager chatSessionManager;
  
    @PostMapping("/chat")
    public void chat(@RequestBody ChatRequest chatRequest, HttpServletRequest request) {
    String sessionId = chatRequest.getSessionId();
    String userId = chatRequest.getUserId();

    // 构造请求体
    ObjectMapper mapper = new ObjectMapper();
    String requestBody = mapper.writeValueAsString(chatCompletion);

    Request request = new Request.Builder()
    .url(this.apiHost)
    .post(RequestBody.create(MediaType.parse("application/json"), requestBody))
    .build();

    SSEEventSourceListener listener = new SSEEventSourceListener(emitter, userId, sessionId);
    EventSource eventSource = EventSources.createFactory(okHttpClient).newEventSource(request, listener);

    // 保存 eventSource 到 session manager
    chatSessionManager.addSession(sessionId, eventSource);
    }

    @PostMapping("/cancel")
    public ResponseEntity<String> cancelChat(@RequestBody Map<String, String> payload) {
        String sessionId = payload.get("sessionId");
        chatSessionManager.cancelSession(sessionId);
        return ResponseEntity.ok("Chat request canceled.");
    }
}
```

```java
@Component
public class ChatSessionManager {

    // session ID -> EventSource
    private final Map<String, EventSource> sessions = new ConcurrentHashMap<>();

    public void addSession(String sessionId, EventSource eventSource) {
        sessions.put(sessionId, eventSource);
    }

    public void removeSession(String sessionId) {
        sessions.remove(sessionId);
    }

    public void cancelSession(String sessionId) {
        EventSource eventSource = sessions.get(sessionId);
        if (eventSource != null) {
            eventSource.cancel(); // 取消 SSE 请求
            sessions.remove(sessionId);
        }
    }
}
```

> 注：如果使用 **Redis**，可以将 session 信息存入 Redis，实现分布式取消。
