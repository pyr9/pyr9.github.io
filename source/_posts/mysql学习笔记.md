---
title: Mysql学习
date: 2021-11-04 21:22:22
tags: mysql索引
categories: 性能调优
---

## 索引的本质

> 索引是帮助Mysql高效获取数据的排好序的数据结构

## 索引的数据结构

#### 二叉树：单边增长的场景会导致全表扫描。

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gwoa8e7thzj30am0dkwek.jpg" style="zoom: 67%;" />

<!-- more -->

#### 红黑树：相对平衡，比二叉树性能好，大数据下，红黑树的高度过高，会造成磁盘IO频繁。

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gwoabk973sj30dy08cjrf.jpg"  />

#### Hash

#### B-tree： 

1. **特点：**

> * 叶子结点具有相同的深度，叶子结点指针为空。
> * 所有索引元素不重复
> * 节点中的数据索引从左到右递增

2. 如果使用了innoDb存储引擎，结点的data元素就可能存储的是除了索引外的其他所有列，会占用比较大的存储空间，对于一个大结点而言，可以存放的结点数量就会比较小

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gwoaf7s92lj30ne07oq3a.jpg"  />

 #### B+tree(B-tree变种)

1. 特点：

   > - 翡叶子结点不存储data ,只存储索引（冗余），可以存放更多的索引。
   > - 叶子结点包含所有的索引字段。
   > - 叶子结点用指针连接，提高区间访问的性能。

   ![](https://tva1.sinaimg.cn/large/008i3skNly1gwoarcvziwj30r80bwjs3.jpg)

## 索引是怎么支持千万级表快速查找？

mysql建议一个结点大小为16kb,这样一次iO速度比较快，一个大结点下的一个索引元素大约是14b，所以一个大结点里面约有1170个索引元素，对于一个高度为3的b+树，可以存储16* 1170* 1170 = 2000万。



