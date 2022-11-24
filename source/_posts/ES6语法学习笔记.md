---
title: ES6语法学习笔记
date: 2022-11-04 15:53:43
tags:
- es6
categories: 前端
---

# 1 Let和const

- Let:  所声明的变量，只在let命令所在的代码块内有效。
- `const`声明一个只读的`常量`。一旦声明，常量的值就不能改变。const一旦声明变量，就必须立即`初始化`，不能留到以后赋值。类似final

# 2 解构赋值

- ES6 允许按照一定模式，从**数组和对象**中提取值，对变量进行赋值，这被称为`解构`（Destructuring）。
- 本质上，这种写法属于`“模式匹配”`，只要等号两边的模式相同，左边的变量就会被赋予对应的值

## 2.1  数组的解构赋值

- 可以从数组中提取值，按照对应位置，对变量赋值。

- 数组的元素是按次序排列的，变量的取值由它的位置决定

  ```js
  let [a, b, c] = [1, 2, 3]
  
  let [ , , third] = ["foo", "bar", "baz"];
  
  let [head, ...tail] = [1, 2, 3, 4];
  head // 1
  tail // [2, 3, 4]
  
  let [x, y, ...z] = ['a'];
  x // "a"
  y // undefined
  z // []
  
  let [x, y, z] = new Set(['a', 'b', 'c'])
  ```

- 解构赋值`允许指定`默认值

  ```js
  let [foo = true] = [];
  foo // true
  let [x, y = 'b'] = ['a']; // x='a', y='b'
  
  // 只有当一个数组成员严格等于undefined，默认值才会生效。
  let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
  
  let [x = 1] = [null];
  x // null
  ```

  ## 2.2 对象的解构赋值

  - 对象的属性没有次序，`变量`必须与`属性`同名，才能取到正确的值。

  ```js
  let { foo, bar } = { foo: 'aaa', bar: 'bbb' };
  foo // "aaa"
  bar // "bbb"
  
  let {foo} = {bar: 'baz'};
  foo // undefined
  
  const { log } = console;
  log('hello') // hello
  ```

  如果变量名与属性名不一致，必须写成下面这样。

  foo是匹配的模式，baz才是变量。真正被赋值的是变量baz，而不是模式foo。

  ```js
  let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
  baz // "aaa"
  ```

- 解构也可以用于嵌套结构的对象

  ```js
  let obj = {
    p: [
      'Hello',
      { y: 'World' }
    ]
  };
  let { p: [x, { y }] } = obj;
  x // "Hello"
  y // "World"
  ```

- 对象`的`解构`也可以指定`默认值

  - 默认值生效的条件是，对象的`属性值`严格等于`undefined`。
  - null与undefined不严格相等，所以是个有效的赋值，导致默认值不会生效。

  ```js
  var {x, y = 5} = {x: 1};
  x // 1
  y // 5
  ```

##  2.3 字符串的解构赋值

- 字符串被转换成了一个类似数组的对象

```js
const [a, b, c, d, e] = 'hello';
```

- 类似数组的对象都有一个`length`属性，因此还可以对这个属性解构赋值

```js
let {length : len} = 'hello';
len // 5
```

## 2.4 用途

1. **交换变量的值**

交换变量`x`和`y`的值

```js
[x, y] = [y, x];
```

2. **从函数返回多个值**

   ```js
   // 返回一个数组
   function example() {
     return [1, 2, 3];
   }
   let [a, b, c] = example();
   
   // 返回一个对象
   function example() {
     return {
       foo: 1,
       bar: 2
     };
   }
   let { foo, bar } = example();
   ```

3. **提取 JSON 数据**

   ```js
   let jsonData = {
     id: 42,
     status: "OK",
     data: [867, 5309]
   };
   let { id, status, data: number } = jsonData;
   console.log(id, status, number);
   // 42, "OK", [867, 5309]
   ```

4. **遍历 Map 结构**

   ```js
   for (let [key, value] of map) {
     console.log(key + " is " + value);
   }
   ```

   ```js
   // 获取键名
   for (let [key] of map) {
     // ...
   }
   // 获取键值
   for (let [,value] of map) {
     // ...
   }
   ```

5. **输入模块的指定方法**

   ```js
   const { SourceMapConsumer, SourceNode } = require("source-map");
   ```

   

# 3. 函数的扩展

## 3.1 参数设置默认值

- ES6 允许为函数的参数设置默认值，即直接写在参数定义的后面。

  ```js
  function log(x, y = 'World') {
    console.log(x, y);
  }
  ```

  ```js
  function foo({x, y = 5}) {
    console.log(x, y);
  }
  foo({}) // undefined 5
  foo({x: 1}) // 1 5
  foo({x: 1, y: 2}) // 1 2
  ```

  

- 由于大括号被解释为代码块，所以如果箭头函数直接返回一个对象，必须在对象外面加上括号，否则会报错。

  ```js
  let getTempItem = id => ({ id: id, name: "Temp" });
  ```

- ES6 引入`rest 参数`（形式为 ...变量名 ），用于获取函数的多余参数，这样就不需要使用 arguments 对象了。rest 参数搭配的变量是一个`数组`，该变量将多余的参数放入数组中。

  ⚠️： rest 参数之后不能再有其他参数（即只能是最后一个参数），否则会报错。

  ```js
  function add(...values) {
    let sum = 0;
    for (var val of values) {
      sum += val;
    }
      return sum;
  }
  
  // 利用 rest 参数，可以向该函数传入任意数目的参数。
  add(2, 5, 3) // 10
  ```

  

# 4 数组的扩展

`扩展运算符`（spread）是三个点（ ... ）。它好比 rest 参数的逆运算，将一个数组转为用逗号分隔的参数序列。

**任何定义了遍历器（Iterator）接口的对象（参阅 Iterator 一章），都可以用扩展运算符转为真正的数组。**

```js
console.log(...[1, 2, 3])
// 1 2 3
```

用途：

- 该运算符主要用于`函数调用`。

  ```js
  function add(x, y) {
    return x + y;
  }
  
  const numbers = [4, 38];
  add(...numbers) // 42
  ```

- 应用 Math.max 方法，简化求出一个数组最大元素的写法。

  ```js
  Math.max(...[14, 3, 77])
  ```

- push 函数，将一个数组添加到另一个数组的尾部。

  ```js
  let arr1 = [0, 1, 2];
  let arr2 = [3, 4, 5];
  arr1.push(...arr2);
  ```

- 复制数组

  ```js
  const a1 = [1, 2];
  // 写法一
  const a2 = [...a1];
  // 写法二
  const [...a2] = a1;
  ```

- **合并数组**

  ```js
  const arr1 = ['a', 'b'];
  const arr2 = ['c'];
  const arr3 = ['d', 'e'];
  [...arr1, ...arr2, ...arr3]
  // [ 'a', 'b', 'c', 'd', 'e' ]
  ```

- **与解构赋值结合**，**生成数组**

  如果将扩展运算符用于数组赋值，只能放在参数的最后一位，否则会报错。

  ```js
  const [first, ...rest] = [1, 2, 3, 4, 5];
  first // 1
  rest  // [2, 3, 4, 5]
  
  const [first, ...rest] = [];
  first // undefined
  rest  // []
  
  const [first, ...rest] = ["foo"];
  first  // "foo"
  rest   // []
  ```

- 将`字符串`转为真正的`数组`

  ```js
  [...'hello']
  // [ "h", "e", "l", "l", "o" ]
  ```

- map获取key

  ```js
  let map = new Map([
    [1, 'one'],
    [2, 'two'],
    [3, 'three'],
  ]);
  let arr = [...map.keys()]; // [1, 2, 3]
  ```

  

# 5 对象的扩展

## 5.1 属性的简洁表示法

```js
function f(x, y) {
  return {x, y};
}

// 等同于
function f(x, y) {
  return {x: x, y: y};
}
```

```js
const Person = {
  name: '张三',
  //等同于birth: birth
  birth,
}
```

## 5.2 日志的打印

输出 user 和 foo 两个对象时，就是两组键值对，可能会混淆。把它们放在大括号里面输出，就变成了对象的简洁表示法，每组键值对前面会打印对象名，这样就比较清晰

```js
let user = {
  name: 'test'
};
let foo = {
  bar: 'baz'
};

console.log(user, foo)
// {name: "test"} {bar: "baz"}
console.log({user, foo})
// {user: {name: "test"}, foo: {bar: "baz"}}
```

## 5.3 解构赋值

- 对象的`解构赋值`用于从一个对象`取值`，所有的键和它们的值，都会拷贝到新对象上面。

- 解构赋值要求等号右边是一个对象，所以如果等号右边是 undefined 或 null ，就会报错，因为它们无法转为对象。

- 解构赋值必须是最后一个参数，否则会报错。
- 解构赋值`的拷贝是`浅拷贝

```js
let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
x // 1
y // 2
z // { a: 3, b: 4 }
```

```js
let z = { a: 3, b: 4 };
let n = { ...z };
n // { a: 3, b: 4 }
```

```js
let foo = { ...['a', 'b', 'c'] };
foo
// {0: "a", 1: "b", 2: "c"}
```

- 如果扩展运算符后面是字符串，它会自动转成一个类似数组的对象

```js
{...'hello'}
// {0: "h", 1: "e", 2: "l", 3: "l", 4: "o"}
```

## 5.4 链判断运算符

```js
const firstName = message?.body?.user?.firstName || 'default';
```

?. 运算符，直接在链式调用的时候判断，左侧的对象是否为 null 或 undefined 。如果是的，就不再往下运算，而是返回 undefined 

```js
a?.b
// 等同于
a == null ? undefined : a.b
a?.[x]
// 等同于
a == null ? undefined : a[x]
a?.b()
// 等同于
a == null ? undefined : a.b()
a?.()
// 等同于
a == null ? undefined : a()

表单可能没有 checkValidity 这个方法，这时 ?. 运算符就会返回 undefined ，判断语句就变成了 undefined === false ，所以就会跳过下面的代码。
if (myForm.checkValidity?.() === false) {
  // 表单校验失败
  return;
}
```

## 5.5   ?? 判断运算符

 `||`: 左侧的值如果为 null，undefined，空字符串， false ， 0 ，默认值会生效

`??`: 左侧的值为 null 或 undefined 时，默认值会生效

```js
const headerText = response.settings.headerText || 'Hello, world!';
const headerText = response.settings.headerText ?? 'Hello, world!';
```

# 6 对象的新增方法

## 6.1 Object.is()

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

    

## 6.2 Object.assign()

Object.assign方法用于对象的合并，将源对象（source）的所有可枚举属性，复制到目标对象（target）

- `Object.assign`方法实行的是`浅拷贝`，

```js
const target = { a: 1 };
const source1 = { b: 2 };
const source2 = { c: 3 };
Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}
```

