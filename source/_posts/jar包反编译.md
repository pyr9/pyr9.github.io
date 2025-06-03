---
title: jar包反编译
date: 2025-06-03 21:54:31
tags:
categories:
---

# 1. idea 安装插件

安装Java Decompiler

![image-20250603215554615](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250603215554615.png)

# 2. 执行反编译命令

```shell
java -cp "D:\software\JetBrains\IntelliJ IDEA 2024.3.5\plugins\java-decompiler\lib\java-decompiler.jar" org.jetbrains.java.decompiler.main.decompiler.ConsoleDecompiler -dgs=true wms-inv-2.0.1.jingao.12.jar "D:\code\wms\database\wms-inv"
```

# 3. 解压jar包

使用上述命令执行后，会在 `D:\code\wms\database\wms-inv`下生成jar包，解压缩即可

```shell
jar xf wms-inv-2.0.1.jingao.12.jar
```

