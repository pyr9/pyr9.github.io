---
title: Oauth2
date: 2021-09-26 22:51:27
tags: 鉴权
categories: Java基础

---

---

# oauth2.0

## 一. oauth2.0 产生

1. 传统方式：用户和第三方共享密码
   * 不安全。未来可能会持久访问资源，第三方存储用户密码不安全，因为他可以访问用户的所有资源
   * 改密码的话，第三方会失效

## 二. oauth2.0是什么？

> - oauth2.0 是一种授权方式，使第三方可以获得对用户资源的访问
>
> - 他的核心就是：向第三方应用颁发令牌
>
> - 不管哪一种授权方式，第三方应用申请令牌之前，都必须先到系统备案，说明自己的身份，然后会拿到两个身份识别码：客户端 ID（client ID）和客户端密钥（client secret）

## 三.OAuth 2.0 规定了四种获得令牌的流程

<!-- more -->

### 3.1 授权码模式

**第三方应用先申请一个授权码，然后再用该码获取令牌。**

1. 第一步，A 网站提供一个链接，用户点击后就会跳转到 B 网站，授权用户数据给 A 网站使用。如：

   ```javascript
   https://b.com/oauth/authorize?
     response_type=code&                        
     client_id=CLIENT_ID&
     redirect_uri=CALLBACK_URL&
     scope=read
   ```

   - response_type表示要求返回授权码
   - client_id让B网站知道是谁在请求
   - redirect_uri 是B网站接受或处理后跳转的URL
   - scope 表示授权的范围，这里是只读

2. 第二步，此时B网站会询问用户是否给予A网站授权，用户表示同意，这时B网站就会跳转到上一步`redirect_uri` 中的地址（也就是我们常说的callback地址），同时返回一个授权码，如下面的地址，其中，`code`参数就是授权码

```
https://a.com/callback?code=AUTHORIZATION_CODE
```

3. 第三步，A网站拿到授权码后，就可以根据该授权码，在后端向B网站请求**令牌**

```
https://b.com/oauth/token?
 client_id=CLIENT_ID&
 client_secret=CLIENT_SECRET&
 grant_type=authorization_code&
 code=AUTHORIZATION_CODE&
 redirect_uri=CALLBACK_URL
```

- `client_id`参数和`client_secret`参数用来让 B 确认 A 的身份（`client_secret`参数是保密的，因此只能在后端发请求）

- `grant_type`参数的值是`AUTHORIZATION_CODE`，表示采用的授权方式是授权码

- `code`参数是上一步拿到的授权码

- `redirect_uri`参数是令牌颁发后的回调网址。

  

4. 第四步，B 网站收到请求以后，就会颁发令牌。具体做法是向`redirect_uri`指定的网址，发送一段 JSON 数据。

   ```
   {    
     "access_token":"ACCESS_TOKEN",
     "token_type":"bearer",
     "expires_in":2592000,
     "refresh_token":"REFRESH_TOKEN",
     "scope":"read",
     "uid":100101,
     "info":{...}
   }
   ```

   上面 JSON 数据中，`access_token`字段就是令牌，A 网站在后端拿到了。

#### 3.2 隐藏式模式（适合没有后台的第三方）

第一步，A 网站提供一个链接，要求用户跳转到 B 网站，授权用户数据给 A 网站使用。

```
https://b.com/oauth/authorize?
  response_type=token&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```

第二步，用户跳转到 B 网站，登录后同意给予 A 网站授权。这时，B 网站就会跳回`redirect_uri`参数指定的跳转网址，并且把令牌作为 URL 参数，传给 A 网站。

```
https://a.com/callback#token=ACCESS_TOKEN
```

上面 URL 中，`token`参数就是令牌，A 网站因此直接在前端拿到令牌。

1. 点击链接，跳转至第三方
2. 第三方直接把令牌给客户端

注意，令牌的位置是 URL 锚点（fragment），而不是查询字符串（querystring），这是因为 OAuth 2.0 允许跳转网址是 HTTP 协议，因此存在"中间人攻击"的风险，而浏览器跳转时，锚点不会发到服务器，就减少了泄漏令牌的风险。

这种方式把令牌直接传给前端，是很不安全的。因此，只能用于一些安全要求不高的场景，并且令牌的有效期必须非常短，通常就是会话期间（session）有效，浏览器关掉，令牌就失效了。

#### 3.3 密码模式（风险极大，适合于用户极其信任第三方）

**如果你高度信任某个应用，RFC 6749 也允许用户把用户名和密码，直接告诉该应用。该应用就使用你的密码，申请令牌**

1. 第一步，A 网站要求用户提供 B 网站的用户名和密码。拿到以后，A 就直接向 B 请求令牌。、

```
https://oauth.b.com/token?
  grant_type=password&
  username=USERNAME&
  password=PASSWORD&
  client_id=CLIENT_ID
```

上面 URL 中，`grant_type`参数是授权方式，这里的`password`表示"密码式"，`username`和`password`是 B 的用户名和密码。

2. 第二步，B 网站验证身份通过后，直接给出令牌。注意，这时不需要跳转，而是把令牌放在 JSON 数据里面，作为 HTTP 回应，A 因此拿到令牌。

#### 3.4 凭证式模式（适合没有前端的第三方）

**命令行下请求令牌。**这种方式给出的令牌，是针对第三方应用的，而不是针对用户的，即有可能多个用户共享同一个令牌。

1. 第一步，A 应用在命令行向 B 发出请求。

```
https://oauth.b.com/token?
  grant_type=client_credentials&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET
```

* `grant_type`参数等于`client_credentials`表示采用凭证式

* `client_id`和`client_secret`用来让 B 确认 A 的身份。

2. 第二步，B 网站验证通过以后，直接返回令牌。

适合场景：向该平台所有用户发送消息提醒



## 四、令牌的使用

A 网站拿到令牌以后，就可以向 B 网站的 API 请求数据了。

此时，每个发到 API 的请求，都必须带有令牌。具体做法是在请求的头信息，加上一个`Authorization`字段，令牌就放在这个字段里面。

```
curl -H "Authorization: Bearer ACCESS_TOKEN" \
"https://api.b.com"
```

上面命令中，`ACCESS_TOKEN`就是拿到的令牌。



## 五、更新令牌

令牌的有效期到了，如果让用户重新走一遍上面的流程，再申请一个新的令牌，很可能体验不好，而且也没有必要。OAuth 2.0 允许用户自动更新令牌。

具体方法是，B 网站颁发令牌的时候，一次性颁发两个令牌，一个用于获取数据，另一个用于获取新的令牌（refresh token 字段）。令牌到期前，用户使用 refresh token 发一个请求，去更新令牌。

```
https://b.com/oauth/token?
  grant_type=refresh_token&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET&
  refresh_token=REFRESH_TOKEN
```



上面 URL 中，`grant_type`参数为`refresh_token`表示要求更新令牌，`client_id`参数和`client_secret`参数用于确认身份，`refresh_token`参数就是用于更新令牌的令牌。

B 网站验证通过以后，就会颁发新的令牌。
