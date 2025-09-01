---
title: "@Async和@Scheduled线程池统一配置"
date: 2025-08-28 21:18:28
tags: []
categories: Java基础
---

# 1. 配置方式

## 1.1. 配置线程池

- Spring 的 @Async 注解会自动查找名为 taskExecutor 或 executor 的 Executor 类型 Bean，但是从 Spring Boot 2.7 开始，Spring Boot 默认自动配置一个名为 taskExecutor 的 ThreadPoolTaskExecutor Bean，所以还需要使用AsyncConfigurer， 指定使用的线程池
- Spring 的 @Scheduled 注解会自动使用你定义的 ScheduledExecutorService 或 TaskScheduler Bean 来执行 @Scheduled 任务

```java
import org.apache.commons.lang3.concurrent.BasicThreadFactory;
import org.jasolar.common.core.config.properties.ThreadPoolProperties;
import org.jasolar.common.core.utils.Threads;
import org.springframework.boot.autoconfigure.AutoConfiguration;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.ScheduledThreadPoolExecutor;
import java.util.concurrent.ThreadPoolExecutor;

@AutoConfiguration
@EnableConfigurationProperties(ThreadPoolProperties.class)
public class ThreadPoolConfig {

    /**
     * 核心线程数 = cpu 核心数 + 1
     */
    private final int core = Runtime.getRuntime().availableProcessors() + 1;

      /**
     * 专门给 @Async 用的线程池
     */
    @Bean(name = "threadPoolTaskExecutor")
    @ConditionalOnProperty(prefix = "thread-pool", name = "enabled", havingValue = "true")
    public ThreadPoolTaskExecutor threadPoolTaskExecutor(ThreadPoolProperties threadPoolProperties) {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setThreadNamePrefix("async-pool-");  // <<< 线程名标识
        executor.setCorePoolSize(core);
        executor.setMaxPoolSize(core * 2);
        executor.setQueueCapacity(threadPoolProperties.getQueueCapacity());
        executor.setKeepAliveSeconds(threadPoolProperties.getKeepAliveSeconds());
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        return executor;
    }

    /**
     * 专门给 @Scheduled 定时任务用的线程池
     */
    @Bean(name = "scheduledExecutorService")
    protected ScheduledExecutorService scheduledExecutorService() {
        return new ScheduledThreadPoolExecutor(core,
            new BasicThreadFactory.Builder().namingPattern("schedule-pool-%d").daemon(true).build(),
            new ThreadPoolExecutor.CallerRunsPolicy()) {
            @Override
            protected void afterExecute(Runnable r, Throwable t) {
                super.afterExecute(r, t);
                Threads.printException(r, t);
            }
        };
    }


}
```

## 1.2. 异步配置

只是为了@async可以使用指定线程池

```java
package org.jasolar.common.core.config;

import cn.hutool.core.util.ArrayUtil;
import org.jasolar.common.core.exception.ServiceException;
import org.springframework.aop.interceptor.AsyncUncaughtExceptionHandler;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.AutoConfiguration;
import org.springframework.scheduling.annotation.AsyncConfigurer;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.Arrays;
import java.util.concurrent.Executor;
import java.util.concurrent.ScheduledExecutorService;

/**
 * 异步配置
 *
 * @author Lion Li
 */
@EnableAsync(proxyTargetClass = true)
@AutoConfiguration
public class AsyncConfig implements AsyncConfigurer {

    @Autowired
    @Qualifier("threadPoolTaskExecutor")
    private ThreadPoolTaskExecutor threadPoolTaskExecutor;

    /**
     * 自定义 @Async 注解使用系统线程池
     */
    @Override
    public Executor getAsyncExecutor() {
        return threadPoolTaskExecutor;
    }

    /**
     * 异步执行异常处理
     */
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (throwable, method, objects) -> {
            throwable.printStackTrace();
            StringBuilder sb = new StringBuilder();
            sb.append("Exception message - ").append(throwable.getMessage())
                .append(", Method name - ").append(method.getName());
            if (ArrayUtil.isNotEmpty(objects)) {
                sb.append(", Parameter value - ").append(Arrays.toString(objects));
            }
            throw new ServiceException(sb.toString());
        };
    }

}
```

## 1.3. 使用

### 1.3.1. 使用@Async

```java
@Async
@Override
public void saveLog(ApiCallHttpRequest request, Object responseData) {
    ApiCallLog log = getApiCallLog(request, responseData);
    log.setResponseCode(HttpStatus.OK.value());
    System.out.println("[当前线程] " + Thread.currentThread().getName());
    baseMapper.insert(log);
}
```

打印线程名：

[当前线程] schedule-pool-2

### 1.3.2. 使用@Scheduled

```java
@Scheduled(fixedRate = 5000)
public void doSomethingScheduled() {
    log.info("执行定时任务，当前线程: {}", Thread.currentThread().getName());
}
```

打印线程名：

执行定时任务，当前线程: schedule-pool-1

# 2. 为什么不能直接使用@ Async

1. **默认线程池很小**

- - 如果你没配置 `Executor`，Spring 会用一个名字叫 `SimpleAsyncTaskExecutor` 的执行器。
  - 它不是一个真正的线程池，而是**每次调用就新建一个线程**，没有复用。
  - 并发高的时候，可能瞬间创建几百上千个线程 → 内存和 CPU 被拖垮。

1. **不可控**

- - 默认的线程策略你没法设置，比如最大线程数、队列大小、拒绝策略等。
  - 如果调用量大，可能会 OOM 或者让系统变卡。

1. **日志和排查困难**

- - 默认线程名字一般是 `task-1`、`task-2` 这样，不区分业务。
  - 出了问题很难快速知道是哪一类任务。

1. **全局唯一**

- - 默认 `@Async` 都走一个线程池，不同业务混在一起。
  - 比如一个很耗时的任务堵住了线程，可能把别的异步任务也卡死。

# 3. 为什么要用 `ThreadPoolTaskExecutor`

1. **可控的线程池大小**

- - 可以根据机器 CPU 配置合理的 `corePoolSize`、`maxPoolSize`。
  - 避免无节制地创建线程。

1. **任务排队策略**

- - 可以配置 `queueCapacity`，让任务排队而不是直接丢弃或者创建过多线程。

1. **拒绝策略**

- - 可以配置 `RejectedExecutionHandler`，比如让调用线程执行（CallerRunsPolicy），避免任务丢失。

1. **线程名字可读**

- - 可以设置 `setThreadNamePrefix("async-log-")`，方便日志和排查。

1. **多池隔离**

- - 可以定义多个线程池，不同业务用不同池，比如：

- - - `@Async("logExecutor")` 专门写日志
    - `@Async("aiExecutor")` 专门跑 AI 接口
    - `@Async("notifyExecutor")` 专门发通知

避免互相抢占资源。
