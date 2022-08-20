---
title: springCloud之集成Hystrix
date: 2022-08-20 14:34:54
tags:
- 分布式
- 服务熔断
categories: springCloud
---

# 服务熔断

## 什么是服务熔断？

- 服务熔断就是对服务的调用执行熔断，对于后续的请求，不再调用目标服务，而是直接返回一个合理的信息，比如说请求量太大，返回一个"此时服务过载"，从而可以快速释放资源

- 保护系统

  

## 为什么会有这种熔断机制的出现?

在微服务相互调用的时候可能会出现服务调用发生异常或调用超时,多此请求造成服务积压导致服务无法使用后接着级联导致调用此服务的消费者也出现异常或调用超时,最后导致整个系统的全部崩溃.这就是雪崩效应.

> [级联](https://so.csdn.net/so/search?q=级联&spm=1001.2101.3001.7020)(失败)雪崩：一个服务失败，导致整条链路的服务都失败的情形。



## 熔断的意义？

- 让系统更加稳定
- 减少性能损耗
  - 响应很简单，所以产生的性能很小。
  - 用户知道服务有问题了，就不再调用，避免一直重试，也可以减少性能损耗。
- 及时响应。不需要其他计算，直接就可以返回一个简单的响应。
- 阀值可定制，比如请求到达10000，就开启熔断器



## 熔断器的功能

- 异常处理。需要可以根据异常的类型，去处理异常
- 日志记录。记录失败的数目，才可以开启熔断
- 测试失败的操作。周期性的检查来测试服务是否健康。
- 手动复位。管理员可以强行关闭断路器，重置故障的计数器
- 并发。断路器可以支持大并发量
- 加速断路：检测出问题的速度要快，及时响应
- 重试失败请求：在服务可用时，可以安排重试



## 熔断和降级的区别

相同点：

- 目的一致。通过技术手段来保护系统
- 表现类似：让用户体验到某些服务暂时不可用
- 粒度相同：都是基于服务的

不同点

- 触发条件不同
  - 服务熔断：服务熔断由链路上某个服务故障引起的
  - 服务降级，整体负荷来考虑。比如：原本有一个系统可以承载100个请求，但是后面由于请求数降低，相应的服务数量也需要降低
- 管理目标层级不同
  - 服务熔断：服务熔断是一个框架层次的处理，
  - 服务降级：服务降级是业务层次的处理。降级一般是从外围服务开始



# hystrix

## hystrix是什么？

- Feign是Netflix开发的一款服务容错和保护组件



## SpringCloud 集成hystrix

1. 搭建Eureka Client并集成Feign ：[springCloud之Feign搭建 - 楼上有只喵 (panyurou.github.io)](https://panyurou.github.io/2022/08/18/springCloud之Feign搭建/)

2. 导入依赖

   ```java
   	implementation 'org.springframework.cloud:spring-cloud-starter-netflix-hystrix'
   ```

3. 启动类加上@EnableCircuitBreaker注解

   ```java
   package com.pyr.spring.cloud.weather;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
   import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
   import org.springframework.cloud.openfeign.EnableFeignClients;
   
   @SpringBootApplication
   @EnableDiscoveryClient
   @EnableFeignClients
   @EnableCircuitBreaker
   public class Application {
     public static void main(String[] args) {
       SpringApplication.run(Application.class, args);
     }
   }
   ```

4. properties修改配置

   ```xml
   server.port=8085
   spring.application.name: micro-weather-eureka-client-feign-hystrix
   eureka.client.serviceUrl.defaultZone: http://localhost:8761//eureka/
   feign.client.config.feignName.connectTimeout: 5000
   feign.client.config.feignName.readTimeout: 5000
   
   # 开启feign.hystrix
   feign.hystrix.enabled=true
   ```

5. 修改CityClient

   ```java
   @FeignClient(name="micro-weather-city-eureka", fallback = CityClientFallBack.class)
   public interface CityClient {
     @RequestMapping("/cities")
     String listCity();
   }
   ```

6.  定义CityClientFallBack实现CityClient

   ```java
   package com.pyr.spring.cloud.weather.service;
   
   import org.springframework.stereotype.Component;
   
   @Component
   public class CityClientFallBack implements CityClient{
     @Override
     public String listCity() {
       return "City is empty!";
     }
   }
   
   ```

7. 访问[localhost:8085/cities](http://localhost:8085/cities)

   <img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h5dkaubuqbj20iw07yjro.jpg" style="zoom:50%;" />

**另一种实现方式**

1. 修改业务类，使用@HystrixCommand(fallbackMethod=XXX)，指明熔断时调用的方法

   ```java
   package com.pyr.spring.cloud.weather.controller;
   
   import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
   import com.pyr.spring.cloud.weather.service.CityClient;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   public class CityController {
     @Autowired
     private CityClient cityClient;
   
     @RequestMapping("/cities")
     @HystrixCommand(fallbackMethod="defaultCities")
     public String listCity(){
       return cityClient.listCity();
     }
   
     public String defaultCities(){
       return "Micro weather city eureka is down!";
     }
   }
   ```

- CityClient不需要修改

  ```java
  package com.pyr.spring.cloud.weather.service;
  
  import org.springframework.cloud.openfeign.FeignClient;
  import org.springframework.web.bind.annotation.RequestMapping;
  
  @FeignClient(name="micro-weather-city-eureka")
  public interface CityClient {
    @RequestMapping("/cities")
    String listCity();
  }
  ```

5. 停止micro-weather-city-eureka服务，访问[localhost:8085/cities](http://localhost:8085/cities)

   <img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h5dbm2suxzj20lm08c74q.jpg" style="zoom:50%;" />

注：这种方式不需要添加

```java
feign.hystrix.enabled=true
```

