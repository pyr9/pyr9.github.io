---
title: 双token认证
date: 2025-08-14 21:40:56
tags: 鉴权
categories: 实战问题复盘
---

# 1. 双token认证流程

## 1.1. 登录阶段

1. 用户输入账号 + 密码。
2. 服务端验证成功后，颁发：

- - **Access Token**（访问令牌，短期有效，典型：15分钟~1小时）
  - **Refresh Token**（刷新令牌，长期有效，典型：7天~90天）

1. 前端保存：

- - Access Token → 放在内存 / Vuex / Redux / Pinia / localStorage（要防止 XSS）。
  - Refresh Token → 优先选择：服务端数据库 + Redis + HttpOnly Cookie（防止被前端JS脚本获取）。

------

## 1.2. 请求接口

1. 前端每次请求时，在 `Authorization` Header 里带上：

```typescript
// 请求拦截器：加上 Access Token
api.interceptors.request.use((config) => {
  const token = getAccessToken();
  if (token) {
    config.headers.Authorization = `Bearer ${Access Token}`;
  }
  return config;
});
```

## 1.3. 使用Refresh Token签发新的 Access Token

服务端验证 Access Token 是否快要过期、快要过期的话，使用Refresh Token重新生成Access Token

- 如果合法 → 返回正常结果。
- 如果将要过期 → 签发新的 Access Token（我们这里的场景是使用clent_id, secret_key去调用第三方的/refresh 接口，签发新的Access Token）

```java
Key:   auth:token:access:{clientId}_{username}   
Value: { token: "xxx", expiresAt: 1727800000000 }

Key:   auth:token:refresh:{clientId}_{username}
Value: { token: "rt_xxx", expiresAt: "...", revoked: false }
```

------

## 1.4.  用户登出

1. 前端清除 Access Token + Refresh Token。
2. 服务端可选择在数据库/Redis 中标记该 Refresh Token 失效，确保不能再被使用。



# 2. 代码实现

## 2.1. 初始方案（生产踩坑）

生产出现接口调用，频繁报错401的问题（核心原因是访问token的过期时间太久+并发，令牌提前20分钟就过期，会在并发情况下出现token多次覆盖，接口使用旧token，导致鉴权失败）

### 2.1.1. 代码

```java
public class TokenServiceImpl implements TokenService 
    private static final Map<String, TokenVo> ToKEN_MAP = new ConcurrentHashMap<>()
        // 获取 token 主方法
        public String getJwtToken(String clientId, String username, boolean refresh) {
            String cacheKey = "Authorization_" + clientId + "_" + username;
            // 先从缓存里获取 Token
            TokenVo tokenVo = TOKEN_MAP.get("Authorization_" + clientId + username);
            
            if (tokenVo == null || refresh) {
                // 缓存不存在 → 从接口获取
                tokenVo = this.getTokenFromSaas(clientId, username);
                return tokenVo.getTokenString();
            } else {
                // 缓存存在 → 检查是否快过期
                long diff = tokenVo.getExpiresDate().getTime() - new Date().getTime();
                if (diff < 20 * 60 * 1000) {  // 剩余时间小于 20 分钟
                    // 刷新 Token
                    tokenVo = getRefreshTokenFromSaas(tokenVo, clientId, username);
                }
                // 返回 Token
                return tokenVo.getTokenString();
        }
}
```

### 2.1.2. 存在问题

#### 1. 缓存被多次覆盖（数据竞争）

```java
tokenVo = getRefreshTokenFromSaas(...);
TOKEN_MAP.put(key, tokenVo); // 多个线程同时 put
```

**现象：**

- 线程 A 刷新成功，`put` 新 token
- 线程 B 刷新成功（或失败），也 `put` 数据
- 结果：**后** `put` **的覆盖先** `put` **的**

**后果：**

- 缓存中的 token 可能不是最新的
- 如果线程 B 刷新失败，它可能把一个**过期的 token 写回去**
- 导致后续请求失败

数据不一致，状态混乱

------

#### 2. 资源浪费（多次调用 SaaS 刷新接口）

**现象：**

- 线程 A：发现 token 快过期 → 调用 `getRefreshTokenFromSaas()`
- 线程 B：也发现 token 快过期 → 也调用 `getRefreshTokenFromSaas()`
- 结果：**同一个** `refresh_token` **被用了两次**

**后果：**

- 即使 SaaS 允许 `refresh_token` 可重复使用
- 你也发起了 **2 次 HTTP 请求**
- 浪费带宽、连接池、CPU、日志容量
- 可能触发 SaaS 的 **限流（Rate Limit）**

#### 3. 访问令牌被吊销（如果它是一次性的）

**现象：**

- 很多 SaaS 系统（如 OAuth2 规范）要求 `refresh_token` **一次性使用**
- 第一次刷新后，旧 `refresh_token` 失效，返回新的 `refresh_token`
- 线程 A 成功刷新，拿到新 token
- 线程 B 使用**同一个旧** `refresh_token` 再次刷新 → **失败！401 Unauthorized**

**后果：**

- 线程 B 抛异常，可能导致业务失败
- 日志中出现大量 `Invalid refresh token` 错误
- 用户看到“系统错误”，体验极差

这是最典型的“并发刷新导致 token 失效”问题

#### 4. **生成多个有效的** `access_token`（多节点）

**现象：**

- 每个 Pod 刷新后，都拿到一个新的 `access_token`
- 导致系统中同时存在 **多个有效的 access_token**
- 结果：**有的 Pod 用新 token，有的用旧 token**

**后果：**

- 如果你要**撤销用户访问**（如用户登出），必须通知 SaaS 吊销所有 token
- 你不知道有多少个 token 在使用 → **无法有效管理**
- 安全审计困难

#### 5. 无法统一控制生命周期（多节点）

- 你想强制所有 Pod 使用新 token？ → 必须重启所有 Pod 或等待缓存过期
- 你想在用户登出时**立即失效 token**？ → 无法通知所有 Pod 清除本地缓存

**失去了对 token 的“全局控制权”**

## 2.2. 最终方案

### 2.2.1. 优化思路

#### 1. **集中缓存**

- 将 Token 存到 Redis 或其他分布式缓存中
- 所有节点统一读取和更新 Token
- 避免各节点刷新覆盖问题

#### 2. 分布式锁控制刷新

- 使用 Redis 分布式锁（Redisson、Spring RedisLock 等）
- 保证同一时间只有一个线程/节点在刷新 Token
- 其他请求阻塞等待刷新完成

#### 3. 增加重试机制

- 接口调用返回 401 时再刷新 Token
- 防止刷新失败或网络异常导致请求失败

### 2.2.2. 代码

```java
@Autowired
private RedissonClient redissonClient;

@Autowired
private RedisTemplate<String, TokenVo> redisTemplate;

private static final String TOKEN_KEY = "Authorization_%s_%s";
private static final String LOCK_KEY = "LOCK_TOKEN_%s_%s";

// 获取 Token
public String getToken(String clientId, String username) {
    String tokenKey = String.format(TOKEN_KEY, clientId, username);
    String lockKey = String.format(LOCK_KEY, clientId, username);

    TokenVo tokenVo = redisTemplate.opsForValue().get(tokenKey);
    long now = System.currentTimeMillis();

    if (tokenVo == null || tokenVo.getExpiresDate().getTime() - now < 20 * 60 * 1000) {
        // 使用分布式锁保证单点刷新
        RLock lock = redissonClient.getLock(lockKey);
        try {
            if (lock.tryLock(10, 5, TimeUnit.SECONDS)) { // 等待10秒，锁5秒过期
                tokenVo = redisTemplate.opsForValue().get(tokenKey); // 再次读取，防止重复刷新
                if (tokenVo == null || tokenVo.getExpiresDate().getTime() - now < 20 * 60 * 1000) {
                    try {
                        tokenVo = getRefreshTokenFromSaas(tokenVo, clientId, username);
                    } catch (Exception e) {
                        log.error("刷新 Token 失败，重新获取", e);
                        tokenVo = getTokenFromSaas(clientId, username);
                    }
                    // 更新 Redis
                    redisTemplate.opsForValue().set(tokenKey, tokenVo, 60, TimeUnit.MINUTES);
                }
            } else {
                // 如果获取锁失败，短等待后再取缓存
                Thread.sleep(50);
                tokenVo = redisTemplate.opsForValue().get(tokenKey);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
    return tokenVo.getTokenString();
}
```
