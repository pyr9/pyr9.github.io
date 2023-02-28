---
title: docker安装clickhouse
date: 2022-02-01 21:44:28
tags: 分布式框架
categories: Clickhouse
---

1.启动容器服务，加载镜像

```sql
➜  ~ docker run -d --name ch-server --ulimit nofile=262144:262144 -p 8123:8123 -p 9000:9000 -p 9009:9009 yandex/clickhouse-server
```

![image-20230228225548754](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228225548754.png)

2.查看服务

```sql
➜  ~ docker ps
```

![image-20230228225555839](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228225555839.png)

3.进入docker容器内，使用用户名和密码连接数据库。

```sql
➜  ~ docker exec -it 46bb792f7762 /bin/bash
root@46bb792f7762:/# clickhouse-client
```

![image-20230228225603858](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228225603858.png)

>如果有密码，输入下面命令：
>
>```sql
>root@46bb792f7762:/# clickhouse-client -u default --password test
>```

4.查看数据库

```sql
46bb792f7762 :) show databases;
```

![image-20230228225615339](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228225615339.png)

5.dbeaver 查看

![image-20230228225624031](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228225624031.png)
