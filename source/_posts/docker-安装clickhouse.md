---
title: docker 安装clickhouse
date: 2022-02-01 21:44:28
tags: 分布式框架
categories: clickhouse
---

1.启动容器服务，加载镜像

```sql
➜  ~ docker run -d --name ch-server --ulimit nofile=262144:262144 -p 8123:8123 -p 9000:9000 -p 9009:9009 yandex/clickhouse-server
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gyycj48p14j31py0g4tcd.jpg)

2.查看服务

```sql
➜  ~ docker ps
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gyycjh5146j31yy05075c.jpg)

3.进入docker容器内，使用用户名和密码连接数据库。

```sql
➜  ~ docker exec -it 46bb792f7762 /bin/bash
root@46bb792f7762:/# clickhouse-client
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gyycl6slqwj30w008qjsx.jpg)

>如果有密码，输入下面命令：
>
>```sql
>root@46bb792f7762:/# clickhouse-client -u default --password test
>```

4.查看数据库

```sql
46bb792f7762 :) show databases;
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gyydfocs7yj30r00fyq3t.jpg)

5.dbeaver 查看

![](https://tva1.sinaimg.cn/large/008i3skNly1gyydcukk8nj318m0u0q6x.jpg)
