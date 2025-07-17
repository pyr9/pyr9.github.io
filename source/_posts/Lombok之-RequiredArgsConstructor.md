---
title: Lombok之@RequiredArgsConstructor
date: 2025-07-17 19:03:03
tags:
categories: Java基础
---

# 1. @RequiredArgsConstructor是什么?

`@RequiredArgsConstructor` 是 Lombok 提供的一个注解，用于自动生成构造函数。这个构造函数只包含那些被声明为 `final` 或者标注了 `@NonNull` 的字段，从而确保这些字段在对象创建时必须被初始化，以避免潜在的空指针异常等问题。

# 2. 通过@RequiredArgsConstructor简化Bean注入

通过类的构造函数来注入依赖。这是最推荐的方式，尤其是在依赖是强制性的情况下。

Spring 使用构造器注入的方式将 `OpenAiStreamClient` 类型的 Bean 注入到 `SseServiceImpl` 中。将可以简化@Autowired 的书写过程， 即不需要再通过@Autowired 或者@Resource 进行Bean注入

二、@RequiredArgsConstructor的使用方式

1、引入依赖

```xml
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
</dependency>
```

### 2、代码示例

#### 2.1、常规 @Autowired注入：

```java
@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/getUser")
    public User getUser(){
        User user = userService.getUser("1");
        return user;
    }
}
```



常规构造器注入

```java
@RestController
@RequestMapping("/user")
public class UserController {

    private UserService userService;
    
    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/getUser")
    public User getUser(){
        User user = userService.getUser("1");
        return user;
    }
}
```

#### 2.3、lombok中的RequiredArgsConstructor注入：

可以看到，使用 `@RequiredArgsConstructor` 省去了 `@Autowired` 或 `@Resource` 注解。

```java
@RestController
@RequestMapping("/user")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping("/getUser")
    public User getUser(){
        User user = userService.getUser("1");
        return user;
    }
}
```

参考：https://blog.csdn.net/q2qwert/article/details/140683375
