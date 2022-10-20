---
title: Vue入门
date: 2022-09-26 15:25:04
tags: 前端
categories: Vue
---

# 1 Vue是什么？

- vue是一套JavaScript框架
- Vue可以简化Dom操作
- 响应式数据驱动

# 2 hello world

1. 导入Vue

```vue
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
```

2. 创建Vue实例对象，设置el属性和data属性
3. 使用简洁的模板语法把数据渲染到页面上

```vue
<body>
  <div id="app">
    {{message}}
  </div>
  <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
  <script>
   var app = new Vue({
     el:"#app",
     data:{
       message: "hello world!"
     }
   })
  </script>
</body>
```

# 3 el挂载点

- el是用来设置**vue实例挂载**的元素

- Vue会管理el选项**命中的元素**以及其内部的**后代元素**

- 可以使用其他的选择器，但是**建议使用id选择器**

- 可以使用其他的双标签，不能使用HTML或者body

# 4 data数据对象

- vue中用到的对象定义在data中
- Data中可以写复杂类型的数据，比如对象，数组
- 渲染复杂类型的数据，遵循js的语法即可

```vue
<body>
  <div id="app">
    {{message}}
    {{school.name}}
    {{school.age}}
    <ul>
      <li>{{hoobies[0]}}</li>
      <li>{{hoobies[2]}}</li>
    </ul>
  </div>
  <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
  <script>
    var app = new Vue({
      el: "#app",
      data: {
        message: "hello world!",
        school: {
          name: "张三",
          age: 14
        },
        hoobies: ["唱歌", "跳舞", "弹琴"]
      }
    })
  </script>
</body>
```



 
