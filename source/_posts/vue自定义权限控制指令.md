---
title: vue自定义权限控制指令v-permission
date: 2025-10-18 20:10:09
tags:
categories: Vue
---

# 1. 指令的基本概念

在 Vue 中，**指令（Directive）** 是带有 `v-` 前缀的特殊属性，用于在模板中声明式地操作 DOM。

Vue 内置了一系列常用指令（如 `v-model`、`v-if`、`v-for`、`v-bind`、`v-on` 等），同时也支持开发者自定义指令，以满足特定业务需求。

当内置指令无法满足需求时，可以通过 `directive` 方法定义自定义指令。自定义指令分为**全局指令**和**局部指令**。

指令的语法格式：

**vue**

```vue
<元素 v-指令名:参数.修饰符="表达式"></元素>
```

- **指令名**：指令的核心标识（如 `v-focus` 中的 `focus`）。
- **参数**：可选，用于指定指令的作用目标（如 `v-bind:href` 中的 `href`）。
- **修饰符**：可选，用于修改指令的行为（如 `v-on:click.prevent` 中的 `prevent`，阻止默认行为）。
- **表达式**：指令的触发条件或依赖数据（如 `v-if="isShow"` 中的 `isShow`）。

# 2. 权限控制指令 `v-permission`

需求：根据用户权限决定元素是否显示。

## 2.1. 全局注册 v-permission 指令

```typescript
import type { App } from 'vue';
import hasPermission from './permission/hasPermission';
import hasRole from './permission/hasRole';

export default function directive(app: App) {
  app.directive('hasRole', hasRole);
  app.directive('hasPermission', hasPermission);
}
export { hasPermission, hasRole };
const app = createApp(App);
directive(app);
app.mount('#app');
import type { Directive, DirectiveBinding } from 'vue';
import { useUserStore } from '@/stores';

const hasPermission: Directive = {
  mounted(el: HTMLElement, binding: DirectiveBinding) {
    const { value } = binding;
    const all_permission = '*:*:*';
    const permissions = useUserStore().permissions;

    if (value && Array.isArray(value) && value.length > 0) {
      const permissionFlag = value;

      const hasPermissions = permissions.some((permission: string) => {
        return all_permission === permission || permissionFlag.includes(permission);
      });

      if (!hasPermissions) {
        el.parentNode && el.parentNode.removeChild(el);
      }
    }
    else {
      throw new Error(`请设置操作权限标签值`);
    }
  },
};

export default hasPermi;
```

## 2.2. 使用方式

```vue
  <el-button
    v-hasPermission="['system:role:edit']"
    plain
    icon="Edit"
    @click="handleUpdate"
  >
    修改
  </el-button>
```
