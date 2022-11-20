---
title: CSS盒子模型
date: 2022-11-20 18:52:44
tags:
- 前端
categories: CSS
---

# 1 块级元素水平居中

1. 必须满足两个条件
   - 盒子必须指明了宽度
   - 盒子左右的外边剧都设置为auto
2. 常规写法
   - margin-left: auto ; margin-right: auto
   - margin: auto
   - margin: 0 auto

# 2 行内元素或者行内块元素水平居中

解决方案：给父元素添加： text-align: center

# 3 嵌套块元素垂直外边距塌陷

对于两个嵌套关系（父子关系）的块元素，父元素和子元素同时都有上外边距，此时父元素会塌陷较大的外边距值

解决方案

- 给父元素定义上边框
- 给父元素定义上内边距
- 给父元素添加：overflow: hidden

# 4 清除内外边距

网页元素很多都带有默认的内外边距，不同的浏览器默认的也不一致。因此我们在布局前，需要清除下网页元素的内外边距。

```html
* {
  padding: 0; /* 清除内边距 */
  margin: 0; /* 清除外边距 */
}
```

# 5 Padding 会影响盒子的大小

- 如果盒子已经有了宽度和高度，此时再指定内边距，会撑大盒子

​		解决方案：让width和height减去多出来的内边距大小

<img src="https://tva1.sinaimg.cn/large/008vxvgGly1h8bv0i7jx1j314g0u0aet.jpg" style="zoom: 33%;" />

- 如果盒子本身没有指定宽度和高度，则此时padding不会撑大盒子

# 6 padding-bottom实现普通元素固定宽高比

将`div`元素的高度设为了`0`，通过`padding-bottom`来撑开盒子的高度

```js
<div class="father">
  <div class="son"></div>
</div>
```

```js
.father {
  width: 50%;
  background-color: #25a4bb;
}

.son {
  height: 0;
  padding-bottom: 50%; //规定基于父元素的宽度的百分比的内边距。
  background-color: red;
}
```



