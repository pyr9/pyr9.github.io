---
title: postgresql索引基础
date: 2022-06-01 22:11:25
tags:
categories:
---

# PostgreSQL索引类型
- PostgreSQL提供了多种索引类型： B-tree、Hash、GiST、SP-GiST 、GIN 和 BRIN。
- 每一种索引类型使用了 一种不同的算法来适应不同类型的查询。
- 默认情况下，CREATE INDEX命令创建适合于大部分情况的B-tree 索引。
# 数据类型
