---
title: 响应式布局中rem的使用
date: 2025-11-09 19:48:49
tags:
categories: 前端布局
---

# 1. 什么是rem?

## 1.1. 定义

rem 是一种相对单位，表示相对于 HTML 根元素的字体大小。它的全称是 "root em"，即根 em。

## 1.2. 为什么要使用rem?

设置 `rem`基准大小主要是为了 **响应式设计** 和 **统一缩放控制**。

```javascript
/* 基础设置 */
html {
  font-size: 16px; /* 桌面端基准 */
}

/* 平板设备 */
@media (max-width: 768px) {
  html {
    font-size: 14px; /* 整体缩小 */
  }
}

/* 手机设备 */
@media (max-width: 480px) {
  html {
    font-size: 12px; /* 进一步缩小 */
  }
}
```

**效果**：只需要修改一个值，所有使用 `rem`的元素都会按比例缩放。

## 1.3. 默认基准值

`rem`的大小取决于 **根元素(html)的字体大小**。

- 大多数现代项目：1rem = 16px
- 移动端优化项目：1rem = 14px
- 简化计算的项目：1rem = 10px

```html
/* 浏览器默认值 */
html {
  font-size: 16px; /* 大多数浏览器默认 1rem = 16px */
}
```

## 1.4. 查看当前项目的 rem 基准值

项目控制台输入： 

```javascript
console.log('1rem =', getComputedStyle(document.documentElement).fontSize);
```

# 2. 推荐使用rem的情况

- 布局容器：width 
- 内外边距: pading、margin、gap
- 字体大小 - 需要随用户字体偏好缩放的元素



注：使用 px： - 边框、阴影、细线 - 图标、固定尺寸控件 - 不需要缩放的效果

## 2.1. 布局和间距

```javascript
.container {
  max-width: 80rem;    /* 布局容器 */
  padding: 2rem;       /* 内边距 */
  margin: 1.5rem;      /* 外边距 */
  gap: 1rem;           /* 网格/弹性布局间距 */
}

.card {
  border-radius: 0.5rem; /* 圆角 */
}
```

## 2.2. 字体大小

```javascript
.title {
  font-size: 2rem;     /* 标题 */
}

.body {
  font-size: 1rem;     /* 正文 */
}

.caption {
  font-size: 0.875rem; /* 小字 */
}
```



TIP： 为了简化写法，可以使用 tailwindcss， 如：max-h-80、w-24、 h-64

https://www.tailwindcss.cn/docs/max-width
