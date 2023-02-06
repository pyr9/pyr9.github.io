---
title: SSH免密登录
date: 2023-02-06 15:47:59
tags:
categories:
---

# 1. 客户端生成公私钥

1. 执行`ssh-keygen`在用户目录.ssh文件夹下创建公私钥:

   - id_rsa （私钥）

   - id_rsa.pub (公钥)

# 2. 上传公钥到服务器

1. 进入服务端.ssh目录下 `cd ~/.ssh`

2. 将客户端生成的id_rsa.pub 复制到 authorized_keys 里 `vim authorized_keys`

# 3. 测试免密登录

`ssh root@localhost`

