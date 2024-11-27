---
title: docker-compose常用命令
date: 2024-11-27 14:43:51
tags:
categories:
---

1. 前台启动, 启动项目中的所有服务 `docker-compose up`

2. 后台启动, 启动所有服务并在后台运行 `docker-compose up -d`
3. 重新编译文件并启动  `docker-compose up --build nginx `

3. 停止所有服务, 保留容器`docker-compose stop `

4. 停止所有服务, 删除容器`docker-compose down`
5. 删除容器`docker-compose rm -f nginx`

5. 重启容器
   - 重启所有服务的容器 `docker-compose restart`
   - 重启指定服务的容器 `docker-compose restart nginx `

6. 启动容器
   - 启动所有服务的容器 `docker-compose start `
   - 启动指定服务的容器 `docker-compose start nginx `

7. 停止容器
   - 停止所有服务的容器` docker-compose stop`
   - 停止指定服务的容器 `docker-compose stop nginx `

8. 进入容器
   -  `进入指定服务的容器` `docker-compose exec nginx bash`
