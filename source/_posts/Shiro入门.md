---
title: Shiro入门
date: 2023-02-26 21:09:10
tags: Shiro
categories: Springboot
---

# 1. shiro 概念

- **Apache Shiro™**是一个强大且易用的Java安全框架,能够用于身份验证、授权、加密和会话管理
- Shiro拥有易于理解的API,您可以快速、轻松地获得任何应用程序——从最小的移动应用程序到最大的网络和企业应用程序。

# 2. shiro优缺点

优点： 

- 提供了一套框架，易于使用
- 更灵活，应对需求能力强，web能力强
- 可与很多框架进行集成

缺点：

- 操作的界面需要自己实现 

# 3 springboot集成shiro

1. 引入依赖

```xml
		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-core</artifactId>
			<version>1.2.3</version>
		</dependency>
		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-spring</artifactId>
			<version>1.2.3</version>
		</dependency>
```

2. 创建自定义AuthRealm和密码校验规则

```java
public class AuthRealm extends AuthorizingRealm {

    @Autowired
    private UserService userService;

    // 认证登录
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        UsernamePasswordToken usernamePasswordToken = (UsernamePasswordToken) token;
        String username = usernamePasswordToken.getUsername();
        User user = userService.findByUsername(username);
        return new SimpleAuthenticationInfo(user, user.getPassword(), this.getClass().getName());
    }

    // 授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        User user = (User) principals.fromRealm(this.getClass().getName()).iterator().next();
        List<String> permissionList = new ArrayList<>();
        List<String> roleNameList = new ArrayList<>();
        Set<Role> roleSet = user.getRoles();
        if (CollectionUtils.isNotEmpty(roleSet)) {
            for (Role role : roleSet) {
                roleNameList.add(role.getRname());
                Set<Permission> permissionSet = role.getPermissions();
                if (CollectionUtils.isNotEmpty(permissionSet)) {
                    for (Permission permission : permissionSet) {
                        permissionList.add(permission.getName());
                    }
                }
            }
        }
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        info.addStringPermissions(permissionList);
        info.addRoles(roleNameList);
        return info;
    }

}
```



```java
/**
 * 自己实现密码校验规则
 */
public class CredentialMatcher extends SimpleCredentialsMatcher {

    @Override
    public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
        UsernamePasswordToken usernamePasswordToken = (UsernamePasswordToken) token;
        String password = new String(usernamePasswordToken.getPassword());
        String dbPassword = (String) info.getCredentials();
        return this.equals(password, dbPassword);
    }
}

```



3. 添加配置类

```java
package com.pyr.shiro.demo1.config;

import com.pyr.shiro.demo1.shiro.AuthRealm;
import com.pyr.shiro.demo1.shiro.CredentialMatcher;
import org.apache.shiro.cache.MemoryConstrainedCacheManager;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.filter.mgt.DefaultFilter;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.LinkedHashMap;

@Configuration
public class ShiroConfiguration {

    //ShiroFilter过滤所有请求
    @Bean
    public ShiroFilterFactoryBean shiroFilter(@Qualifier("securityManager") SecurityManager manager) {
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        bean.setSecurityManager(manager);
        bean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return bean;
    }

    // 创建安全管理器
    @Bean("securityManager")
    public SecurityManager securityManager(@Qualifier("authRealm") AuthRealm authRealm) {
        DefaultWebSecurityManager manager = new DefaultWebSecurityManager();
        manager.setRealm(authRealm);
        return manager;
    }

    // 创建自定义Realm
    @Bean("authRealm")
    public AuthRealm authRealm(@Qualifier("credentialMatcher") CredentialMatcher matcher) {
        AuthRealm authRealm = new AuthRealm();
        authRealm.setCacheManager(new MemoryConstrainedCacheManager());
        authRealm.setCredentialsMatcher(matcher);
        return authRealm;
    }

    @Bean("credentialMatcher")
    public CredentialMatcher credentialMatcher() {
        return new CredentialMatcher();
    }

    /**
     * shiro和spring的配置
     * 设置spring使用的securityManager是我们自定义的securityManager
     */
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(@Qualifier("securityManager") SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor advisor = new AuthorizationAttributeSourceAdvisor();
        advisor.setSecurityManager(securityManager);
        return advisor;
    }

    @Bean
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator creator = new DefaultAdvisorAutoProxyCreator();
        creator.setProxyTargetClass(true);
        return creator;
    }
}

```

# 4. 使用

代码地址： https://github.com/pyr9/springboot-shiro-demo
