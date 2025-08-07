---
title: ESLint
date: 2022-11-18 16:09:43
tags:
- 前端
- 代码规范
- eslint
categories: 前端布局
---

# 1 ESLint是什么？

ESLint最初是由[Nicholas C. Zakas](http://nczonline.net/) 于2013年6月创建的开源项目。它的目标是提供一个插件化的javascript代码检测工具。

- ESLint 的初衷是为了让程序员可以创建自己的检测规则
- ESLint 并不推荐任何编码风格，规则是自由的

**官网：**[ESLint - 插件化的 JavaScript 代码检测工具 - ESLint中文文档 (bootcss.com)](https://eslint.bootcss.com/)

# 2 优势

### 2.1. 避免低级bug，找出可能发生的语法错误

> 使用未声明变量、修改 const 变量……

### 2.2. 提示删除多余的代码

> 声明而未使用的变量、重复的 case ……

### 2.3. 统一团队的代码风格

> 加不加分号？使用 tab 还是空格？



# 3 ESLint检查语法的规则

[List of available rules - ESLint中文](http://eslint.cn/docs/rules/)



# 4 Eslint 配置

**1 可以通过vue脚手架创建项目时自动下载配置**

**2 项目已经创建好，手动下载配置**

- 安装并初始化配置文件

  ```js
  npm install eslint --save-dev   // 安装并保存到项目开发依赖
  ./node_modules/.bin/eslint -- --init // 初始化命令
  ```

-  ESLint 会询问一系列问题，根据你的回答生成对应的`.eslintrc.js`配置文件。	![](https://tva1.sinaimg.cn/large/008vxvgGly1h8bjfywzaxj31k80iqgr7.jpg)



# 5 配置文件介绍

```js
module.exports = {
    "env": {
        "browser": true,
        "es6": true
    },
    "extends": "standard",
    "globals": {
        "Atomics": "readonly",
        "SharedArrayBuffer": "readonly"
    },
    "parserOptions": {
        "ecmaVersion": 2018,
        "sourceType": "module"
    },
    "plugins": [
        "vue"
    ],
    "rules": {
    }
};
```

## 5.1 env

告诉eslint，当前代码是运行在哪些环境中

- 开发中经常会用到 一些运行环境自带的 api，如：

  - 浏览器中的 **window/document** 等

  - nodejs中的 **__dirname** 等

- 如果不指定的话，检查时就会报错

## 5.2 extends

 直接使用别人已经写好的 lint 规则，方便快捷。扩展一般支持三种类型：

```js
{
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended",
    "eslint-config-standard",
  ]
}
```

- ESLint 官方的扩展:  一共有两个：`eslint:recommended` 、`eslint:all`
- 插件类型。
-  npm 包的扩展。官方规定 npm 包的扩展必须以 `eslint-config-` 开头，使用时可以省略这个头，上面案例中 `eslint-config-standard` 可以直接简写成 `standard`。

## 5.3 globals

在 ESLint 中定义全局变量

-  当访问当前源文件内未定义的变量时，[no-undef](https://link.juejin.cn/?target=https%3A%2F%2Feslint.bootcss.com%2Fdocs%2Frules%2Fno-undef) 规则将发出警告。
- 如果你想在一个源文件里使用全局变量，推荐你在 ESLint 中定义这些全局变量，这样 ESLint 就不会发出警告了。

## 5.4  parserOptions

ESLint 解析器 解析代码时，可以指定 用哪个 js 的版本

- ecmaVersion： *你可以使用 6、7、8、9 或 10 来指定你想要使用的 ECMAScript 版本。你也可以用使用年份命名的版本号指定为 2015（同 6），2016（同 7），或 2017（同 8）或 2018（同 9）或 2019 (same as 10)*
- sourceType： sourceType 有两个值，script 和 module。 对于 ES6+ 的语法和用 import / export 的语法必须用 module. 

## 5.5 plugins

许多 ESLint 插件为使用特定的库和框架提供了额外的规则。

- 官方的规则只能检查标准的 JavaScript 语法，如果你写的是 JSX 或者 Vue 单文件组件，就需要安装 ESLint 的插件，来定制一些特定的规则进行检查。

- ESLint 的插件与扩展一样有固定的命名格式，以 `eslint-plugin-` 开头，使用的时候也可以省略这个头。

```js
{
  "plugins": [
    "react", // eslint-plugin-react
    "vue",   // eslint-plugin-vue
  ]
}
```

或者是在扩展中引入插件，前面有提到 `plugin:` 开头的是扩展是进行插件的加载。

```js
{
  "extends": [
    "plugin:react/recommended",
  ]
}
```

## 5.6 rules

ESLint 附带有[大量的规则](https://link.zhihu.com/?target=https%3A//cn.eslint.org/docs/rules/)，你可以在配置文件的 `rules` 属性中配置你想要的规则
