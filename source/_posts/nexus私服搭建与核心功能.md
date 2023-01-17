---
title: nexus私服搭建与核心功能
date: 2023-01-15 20:42:13
tags:
categories: maven
---

# 1. 私服是什么？

私服是一个特殊的远程仓库，它是架设在局域网内的仓库服务。私服代理广域网上的远程仓库，供局域网内的Maven用户使用。

当Maven需要下载构建的使用，它先从私服请求，如果私服上没有的话，则从外部的远程仓库下载，然后缓存在私服上，再为Maven的下载请求提供服务。

**使用私服的好处**

- 节省自己的外网带宽：大量的对于外部远程仓库的重复请求会消耗很大的带宽
- 加速Maven的构建：不停的请求外部仓库无疑是比较耗时的， 但Maven的一些内部机制（如快照检测）要求Maven在执行构建的时候不停地检查远程仓库的数据。因此当配置了很多远程仓库时，构建的速度会被大大降低。
- 部署第三方构件：当某个构件无法从外部远程仓库下载。如Oracle的JDBC驱动由于版权原因不能发布到外网的中心仓库，建立私服之后便可以将这些构件部署到本地私服中，供内部的Maven项目使用。
- 提高稳定行，增强控制。Maven构建搞定依赖于远程仓库，因此，当Internet不稳定的时候，Maven构建也会变的不稳定，甚至无法构建。使用私服后即使暂时没有Internet连接Maven也可以正常运行，因为私服中缓存了大量的构件。此外一些私服软件（如：Nexus）还提供了很多额外的功能，如权限管理，RELEASE/SNAPSHOT区分等，管理员可以对仓库进行一些更高级的控制。
- 降低中央仓库的负荷。数百万的请求，存储数T的数据，需要相相当大的财力。使用私服可以避免很多对中央仓库的重复请求。

# 2 nexus 是什么？

Nexus 是 Sonatype 公司发布的一款仓库（Repository）管理软件，常用来搭建 Maven 私服，所以也有人将 Nexus 称为“Maven仓库管理器”。

Nexus 开源版具有以下特性：

- 占用内存小（28 M 左右）
- 具有基于 ExtJs 得操作界面，用户体验较好
- 使用基于 Restlet 的完全 REST API
- 支持代理仓库、宿主仓库和仓库组
- 基于文件系统，不需要依赖数据库
- 支持仓库索引以及搜索
- 支持在界面上上传构件
- 安全控制

# 3 nexus安装

1. 进入[Repository Manager 2 (sonatype.com)](https://help.sonatype.com/repomanager2)，选择对应的版本进行下载

2. 下载解压后，进入bin目录，运行 `./nexus start`

3. 访问http://localhost:8081/nexus

   ![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230116212514.png)

注：nexus启动失败，查看报错日志（nexus-2.15.1-02/logs/wrapper.log）

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230116212959.png)

修复：将javax.activation-1.2.0.jar 和jaxb-api-2.3.0.jar包放到nexus的lib下

4  登陆nexus，nexus2 默认用户名和密码是`admin` and `admin123` .不确定的可以查看官方文档[Running (sonatype.com)](https://help.sonatype.com/repomanager2/installing-and-running/running)

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230116215507.png)

5. 查看仓库

   ![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230116220006.png)

​       如果私服下载不到，就会去远端仓库下载，远端仓库的下载路径就是Central配置的地址，这里我修改成阿里镜像地址

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230117095523.png)



# 4 配置从nexus下载依赖

1.  pom.xml里配置仓库，仓库地址就是nexus的public Repositories的地址，如果有多个项目可以直接配到setting.xml中

   ```java
   	<repositories>
   		<repository>
   			<id>nexus-repository</id>
   			<name>nexus repository</name>
   			<url>http://localhost:8081/nexus/content/groups/public/</url>
   		</repository>
   	</repositories>
   ```

2. 测试

   删除项目已有的依赖，重新compile项目，可以看到是从nexus去下载的依赖，我这里是测试的junit4

   ![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230117100714.png)

​      此时进入public仓库，可以看到私服里已经多了junit

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230117100859.png)

# 5 配置发布到nexus

1.  修改pom.xml，配置发布版本和快照版本的仓库地址

   ```xml
   <distributionManagement>
           <repository>
               <id>release-repository</id>
               <name>release repository</name>
               <url>http://localhost:8081/nexus/content/repositories/releases/</url>
           </repository>
           <snapshotRepository>
               <id>snapshots-repository</id>
               <name>snapshots repository</name>
               <url>http://localhost:8081/nexus/content/repositories/snapshots/</url>
           </snapshotRepository>
    </distributionManagement>
   ```

2. 修改setting.xml，配置服务用户名和密码（上传到私服需要有权限）

   ```xml
   <servers>
          <server>
           <id>release-repository</id>
           <username>deployment</username>
           <password>deployment123</password>
          </server>
         <server>
           <id>snapshots-repository</id>
           <username>deployment</username>
           <password>deployment123</password>
          </server>
   </servers>
   ```

3. 部署项目

   ![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230117111441.png)



​        由于我的项目是快照版本（pom.xm配置的<version>1.0-SNAPSHOT</version>），所以上传到了快照仓库里如果需要上传到发               布仓库里修改pom.xml

![](https://panyuro.oss-cn-beijing.aliyuncs.com/202301171117149.png)

可以看到目前项目上传到了release仓库

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230117113859.png)
