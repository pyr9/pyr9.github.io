---
title: hooks的使用
date: 2022-05-12 23:23:38
tags: hook 
categories: React
---
# useRef
案例效果：
- 点击按钮，文本title修改
- 点击按钮，input聚焦
```js
import React, { useRef, PureComponent } from 'react'

class TestChild extends PureComponent {
  render() {
    return (
      <div>UseRefDemo</div>
    )
  }
}

export default function UseRefDemo() {
  const titleRef = useRef()
  const inputRef = useRef()
  const testChild = useRef()
  return (
    <>
    <div>UseRefDemo</div>
    <div ref= {titleRef}>title</div>
    <input ref = {inputRef} type = "text"></input>
    <button onClick={e => changeDom()}>按钮</button>
    <TestChild ref = {testChild}></TestChild>
    </>
  )

function changeDom(){
    titleRef.current.innerHTML = "改变后的title"
    inputRef.current.focus()
    console.log(testChild)
  }
}
```

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h261x90pltj20kc0cqmxn.jpg )
