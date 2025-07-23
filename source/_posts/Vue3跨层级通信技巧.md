---
title: Vue3跨层级通信技巧
date: 2025-07-19 15:58:34
tags:
categories: Vue
---

# 1. **Provide 和 Inject**

- `provide`和`inject`允许祖先组件向任意深度的后代组件提供数据，而不需要经过中间组件 props逐层传递。
- 祖先组件使用`provide`来提供数据，后代组件使用`inject`来接收这些数据。这种方式非常适合用于主题、用户偏好设置等全局或半全局的数据共享。

```java
<script setup lang="ts">
    import { provide } from 'vue'
    const rules = { name: 'required', age: 'number' }
    provide('rules', rules)
</script>
<script setup lang="ts">
    import { inject } from 'vue'
    const rules = inject('rules')
</script>
```

# 2. Pinia 

- Pinia则是Vue 3的新一代状态管理库，它更轻量且易于使用。
- 实现思路：定义store存储变量，在子组件中更新store的值，在父组件监听

```java
export const useModelStore = defineStore('model', () => {
  // 当前模型
  const currentModelInfo = ref<GetSessionListVO>(<GetSessionListVO>{});

  // 设置当前模型
  const setCurrentModelInfo = (modelInfo: GetSessionListVO) => {
    currentModelInfo.value = modelInfo;
  };
  return {
    currentModelInfo,
    setCurrentModelInfo,
  }
});
<script setup lang="ts">
  function handleClick(item: GetSessionListVO) {
    modelStore.setCurrentModelInfo(item);
  }
</script>

<template>
    <div
      class="model-select"
      :class="{ 'bg-[rgba(0,0,0,.04)] is-select': item.modelName === currentModelName }"
      @click="handleClick(item)"
    >
</template>
<script setup lang="ts">
  watch(
      () => modelStore.currentModelInfo,
      (val) => {
        if (val.modelName === modelStore.defaultModel.modelName) {
          return;
        }
        senderRef.value.openHeader();
      },
    );
</script>
```
