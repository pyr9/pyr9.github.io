---
title: 在vue2的项目上使用vue3的库
date: 2025-02-12 15:56:02
tags: xss跨站漏洞
categories: Vue
---

# 1. 业务场景

当前前端项目是基于vue2的，使用的pdf预览的组件为vue-pdf-embed， 但是vue2下，这个组件只兼容到v1版本，会有xss漏洞，在vue3下对应的v2版本才修复了这个问题。

由于项目是个很老的项目，整体升级vue3工作量太大，所以选择使用Web Components 封装组件的方式，将vue-pdf-embed 封装为一个 Web Component，然后在 Vue 2 中使用。

# 2. 步骤

## 1. 创建一个 独立的Vue 3 项目

1. 创建项目 `vue create simple-vue3-project`
2. 在 Vue 3 项目中安装 vue-pdf-embed `npm install vue-pdf-embed`

## 2. 创建 Web Components 封装

1. 创建PdfViewer.ce.vue文件

   ```vue
   <template>
       <vue-pdf-embed :source="source"/>
   </template>
   
   <script>
   import VuePdfEmbed from 'vue-pdf-embed';
   
   export default {
       props: {
           source: {
               type: String,
               required: true,
           },
       },
       components: {
           VuePdfEmbed,
       },
   };
   </script>
   ```

   

2. 在 Vue 3 项目的入口文件（如 main.js）中注册 Web Components。

   ```js
   import {defineCustomElement} from 'vue'
   
   import PdfViewer from './components/PdfViewer.ce.vue';
   
   // 将 Vue 组件封装为 Web Component
   const PdfViewerElement = defineCustomElement(PdfViewer);
   
   // 注册自定义元素
   customElements.define('pdf-viewer', PdfViewerElement);
   
   console.log('Web Component registered:', customElements.get('pdf-viewer'));
   ```

   这里为了减小js文件大小，这里我删除了main.js模版的本身内容

3. 修改vue.config.js

   ```js
   const { defineConfig } = require('@vue/cli-service');
   
   module.exports = defineConfig({
     transpileDependencies: true, // 确保依赖库被正确转译
   
     configureWebpack: {
       entry: './src/main.js', // 指定入口文件
       output: {
         filename: 'pdf-viewer.js', // 输出文件名
         library: 'PdfViewer', // 设置生成的库名称
         libraryTarget: 'window', // 将库挂载到 window 上
         globalObject: 'this', // 兼容非浏览器环境（如 Node.js）
       },
       optimization: {
         splitChunks: false, // 禁用代码分割
       },
     },
   
     css: {
       extract: true,
       loaderOptions: {
         css: {
           minimize: true, // 压缩 CSS
         },
       },
     },
   
     productionSourceMap: false, // 关闭生产环境下的 source map，减少文件大小
   });
   ```

   

4. 使用npm run build构建成一个js文件如 （dist/pdf-viewer.js），其中包含了封装好的 Web Components。

## 3. 在 Vue 2 项目中使用 Web Components

1. 将打包好的js文件放在，public目录下

2. 在index.html中引入

   ```html
   <!-- public/index.html -->
   <script type="module" src="/pdf-viewer.js"></script>
   ```

3. 在其他视图或组件使用

   ```vue
   <pdf-viewer :source="url" class="m-auto"></pdf-viewer>
   ```

4. 在main.js 忽略这个自定义元素，避免报错

   ```js
   Vue.config.ignoredElements = ['pdf-viewer'];
   ```

   
