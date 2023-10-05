---
title: docker+jenkins实现SpringBoot项目自动化部署
date: 2023-10-03 18:41:50
tags:
categories: jenkins
password: 19980617pyr
---

# 1 背景

当前有一个微服务项目，共包含4个常用的微服务子项目，原始项目升级方式为:

1. 修改application.yml文件，profile为product
2. 在自己的电脑上手动利用maven打包，分为clean, package
3. 分别进入4个项目的target目录下，将打好的jar包拷贝出来。
4. 将本机拷贝出来的4个jar包上传到跳板机上
5. 将上传到跳班机上的4个jar包拷贝出来，传到客户服务器上的指定目录
6. 在服务器上依次执行jps -l（列出当前正在运行的Java进程和它们的进程PID），kill -9 , jar -jar xxx.jar命令
7. 再次执行jps -l，查看服务是否正常启动

流程不仅复杂，而且耗费时间，所以考虑自动化部署，来简化这一体系流程

# 2 原理

1. 从git仓库拉取最新代码

2. 利用maven执行clean , package操作，进行打包
3. 打包结果通知到飞书（可有可无）

3. 将打好的jar包，发送到指定服务器上

4. 执行部署脚本 包括kill -9 , jar -jar xxx.jar

# 3 具体步骤

## 1. 配置maven和jdk

![image-20231005191707787](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231005191707787.png)

![image-20231005191548704](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231005191548704.png)



![image-20231005191644036](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231005191644036.png)

## 2. 新建Job

![image-20231005191359836](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231005191359836.png)

## 3. 配置git仓库地址和分支名

![img](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230315160535916.png)

## 4. 配置构建环境

每次构建前删除工作空间（避免由于类转移位置后，两个位置都存在类，引起同名bean冲突）

打印控制台输出时间戳，便于观察构建进度

![image-20231005191939745](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231005191939745.png)

## 5 配置编译使用的pom.xml文件

![image-20231005192206486](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231005192206486.png)

## 6 配置给飞书发送通知

由于不论成功还是失败都需要给飞书发送通知，这里使用了**Post build task** 插件

![image-20231005192341444](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231005192341444.png)

feishu.sh:

```sh
#！/bin/bash
jobName=$JOB_BASE_NAME
json=$(curl -u admin:password https://xxx.com.cn/jenkins/job/$jobName/${BUILD_NUMBER}/api/json)
state=$(echo $json | sed 's/,/\n/g' | grep "result" | sed 's/:/\n/g' | sed '1d' | sed 's/}//g')
nowTime=$(TZ=UTC-8 date +%Y-%m-%d" "%H:%M:%S)
if [ ${state} == "\"SUCCESS\"" ] ; then stateText="成功"; else stateText="失败"; fi
curl -X POST -H "Content-Type: application/json" \
        -d '{"msg_type":"post","content": {"post": {"zh_cn": {"title": "jenkins构建报告","content": [[{"tag": "text","text": "'"名  称：$jobName\n编  号：${BUILD_NUMBER}\n状  态:  $stateText\n时  间:  $nowTime\n详  情:  https://xxx.com.cn/jenkins/job/$jobName/${BUILD_NUMBER}/console\n日  志：https://b1ortdxsoc.feishu.cn/wiki/RbAowYec2iUw72kWlJocapX8n2f"'"}]]} } }}' \
https://open.feishu.cn/open-apis/bot/v2/hook/ffb88764-379c-4a98-8496-b1a713114f3
```

飞书收到的格式为：

![image-20231005192946100](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231005192946100.png)

## 7 每个微服务单独建一个执行job

配置运行这个job时，将构建好的jar包上传到客户服务器上，并且执行部署脚本

![image-20231005193534214](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231005193534214.png)

deploy.sh：

```sh
source /etc/profile
java -version
BUILD_ID=dontKillMe
export TERM=xterm
CURRENT_TIMESTAMP=$(date +%s)
NOW=$(date -d @$CURRENT_TIMESTAMP "+%Y-%m-%d_%H-%M-%S")
jar=$1-1.0.0-SNAPSHOT.jar
cp $jar $1-$NOW.jar
jar=`ls $1*.jar |sort -r|head -n 1`
echo $jar

profile_active=sithMes
if [ -n "$2" ]
then
    profile_active=$2
    echo "set profile_active: $profile_active"
fi

log_output=/dev/null
if [ -n "$3" ]
then
    log_output=$3
    echo "set log_output: $log_output"
fi

ps -ef | grep java | grep $1 | grep -v grep | awk '{print $2}' | xargs kill -9
nohup java -jar $jar --spring.profiles.active=$profile_active >$log_output 2>&1 &
sleep 30
if test $(pgrep -f $jar|wc -l) -eq 0
then
   echo "Start Failed"
   exit 1
else
   echo "Start Success"
fi
echo '============================== 执行 top | head -50 ====================='

top -b| head -40

echo '================================== 执行 jps -l ========================'

jps -l
```

> 这里 没查到jar包执行时，执行exit 1，会时jenkins当前job的状态变成黄色，而不是成功时的绿色。

# 4 常见问题

## 1 jenkins迁移时需要迁移哪些文件夹？

将老服务器[jenkins](https://so.csdn.net/so/search?q=jenkins&spm=1001.2101.3001.7020)主目录下的config.xml文件以及jobs、users、workspace、plugins四个目录拷贝到新机器的jenkins主目录下。

```sh
docker cp /Users/panyurou/.jenkins/jobs  6257c7abfa11:/var/jenkins_home/jobs
```

## 2 使用docker增加前缀 /jenkins

```sh
 docker pull jenkins/jenkins:2.346
```

```sh
docker run -d -p 8089:8080  --name jenkins4 -e JENKINS_OPTS="--prefix=/jenkins" jenkins/jenkins:2.346
```

## 3 没权限 

报错：I have experienced this when the $JENKINS_HOME/jobs directory had incorrect permissions or ownership.

```sh
docker exec -it --user root 7666058a5629 bash
```

```sh
chown -R jenkins:jenkins /var/jenkins_home
```

## 4 SSH服务器验证失败

报错： stderr: No RSA host key is known for [xxx.xxx.xxx.xxx]:7999 and you have requested strict checking.

Manage Jenkins -> Security -> Configure Global Security

![image-20231003184529837](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231003184529837.png)

## 5 Docker部署jenkins配置公私钥拉取代码

https://blog.csdn.net/weixin_38080573/article/details/128482482

## 6 jdk，maven需要重新配置docker里安装的路径

maven settings.xml 仓库需要配置成docker里的仓库

## 7 设置开机自启docker

```
systemctl enable docker
```

## 8 设置docker启动时，启动jenkins容器

```shell
docker update --restart=always jenkins
```

