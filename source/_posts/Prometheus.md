---
title: Prometheus
date: 2025-03-21 08:09:51
tags:
categories: 应用监控工具
---

# 通过 Prometheus 和 Grafana 监控 Spring Cloud 服务

实现的效果：当em-gateway服务宕机后, 飞书通知到群里。

- prometheus（/proˈmiθɪəs/）: Prometheus 负责从各个目标服务中收集监控数据。
- grafana: 开源的可视化系统,可实现Prometheus、Elasticsearch、Postgres、InfluxDB等数据源的可视化，grafana 通过连接 Prometheus 数据源，利用其丰富的可视化选项创建直观的监控仪表板。
- prometheusalert: prometheusalert是一个开源的告警通知工具，旨在简化 Prometheus 告警的通知流程，并支持多种通知渠道。它可以帮助用户将 Prometheus 的告警信息发送到各种常用的消息平台和通知系统，如微信、钉钉、飞书、Slack、**邮件等**

# 1. 准备工作：

需要监控的项目，都需要集成actuator， 可以使Prometheus通过http请求收集数据。[参考博客](https://pyr9.github.io/Actuator/)

# 2. docker-compose 安装

```yml
services:
  prometheus:
    image: prom/prometheus:v2.53.0
    container_name: prometheus
    volumes:
      - ./config/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus-data:/prometheu
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:11.4.2
    container_name: grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin  # 设置Grafana管理员密码
    ports:
      - "3000:3000"
    depends_on:
      - prometheus
    volumes:
      - ./grafana-storage:/var/lib/grafana

  prometheusalert:
    image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/feiyu563/prometheus-alert:v4.9.1
    container_name: prometheusalert
    ports:
      - "8089:8080"
    environment:
      - PA_LOGIN_USER=prometheusalert
      - PA_LOGIN_PASSWORD=prometheusalert
      - PA_TITLE=PrometheusAlert
      - PA_OPEN_FEISHU=1
      - PA_OPEN_DINGDING=1
      - PA_OPEN_WEIXIN=1
    volumes:
      - ./prometheus-alert/db:/app/db
    command: ["-c", "/app/prometheusalert.yml"]

```

# 3. 配置 Prometheus

## 1. 修改配置文件

这里使用到了`host.docker.internal`， 因为我的Java服务是运行在宿主机的，而prometheus是运行在容器里的，使用`host.docker.internal`来访问宿主机

prometheus.yml 

```yml
global:
  scrape_interval: 15s # 默认抓取间隔

scrape_configs:
  - job_name: 'em-gateway'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: 
        - 'host.docker.internal:9998'  # 替换为你的Spring Cloud服务地址
  - job_name: 'sith-mes'
    metrics_path: '/sith-mes/actuator/prometheus'
    static_configs:
      - targets: 
        - 'host.docker.internal:6500'  # 替换为你的Spring Cloud服务地址 
```

## 2.  配置 Spring Cloud 应用

1. 引入依赖

   ```xml
   <dependency>
       <groupId>io.micrometer</groupId>
       <artifactId>micrometer-registry-prometheus</artifactId>
   </dependency>
   ```

2. 修改配置

   ```yml
   ## 开启所有actuator-endpoint
   management:
     endpoint:
       health:
         show-details: always
     endpoints:
       web:
         exposure:
           include:
             - '*'
             - 'prometheus'
   ```

3. 访问项目 `/actuator/prometheus`

![image-20250313131253269](C:\Users\z004nnre\AppData\Roaming\Typora\typora-user-images\image-20250313131253269.png)

4. 再次访问`http://localhost:9090/targets`确保当前服务成功注册到了prometheus上

![image-20250313145316614](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250313145316614.png)

5. 配置属性在prometheus简单查看

![image-20250313145416414](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250313145416414.png)

# 4. 配置 Grafana

访问 Grafana 打开浏览器访问 http://localhost:3000，默认用户名和密码是 admin/admin

## 1. 配置Prometheus作为datasource

![image-20250313131723923](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250313131723923.png)

![image-20250313134156998](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250313134156998.png)

## 2. 测试展示系统的CPU使用情况

![image-20250313134357862](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250313134357862.png)

![image-20250313141328486](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250313141328486.png)

## 3. dashborad模板

网址：[Grafana dashboards | Grafana Labs](https://grafana.com/grafana/dashboards/)

1. dashboard 市场有很多现成的dashborad，可以直接使用，比如：

- Actuator: https://grafana.com/grafana/dashboards/12856

- jmx_exporter: https://grafana.com/grafana/dashboards/3457
- node_exporter: https://grafana.com/grafana/dashboards/8919
- mysqld_exporter:
  - https://grafana.com/grafana/dashboards/13106
  - https://grafana.com/grafana/dashboards/11323
- redis_exporter: https://grafana.com/grafana/dashboards/11835
- k8s: https://grafana.com/grafana/dashboards/13105

2. 具体使用步骤
   1. 关键词搜索

![image-20250313141833859](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250313141833859.png)

2. 复制模板ID

![image-20250313141924350](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250313141924350.png)

3. 填入ID导入

![image-20250313142044330](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250313142044330.png)



## 4. 引入Prometheusalert

因为grafana的github的issue上写了他们短期不支持飞书，我也尝试配置webhook，飞书通知失败，所以这里引入了prometheusalert来做飞书通知。

1. 在模板管理里，找到自己需要的模板，复制URL， 后续作为webhook地址，供grafana使用。这是我下面示例使用的[URL](http://host.docker.internal:8089/prometheusalert?type=fs&tpl=grafana-fs&fsurl=https://open.feishu.cn/open-apis/bot/v2/hook/ae1eaeaa-fab0-4815-bc7f-f96110281df4)

![image-20250319112015718](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250319112015718.png)

## 4. 配置grafana告警

1. 修改 prometheusalert.yml (不确定是否真的需要，可以先不配试试)

```yml
feishu:
  - name: "feishu"
    webhook: "https://open.feishu.cn/open-apis/bot/v2/hook/ae1eaeaa-fab0-4815-bc7f-f96110281df4"
```

1. 增加 Contact points，配置红框中的属性

![image-20250319110857781](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250319110857781.png)

2. 创建看板，展示em-gateway的运行情况

![image-20250319112616203](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250319112616203.png)

3. 配置Alert

![image-20250319112727528](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250319112727528.png)

![image-20250319112757644](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250319112757644.png)

![image-20250319112829866](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250319112829866.png)

停止服务，查看飞书通知;

![image-20250319113318248](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250319113318248.png)

