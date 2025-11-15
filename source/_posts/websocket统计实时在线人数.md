---
title: websocket统计实时在线人数(java&vue)
date: 2025-11-01 19:35:21
tags:
categories: 
---

# 1. 后端

## 1.1. 引入依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

## 1.2. 配置类  

```java
package org.jasolar.common.chat.config;

import cn.hutool.core.util.StrUtil;
import org.jasolar.common.chat.config.properties.WebSocketProperties;
import org.jasolar.common.chat.handler.PlusWebSocketHandler;
import org.jasolar.common.chat.interceptor.PlusWebSocketInterceptor;
import org.jasolar.common.chat.listener.WebSocketTopicListener;
import org.springframework.boot.autoconfigure.AutoConfiguration;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.server.HandshakeInterceptor;

/**
 * WebSocket 配置
 *
 */
@AutoConfiguration
@ConditionalOnProperty(value = "websocket.enabled", havingValue = "true")
@EnableConfigurationProperties(WebSocketProperties.class)
@EnableWebSocket
public class WebSocketConfig {

    @Bean
    public WebSocketConfigurer webSocketConfigurer(HandshakeInterceptor handshakeInterceptor,
                                                   WebSocketHandler webSocketHandler,
                                                   WebSocketProperties webSocketProperties) {
        // 如果WebSocket的路径为空，则设置默认路径为 "/websocket"
        if (StrUtil.isBlank(webSocketProperties.getPath())) {
            webSocketProperties.setPath("/websocket");
        }
        // 如果允许跨域访问的地址为空，则设置为 "*"，表示允许所有来源的跨域请求
        if (StrUtil.isBlank(webSocketProperties.getAllowedOrigins())) {
            webSocketProperties.setAllowedOrigins("*");
        }
        // 返回一个WebSocketConfigurer对象，用于配置WebSocket
        return registry -> registry
            // 添加WebSocket处理程序和拦截器到指定路径，设置允许的跨域来源
            .addHandler(webSocketHandler, webSocketProperties.getPath())
            .addInterceptors(handshakeInterceptor)
            .setAllowedOrigins(webSocketProperties.getAllowedOrigins());
    }

    @Bean
    public HandshakeInterceptor handshakeInterceptor() {
        return new PlusWebSocketInterceptor();
    }

    @Bean
    public WebSocketHandler webSocketHandler() {
        return new PlusWebSocketHandler();
    }

    @Bean
    public WebSocketTopicListener topicListener() {
        return new WebSocketTopicListener();
    }
}
```

## 1.3. 处理器 

```java
package org.jasolar.common.chat.handler;

import lombok.extern.slf4j.Slf4j;
import org.jasolar.common.chat.holder.WebSocketSessionHolder;
import org.jasolar.common.chat.utils.WebSocketUtils;
import org.springframework.web.socket.BinaryMessage;
import org.springframework.web.socket.CloseStatus;
import org.springframework.web.socket.PongMessage;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.AbstractWebSocketHandler;

/**
 * WebSocketHandler 实现类
 *
 */
@Slf4j
public class PlusWebSocketHandler extends AbstractWebSocketHandler {

    public static final String USER_ID = "userId=";

    /**
     * 连接成功后
     */
    @Override
    public void afterConnectionEstablished(WebSocketSession session) {
        String userId = getUserId(session);
        WebSocketSessionHolder.addSession(userId, session);
    }


    /**
     * 连接出错时
     *
     */
    @Override
    public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {
        log.error("[transport error] sessionId: {} , exception:{}", session.getId(), exception.getMessage());
    }

    /**
     * 连接关闭后
     *
     * @param session
     * @param status
     */
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) {
        WebSocketSessionHolder.removeSession(session.getId());
    }

    private String getUserId(WebSocketSession session) {
        String query = session.getUri().getQuery(); // userId=123
        if (query != null && query.startsWith(USER_ID)) {
            return query.substring(USER_ID.length());
        }
        return null;
    }
}
```

## 1.4. 发送消息工具类

```java
package org.jasolar.common.chat.utils;

import cn.hutool.core.collection.CollUtil;
import lombok.AccessLevel;
import lombok.NoArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.jasolar.common.chat.entity.dto.WebSocketMessageDto;
import org.jasolar.common.chat.holder.WebSocketSessionHolder;
import org.jasolar.common.redis.utils.RedisUtils;
import org.springframework.web.socket.PongMessage;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketMessage;
import org.springframework.web.socket.WebSocketSession;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.function.Consumer;

import static org.jasolar.common.chat.constant.WebSocketConstants.WEB_SOCKET_TOPIC;

/**
 * 工具类
 *
 */
@Slf4j
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public class WebSocketUtils {

    /**
     * 发送消息
     *
     * @param sessionKey session主键 一般为用户id
     * @param message    消息文本
     */
    public static void sendMessage(String sessionKey, String message) {
        WebSocketSession session = WebSocketSessionHolder.getSessions(sessionKey);
        sendMessage(session, message);
    }

    public static void sendMessage(WebSocketSession session, String message) {
        sendMessage(session, new TextMessage(message));
    }

    private static void sendMessage(WebSocketSession session, WebSocketMessage<?> message) {
        if (session == null || !session.isOpen()) {
            log.error("[send] session会话已经关闭");
        } else {
            try {
                // 获取当前会话中的用户
                session.sendMessage(message);
                log.info("已发送消息给客户端：" + session.getId());
            } catch (IOException e) {
                log.error("[send] session({}) 发送消息({}) 异常", session, message, e);
            }
        }
    }
}
```

## 1.5. 使用

```java
@Transactional(rollbackFor = Exception.class)
public int updateRole(SysRoleBo bo) {
    String message = "{\"type\":\"permission_update\"}";
    WebSocketUtils.sendMessage(baseChatRequest.getUserId().toString(), message);
}
```

# 2. 前端连接

## 2.1. 统一管理websocket

- 通过 `onmessage` 判断消息类型，分发给不同模块。

```java
import { defineStore } from 'pinia';
import { useUserStore } from '@/stores';

export const useWsStore = defineStore('ws', () => {
  const ws = ref<WebSocket>();
  const subscriptions = ref<WSSubscription[]>([]);
  const reconnectInterval = 5000; // 5秒重连

  const initWebSocket = () => {
    const userStore = useUserStore();
    const userId = userStore.userInfo?.userId;
    if (!userId)
      return;

    // VITE_API_URL 是http://localhost:6039
    ws.value = new WebSocket(`${import.meta.env.VITE_API_URL.replace('http', 'ws')}/resource/websocket?userId=${userId}`);

    ws.value.onopen = () => {
      console.log('WebSocket 已连接');
    };

    ws.value.onmessage = (event: MessageEvent) => {
      console.log('WebSocket 收到消息:', event.data);
      const data = JSON.parse(event.data);
      // 遍历所有订阅，根据 type 分发
      subscriptions.value?.forEach((sub) => {
        if (sub.type === data.type) {
          sub.handler(data.payload);
        }
      });
    };

    ws.value.onclose = () => {
      console.log('WebSocket 断开，5秒后尝试重连...');
      setTimeout(() => {
        initWebSocket();
      }, reconnectInterval);
    };

    ws.value.onerror = (err) => {
      console.error('WebSocket 错误:', err);
      ws.value?.close();
    };
  };

  /**
   * 注册消息订阅
   * @param type 消息类型
   * @param handler 消息回调
   */
  const subscribe = (type: string, handler: WSMessageHandler) => {
    subscriptions.value.push({ type, handler });
  };

  /**
   * 取消订阅
   */
  const unsubscribe = (type: string, handler?: WSMessageHandler) => {
    subscriptions.value = subscriptions.value.filter(sub =>
      sub.type !== type || (handler && sub.handler !== handler),
    );
  };

  return {
    ws,
    initWebSocket,
    subscribe,
    unsubscribe,
  };
});

type WSMessageHandler = (payload: any) => void;

interface WSSubscription {
  type: string;
  handler: WSMessageHandler;
}
```

## 2.2. 全局初始化

```java
const wsStore = useWsStore();
wsStore.initWebSocket();
```

## 2.3. 使用

```java
    const wsStore = useWsStore();
    wsStore.subscribe('permission_update', async () => {
      await authStore.loadUserPermissions(true); 
    });
```



# 3. 实战：统计实时在线人数

## 3.1. 核心思路

- 后端不要在 `afterConnectionEstablished` 立即发送消息
- 客户端在 WebSocket 打开后，发送一个“ready”消息通知服务端
- 后端收到“ready”后，再发送首次消息

不能后端一判断建立连接就立即广播在线人数，因为此时有可能 前端还没订阅完，此时还没来得及处理消息，会产生如下问题：

- **后端**：WebSocket 一建立连接就发送消息（如在线人数）
- **前端**：组件在 `setup` 或 `onMounted` 才注册订阅
- **结果**：首次消息到达时没有订阅处理 → 消息丢失

## 3.2. 后端

```java
import lombok.Getter;

@Getter
public enum WebSocketMessageType {
    READY("ready"),
    ONLINE_COUNT("online_count")
    ;

    private final String type;
    WebSocketMessageType(String type) { this.type = type; }
}
```



```java
@Slf4j
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public class WebSocketUtils {
        /**
     * 广播消息给所有在线用户
     */
    public static void broadcast(String message) {
        var sessions = WebSocketSessionHolder.getAllSessions();
        if (CollUtil.isEmpty(sessions)) {
            return;
        }
        for (WebSocketSession session : sessions) {
            sendMessage(session, message);
        }
    }
}
public class PlusWebSocketHandler extends AbstractWebSocketHandler {
    private final ObjectMapper mapper = new ObjectMapper();
    /**
     * 连接成功后
     */
    @Override
    public void afterConnectionEstablished(WebSocketSession session) {
        String userId = getUserId(session);
        WebSocketSessionHolder.addSession(userId, session);
    }

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        WebSocketMessagePayLoad<?> data = mapper.readValue(message.getPayload(), WebSocketMessagePayLoad.class);
        if (ObjectUtil.equals(data.getType(),WebSocketMessageType.READY.getType())) {
            broadcastOnlineCount();
            log.info("当前在线人数：{}", WebSocketSessionHolder.getSessionsSize());
        }
    }
     /**
     * 连接关闭后
     *
     * @param session
     * @param status
     */
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) {
        WebSocketSessionHolder.removeSession(session.getId());
        log.info("用户断开，当前在线人数：{}", WebSocketSessionHolder.getSessionsSize());
        broadcastOnlineCount();
    }

    private void broadcastOnlineCount() {
        try {
            WebSocketMessagePayLoad<Object> message = WebSocketMessagePayLoad.builder()
                    .type(WebSocketMessageType.ONLINE_COUNT.getType())
                    .payload(WebSocketSessionHolder.getSessionsSize())
                    .build();
            String json = mapper.writeValueAsString(message);
            WebSocketUtils.broadcast(json);
        } catch (Exception e) {
            log.error("广播在线人数失败", e);
        }
    }
}
```



## 3.3. 前端

```vue
<script setup lang="ts">
  import { useOnlineStore } from '@/stores/modules/online.ts';
  const { onlineUserCount } = useOnlineStore();
</script>
<template>
 <el-card shadow="never" class="h-full flex flex-col">
      <template #header>
        <div class="flex-x-between">
          <span class="text-gray mr-2">在线用户</span>
          <el-tag type="danger" size="small">
            实时
          </el-tag>
        </div>
      </template>
  
      <div class="flex-x-between mt-2 flex-1">
        <div class="flex-y-center">
          <span class="text-lg transition-all duration-300 hover:scale-110">
            {{ onlineUserCount }}
          </span>
          <span class="ml-2 text-xs text-[#67c23a]">
            <el-icon><Connection /></el-icon>
            已连接
          </span>
        </div>
        <div class="i-svg:people w-8 h-8 animate-[pulse_2s_infinite]" />
      </div>
   </template>
import { defineStore } from 'pinia';
import { ref } from 'vue';
import { useWsStore } from '@/stores/modules/websocket.ts';

export const useOnlineStore = defineStore('online', () => {
  const onlineUserCount = ref(0);
  const wsStore = useWsStore();

  const init = () => {
    wsStore.initWebSocket();
    wsStore.subscribe('online_count', (count) => {
      onlineUserCount.value = count;
    });
  };

  return {
    onlineUserCount,
    init,
  };
});
```



```typescript
    ws.value.onopen = () => {
      console.log('WebSocket 已连接');
      ws.value?.send(JSON.stringify({ type: 'ready' }));
    };
```



```typescript
// 初始化全局在线人数 WebSocket
const onlineStore = useOnlineStore();
const userStore = useUserStore();

watch(
  () => userStore.userInfo?.userId,
  (userId) => {
    if (userId) {
      onlineStore.init();
    }
  },
  { immediate: true },
);
```

# 4. Nginx配置

```shell
# HTTP server (用于重定向至HTTPS)
server {
    listen 80;
    server_name chat.mytest.com;
    
    client_max_body_size 100M;
    
    # 将所有HTTP请求重定向到HTTPS
    return 301 https://$host:$request_uri;
}

# HTTPS server
server {
    listen 443 ssl;
    server_name chat.mytest.com;

    client_max_body_size 100M;

    ssl_certificate /etc/nginx/sslcert/mytest.com.pem; 
    ssl_certificate_key /etc/nginx/sslcert/mytest.com.key; 

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";

    location / {
        proxy_pass http://localhost:8081; # 转发到Docker容器前端暴露的端口 jai-brain-front
        
        # 与后端建立连接的超时
        proxy_connect_timeout 300s;
        # 向后端发送请求的超时（上传文件等）
        proxy_send_timeout    300s;
        # 从后端读取响应的超时（最关键！）
        proxy_read_timeout    300s;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /test-api/resource/websocket {
        proxy_pass http://localhost:8081/test-api/resource/websocket;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
         proxy_set_header Connection "Upgrade";

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }

    location /api/translate/ {
    proxy_pass http://172.28.49.25:8026/api/translate/;  # 注意：proxy_pass 后面加 / 表示路径替换
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
   }
    error_log /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
}
```
