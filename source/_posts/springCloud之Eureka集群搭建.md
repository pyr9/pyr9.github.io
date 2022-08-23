---
title: springCloud之Eureka集群搭建
date: 2022-07-22 22:35:16
tags:
- 分布式
- 集群
- eureka
categories: springCloud
---

# springCloud之Eureka集群搭建

## 1. 搭建3个Eureka server

- 参考：[springCloud之Eureka搭建 - 楼上有只喵 (panyurou.github.io)](https://panyurou.github.io/2022/08/17/springCloud之Eureka搭建/)

- 三个eureka-server的application.yml文件如下：

  - eureka-server

    ```xml
    server.port: 8761
    
    eureka.instance.hostname: localhost
    #false表示不向注册中心注册自己。
    eureka.client.registerWithEureka: false
     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    eureka.client.fetchRegistry: false
    #eureka.client.serviceUrl.defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
    eureka.client.serviceUrl.defaultZone: http://eureka8762.com:8762/eureka/,http://eureka8763.com:8763/eureka/
    ```

  - eureka-server1

    ```xml
    server.port: 8762
    spring.application.name = micro-weather-eureka-server2
    
    eureka.instance.hostname: localhost
    eureka.client.registerWithEureka: false
    eureka.client.fetchRegistry: false
    eureka.client.serviceUrl.defaultZone: http://eureka8761.com:8761/eureka/,http://eureka8763.com:8763/eureka/
    ```

  - eureka-server2

    ```xml
    server.port: 8763
    spring.application.name = micro-weather-eureka-server1
    
    eureka.instance.hostname: localhost
    eureka.client.registerWithEureka: false
    eureka.client.fetchRegistry: false
    eureka.client.serviceUrl.defaultZone: http://eureka8761.com:8761/eureka/,http://eureka8762.com:8762/eureka/
    ```



## 2.配置三个hostname

sudo vim /etc/hosts

```xml
127.0.0.1       eureka8761.com
127.0.0.1       eureka8762.com
127.0.0.1       eureka8763.com
```



## 3. 启动三个eureka-server，并访问

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5fas7jqlnj21vc0u0wjx.jpg)

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5fassbxfuj21uk0u00xx.jpg)

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5fat91b5gj21uu0u0wjo.jpg)

注意⚠️：

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5fauhebrjj228k09udgs.jpg)

- 这里的意思就是这两个注册中心是当前注册中心的集群节点，当前注册中心会从这两个节点同步服务

- 这里是通过hostname辨别的，所以配置yml参数的时候需要配置不同的hostname。
- 这里有显示配置的集群节点，就证明集群配置成功了。
