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

## 5 模版语法

-  ` {{ message }}` 展示字符串文本
- `v-bind:class="'btnb tn-sm'"` 绑定属性

<img src="/Users/panyurou/Library/Application Support/typora-user-images/image-20221023232100807.png" alt="image-20221023232100807" style="zoom:50%;" />

- `v-html`: 翻译成html标签
- `{{ ok ? 'YES':'No' }}` 三元运算
- `{{ message.spilt('').join(',') }}`:  vue支持js表达式



- V-if 控制元素是否切换

```vue
<div id="app-3">
  <p v-if="seen">现在你看到我了</p>
</div>
```

```js
var app3 = new Vue({
  el: '#app-3',
  data: {
    seen: true
  }
})
```

- @click 绑定事件

<img src="/Users/panyurou/Library/Application Support/typora-user-images/image-20221023231840513.png" alt="image-20221023231840513" style="zoom:50%;" />

 

![image-20221023232442286](/Users/panyurou/Library/Application Support/typora-user-images/image-20221023232442286.png)





- 可以定义请求返回的数据的格式

```
data: function () {
    return {
        isEdit: false,
        businessDataFromApi: null
    }
}

 getBusinessDataByQueryApi() {
            if (this.queryApi !== undefined) {
                let url = `${this.$store.state[this.entity].baseModuleUrl}/${this.entity}/` + this.queryApi;
                this.$store.dispatch('api/post', {
                    url: url,
                    data: {
                        idFilter: this.businessData.id === undefined ? this.$route.query.id : this.businessData.id
                    }
                }).then(response => {
                    if (response.data.list.length === 1) {
                        this.businessDataFromApi = response.data.list[0];
                    }
                })
            }
        }
```

- ![image-20221024231200647](/Users/panyurou/Library/Application Support/typora-user-images/image-20221024231200647.png)

![image-20221024231215526](/Users/panyurou/Library/Application Support/typora-user-images/image-20221024231215526.png)

- 

- ```
  handleClick(col, $event) 
  ```

- @click="handleClick()" 和@click="handleClick"的区别在于：第一个拿不到event事件，如果要拿到事件，需要写成@click="handleClick($event)"

- 阻止默认行为@click.prevent

- @click.once 只执行一次事件触发

- @click.self 只有e.target = e.currentTarget的时候才会执行

  <img src="/Users/panyurou/Library/Application Support/typora-user-images/image-20221108210854373.png" alt="image-20221108210854373" style="zoom: 33%;" /> 
