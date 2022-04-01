---
title: react基础
date: 2022-04-04 16:03:29
tags: 组件
categories: react 
---

# react组件基础

## 组件概念

- 使用 React 可以将一些简短、独立的代码片段组合成复杂的 UI 界面，这些代码片段被称作“组件”。
- 所谓组件，即封装起来的具有独立功能的UI部件。组件就是页面上的一部分，大大小小的各种组件拼在一起就变成了一个完整的页面，就像我们玩的拼图，需要一块一块的拼接在一起才能变成一副完整的拼图。

## 函数组件

### 概念

使用 JS 的函数（或箭头函数）创建的组件，就叫做`函数组件`

```react
function HelloComponent(){
  return (
    <div>hello Componnet</div>
  )
}
```

- 组件的名称**必须首字母大写**，react内部会根据这个来判断是组件还是普通的HTML标签

- 函数组件**必须有返回值**，表示该组件的 UI 结构；如果不需要渲染任何内容，则返回 null

- 组件就像 HTML 标签一样可以被渲染到页面中。组件表示的是一段结构内容，对于函数组件来说，渲染的内容是函数的**返回值**就是对应的内容

- 使用函数名称作为组件标签名称，可以成对出现也可以自闭合

  ```react
  function App() {
    return (
      <div>
      <HelloComponent/>
      <HelloComponent></HelloComponent>
      </div>
    );
  }
  ```

## 类组件

使用 ES6 的 class 创建的组件，叫做类（class）组件

```react
class HelloComponent2 extends React.Component{
  //必须有一个render方法
  //在这个方法里有return UI 结构
  render(){
    return (
      <div>hello Componnet2</div>
    )
  }
}
```

- **类名称也必须以大写字母开头**
- 类组件应该继承 React.Component 父类，从而使用父类中提供的方法或属性
- 类组件必须提供 render 方法**render 方法必须有返回值，表示该组件的 UI 结构**

## 事件绑定

react事件采用驼峰命名法

```react
// 事件触发函数组件
function EventDemoClass1(){
  function handle(){
    console.log("事件触发了EventDemoClass1")
  }
  return (
    <button onClick={handle}>事件触发函数组件</button>
  )
}


//事件触发类组件
class EventDemoClass2 extends React.Component{
  clickHandler = () => {
    console.log("事件触发了EventDemoClass2")
  }
  render(){
    return (
      <div>
        <button onClick={this.clickHandler}>事件触发类组件</button>
      </div>
    )
  }
}

// 根组件
function App() {
  return (
    <div>
      <EventDemoClass1></EventDemoClass1>
     <EventDemoClass2></EventDemoClass2>
    </div>
  );
}
```

