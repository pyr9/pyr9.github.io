---
title: SpringSecurity
date: 2023-02-18 23:01:38
tags: SpringSecurity
categories: 权限管理
---

# 1. 权限管理

## 1.1. 为什么需要权限管理

- 安全性：误操作、认为破坏、数据泄漏等
- 数据隔离：不同的权限可以看到不同的数据
- 明确职责：销售、开发等不同角色，leader和dev不同级别，不同职责可以看到不同的数据

## 1.2. 理想的权限管理

- 能实现角色级权限
- 能实现功能级、数据级权限
- 简单、易操作、能够应对各种需求

# 2. SpringSecurity

## 2.1.  概念

-  Spring Security 是 Spring 家族中的一个**安全管理框架**。相比与另外一个安全框架Shiro，它提供了更丰富的功能，社区资源也比Shiro丰富。

- 一般来说中大型的项目都是使用SpringSecurity 来做安全框架。小项目有Shiro的比较多，因为相比与SpringSecurity，Shiro的上手更加的简单。

- 一般Web应用的需要进行认证和授权。而**认证和授权也是SpringSecurity作为安全框架的核心功能**。
  - 认证：验证当前访问系统的是不是本系统的用户，并且要确认具体是哪个用户
  - 授权：经过认证后判断当前用户是否有权限进行某个操作

 ## 2.2 优点和缺点

优点：

- 提供了很多用户认证的功能，实现相关接口即可，节省了大量开发工作
- 基于spring，易于集成

缺点：

- 配置文件多，角色被 “编码" 到配置文件和源文件中
- 对于系统中的用户、角色、权限之间的关系，没有可操作的界面。后台管理员没办法知道每个人的角色权限是什么样的，增大了管理的难度和分配权限的错误率。
- 由于配置多，没有可操作的界面，大数据量情况下，基本不可用。

## 2.3  springboot 整合 springSecurity

1. 添加依赖

```xml
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

2. 添加配置文件

```java
package com.pyr.security.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/").permitAll()
                .anyRequest().authenticated()
                .and()
                .logout().permitAll()
                .and()
                .formLogin();
        http.csrf().disable();
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/js/**", "/css/**", "/images/**");
    }
}
```

3. 测试

根据配置文件，可以得知，所有人可访问"/"，其他请求都需要登录。

```java
@RestController
public class HelloController {
    @RequestMapping("/")
    public String index(){
        return "index!";
    }

    @RequestMapping("/hello")
    public String hello(){
        return "hello springboot!";
    }
}
```

- 测试访问 "/" 路径

![image-20230226185258468](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230226185258468.png)

- 测试访问 "/hello" ，会302重定向到shiro框架，默认的login界面（依赖版本不同，login页面会有差异）

  ![image-20230226185524573](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230226185524573.png)

## 2.4 存储用户信息

### 2.4.1 直接存储在内存中

在SpringSecurityConfig，中重写configure方法，指定用户和角色：

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder()).withUser("admin").password(new BCryptPasswordEncoder().encode("111")).roles("USER");
  auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder()).withUser("pyr").password(new BCryptPasswordEncoder().encode("pyr")).roles("ADMIN");
}

```

### 2.4.2 存储在数据库中

1. 定义UserService实现UserDetailsService

```java
@Component
public class MyUserService implements UserDetailsService {
    @Resource
    private UserMapper userMapper;
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return  userMapper.findByUsername(username);
    }
}
```

## 2.5 使用角色控制方法的访问权限

1. 当ADMIN角色的用户访问/roleAuth时，正常访问。当其他角色用户，访问/roleAuth时，返回403

```java
    // RoleVoter 里定义了角色名，使用的时候，都必须使用ROLE_作为前缀
    @PreAuthorize("hasRole('ROLE_ADMIN')")
    @RequestMapping("/roleAuth")
    public String role() {
        return "admin Auth!";
    }
```

2. 当前登录的用户，有ADMIN或者MANAGER的角色

```java
    @PreAuthorize("hasRole('ROLE_ADMIN') or hasRole('ROLE_MANAGER')")
    @RequestMapping("/roleAuth2")
    public String role2() {
        return "admin Auth  222!";
    }
```

3. 传过来的id小于10 且 传过来的userName和数据库查出来的username相等

```java
    @PreAuthorize("#id< 10 and principal.username.equals(#username)")
    @RequestMapping("/roleAuth3")
    public String role3(Integer id, String username) {
        return "admin Auth  33!";
    }
```

代码地址：

https://github.com/pyr9/springboot-security-demo
