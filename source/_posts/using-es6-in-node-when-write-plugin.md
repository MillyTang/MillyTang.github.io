---
title: 开发vite相关插件的问题
date: 2021-04-15 15:50:22
tags: vite-plugin-vue, cjs, es6
---

本项目的技术栈是：`vite vue3 ant-desing-vue`。
项目中需要用到的`ant-desing-vue`UI 组件依赖`moment`,我看官网提示有插件可以改成`dayjs`,可以使整个包小一点，看了`antd-dayjs-webpack-plugin`的代码后发现没有针对`vite ant-desing-vue`的,就跃跃欲试想写一个`vite-plugin-vue-ant-desing-vue-dayjs`插件，专门用在`vite vue3 ant-desing-vue`项目中。后面其实发现没必要，当然，写完才发现-\_-||

## 主要问题

### 叠了 3 个 debuff 导致运行时出问题

1. 由于我 fork 了一部分`antd-dayjs-webpack-plugin`代码，内部是`module.exports`
2. 同时入口文件里是直接`export default vitePluginVueAntDVueDayjs(Options={}) {...}`
3. 而且是直接`yarn add git+ssh://git@github.com:xxx/some-project.git --dev`，这个`some-project`就是写的插件，`package.json`里暴露的直接是`main: src/index.js`,

第一次写插件，心情十分激动结果刚运行就报错,搜了一下原来是 node 环境里不能直接加载`ES6`模块。。。于是我重新看了一次 es6 文档`node 环境如何加载es6模块`，然后查了一下`@vite/plugin-vue`的解决办法

## 解决方案

1. 完全采用 cjs 模块方式编写，一水儿的`module.export` -> 针对入口文件非打包后的插件。如`package.json中的main: src/index.js`
2. 将 es6 模块打包编译成 node 模块（@vite/plugin-vue）就是这么做的：`esbuild src/index.ts --bundle --platform=node --target=node12 --external:@vue/compiler-sfc --outfile=dist/index.js` -> 完全使用 es6 模块编写插件，然后使用打包后的 cjs 模块, `package.json中的main: dist/index.js`

## 没必要写是因为直接配置即可

    ```bash
    # 命令行
    # npm i dayjs
    yarn add dayjs
    ```
    ```js
    // vite.config.js文件
    {
      resolve: {
        alias: {
          moment: 'dayjs'
        }
      },
    }
    ```
    ```ts
    // main.ts
    import dayjs from 'dayjs'
    import isSameOrBefore from 'dayjs/plugin/isSameOrBefore'
    import isSameOrAfter from 'dayjs/plugin/isSameOrAfter'
    import advancedFormat from 'dayjs/plugin/advancedFormat'
    import customParseFormat from 'dayjs/plugin/customParseFormat'
    import weekday from 'dayjs/plugin/weekday'
    import weekOfYear from 'dayjs/plugin/weekOfYear'
    import isMoment from 'dayjs/plugin/isMoment'
    import localeData from 'dayjs/plugin/localeData'
    import localizedFormat from 'dayjs/plugin/localizedFormat'
    import 'dayjs/locale/zh-cn' // 导入本地化语言

    dayjs.extend(isSameOrBefore)
    dayjs.extend(isSameOrAfter)
    dayjs.extend(advancedFormat)
    dayjs.extend(customParseFormat)
    dayjs.extend(weekday)
    dayjs.extend(weekOfYear)
    dayjs.extend(isMoment)
    dayjs.extend(localeData)
    dayjs.extend(localizedFormat)
    dayjs.locale('zh-cn') // 使用本地化语言
    ```

## 插件步骤

如果要写也不是不行：看上面的配置步骤即可知道思路，可参考项目[vite-plugin-vue-antd-dayjs](https://github.com/MillyTang/vite-plugin-for-antd-dayjs)

### 第一种： 使用非打包插件

> 有个直接的好处，安装后可以直接修改包文件来测试代码逻辑是否正确，即改即用,我当时采用的就是这种写法

#### 1. 新建项目

```bash
# 项目结构
|-- src
|----|--index.js
|-- package.json
|-- README.md
```

#### 2. package.json 配置

```json
// packgae.json
{
  "name": "vite-plugin-xxx",
  "version": "1.0.0",
  "description": "Description of the vite-plugin-xxx",
  "main": "src/index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": ["vite-plugin", "vite-plugin-vue", "rollup-plugin", "xxx"],
  "engines": {
    "node": ">=12.0.0"
  },
  "devDependencies": {
    "dayjs": "*"
  },
  "peerDependencies": {
    "dayjs": "*"
  },
  "homepage": "",
  "bugs": {
    "url": ""
  },
  "repository": {
    "type": "git",
    "url": ""
  },
  "author": "your Name",
  "license": ""
}
```

#### 3. 入口文件

```js
// src/index.js
module.exports = function VitePluginVueXXX(Options = {}) {
  const virtualFile = "@virtual-file";
  return {
    name: "vite-plugin-vue-xxx",
    enforce: "pre", // 加载vite内置插件之前加载
    config: () => ({
      "resolve.alias": {
        moment: "dayjs", // set dayjs alias
      },
    }), // 在被解析之前修改 Vite 配置
    configResolved(resolvedConfig) {
      // 存储最终解析的配置
      config = resolvedConfig;
    }, // 在解析 Vite 配置后调用
    resolveId(id) {
      // 每个传入的模块请求时被调用
      if (id === virtualFile) {
        console.log("resolveid id test", id);
        return id;
      }
    },
    load(id) {
      // 每个传入的模块请求时被调用
      if (id === virtualFile) {
        // 在这里return上面main.js引入的插件和依赖
        // 如果需要知道哪些插件必须引入，直接查看dayjs官网的插件合集
        // return 出去的是 `@virtual-file`的文件内容，这里写成 `const xxx = require('xxx')` or `import xxx from 'xxxx'`都是可以的，注意在`main.js`内的引用形式即可
        // 例如：isMoment plugin
        return `const isMoment =  require('dayjs/plugin/isMoment'); dayjs.extend(isMoment);module.exports = {isMoment}`;
      }
    },
  };
};
```

#### 4. 安装 没有发布的包文件

```bash
# 安装 插件的 peerDependencies
yarn add dayjs
# 安没有发布的装插件包
yarn add git+ssh://git@github.com:xxx.git --dev
```

```js
// 在 vite.config内配置
// 引入插件
  const vitePluginVueForAntdDayjs = require('vite-plugin-vue-antd-dayjs')
 {
    plugins:  [vue(),vitePluginVueForAntdDayjs()],
    resolve: {
      alias: {
        moment: 'dayjs'
      }
    },
  }
```

#### 5. 在 main.js 里引入虚拟文件即可

```ts
// 以 isMoment 为例
// 在 mjs 内 引入 cjs 模块
import isMoment from "@virtual-file";
// 如果模块很多，一次引入很多模块, 直接引入文件
import "@virtual-file";
```

> 启动项目： `yarn dev`, vite 打包会发现有提示：`[vite] new dependencies found: dayjs/plugin/isSameOrAfter, dayjs/plugin/advancedFormat, dayjs/plugin/customParseFormat, dayjs/plugin/weekday, dayjs/plugin/weekYear, dayjs/plugin/weekOfYear, dayjs/plugin/isMoment, dayjs/plugin/localeData, dayjs/plugin/localizedFormat, dayjs/locale/zh-cn, updating...` 说明已经成功引入

#### 6. 打包比较前后包的差异

### 第二种：使用打包后的插件

> 步骤跟上面的差不多，只是写的时候使用 es6 模块,多一步打包&入口文件有点不同，可参考项目[vite-plugin-antd-vue-ts](https://github.com/MillyTang/vite-plugin-antd-vue-ts)

1. packgae.json 的配置

   ```json
   {
     "main": "dist/index.js",
     "types": "dist/index.d.ts",
     "scripts": {
       "build:boundle": "`esbuild src/index.ts --bundle --platform=node --target=node12 --external:dayjs --outfile=dist/index.js"
     },
     "dependencies": {
       "esbuild": "*"
     },
     "devDependencies": {
       "dayjs": "*"
     },
     "peerDependencies": {
       "dayjs": "*"
     }
   }
   ```

2. 入口文件

```js
export default function VitePluginVueXXX(Options = {}) {}
```

3. 执行打包命令

```bash
npm run build:boudle
```

4. 配置 `vite.config.js` & 在`main.js`引入文件步骤同上

> 如果要实现在入口文件的引用方式为 `import someplugin from 'path/to/all-plugin-file-name'`,在插件内使用 `export default` 和 `module.exports`是一样的效果
