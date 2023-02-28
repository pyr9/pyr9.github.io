---
title: react初体验
date: 2022-04-27 22:52:51
tags: 概念
categories: React
---

# React是什么？

React 是一个声明式，高效且灵活的用于构建用户界面的 JavaScript 库。

> 使用最原生的HTML、CSS、JavaScript可以构建完整的用户界面吗？当然可以，但是会存在很多问题
>
> - 操作DOM兼容性的问题；
> - 过多兼容性代码的冗余问题；
> - 代码组织和规范的问题；

# React的特点 

## 声明式编程

- 声明式编程是目前整个大前端开发的模式：Vue、React、Flutter、SwiftUI
- 它允许我们只需要维护自己的状态，当状态改变时，React可以根据最新的状态去渲染我们的UI界面；

## 组件化开发

- 组件化开发页面目前前端的流行趋势

## 多平台适配

- 2013年，React发布之初主要是开发Web页面
- 2015年，Facebook推出了ReactNative，用于开发移动端跨平台（虽然目前Flutter非常火爆，但是还是有很多公司在使用ReactNative）
- 2017年，Facebook推出ReactVR，用于开发虚拟现实Web应用程序

## react 开发依赖

### 开发React必须依赖三个库：

- react：包含react所必须的核心代码
- react-dom：react渲染在不同平台所需要的核心代码
- babel：将jsx转换成React代码的工具

## 如何引入

- cdn引入
- 下载后，添加本地依赖
- npm依赖（脚手架使用）

## 认识Babel

- babel是目前前端使用非常广泛的编辑器、转移器。
- 比如当下很多浏览器并不支持ES6的语法，但是确实ES6的语法非常的简洁和方便，我们开发时希望使用它。
- 那么编写源码时我们就可以使用ES6来编写，之后通过Babel工具，将ES6转成大多数浏览器都支持的ES5的语法。

> React和Babel的关系：
>
> - 默认情况下开发React其实可以不使用babel。
> - 但是前提是我们自己使用React.createElement来编写源代码，它编写的代码非常的繁琐和可读性差。
> - 那么我们就可以直接编写jsx（JavaScriptXML）的语法，并且让babel帮助我们转换成React.createElement。

# Hello world案例

- 这里我们编写React的script代码中，必须添加type="text/babel"，作用是可以让babel解析jsx的语法
- ReactDOM.render函数：
  - 参数一：传递要渲染的内容，这个内容可以是HTML元素，也可以是React的组件
  - 参数二：将渲染的内容，挂载到哪一个HTML元素上
- 通过{}语法来引入外部的变量或者表达式

```js
<body>
  <!--添加react 依赖-->
  <!-- // crossorigin 将远程js的一些错误，显示到本地 -->
  <script src="https://unpkg.com/react@17/umd/react.development.js" crossorigin></script>
  <script src="https://unpkg.com/react-dom@17/umd/react-dom.development.js" crossorigin></script>
  <script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
<div id="app">【内容会被覆盖】</div>
<script type="text/babel">
  // render（渲染的内容，挂载的位置）
  // ReactDOM.render(<h2>hello World</h2>,document.getElementById("app"))

  let message = "hello world"
  ReactDOM.render(<h2>{message}</h2>,document.getElementById("app"))
</script>
</body>
```

![image-20230228233834887](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228233834887.png)

# HelloReact– 组件化开发

- 这里我们暂时使用类的方式封装组件：
  - 1.定义一个类（类名大写，组件的名称是必须大写的，小写会被认为是HTML元素），继承自React.Component
  - 2.实现当前组件的render函数。render当中返回的jsx内容，就是之后React会帮助我们渲染的内容

```js
<script type="text/babel">
 class App extends React.Component {
   render(){
     return (
        <h2>hello world</h2>
     )
   }
 }
 ReactDOM.render(<App/>, document.getElementById("app"))
</script>
```

## 组件化 - 数据依赖

- 组件中的数据，我们可以分成两类：
  - 参与界面更新的数据：当数据变量时，需要更新组件渲染的内容
  - 不参与界面更新的数据：当数据变量时，不需要更新将组建渲染的内容
- 参与界面更新的数据我们也可以称之为是参与数据流，这个数据是定义在当前对象的state中
  - 我们可以通过在构造函数中this.state={定义的数据}
  - 当我们的数据发生变化时，我们可以调用this.setState来更新数据，并且通知React进行update操作。
  - 在进行update操作时，就会重新调用render函数，并且使用最新的数据，来渲染界面

```react
 class App extends React.Component {
   constructor(){
     super()
     this.state = {
      message: "hello world"
     }
   }
 }
```

## 	组件化 – 事件绑定

- 在类中直接定义一个函数，并且将这个函数绑定到html原生的onClick事件上，当前这个函数的this指向默认情况下是undefined
  - 因为在正常的DOM操作中，监听点击，监听函数中的this其实是节点对象（比如说是button对象）；
  - React并不是直接渲染成真实的DOM，我们所编写的button只是一个语法糖，它的本质React的Element对象；

- 我们在绑定的函数中，可能想要使用当前对象，比如执行this.setState函数，就必须拿到当前对象的this

  - 我们就需要在传入函数时，给这个函数直接绑定this

  - 类似于下面的写法：

    ```react
    <button onClick={this.btnClick.bind(this)}>改变文本</button>
    ```

    

# 电影列表案例

```js
<script type="text/babel">
 class App extends React.Component {
   constructor(){
     super()
     this.state = {
      movies: ["大话西游", "开心农场"]
     }
   }
   render(){
     return (
       <div>
        <h2>电影列表</h2>
        <ul>
          {
            this.state.movies.map((item, index)=>{
            return (<li>{item}</li>)
            })
          }
        </ul>
       </div>
     )
   }
 }
 ReactDOM.render(<App/>, document.getElementById("app"))
</script>
```

![image-20230228233846219](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228233846219.png)

# 计数器案例

```js
<script type="text/babel">
 class App extends React.Component {
   constructor(){
     super()
     this.state = {
      count: 0
     }
   }
   render(){
     return (
       <div>
        <h2>当前计数:{this.state.count}</h2>
        <button onClick = {this.incr.bind(this)}>+1</button>
        <button onClick = {this.decr.bind(this)}>-1</button>
       </div>
     )
   }

   incr(){
     this.setState({
       count: this.state.count + 1
     })
   }

   decr(){
     this.setState({
       count: this.state.count - 1
     })
   }

 }
 ReactDOM.render(<App/>, document.getElementById("app"))
</script>
```

![image-20230228233857215](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228233857215.png)

2ED3-DF39
