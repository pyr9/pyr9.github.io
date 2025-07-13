---
title: gitignore文件添加
date: 2025-07-13 11:17:01
tags:
categories: Java基础
---

# 1. 删除缓存

```shell
 git rm -r --cached .idea/
```

## 2. gitignore 文件示例

```yml
# 编译产生的文件
/target/
/bin/

# IntelliJ IDEA IDE 配置文件（如果使用）
/.idea/
/*.iml

# Eclipse IDE 配译文件（如果使用）
/.settings/
/.classpath
/.project

# NetBeans IDE 配置文件（如果使用）
/nbproject/private/
/nbbuild/
/dist/
/build/

# Maven 配置文件
/target/

# Gradle 构建工具配置文件
.gradle/
build/

# macOS 系统文件
.DS_Store

# Windows 系统文件
Thumbs.db

# Linux 下的临时文件
*~

# 日志文件
*.log
```

