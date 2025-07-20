---
title: Vue3跨层级通信技巧
date: 2025-07-19 15:58:34
tags:
categories: Vue
---

# 1. 从父到孙

`provide` / `inject` 是 Vue 提供的**跨层级通信机制**，不再依赖 props 层层传递；

- 父

```typescript
<script setup lang="ts">
    import { provide } from 'vue'
    const rules = { name: 'required', age: 'number' }
    provide('rules', rules)
</script>
```

- 孙

```typescript
<script setup lang="ts">
    import { inject } from 'vue'
    const rules = inject('rules')
</script>
```



