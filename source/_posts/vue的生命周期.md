---
title: vue的生命周期选项
date: 2022-12-01 15:10:25
tags:
categories: Vue
---

## 1. beforeCreate

- 在组件实例初始化之后调用，也就是new Vue()之后

- 初始化了主要的生命周期函数

## 2. created

- 初始化了data()，computed()，methods()，watch()
- 使用场景：调用ajax来获取自己的首屏数据

## 3.beforeMount

- 内存中已经编译生成好了html结构，但是还没有渲染。

> mount表示挂载的意思，就是说要将内存中渲染的html结构，渲染到页面上。

## 4. mounted

- 内存中渲染好的html结构，替换到页面上
- 其自身的 DOM 树已经创建完成并插入了父容器中。
- 所有同步子组件都已经被挂载

**使用场景**：

在此处发送 异步请求 （ajax，fetch，axios等），获取服务器上的数据，显示在DOM里。

比如：根据输入框里输入的元素，去服务器查询数据

## 5. **beforeUpdate**

- 组件更新（数据更新）之前执行的函数

## 6.**updated**

组件更新（数据更新）后执行的函数

## 7. beforeDestroy

- vue（组件）对象销毁之前

## 8. destroyed

- 组件销毁后

