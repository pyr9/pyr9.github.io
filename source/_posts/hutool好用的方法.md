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

# 3. 常量

```java
public class StrUtil {
    public static final String SPACE = " ";
    public static final String DOT = ".";
    public static final String SLASH = "/";
    public static final String BACKSLASH = "\\";
    public static final String EMPTY = "";
    public static final String NULL = "null";
    public static final String CR = "\r";
    public static final String LF = "\n";
    public static final String CRLF = "\r\n";
    public static final String UNDERLINE = "_";
    public static final String DASHED = "-";
    public static final String COMMA = ",";
    public static final String DELIM_START = "{";
    public static final String DELIM_END = "}";
    public static final String BRACKET_START = "[";
    public static final String BRACKET_END = "]";
    public static final String COLON = ":";
    public static final String EMPTY_JSON = "{}";
```
