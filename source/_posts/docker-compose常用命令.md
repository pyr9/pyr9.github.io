---
title: docker-compose
date: 2024-11-27 14:43:51
tags:
categories:
---

# 1. 常用命令

1. 前台启动, 启动项目中的所有服务 `docker-compose up`

2. 后台启动, 启动所有服务并在后台运行 `docker-compose up -d`
3. 重新编译文件并启动  `docker-compose up --build nginx `

4. 停止所有服务, 保留容器`docker-compose stop `

5. 停止所有服务, 删除容器`docker-compose down`
6. 删除容器`docker-compose rm -f nginx`

7. 重启容器
   - 重启所有服务的容器 `docker-compose restart`
   - 重启指定服务的容器 `docker-compose restart nginx `

8. 启动容器
   - 启动所有服务的容器 `docker-compose start `
   - 启动指定服务的容器 `docker-compose start nginx `

9. 停止容器
   - 停止所有服务的容器` docker-compose stop`
   - 停止指定服务的容器 `docker-compose stop nginx `

10. 进入容器
   -  `进入指定服务的容器` `docker-compose exec nginx bash`

# 2. 实战使用

部署项目tms

## 1. 准备工作

1. 在目录home/userxxx 下创建文件夹 tms及相关前后端文件夹

   ```shell
   mkdir tms
   cd tms
   mkdir deploy
   cd deploy
   mkdir backend
   mkdir frontend
   ```

2. 安装docker, docker-compose

   Docker 注意修改默认挂载地址

## 2. 部署中间件

1. middleware目录下上传docker-compose.yml

   ```
   cd deploy
   mkdir middleware
   ```

2. 使用`docker-compose up prod`启动redis，rabbitMQ，postgresql(需开放对应端口或后续修改nginx配置通过nginx访问)

   docker-compose.yml

   ```yml
   services:
     prod:
       image: alpine:latest
       links:
         - postgres
         - redis
         - rabbitmq
         - nginx
       command: echo 'prod environment started'
     rabbitmq:
       image: rabbitmq:3-management-alpine
       hostname: rabbitmq
       environment:
         RABBITMQ_DEFAULT_USER: in_tms_ad
         RABBITMQ_DEFAULT_PASS: INNOMOTICS.PE@2024
       ports:
         - "5672:5672"
         - "15672:15672"
       restart: always
       networks:
         - tms
     redis:
       image: redis
       ports:
         - 6379:6379
       environment:
         - REDIS_PASSWORD=INNOMOTICS.SO@2024
       networks:
         - tms
     postgres:
       image: postgres
       environment:
         POSTGRES_USER: in_tms_ad
         POSTGRES_PASSWORD: INNOMOTICS.Dig@2024
         POSTGRES_DB: db_tms
       ports:
         - "5432:5432"
       restart: always
       volumes:
         - ./postgresql/data:/var/lib/postgresql/data
         - ./postgresql/logs:/var/log/postgresql
       networks:
         - tms
     nginx:
       image: nginx:latest
       ports:
         - "80:80"
       restart: always
       volumes:
         - ./nginx/config/nginx.conf:/etc/nginx/nginx.conf
         - ./nginx/config/conf.d:/etc/nginx/conf.d
         - ./nginx/log/:/var/log/nginx
         - ../frontend:/home/deploy/tms/frontend
       networks:
         - tms
   ```

## 3. 部署vue

1. 进入前端目录，上传dist目录到web目录

   ```
   cd frontend
   mkdir web
   ```

2. 更新nginx.conf配置, 并重启

```yml
server {
    listen 80;
    server_name localhost;

    location / {
        root /home/deploy/tms/frontend;
        index index.html;
        try_files $uri $uri/ /web/index.html;
    }
}
```

## 4. docker-compose部署后端

1. backend对应目录下上传jar包和docker-compose.yml

   ```yml
   version: '3.8'
   
   services:
     ureka-server:
       image: openjdk:17
       container_name: ureka-server
       networks:
         - tms
       ports:
         - "8001:8001"
       volumes:
         - ./eureka-server:/home
       environment:
         - SPRING_PROFILES_ACTIVE=dev
       command: ["java", "-jar", "/home/eureka-server-1.0-SNAPSHOT.jar"]
   
     gateway-server:
       image: openjdk:17
       container_name: gateway-server
       networks:
         - tms
       ports:
         - "8002:8002"
       volumes:
         - ./gateway-server:/home
       environment:
         - SPRING_PROFILES_ACTIVE=dev
       command: ["java", "-jar", "/home/gateway-server-1.0-SNAPSHOT.jar"]
   
     auth-service:
       image: openjdk:17
       container_name: auth-service
       networks:
         - tms
       ports:
         - "8003:8003"
       volumes:
         - ./auth-service:/home
       environment:
         - SPRING_PROFILES_ACTIVE=dev
       command: ["java", "-jar", "/home/auth-service-1.0-SNAPSHOT.jar"]
   
     user-service:
       image: openjdk:17
       container_name: user-service
       networks:
         - tms
       ports:
         - "8101:8101"
       volumes:
         - ./user-service:/home
       environment:
         - SPRING_PROFILES_ACTIVE=dev
       command: ["java", "-jar", "/home/user-service-1.0-SNAPSHOT.jar"]
   
     ssistant-service:
       image: openjdk:17
       container_name: ssistant-service
       networks:
         - tms
       ports:
         - "8004:8004"
       volumes:
         - ./assistant-service:/home
       environment:
         - SPRING_PROFILES_ACTIVE=dev
       command: ["java", "-jar", "/home/assistant-service-1.0-SNAPSHOT.jar"]
   
     tms-service:
       image: openjdk:17
       container_name: tms-service
       networks:
         - tms
       ports:
         - "8103:8103"
       volumes:
         - ./tms-service:/home
       environment:
         - SPRING_PROFILES_ACTIVE=dev
       command: ["java", "-jar", "/home/tms-service-1.0-SNAPSHOT.jar"]
   
   networks:
     tms:
       driver: bridge
   ```

   
