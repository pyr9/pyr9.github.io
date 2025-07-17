---
title: hutool好用的方法
date: 2025-07-17 19:05:04
tags:
categories: Java基础
---

# 1. 动态获取bean对象

```java
    private static final ChatSessionManager chatSessionManager = SpringUtils.getBean(ChatSessionManager.class);
```

# 2. 生成UUID

```java
String uuid = IdUtil.simpleUUID();
```
