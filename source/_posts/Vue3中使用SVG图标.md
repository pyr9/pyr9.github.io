---
title: Vue3中使用SVG图标
date: 2025-11-05 19:36:55
tags:
categories: Vue
---

# 1. 准备svg文件

webstorm 可以安装插件如：icon viewer 查看图标效果

![img](https://panyuro.oss-cn-beijing.aliyuncs.com/1762997518081-bb0d9732-3334-42d2-b344-95d932f4dadc.png)

# 2. 封装 SVG 为组件

```typescript
<script setup lang="ts">
const props = defineProps<{
  className?: string;
  name: string;
  color?: string;
  size?: string;
}>();

const iconName = computed(() => `#icon-${props.name}`);
const svgClass = computed(() => {
  if (props.className) {
    return `svg-icon ${props.className}`;
  }
  return 'svg-icon';
});
</script>

<template>
  <svg :class="svgClass" aria-hidden="true" :style="{ fontSize: size }">
    <use :xlink:href="iconName" :fill="color" />
  </svg>
</template>

<style scope lang="scss">
.sub-el-icon,
.nav-icon {
  position: relative;
  display: inline-block;
  margin-right: 12px;
  font-size: 15px;
}
.svg-icon {
  position: relative;
  width: 1em;
  height: 1em;
  vertical-align: -2px;
  fill: currentColor;
}
</style>
```

1. 注册为插件

```typescript
import path from 'node:path';
import { createSvgIconsPlugin } from 'vite-plugin-svg-icons';

const root = path.resolve(__dirname, '../../');

export default function createSvgIcon(isBuild: boolean) {
  return createSvgIconsPlugin({
    iconDirs: [
      path.join(root, 'src/assets/icons/svg'),
      path.join(root, 'src/assets/icons/Buildings'),
      path.join(root, 'src/assets/icons/Business'),
      path.join(root, 'src/assets/icons/Device'),
      path.join(root, 'src/assets/icons/Document'),
      path.join(root, 'src/assets/icons/Others'),
      path.join(root, 'src/assets/icons/System'),
      path.join(root, 'src/assets/icons/User'),
    ],
    symbolId: 'icon-[dir]-[name]',
    svgoOptions: isBuild,
  });
}
import type { ConfigEnv, PluginOption } from 'vite';
import path from 'node:path';
import createSvgIcon from './svg-icon';

const root = path.resolve(__dirname, '../../');

function plugins({ mode, command }: ConfigEnv): PluginOption[] {
  return [
    ...
    createSvgIcon(command === 'build'),
  ];
}

export default plugins;
import { defineConfig, loadEnv } from "vite";
import path from "path";
import plugins from "./.build/plugins";

// https://vite.dev/config/
export default defineConfig((cnf) => {
  const { mode } = cnf;
  const env = loadEnv(mode, process.cwd());
  const { VITE_APP_ENV } = env;
  return {
    ...
    plugins: plugins(cnf),
    ...
  };
});
```

1. 使用

```vue
<SvgIcon name="xlsx" />
```
