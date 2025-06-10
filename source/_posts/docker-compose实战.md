---
title: docker-compose实战
date: 2024-11-27 14:43:51
tags:
categories: docker    
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
   networks:
     tms:
       external: true
   services:
     rabbitmq:
       image: rabbitmq:3-management-alpine
       hostname: rabbitmq
       container_name: rabbitmq
       environment:
         RABBITMQ_DEFAULT_USER: guest
         RABBITMQ_DEFAULT_PASS: guest
         TZ: Asia/Shanghai
       ports:
         - "5672:5672"
         - "15672:15672"
       restart: always
       networks:
         - tms
       volumes:
         - /etc/localtime:/etc/localtime:ro
   
     redis:
       image: redis
       container_name: redis
       ports:
         - 6379:6379
       restart: always
       networks:
         - tms
       environment:
         TZ: Asia/Shanghai
       volumes:
         - /etc/localtime:/etc/localtime:ro
   
     nginx:
       image: nginx:latest
       container_name: nginx
       ports:
         - "80:80"
         - "443:443"
       restart: always
       volumes:
         - ./nginx/config/nginx.conf:/etc/nginx/nginx.conf
         - ./nginx/config/conf.d:/etc/nginx/conf.d
         - ./nginx/ssl:/etc/nginx/ssl
         - ./nginx/logs:/var/log/nginx
         - ./nginx/html:/usr/share/nginx/html
         - ../frontend:/home/deploy/tms/frontend
         - /etc/localtime:/etc/localtime:ro
       networks:
         - tms
       environment:
         TZ: Asia/Shanghai
   
     mysql:
       image: mysql:8.0
       container_name: mysql
       environment:
         MYSQL_DATABASE: xxl_job
         MYSQL_USER: root
         MYSQL_PASSWORD: root@1234
         MYSQL_ROOT_PASSWORD: root@1234
         TZ: Asia/Shanghai
       ports:
         - "3306:3306"
       restart: always
       volumes:
         - mysql_data:/var/lib/mysql
         - /etc/localtime:/etc/localtime:ro
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
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;
	autoindex on;

    keepalive_timeout  65;

    #gzip  on;

	upstream gatewayservice {
		server 10.64.192.112:8001;
	}
	upstream xxlJobAdmin {
		server 10.64.192.112:8200;
	}
	upstream kibanaService {
		server 10.64.192.112:5601;
	}
	upstream gitlabService {
		server 10.64.192.112:8090;
	}
	upstream rabbitmqService {
		server 10.64.192.112:15672;
	}

    server {
        listen       80;
        server_name  plm.innomotics.net.cn;
        rewrite ^(.*)$ https://$host$1 permanent;
    }


    server {
        listen       443 ssl;
        server_name  plm.innomotics.net.cn;

		#ssl证书的pem文件路径
        ssl_certificate  /etc/nginx/ssl/S0004A05_plm.innomotics.net.cn.cer;
        #ssl证书的key文件路径
        ssl_certificate_key /etc/nginx/ssl/plm.innomotics.net.cn.key;

        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Host $server_name;

        # Header for SocketJS
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

        # 访问所有目录都默认走到
        location / {
            root   /home/deploy/tms/frontend/prod;
            index  index.html index.htm;
            add_header "Access-Control-Allow-Origin"  *;
            try_files $uri $uri/ /index.html;
        }   
        location /api {
		    rewrite ^/api/(.*)$ /$1 break;
          proxy_pass         http://gatewayservice;
					proxy_redirect     off;
        }
        location /xxl-job-admin/ {
          proxy_pass			http://xxlJobAdmin;
          proxy_redirect		off;
        }
        location /so-kibana/{
          proxy_pass			http://kibanaService;
          proxy_redirect		off;
        }
        location /so-gitlab/{
          proxy_pass			http://gitlabService;
          proxy_redirect		off;
        }
        location /so-rabbitmq/{
          rewrite ^/so-rabbitmq/(.*) /$1 break; 
          proxy_pass			http://rabbitmqService;
          proxy_redirect		off;
        }
        location @router {
          rewrite ^.*$ /index.html last;
        } 

    }
}
```

## 4. docker-compose部署后端

1. backend对应目录下上传jar包和docker-compose.yml

   ```yml
   networks:
     tms:
       external: true
   services:
     eureka-server:
       image: openjdk:17
       container_name: eureka-server
       restart: always
       networks:
         - tms
       ports:
         - "8002:8002"
       volumes:
         - ./eureka-server:/home
       environment:
         - SPRING_PROFILES_ACTIVE=prod
       command: ["java", "-jar", "/home/eureka-server-1.0-SNAPSHOT.jar"]
   
     gateway-server:
       image: openjdk:17
       container_name: gateway-server
       restart: always
       networks:
         - tms
       ports:
         - "8001:8001"
       volumes:
         - ./gateway-server:/home
       environment:
         - SPRING_PROFILES_ACTIVE=prod
       command: ["java", "-jar", "/home/gateway-server-1.0-SNAPSHOT.jar"]
   
     auth-service:
       image: openjdk:17
       container_name: auth-service
       restart: always
       networks:
         - tms
       ports:
         - "8003:8003"
       volumes:
         - ./auth-service:/home
       environment:
         - SPRING_PROFILES_ACTIVE=prod
       command: ["java", "-jar", "/home/auth-service-1.0-SNAPSHOT.jar"]
   
     user-service:
       image: openjdk:17
       container_name: user-service
       restart: always
       networks:
         - tms
       ports:
         - "8101:8101"
       volumes:
         - ./user-service:/home
       environment:
         - SPRING_PROFILES_ACTIVE=prod
       command: ["java", "-jar", "/home/user-service-1.0-SNAPSHOT.jar"]
   
     assistant-service:
       image: openjdk:17
       container_name: assistant-service
       restart: always
       networks:
         - tms
       ports:
         - "8004:8004"
       volumes:
         - ./assistant-service:/home
       environment:
         - SPRING_PROFILES_ACTIVE=prod
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
         - SPRING_PROFILES_ACTIVE=prod
       command: ["java", "-jar", "/home/tms-service-1.0-SNAPSHOT.jar"]
   ```

   
