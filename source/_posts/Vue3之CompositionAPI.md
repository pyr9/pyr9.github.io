---
title: Vue3之CompositionAPI
date: 2025-10-15 19:09:10
tags:
categories: Vue
---

Composition API 是 Vue 3 引入的新特性，它的核心思想是：**基于逻辑功能来组织代码，而不是基于选项类型**。

Composition API主要解决了 Options API 在复杂组件中面临的几个主要问题：

## 1. 解决逻辑碎片化，优化代码组织

### 1.1. 问题

Options API 中，逻辑需分散在 `data`、`methods`、`computed` 等不同选项中，当组件功能复杂（如同时包含表单验证、数据请求、状态管理）时，相关代码会被 “拆分” 到各个选项，形成 “碎片化” 结构 —— 开发者需在多个选项间跳转才能理解完整逻辑。

### 1.2. 解决

Composition API 允许将**同一功能的相关代码（数据、方法、计算属性）聚合在一起**，通过 `setup` 函数（或 `<script setup>` 语法糖）按 “逻辑关注点” 组织，而非按 “选项类型” 拆分。

### 1.3. 例子说明

一个简单的**计数器**和一个**搜索框，**对比 Options API 和 Composition API。

#### 1.3.1. **vue2 Options API  实现**

**在 Options API 中**，代码会是这样混杂的：

- 两个功能的代码像打散了的拼图一样混在一起。

```typescript
export default {
  data() {
    return {
      count: 0,        // 计数器功能
      searchQuery: ''  // 搜索功能
    }
  },
  methods: {
    increment() { ... }, // 计数器
    decrement() { ... }, // 计数器
    reset() { ... },     // 计数器
    // 搜索功能没有方法，它的逻辑在watch里
  },
  computed: {
    searchUpperCase() { ... } // 搜索功能
  },
  watch: {
    searchQuery() { ... } // 搜索功能
  }
}
```

#### 1.3.2. **vue3** **Composition API 实现**

**在 Composition API 中**，代码结构非常清晰：

- 你可以一眼看出这个组件由哪两大功能组成，并且修改或删除其中一个功能时，完全不会影响到另一个。

```typescript
<script setup>
  import { ref, computed, watch } from 'vue';

// ------ 计数器功能逻辑块 ------
const count = ref(0);
const increment = () => { count.value++; };
const decrement = () => { count.value--; };
const reset = () => { count.value = 0; };

// ------ 搜索功能逻辑块 ------
const searchQuery = ref('');
const searchUpperCase = computed(() => { ... });
watch(searchQuery, (newQuery) => { ... });

</script>
```

## 2. 解决逻辑复用，替代复杂的 mixin

### 2.1. 问题

在 Vue 2 Options API 中，逻辑复用主要依赖 `mixin`，但存在三大痛点：

- **命名冲突**：多个 mixin 可能定义同名的 `data` 或 `methods`，合并时会覆盖且难以排查；
- **来源不明**：组件中使用的属性 / 方法，无法快速判断是来自自身还是某个 mixin；
- **依赖模糊**：mixin 与组件间可能存在隐式依赖（如 mixin 依赖组件的某个 `data`），维护时容易出错。

### 2.2. 解决

- 组合函数是返回响应式状态和方法的普通函数（如 `useUserData()`、`useFormValidation()`），可直接在组件中调用；
- 不存在命名冲突：组合函数返回的内容需显式解构或赋值给变量，名称由组件自主控制；
- 来源清晰：组件中使用的逻辑可直接追溯到对应的组合函数，依赖关系透明；
- 灵活性高：可根据需求选择性引入部分逻辑，无需像 mixin 那样 “全量混入”。

### 2.3. 例子说明

以计数器为例子

#### 2.3.1. **vue2** mixins 实现

```typescript
<template>
  <div>
    <h3>Component A</h3>
    <p>Count: {{ Count }}</p>
    <button @click="Increment">+1</button>
    <button @click="Decrement">-1</button>
    <button @click="Reset">Reset</button>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import { counterMixin } from '@/mixins/counterMixin'

export default defineComponent({
  mixins: [counterMixin],
  
  mounted() {
    // 使用默认配置
    this.InitCounter()
  }
})
</script>
import { Component, Vue } from 'vue-facing-decorator' // 或者使用传统的 Options API

// 使用 Options API 方式的 Mixin
export const counterMixin = {
  data() {
    return {
      count: 0,
      min: -Infinity,
      max: Infinity
    }
  },
  
  computed: {
    isMin() {
      return this.count <= this.min
    },
    isMax() {
      return this.count >= this.max
    },
    isEven() {
      return this.count % 2 === 0
    }
  },
  
  methods: {
    Increment() {
      if (this.count < this.max) {
        this.count++
      }
    },
    
    Decrement() {
      if (this.count > this.min) {
        this.count--
      }
    },
    
    Reset() {
      this.count = 0
    },
    
    SetValue(value: number) {
      if (value >= this.min && value <= this.max) {
        this.count = value
      }
    },
    
    // 初始化配置的方法
    initCounter(options: { min?: number; max?: number; initialValue?: number } = {}) {
      this.min = options.min ?? -Infinity
      this.max = options.max ?? Infinity
      this.count = options.initialValue ?? 0
    }
  },
  
  mounted() {
    console.log('Counter mixin mounted')
  }
}
```

#### 2.3.2. **vue3** Composition API 实现

```typescript
<template>
  <div>
    <h3>Component A</h3>
    <p>Count: {{ count }}</p>
    <button @click="increment">+1</button>
    <button @click="decrement">-1</button>
    <button @click="reset">Reset</button>
    <p v-if="isEven">Even number!</p>
  </div>
</template>

<script setup lang="ts">
import { useCounter } from '@/composables/useCounter'

// 使用默认配置   每个组件可以只使用需要的部分功能, 通过解构重命名返回值
const { count, increment, decrement, reset, isEven } = useCounter()
</script>
import { ref, computed } from 'vue'

interface UseCounterOptions {
  initialValue?: number
  min?: number
  max?: number
}

export function useCounter(options: UseCounterOptions = {}) {
  const { initialValue = 0, min = -Infinity, max = Infinity } = options

  const count = ref(initialValue)

  const isMin = computed(() => count.value <= min)
  const isMax = computed(() => count.value >= max)
  const isEven = computed(() => count.value % 2 === 0)

  function increment() {
    if (count.value < max) {
      count.value++
    }
  }

  function decrement() {
    if (count.value > min) {
      count.value--
    }
  }

  function reset() {
    count.value = initialValue
  }

  function setValue(value: number) {
    if (value >= min && value <= max) {
      count.value = value
    }
  }

  return {
    // 状态
    count,
    
    // 计算属性
    isMin,
    isMax,
    isEven,
    
    // 方法
    increment,
    decrement,
    reset,
    setValue
  }
}
```



## 3. 解决类型推导困难，提升 TypeScript 支持

### 3.1. 问题

Options API 的 “选项式” 结构与 TypeScript 的 “类型系统” 天然适配性差，对 TypeScript 的类型推断不太友好。

- 计算属性、方法返回值需要手动声明类型
- `methods` 中的 `this` 指向模糊，TypeScript 难以准确推断 `this` 上的属性和方法类型，需频繁使用 `this as 类型` 断言，代码冗余且易出错。

### 3.2. 解决

Composition API 天生对 TypeScript 友好：

- 响应式数据（`ref`/`reactive`）可自动推导类型，无需额外定义接口（如 `const count = ref(0)` 会自动推导为 `Ref<number>` 类型）；
- `setup` 函数中无 `this` 依赖，所有变量和方法均通过显式定义或函数返回获取，TypeScript 可精准推导类型，减少类型断言，提升开发体验和代码健壮性。

### 3.3. 例子说明

#### 3.3.1. vue2

```typescript
<script lang="ts">
  import { defineComponent } from 'vue'
  import type { User, UserFormData } from '@/types/user'

  export default defineComponent({
    data() {
      return {
        // 需要重复定义类型
        user: null as User | null,
        formData: {
          name: '',
          email: '',
          role: 'user',
          password: ''
        } as UserFormData,
        loading: false
      }
    },

    computed: {
      // 计算属性的类型推断有限
      isFormValid(): boolean {
        return this.formData.name.length > 0 && 
          this.formData.email.includes('@') &&
          this.formData.password.length >= 6
      },

      // 需要手动声明返回类型
      userAgeDescription(): string {
        return this.user?.age ? `Age: ${this.user.age}` : 'Age not specified'
      }
    },

    methods: {
      // 参数和返回值类型需要手动声明
      async submitForm(): Promise<void> {
        this.loading = true
      try {
    // this.formData 的类型提示有限
    const response = await api.createUser(this.formData)
    this.user = response.data
  } catch (error) {
    console.error('Failed to create user', error)
  } finally {
    this.loading = false
  }
  },

  // 方法间的类型共享困难
  updateUser(updates: Partial<User>): void {
    if (this.user) {
      this.user = { ...this.user, ...updates }
    }
  }
 },

  mounted() {
    // 生命周期中的 this 类型提示不完整
    this.loadUser(1)
  }
  })
</script>
```



#### 3.3.2. vue3



```typescript
import { ref, computed, reactive } from 'vue'
import type { User, UserFormData } from '@/types/user'
import { userApi } from '@/api/userApi'

interface UseUserFormOptions {
  initialRole?: User['role']
}

export function useUserForm(options: UseUserFormOptions = {}) {
  const { initialRole = 'user' } = options

  // 响应式数据 with explicit types
  const user = ref<User | null>(null)
  const loading = ref(false)
  const error = ref<string | null>(null)

  // 表单数据 with type inference
  const formData = reactive<UserFormData>({
    name: '',
    email: '',
    role: initialRole,
    password: ''
  })

  // 计算属性 with full type inference
  const isFormValid = computed(() => {
    return formData.name.length > 0 && 
           formData.email.includes('@') &&
           formData.password.length >= 6
  })


  // 异步方法 with proper typing
  async function submitForm(): Promise<User> {
    loading.value = true
    error.value = null
    
    try {
      const response = await userApi.createUser(formData)
      user.value = response.data
      return response.data
    } catch (err) {
      error.value = 'Failed to create user'
      throw err
    } finally {
      loading.value = false
    }
  }


  return {
    // State
    user,
    formData,
    loading,
    error,
    
    // Computed
    isFormValid,
    userAgeDescription,
    
    // Methods
    submitForm,
  }
}
```
