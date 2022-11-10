---
title: ES6语法学习笔记(二)
date: 2022-11-06 11:53:54
tags:
- es6
categories: 前端
---

# 1 ES6 对象的新增方法

## 1.1 Object.is()

- `严格相等运算符`（ === ） 

  - 类型不同，直接返回false
  - 两个复合类型（对象、数组、函数）的数据比较时，不是比较它们的值是否相等，而是比较它们是否指向同一个对象。
  - 同一类型的原始类型的值（数值、字符串、布尔值）比较时，值相同就返回true，值不同就返回false
  - undefined 和 null 与自身严格相等

- `相等运算符`（ == ）

  - 在比较不同类型的数据时，相等运算符会先将数据进行类型转换，然后再用严格相等运算符比较

- Object.is()

  - 它用来比较两个值是否严格相等，与严格比较运算符（===）的行为基本一致。

  - 不同之处只有两个：一是 +0 不等于 -0 ，二是 NaN 等于自身。

    ```js
    Object.is(+0, -0) // false
    Object.is(NaN, NaN) // true	
    ```

    

## 1.2 Object.assign()

Object.assign方法用于对象的合并，将源对象（source）的所有可枚举属性，复制到目标对象（target）

- `Object.assign`方法实行的是`浅拷贝`，

```js
const target = { a: 1 };
const source1 = { b: 2 };
const source2 = { c: 3 };
Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}
```



# promise

- Promise 相当于一个容器，保存着异步操作的结果。有三个状态：pending（进行），resolved（成功），rejected（失败）
