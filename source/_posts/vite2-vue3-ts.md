---
title: 构建 vite2 & Vue3 & ts项目
date: 2021-02-18 17:27:53
tags:
---

## 准备工作

> 使用nvm切换不同版本的node来适用不同项目

### 命令行使用vite生成项目模板

```bash
node -v
# v8.9.4
nvm use 12
# Now using node v12.19.0 (npm v6.14.8)
yarn create @vitejs/app
# 选择 vue-ts 预设模板
cd my-vue3-vite2-app
yarn
yarn dev
```

### 配置 TypeScript

#### tsconfig.json文件 增加2项配置 

```json
{
   "compilerOptions": {
    "isolatedModules": true,
    "types": ["vite/client"]
  }
}
```

#### 安装 ts 类型检查

> 打开 vite.config.ts文件时 控制台就会报  failed to load the eslint library for the vite.config.ts 错误，这是因为本地eslint无法解析ts文件，需要安装ts依赖 & vite不提供ts类型检查

* 详见[typescript-eslint配置](https://github.com/typescript-eslint/typescript-eslint/blob/master/docs/getting-started/linting/README.md)

```bash
# 安装ts eslint 依赖
# node version >= 12
yarn add -D eslint typescript @typescript-eslint/parser @typescript-eslint/eslint-plugin
# 安装node变量的类型声明
npm i --save-dev @types/node
```

> 下载额外的`npm`包需要添加其他类型定义：1.安装`npm i --save-dev @types/xxx`；2.在`tsconfig.json`中添加``types`的配置『 ***由于vite2创建的ts项目中已添加默认的types定义，标明只使用该类型定义，所以如果需要其他类型（不在@vite/client中），需要增加配置*** 』

```json
// tsconfig.json
// 在 node 后面增加一项xxx即可
{
  "types": ["vite/client", "node", "xxx"],
}
```

## 注意：vite2自动构建 typescript 模板项目文件，部分同学会在build阶段报错，详情请移步

* [error VueDX/TS: context.imports.add is not a function](https://github.com/znck/vue-developer-experience/issues/208)
* [@vuedx/typecheck vite build error with `const FSP = require('fs/promises');`](https://github.com/znck/vue-developer-experience/issues/144)

> 本项目使用 eslint:recommended 规范，如需修改请参照tslint官方提示

1. 工作目录下新增 `.eslintrc.js` 文件

    ```js
    module.exports = {
      root: true,
      parser: '@typescript-eslint/parser',
      plugins: [
        '@typescript-eslint',
      ],
      extends: [
        'eslint:recommended',
        'plugin:@typescript-eslint/recommended',
      ],
    }
    ```

2. 工作目录下新增 `.eslintignore` 文件

    ```ignore
    # don't ever lint node_modules
    node_modules
    # don't lint build output (make sure it's set to your correct build folder name)
    dist
    # don't lint nyc coverage output
    coverage
    # don't lint .vscode
    .vscode
    # don't lint .eslintrc.js
    .eslintrc.js
    ```

### 安装CSS预处理器

```bash
# .scss and .sass
npm install -D sass
# .less
npm install -D less
# .styl and .stylus
npm install -D stylus
```

## 开发

### 配置环境文件

* 构建包文件会通过环境文件来确定环境变量，在项目下新增:
  * `.env.production`文件
  * `.env.development`文件
  * `.env.stagging`文件

### 配置别名【可选】

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from "path";

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src")
    }
  }
})
```

### 配置路由

```bash
yarn add vue-router@4
```

#### 使用

```ts
import HelloWorld from '@/components/hello-world.vue'
import { createRouter, createWebHashHistory, RouteRecordRaw } from 'vue-router'

// 路由懒加载
const About = () => import('../components/about.vue')
const routes: RouteRecordRaw[] = [
  {
    path: '/',
    component: HelloWorld // 非懒加载
  },
  {
    path: '/about',
    component: About,
    // meta类型需要额外声明，此声明在 src/typings/augmenation.d.ts内
    meta: {
      isAdmin: false,
      title: '关于'
    }
  }
]
const router = createRouter({
  history: createWebHashHistory(),
  routes
})
export default router
```

#### 添加额外的 meta 字段的类型声明

```ts
// augmenation.d.ts
// Ensure this file is parsed as a module regardless of dependencies.
export {}
declare module 'vue-router' {
  // 自定义元字段声明
  interface RouteMeta {
    // is optional
    isAdmin?: boolean
    // is optional
    requiresAuth?: boolean,
    // must be declared by every route
    title: string
  }
}
```

#### 添加路由元信息声明

### 选择组件库 ☞ 本项目使用 `ant-design-vue`, 所以选用 less, 方便覆盖默认主题

#### 配置主题文件

```js
// 使用插件转换文件
import lessToJS from 'less-vars-to-js'
const themeVariables = lessToJS(
  fs.readFileSync(path.resolve(__dirname, './src/styles/variable.less'), 'utf8')
)
// vite.config.js
{
  css: {
    preprocessorOptions: {
      // 在这里覆盖css全局变量, 可覆盖变量可在varible.less中查看
      less: {
        modifyVars: themeVariables,
        javascriptEnabled: true,
      }
    }
  },
}
```

#### 替换 moment 为 dayjs

1. `yarn add dayjs` or  `npm i dayjs`
2. 在 `vite.config`中添加alias

    ```js
    {
      resolve: {
        alias: {
          moment: 'dayjs'
        }
      },
    }
    ```

3. 在 `main.ts`文件里添加相关插件, 添加前后分别打包进行比较

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

### 响应式API

* `reaåctive()`: 返回对象的响应式副本

### SetUp & ref() || unRef() & toRefs()

***SetUp(props, context)*** 逻辑块组合，可组合:`生命周期函数/watch/computed`，返回**对象**，对象可包含 `data/methods/computed`

#### 生命周期映射关系

* ~~`beforeCreate`~~ -> use `setup()`
* ~~`created`~~ -> use `setup()`
* `beforeMount` -> `onBeforeMount`
* `mounted` -> `onMounted`
* `beforeUpdate` -> `onBeforeUpdate`
* `updated` -> `onUpdated`
* `beforeUnmount` -> `onBeforeUnmount`
* `unmounted` -> `onUnmounted`
* `errorCaptured` -> `onErrorCaptured`
* `renderTracked` -> `onRenderTracked`
* `renderTriggered` -> `onRenderTriggered`

> 由于 `props` 是响应式的，所以不能使用`ES6解构`，因为 ***解构是深复制，跟原prop已经脱离了引用*** 所以会消除prop的响应性，如果需要解构prop,使用 `toRefs`

> 若要获取传递给 setup() 的参数的类型推断，请使用 `defineComponent`

#### 在Setup中使用Router

> 请使用`useRouter()` || `useRoute()`, 模板中可以访问 `$router` 和 `$route`，所以不需要在 setup 中返回 `router` 或 `route`

* `userLink & RouterLink`: 内部行为已经作为一个组合式 API 函数公开

### ts相关

* `setup()` 函数中，不需要将类型传递给 props 参数，因为它将从 props 组件选项推断类型
* `refs()` 1.初始值推断类型;2.传递一个泛型参数;3. 使用`Ref<T>`替代`ref`
* `readtive()` 使用接口(`Interface`)
* `computed`自动推断

#### 与 Options api 一起使用

* 复杂的类型或接口使用 **as** 强制转换

#### 注解返回类型

* computed 的类型难以推断 需要注解

#### 注解 props

* 使用 PropType 强制转换构造函数

```ts
import { defineComponent, PropType } from 'vue'

// 这里是接口，也可以是type
interface Book {
  title: string
  author: string
  year: number
}

const Component = defineComponent({
  props: {
    name: String,
    success: { type: String },
    callback: {
      type: Function as PropType<() => void>
    },
    book: {
      type: Object as PropType<Book>,
      required: true
    }
  }
})
```

* 由于 ts的设计限制：涉及到为了对函数表达式进行类型推理，你必须注意·对象和数组的`validators` 和 `default` 值

```ts
import { defineComponent, PropType } from 'vue'

interface Book {
  title: string
  year?: number
}

const Component = defineComponent({
  props: {
    bookA: {
      type: Object as PropType<Book>,
      // 请务必使用箭头函数
      default: () => ({
        title: 'Arrow Function Expression'
      }),
      validator: (book: Book) => !!book.title
    },
    bookB: {
      type: Object as PropType<Book>,
      // 或者提供一个明确的 this 参数
      default(this: void) {
        return {
          title: 'Function Expression'
        }
      },
      validator(this: void, book: Book) {
        return !!book.title
      }
    }
  }
})
```

#### 注解 emit

* 为触发的事件注解一个有效载荷。另外，所有未声明的触发事件在调用时都会抛出一个类型错误。

```ts
const Component = defineComponent({
  emits: {
    addBook(payload: { bookName: string }) {
      // perform runtime 验证
      return payload.bookName.length > 0
    }
  },
  methods: {
    onSubmit() {
      this.$emit('addBook', {
        bookName: 123 // 类型错误！
      })
      this.$emit('non-declared-event') // 类型错误！
    }
  }
})
```

#### 类型声明 refs

* 有时我们可能需要为 ref 的内部值指定复杂类型。我们可以在调用 ref 重写默认推理时简单地传递一个泛型参数

```ts
const year = ref<string | number>('2020') // year's type: Ref<string | number>
year.value = 2020 // ok!
// 如果泛型的类型未知，建议将 ref 转换为 Ref<T>
```

#### 类型声明 reactive

> 与 `ref` 的区别： 当变量基础类型时使用`ref`, 引用类型-对象/数组等使用 `reactive`

* 当声明类型 reactive property，我们可以使用接口

```ts
import { defineComponent, reactive } from 'vue'

interface Book {
  title: string
  year?: number
}

export default defineComponent({
  name: 'HelloWorld',
  setup() {
    const book = reactive<Book>({ title: 'Vue 3 Guide' })
    // or
    const book: Book = reactive({ title: 'Vue 3 Guide' })
    // or
    const book = reactive({ title: 'Vue 3 Guide' }) as Book
  }
})
```

### 开发过程中遇到的问题[TroubleShooting]

#### 系统问题？

 现发现 windows 下 build 会有问题，windows系统的同学可将 build 打包命令由`vuedx-typecheck . && vite build` 改为 `vite build && tsc`,ci脚本内加一条：`run: npx --no-install vuedx-typecheck`


#### UI组件的问题

* ant-design-vue 按需加载问题:
  查看[Imports using @ant-design/icons-vue should be importing the ESM version](https://github.com/vueComponent/ant-design-vue/issues/3570)

* and-design-vue 版本问题！！！现发现 `form & input`组件在引入es包时一直报错`The requesed module '/node_modules/is-mobile/index.js' is dose not provide an export named 'isMobile'`，遂升级至2.1.1版本，同时删除 vite.config里optimizeDeps的exclude的配置,才能使用:
  详见[组件库依赖的ismobilejs存在兼容性问题](https://github.com/vueComponent/ant-design-vue/issues/1936)

* Element-plus 包问题:
  查看[element plus issues: [Bug Report] error: [ vite:dep-scan ] Failed to resolve entry for package "element-plus". The package may have incorrect main/module/exports specified in its package.json](https://github.com/element-plus/element-plus/issues/1632)

### 依赖强缓存

* 通过浏览器 devtools 的 Network 选项卡暂时禁用缓存；
* 重启 Vite dev server，使用 --force 标志重新打包依赖；
* 重新载入页面。

## esbuild 预构建

## Filed Under

* [tslint配置](https://github.com/typescript-eslint/typescript-eslint/blob/master/docs/getting-started/linting/README.md)
* [esbuild](https://esbuild.github.io/getting-started/#bundling-for-the-browser)
* [vite](https://cn.vitejs.dev/guide/)
