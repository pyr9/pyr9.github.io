---
title: springCloud
date: 2022-08-17 14:06:30
tags:
categories:
---

# springcloud

## springCloud和springboot

- springCloud 是基于Springboot 构建的，他里面的每个子项目就是一个springboot项目



spring cloud config

- 配置中心，利用gi t



访问服务的方式

- 通过url访问服务 : http://localhost:8081/weather/cityId/101280101: ip地址+端口号+资源的位置
  - 缺点：
    - IP地址需要绑定主机，而且是固定的，动态的就不行
    - IP难记
    - 2个IP很难做负载均衡
- 服务的注册和发现：用名称来访问 eueka
  - 名字好记
  - 一个名字可以映射到多个IP

![image-20220817224413639](/Users/panyurou/Library/Application Support/typora-user-images/image-20220817224413639.png)





负载均衡器在客户端

![image-20220817224557171](/Users/panyurou/Library/Application Support/typora-user-images/image-20220817224557171.png)





![image-20220817224619763](/Users/panyurou/Library/Application Support/typora-user-images/image-20220817224619763.png)



负载均衡器在服务端
