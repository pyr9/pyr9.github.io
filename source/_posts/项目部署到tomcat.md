---
title: 项目部署到tomcat的三种方式
date: 2023-01-24 20:00:20
tags:
categories:
---

# 1.**项目直接放入 webapps 目录中**

1. 把项目打包，放入webapps目录下

![](https://panyuro.oss-cn-beijing.aliyuncs.com/202301242001681.png)

2. 依次运行tomcat 的bin目录下`shutdown.sh`,`startup.sh`

   ![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230124200306.png)

3. 在浏览器输入：http://localhost:8080/项目名，就可以进入到项目的index页面

   <img src="https://panyuro.oss-cn-beijing.aliyuncs.com/20230124200912.png"  />

   ![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230124200828.png)       



# 2 **修改 conf/server.xml 文件** 

进入tomcat下conf/server.xml，在`<Host> </Host>`标签之间输入项目配置信息

```xml
<Context path="/test01" docBase="/Users/xxx/test-jenkins-1.0-RELEASE" reloadable="true" />
```

- **path**:浏览器访问时的路径名

- **docBase**:web项目的WebRoot所在的路径，注意是WebRoot的路径，不是项目的路径，其实也**就是编译后的项目路径**（解压后的j ar包路径）。

- ***reloadble**:设定项目有改动时，tomcat是否重新加载该项目*

# 3 **conf/Catalina** 

1. 入到 apache-tomcat-7.0.52\conf\Catalina\localhost 目录，新建一个 `ROOT.xml` 文件

2. 在那个新建的 xml 文件中，增加下面配置语句

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <Context docBase="/Users/panyurou/test-jenkins-1.0-RELEASE" reloadable="true" />
   ```

3. 在浏览器输入路径：localhost:8080

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230126154643.png)
