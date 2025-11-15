---
title: Vue常见面试题
date: 2025-10-20 19:06:55
tags:
categories: Vue
---

## 1. Vue3

### 1.1. Vue 3 的主要新特性是什么？

Vue 3 的主要新特性包括：

- **Composition API**：提供更灵活的代码组织方式。
- **性能优化**：使用 Proxy 实现响应式，性能更好。
- **Tree-shaking**：支持按需引入，减小打包体积。
- **TypeScript 支持**：对 TypeScript 的支持更好。
- **Fragment、Teleport、Suspense**：新增内置组件。

### 1.2. Vue3 Composition API 解决的问题

Composition API 是 Vue 3 引入的新特性，它的核心思想是：**基于逻辑功能来组织代码，而不是基于选项类型**。主要解决了 Options API 在复杂组件中面临的几个主要问题：

#### 1.2.1. 解决 “逻辑碎片化” 问题，优化代码组织

Options API 中，逻辑需分散在 data、methods、computed 等不同选项中，当组件功能复杂（如同时包含表单验证、数据请求、状态管理）时，相关代码会被 “拆分” 到各个选项，形成 “碎片化” 结构 —— 开发者需在多个选项间跳转才能理解完整逻辑。
Composition API 允许将同一功能的相关代码（数据、方法、计算属性）聚合在一起，通过 setup 函数（或 `<script setup>` 语法糖）按 “逻辑关注点” 组织，而非按 “选项类型” 拆分。

#### 1.2.2. 解决 “逻辑复用” 难题，替代复杂的 mixin

Options API 中，逻辑复用主要依赖 mixin，但存在三大痛点：

- 命名冲突：多个 mixin 可能定义同名的 data 或 methods，合并时会覆盖且难以排查；
- 来源不明：组件中使用的属性 / 方法，无法快速判断是来自自身还是某个 mixin；
- 依赖模糊：mixin 与组件间可能存在隐式依赖（如 mixin 依赖组件的某个 data），维护时容易出错

Composition API 通过组合函数（Composables） 实现逻辑复用：

- 组合函数是返回响应式状态和方法的普通函数（如 useUserData()、useFormValidation()），可直接在组件中调用；
- 不存在命名冲突：组合函数返回的内容需显式解构或赋值给变量，名称由组件自主控制；
-  来源清晰：组件中使用的逻辑可直接追溯到对应的组合函数，依赖关系透明；
- 灵活性高：可根据需求选择性引入部分逻辑，无需像 mixin 那样 “全量混入”。

#### 1.2.3. 提升 TypeScript 支持，解决类型推导困难

Options API 的 “选项式” 结构与 TypeScript 的 “类型系统” 天然适配性差，对 TypeScript 的类型推断不太友好。

- 计算属性、方法返回值需要手动声明类型。
- methods 中的 this 指向模糊，TypeScript 难以准确推断 this 上的属性和方法类型，需频繁使用 this as 类型 断言，代码冗余且易出错。

Composition API 天生对 TypeScript 友好：

- 响应式数据（ref/reactive）可自动推导类型，无需额外定义接口（如 const count = ref(0) 会自动推导为 Ref 类型）；
-  setup 函数中无 this 依赖，所有变量和方法均通过显式定义或函数返回获取，TypeScript 可精准推导类型，减少类型断言，提升开发体验和代码健壮性。

### 1.3. reactive() 和 ref() 对比

实际开发中，`ref()` 因灵活性更高（支持所有类型），使用频率通常高于 `reactive()`。

- `reactive()` **不支持基本类型**：若传入 `number`、`string` 等，会返回原始值且不具备响应性（Vue 会警告）。
- `ref()` **无类型限制**：基本类型会被包装为 `Ref` 对象（通过 `.value` 访问），对象 / 数组会被自动转为 `reactive` 代理，兼顾灵活性。

| **维度**     | `reactive()`                      | `ref()`                              |
| ------------ | --------------------------------- | ------------------------------------ |
| 数据类型支持 | 仅对象 / 数组                     | 所有类型（基本类型 / 对象 / 数组）   |
| 访问方式     | 直接访问属性                      | 需通过 `.value`（模板中自动解包）    |
| 解构响应性   | 解构后丢失响应性                  | 需配合 `toRefs()` 保持响应性         |
| 重新赋值     | 直接赋值会丢失响应性              | 通过 `.value` 重新赋值，响应性保留   |
| 适用场景     | 复杂对象 / 数组，无需整体重新赋值 | 基本类型、需重新赋值的变量、通用场景 |



### 1.4. Vue 3 中的 `provide` 和 `inject` 是什么？

`provide` 和 `inject` 用于跨层级组件通信。

```typescript
 <!-- 父组件 -->
import { provide, ref } from 'vue';

export default {
  setup() {
    const message = ref('Hello from parent');
    provide('message', message); // 提供数据
    }
};

// 子组件
import { inject } from 'vue';

export default {
  setup() {
    const message = inject('message'); // 注入数据
    return { message };
  }
};
```

### 1.5.  Vue 3 中的 `v-bind` 和 `v-model` 有什么区别？

- `v-bind`：适用于 **非表单元素的属性绑定** 或 **表单元素的单向数据展示**。例如：

- - 绑定图片的 `src`、链接的 `href`、元素的 `style`/`class` 等。
  - 给组件传递 props（父组件向子组件传值）。

- `v-modes`：仅适用于 **表单元素或自定义组件**，用于处理用户输入交互，实现数据的双向同步。例如：

- - 文本输入框（input [type="text"]）、多行文本框（textarea）。
  - 复选框（input [type="checkbox"]）、单选按钮（input [type="radio"]）。
  - 下拉选择框（select）。
  - 自定义组件（通过 `modelValue` 和 `update:modelValue` 事件实现双向绑定）。

简单来说，`v-bind` 用于 “展示数据”，`v-model` 用于 “交互数据”，根据是否需要用户输入反向影响数据来选择使用。

| **特性** | `v-bind`             | `v-model`                |
| -------- | -------------------- | ------------------------ |
| 绑定方向 | 单向（数据 → DOM）   | 双向（数据 ↔ DOM）       |
| 适用元素 | 所有元素（属性绑定） | 表单元素、自定义组件     |
| 功能     | 绑定属性值           | 同步用户输入与数据       |
| 语法     | `:属性="数据"`       | `v-model="数据"`         |
| 本质     | 单一属性绑定         | `v-bind + v-on` 的语法糖 |

### 1.6. 状态管理库 Pinia是什么？

Pinia 是 Vue 3 的一个状态管理库，它专门用于管理 Vue 应用中跨组件、跨页面共享的状态（如用户信息）。相比 Vuex，Pinia 提供了更简洁的 API 和更好的 TypeScript 支持。

- **Vuex**：必须通过 `mutations` 修改状态，通过 `getters` 获取值（或直接访问 `state.token`）。
- **Pinia**：可直接访问 `token` 状态值（`tokenStore.token`），修改通过普通方法直接操作 `token.value`，更简洁。

**a. Vuex 4 写法**

Vuex 的核心文件是 `store`，其典型结构包含以下部分：

- State：存储全局共享数据，类似组件的 `data`。
- Getters：从 `state` 派生计算属性，对状态进行加工或过滤。
- Mutations：修改 `state` 的唯一途径，必须是同步函数。
- Actions：处理异步操作，通过提交 `mutations` 修改状态。
- Modules：将状态分模块管理，适合大型项目。

**typescript**

```typescript
import { createStore } from 'vuex'

export const store = createStore({
  state: {
    token: '' // 初始为空字符串
  },
  mutations: {
    setToken(state, newToken: string) {
      state.token = newToken
    }
  },
  actions: {
    // 异步获取token
    async generateToken({ commit }) {
      // 模拟简单接口请求
      const mockToken = await new Promise<string>(resolve => {
        setTimeout(() => resolve('mock_token_123'), 500)
      })
      commit('setToken', mockToken)
    }
  },
  getters: {
    getToken(state) {
      return state.token
    }
  }
})

// 组件中使用
// const store = useStore()
// store.commit('setToken', 'abc123') // 设置token
// console.log(store.getters.getToken) // 获取token值
```

**b. Pinia 写法**

**typescript**

```typescript
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useTokenStore = defineStore('token', () => {
  const token = ref<string>('') // 初始为空字符串

  const setToken = (newToken: string) => {
    token.value = newToken
  }
  
  // 最基础的异步获取token
  const generateToken = async () => {
    // 模拟简单接口请求
    const mockToken = await new Promise<string>(resolve => {
      setTimeout(() => resolve('mock_token_123'), 500)
    })
    token.value = mockToken
  }
  return { token, setToken }
})

// 组件中使用
// const tokenStore = useTokenStore()
// tokenStore.setToken('abc123') // 设置token
// console.log(tokenStore.token) // 直接获取token值（或 tokenStore.getToken）
```



### 1.7. 父子组件传参、跨层级传参？

![img](https://panyuro.oss-cn-beijing.aliyuncs.com/1747878513386-ac8cbf03-1e14-47bf-9884-83afd75e384d.png)

### 1.8. Vue 生命周期

1. **beforeCreate**：实例初始化之后，数据观测(data observer)和event/watcher事件配置之前被调用。vue3 setup() 可以替代它。
2. **created**：实例创建完成后被调用。此时已完成数据观测(data observer)，属性和方法的运算，watch/event事件回调。但是尚未挂载到DOM上。vue3 setup() 可以替代它
3. **beforeMount**：在挂载开始之前被调用：相关的render函数首次被调用。
4. **mounted**：el 被新创建的 vm.$el 替换，并挂载到实例上去之后调用该钩子。
5. **beforeUpdate**：数据更新时调用，发生在虚拟DOM重新渲染和打补丁之前。你可以在这个钩子里进一步地更改状态，这不会触发附加的重渲染过程。
6. **updated**：由于数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子。当这个钩子被调用时，组件DOM已经更新，所以你现在可以执行依赖于DOM的操作。
7. **beforeDestroy**（现在称为 `beforeUnmount` 在 Vue 3）：实例销毁之前调用。在这一步，实例仍然完全可用。
8. **destroyed**（现在称为 `unmounted` 在 Vue 3）：实例销毁后调用。调用后，所有的事件监听器会被移除，所有的子实例也会被销毁。

### 1.9. computed、watch（自动监听、深度监听）、methods区别？

- `computed`：适合于依赖其他状态的状态计算，具有缓存机制，避免不必要的重复计算。
- `watch`：用于监听特定数据的变化并做出响应，特别适用于需要执行副作用（如网络请求）的情况。可以通过设置选项实现深度监听。
- `methods`：提供了一种直接定义可调用逻辑的方式，适用于事件处理器和其他不需要缓存的操作。

选择合适的方式来组织你的逻辑取决于具体的使用场景。例如，当你有一个基于其他状态的值需要频繁访问时，`computed` 是最好的选择；而当你需要对某个状态的变化做出反应，尤其是涉及到异步操作时，`watch` 更加适用；对于简单的事件处理或不需要缓存的操作，则可以直接使用 `methods`。

### 1.10.  useRouter 和 useRoute 有啥区别？

1. **useRouter**

- **作用**：获取全局路由实例（`router` 实例），用于执行路由跳转、路由配置修改等**全局路由操作**。
- **返回值**：Vue Router 的实例对象，包含路由的核心方法（如 `push`、`replace`、`go` 等）。
- **常用方法：**

- - `router.push(location)`：跳转到指定路由（类似 `window.history.pushState`）。
  - `router.replace(location)`：替换当前路由（类似 `window.history.replaceState`，不会留下历史记录）。

1. **useRoute**

- **作用**：获取当前激活的路由信息对象（`route` 对象），用于访问当前路由的参数、路径、查询参数等**路由状态信息**。
- **返回值**：当前路由的信息对象，包含 `params`、`query`、`path`、`name` 等属性。
- **常用属性：**

- - `route.params`：路由动态参数（如 `/user/:id` 中的 `id`）。
  - `route.query`：URL 查询参数（如 `?page=1&size=10`）。
  - `route.path`：当前路由的路径（字符串，如 `/user/123`）。

### 1.11. 组件插槽`v-slot` 是什么？什么场景用？

在 Vue.js 中，插槽（Slots）提供了一种内容分发机制，允许你在组件内部定义可替换的内容区域。通过使用插槽，父组件可以向子组件传递内容，从而使组件更加灵活和可复用。

### 1.12. Vue要做权限管理该怎么做？控制到按钮级别的权限怎么做？

总的来说，所有的请求发起都触发自前端路由或视图

所以我们可以从这两方面入手，对触发权限的源头进行控制，最终要实现的目标是：

-  路由方面，用户登录后只能看到自己有权访问的导航菜单，也只能访问自己有权访问的路由地址，否则将跳转 4xx 提示页 -> 菜单和路由都由后端返回
-  视图方面，用户只能看到自己有权浏览的内容和有权操作的控件 -> 全局路由守卫里，每次路由跳转都要做权限判断。
- 接口权限：接口权限目前一般采用`jwt`的形式来验证，没有通过的话一般返回`401`，跳转到登录页面重新进行登录

​        登录完拿到`token`，将`token`存起来，通过`axios`请求拦截器进行拦截，每次请求的时候头部携带`token`



• 最后再加上请求控制作为最后一道防线，路由可能配置失误，按钮可能忘了加权限，这种时候请求控制可以用来兜底，越权请求将在前端被拦截

### 1.13. v-if 和 v-show 有什么区别？

`v-if` 是“真正”的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建， 操作的实际上是 dom 元素的创建或销毁。

`v-show` 就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换， 它操作的是`display:none/block`属性。

一般来说， v-if 有更高的切换开销，而 v-show 有更高的初始渲染开销。因此，如果需要非常**频繁地切换**，则使用` v-show` 较好； 如果在运行时条件很少改变，则使用 v-if 较好。

### 1.14.  lodash常见的 API ？

[官网](https://link.juejin.cn?target=https%3A%2F%2Fwww.lodashjs.com%2F)

Lodash 是一个一致性、模块化、高性能的 JavaScript 实用工具库。 _.cloneDeep 深度拷贝 _.reject 根据条件去除某个元素。 _.drop(array, [n=1] ) 作用：将 array 中的前 n 个元素去掉，然后返回剩余的部分.

### 1.15. cookie 、localstorage 、 sessionstrorage 之间有什么区别？

- 与服务器交互： 

- - cookie 是网站为了标示用户身份而储存在用户本地终端上的数据（通常经过加密）
  - cookie 始终会在同源 http 请求头中携带（即使不需要），在浏览器和服务器间来回传递
  - sessionStorage 和 localStorage 不会自动把数据发给服务器，仅在本地保存

- 存储大小：
- cookie 数据根据不同浏览器限制，大小一般不能超过 4k
- sessionStorage 和 localStorage 虽然也有存储大小的限制，但比 cookie 大得多，可以达到 5M 或更大
- 有期时间： 

- - ocalStorage 存储持久数据，浏览器关闭后数据不丢失除非主动删除数据
  - sessionStorage 数据在当前浏览器窗口关闭后自动删除
  - cookie 设置的 cookie 过期时间之前一直有效，与浏览器是否关闭无关

- **优先选 Cookie**：需要与服务器交互（如认证）、需自动过期、需严格限制作用域的场景。
- **优先选 LocalStorage**：纯前端存储、需大容量、需长期保留的非敏感数据场景。

## 2. Javascript

### 2.1. == 和 ===区别，分别在什么情况使用？

等于操作符用两个等于号（ == ）表示，如果操作数相等，则会返回 `true`

前面文章，我们提到在`JavaScript`中存在隐式转换。等于操作符（==）在比较中会先进行类型转换，再确定操作数是否相等

全等操作符由 3 个等于号（ === ）表示，只有两个操作数在不转换的前提下相等才返回 `true`。即类型相同，值也需相同

## 3. CSS

### 3.1. 响应式布局有哪些方案？

| **方法**                                  | **说明**                                            | **优点**             | **缺点**                  |
| ----------------------------------------- | --------------------------------------------------- | -------------------- | ------------------------- |
| **媒体查询（Media Queries）**             | 使用 CSS 的 `@media` 查询不同屏幕尺寸，应用不同样式 | 简单、原生支持       | 需要手动维护多套样式      |
| **弹性布局（Flexbox）**                   | 使用 `display: flex` 实现灵活的布局排列             | 简洁、易用           | 主要适用于一维布局        |
| **网格布局（Grid）**                      | 使用 `display: grid` 实现二维布局                   | 强大、结构清晰       | 学习成本略高              |
| **百分比 / vw/vh 单位**                   | 使用相对单位（如 `width: 100%`）布局                | 适应性强             | 需注意容器嵌套            |
| **rem/vw 动态适配字体大小**               | 动态设置根字体大小，适配不同设备                    | 字体、间距统一适配   | 需配合 JS 或 CSS 预处理器 |
| **CSS 框架（如 Bootstrap、Tailwind）**    | 使用成熟的响应式框架快速开发                        | 快速上手、功能丰富   | 可能引入冗余代码          |
| **JavaScript 动态控制（如 resize 事件）** | 通过 JS 监听窗口变化，动态调整布局                  | 灵活、可实现复杂交互 | 不推荐用于基础布局        |

### 3.2. flexbox（弹性盒布局模型）,以及适用场景？

`Flexible Box` 简称 `flex`，意为”弹性布局”，可以简便、完整、响应式地实现各种页面布局

采用Flex布局的元素，称为`flex`容器`container`

它的所有子元素自动成为容器成员，称为`flex`项目`item`

容器中默认存在两条轴，主轴和交叉轴，呈90度关系。项目默认沿主轴排列，通过`flex-direction`来决定主轴的方向

每根轴都有起点和终点，这对于元素的对齐非常重要

关于`flex`常用的属性，我们可以划分为容器属性和容器成员属性

容器属性有：

- flex-direction
- flex-wrap
- flex-flow
- justify-content
- align-items
- align-content

### 3.3. css选择器有哪些？优先级？

内联 > ID选择器 > 类选择器 > 标签选择器

关于css属性选择器常用的有：

• id选择器（#box），选择id为box的元素

• 类选择器（.one），选择类名为one的所有元素

• 标签选择器（div），选择标签为div的所有元素

• 后代选择器（#box div），选择id为box元素内部所有的div元素

• 子选择器（.one>one_1），选择父元素为.one的所有.one_1的元素

• 相邻同胞选择器（.one+.two），选择紧接在.one之后的所有.two元素

• 群组选择器（div,p），选择div、p的所有元素

### 3.4. `z-index` 的作用是什么？它的取值范围和默认值是什么？

- **作用**：控制元素的堆叠顺序（层叠上下文），值越大越靠前。
- **取值范围**：任意整数（正数、负数、0），默认值为 `auto`（相当于 `0`）。
- **注意**：仅对定位元素（`position` 非 `static`）生效，且会创建新的层叠上下文（如 `opacity < 1`、`transform` 等属性也会触发）。

### 3.5. `position: relative` 和 `position: absolute` 的核心区别是什么？

| **特性**           | `**position: relative**`                        | `**position: absolute**`                       |
| ------------------ | ----------------------------------------------- | ---------------------------------------------- |
| **是否脱离文档流** | 否（保留原空间）                                | 是（脱离文档流，不占据空间）                   |
| **定位基准**       | 相对于自身原始位置偏移                          | 相对于最近的非 `static` 定位祖先元素           |
| **常见用途**       | 微调元素位置（如对齐、覆盖）                    | 创建浮动层（如弹窗、下拉菜单）                 |
| **对子元素的影响** | 子元素仍以父元素为基准（若子元素为 `absolute`） | 子元素会以该父元素为定位基准（形成定位上下文） |

### 
