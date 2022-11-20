---
title: bootstrap基本样式
date: 2022-10-28 09:43:26
tags: 
- CSS
categories: 前端
---

# 边框

- class="border"  添加边框属性，
- class="border-top” 显示指定边框。
- class=""border-0"：边框-删除
- class="border-top-0“： 删除或显示特定边框
- `.rounded`元素可以轻松的定义四个圆角的孤度及显示效果

# 文本颜色

- .text-danger
- .text-secondary
- .text-success

![](https://tva1.sinaimg.cn/large/008vxvgGly1h7ku4yzhrwj305n05fjrb.jpg)

# Flex弹性布局

- `d-flex `将 **直属内部子元素** 转换为flex属性

-  `.flex-row` 可设置子元素水平方向排版呈现【单行】

- `.flex-row-reverse` 可实现元素在水平上作反方向排列, 右对齐

-  `.flex-column` 设置垂直方向布局

- `.flex-column-reverse` 实现垂直方向的反转布局（从底向上铺开）

- `justify-content-*` 通用样式可以改变flx项目在主轴上的对齐，选方向（值）包括： `start` (浏览器默认值),、`end`、 `center`、 `between`、 `around`

- `.flex-fill`在一系列兄弟元素上使用该类来强制它们变成相等的宽度，同时占据所有可用的水平空间。[特别适用于等宽或正确的导航。](https://getbootstrap.net/docs/4.1/components/navs/#working-with-flex-utilities)

- 向右推两个项目(`.mr-auto`)、向左推两个项目 (`.ml-auto`)，**IE10和IE11不能正确支持在父层具有非默认的 `justify-content` 值自边距浮动auto margin** 

  ![](https://tva1.sinaimg.cn/large/008vxvgGly1h7ksut55d5j30vj01m0sm.jpg)

  ```js
  <div class="d-flex">
    <div class="mr-auto p-2">Flex item</div>
    <div class="p-2">Flex item</div>
    <div class="p-2">Flex item</div>
  </div>
  ```

- `order-*`排序方法的响应式属性：

- `align-content` 通用样式定义，可以将flex物价于横轴上 *一起对齐*，适用于多行的Flex项目

   

