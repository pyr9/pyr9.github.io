---
title: Pinia持久化存储
date: 2025-10-21 19:32:45
tags:
categories: Vue
---

为了在页面刷新后保留状态，我们可以使用 `pinia-plugin-persistedstate` 插件来实现状态持久化存储。

### 1. 安装持久化插件pinia-plugin-persistedstate

#### 1. 安装依赖

```plain
npm install pinia-plugin-persistedstate
```

#### 2. 将插件添加到 pinia 实例上

在 `main.js` 中添加插件配置。

```typescript
import { createPinia } from 'pinia';
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate';

const store = createPinia();

store.use(piniaPluginPersistedstate);
```

### 2. 用法

#### 1. 基本使用

在 store 中启用状态持久化。

```typescript
import { defineStore } from 'pinia';
export const useUserStore = defineStore(
  'user',
  () => {
    const token = ref<string>();
    const setToken = (value: string) => {
      token.value = value;
    };
    const clearToken = () => {
      token.value = void 0;
    };
    return {
      token,
      setToken,
      clearToken,
    };
  },
  {
    persist: true,
  },
);
```

#### 2. 高级使用

可以自定义持久化的配置，如存储的 key 和使用的存储方式。

```typescript
import { defineStore } from 'pinia';
export const useUserStore = defineStore(
  'user',
  () => {
    const token = ref<string>();
    const setToken = (value: string) => {
      token.value = value;
    };
    const clearToken = () => {
      token.value = void 0;
    };
    return {
      token,
      setToken,
      clearToken,
    };
  },
 {
    persist: {
      key: 'userStore',
      storage: localStorage,
    },
  },
);
```
