---
title: 构建 vite2 & Vue3 & ts项目
date: 2021-02-18 17:27:53
tags: vite, vue3, ts
---

## 步骤总览

### 开发前配置

1. 命令行使用 vite 生成项目模板
2. 配置 TypeScript:
   1. tsconfig 配置
   2. 安装 ts 类型检查
3. 集中处理请求【Axios 库】
4. 分支管理保护与自动化的2种方式，1个必要

### 开始开发

1. 配置环境文件
2. 配置别名【可选】
3. 配置路由
4. 选择&调研组件库
5. 配置主题文件
6. 安装 CSS 预处理器
7. 开发过程中与 vue3 开发文档相关的一些知识点
8. 开发过程中常见的问题
9. [在项目中写出可维护可拓展的类型](https://github.com/MillyTang/ts-in-vue3-project)

> 结尾提供了其他相关链接

## 准备工作

> 使用 nvm 切换不同版本的 node 来适用不同项目

### 命令行使用 vite 生成项目模板

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

#### tsconfig.json 文件 增加 2 项配置

```json
{
  "compilerOptions": {
    "isolatedModules": true,
    "types": ["vite/client"]
  }
}
```

#### 安装 ts 类型检查

> 打开 vite.config.ts 文件时 控制台就会报 failed to load the eslint library for the vite.config.ts 错误，这是因为本地 eslint 无法解析 ts 文件，需要安装 ts 依赖 & vite 不提供 ts 类型检查

- 详见[typescript-eslint 配置](https://github.com/typescript-eslint/typescript-eslint/blob/master/docs/getting-started/linting/README.md)

```bash
# 安装ts eslint 依赖
# node version >= 12
# 注意：以下包不提供对 *.vue 文件的检查
yarn add -D eslint typescript @typescript-eslint/parser @typescript-eslint/eslint-plugin
# 如需在开发过程中对 *.vue 文件进行类型检查，请增加以下包支持
# yarn add -D eslint-plugin-vue
# 安装node变量的类型声明
npm i --save-dev @types/node
```

> 下载额外的`npm`包需要添加其他类型定义：1.安装`npm i --save-dev @types/xxx`；2.在`tsconfig.json`中添加``types`的配置『 **_由于 vite2 创建的 ts 项目中已添加默认的 types 定义，标明只使用该类型定义，所以如果需要其他类型（不在@vite/client 中），需要增加配置_** 』

```json
// tsconfig.json
// 在 node 后面增加一项xxx即可
{
  "types": ["vite/client", "node", "xxx"]
}
```

> 本项目使用 eslint:recommended 规范，如需修改请参照 tslint 官方提示

1. 工作目录下新增 `.eslintrc.js` 文件

   ```js
   // 不含  eslint-plugin-vue 插件的配置
   module.exports = {
     root: true,
     parser: "@typescript-eslint/parser",
     plugins: ["@typescript-eslint"],
     extends: ["eslint:recommended", "plugin:@typescript-eslint/recommended"],
   };
   // 包含  eslint-plugin-vue 插件的配置
   module.exports = {
     root: true,
     parser: "vue-eslint-parser",
     parserOptions: {
       parser: "@typescript-eslint/parser",
       ecmaVersion: 2020,
       sourceType: "module",
       ecmaFeatures: {
         jsx: true,
         tsx: true,
       },
     },
     plugins: ["@typescript-eslint"],
     extends: [
       "plugin:vue/vue3-recommended",
       "eslint:recommended",
       "plugin:@typescript-eslint/recommended",
     ],
   };
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

### 配置请求 Axios 集中处理请求

#### 定义

```ts
// server/index.ts 文件
import axios, { AxiosRequestConfig, Method } from "axios";
import qs from "qs";
// 请求常量配置
const BASE_URL = import.meta.env.VITE_APP_BASE_URL as string;
const SEUCCESS_CODE = ["200"]; // 多个code可兼容历史api的成功返回值
const SPECIAL_CODE_403 = []; // 添加业务要求的 无权限的code码
const LOGOUT_CODE = []; // 添加业务要求登出的code码

export type resType = "json" | "blob";

const instance = axios.create({
  baseURL: BASE_URL,
  timeout: 30000,
  withCredentials: true, // 表示跨域请求时是否需要使用凭证
  validateStatus: function (status) {
    return status >= 200 && status <= 304;
  },
  paramsSerializer: function (params) {
    return qs.stringify(params);
  },
});
// 创建一个基本函数，后续的 post & get方法都基于baseFunc
// 添加请求拦截器
instance.interceptors.request.use(
  function (config) {
    // 如需自定义config,可在调用api时传入，在此处修改、增加、覆盖
    const options = {
      ...config,
      headers: {
        ...config.headers,
        // 'X-TOKEN': config.headers['X-TOKEN'] || 'default value or test token for testing' // 请求头自定义token, 重置 responseType || Content-Type
      },
    };
    return options;
  },
  function (error) {
    return Promise.reject(error);
  }
);

// 添加响应拦截器
instance.interceptors.response.use(
  function (response) {
    const { status, data }: { status: number; data: any } = response;
    // 特殊处理返回类型是 blob 类型的请求
    if (status === 200 && response.config.responseType === "blob") {
      return Promise.resolve(response);
    }
    const { code } = data;
    if (SEUCCESS_CODE.includes(code + "")) {
      // 符合请求成功标准
      return Promise.resolve(data.data || {});
    } else if (SPECIAL_CODE_403.includes(code + "")) {
      // 403需要重新授权
      return Promise.resolve(data);
    } else if (LOGOUT_CODE.includes(code + "")) {
      // 意外登出，重新登录
      // reLogin
      return Promise.reject(data);
      // return Promise.resolve(data)
    } else {
      // 统一弹窗提示错误
      return Promise.reject(data);
    }
  },
  function (error) {
    // 集中处理超时错误
    // 或者 统一向上抛出
    return Promise.reject(error);
  }
);
// 基本配置
// 这里有2种方式可以返回 使用api时配置的返回类型
// 1. 直接在根目录下添加 Axios 的类型声明，改写返回类型
// 2. 引入Axios是点击查看 Axios 定义的类型，在使用 `instance.request` 时改成返回传入泛型即可
// 这里我们使用第二种方式
function request<T>(method: Method, url: string, config: AxiosRequestConfig) {
  return new Promise<T>((resolve, reject) => {
    instance
      .request<T, T>({
        method: method,
        url: url,
        ...config,
      })
      .then((res) => {
        resolve(res);
      })
      .catch((err) => {
        reject(err);
      });
  });
}
// 后续根据具体参数类型返回
export function put<T>(url: string, data?: any) {
  return request<T>("PUT", url, { data });
}
export function post<T>(url: string, data?: any, responseType?: resType) {
  return request<T>("POST", url, { data, responseType });
}
export function del<T>(url: string, data?: any) {
  return request<T>("DELETE", url, { data });
}
export function get<T>(url: string, params?: any, responseType?: resType) {
  return request<T>("GET", url, { params, responseType });
}

export default instance;
```

#### 使用定义好的 `post, get`等方法

```ts
// const-api-path.ts
export const POST_SOME_LIST = "/path/to/some/list";
// api.ts
import { post, get } from "./index";
import { POST_SOME_LIST } from "./const-api-path";
import { ListReturnType, ListParamType } from "./api-types";

export const getSomeList = (data: ListParamType) => {
  return post<ListReturnType>(POST_SOME_LIST, data);
};
```


### 分支管理保护与自动化

> 测试环境的发布分支只有 2 个： dev, hotfix

#### 分支管理、保护的2种方式，1个必要

> 基础仓库影响范围较大，分别对应多个项目的不同阶段，则需要对该项目进行分支管理和保护，通常只保护3个主要分支： main, dev,hotfix

##### 第一种方式

> 一个新需求的全部工作流程

1. 新建功能分支，以 `feature-` 开头
2. 功能开发结束给 `dev` 提 `merge request`
3. 代码审核结束成功合并到 `dev`后，开始测试
4. 通过测试后提交 `merge request` 到 `main`分支准备发布
5. 发布成功且线上验收成功后，在最新提交上打 `tag`, 并推送 tags
6. 保留该功能分支 2 周删除该分支对应的远程分支，以防后续开发功能分支越来越多

> 一个紧急修复的全部流程

1. `git checkout hotfix`
2. `git pull`
3. 开始修复，修复结束后提交代码发布到测试环境测试
4. 通过测试后提交 mr 到 `main`分支准备发布
5. 发布成功且线上验收成功后，在最新提交上打 `tag`, 并推送 tags

##### 第二种方式

> 一个主仓库分别由各个开发fork到自己的group, 主仓库只保留3个分支：main(稳定版本-线上环境), hotfix(紧急修复-发布测试环境), dev(开发时-发布测试环境), fork仓库完全由开发者自行管理

```bash
# fork后的项目新增一个 remote
git remote add source-hub-name source-hub-ssh-path-or-https-path
# 拉取 主仓库所有分支
git fetch source-hub-name
# 建立开发进度对应分支
git checkout -b dev-source-hub-name
# 拉取 主仓库 dev 分支所有commit
git pull source-hub-name dev
# 回到dev分支，合并主仓库进度
git checkout dev
# 或者仅pick部分提交
git merge dev-source-hub-name
```

## 开发

### 配置环境文件

- 构建包文件会通过环境文件来确定环境变量，在项目下新增:
  - `.env.production`文件
  - `.env.development`文件
  - `.env.stagging`文件

### 配置别名【可选】

```js
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import path from "path";

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});
```

### 配置路由

```bash
yarn add vue-router@4
```

#### 使用

```ts
import HelloWorld from "@/components/hello-world.vue";
import { createRouter, createWebHashHistory, RouteRecordRaw } from "vue-router";

// 路由懒加载
const About = () => import("../components/about.vue");
const routes: RouteRecordRaw[] = [
  {
    path: "/",
    component: HelloWorld, // 非懒加载
  },
  {
    path: "/about",
    component: About,
    // meta类型需要额外声明，此声明在 src/typings/augmenation.d.ts内
    meta: {
      isAdmin: false,
      title: "关于",
    },
  },
];
const router = createRouter({
  history: createWebHashHistory(),
  routes,
});
export default router;
```

#### 添加额外的 meta 字段的类型声明

```ts
// augmenation.d.ts
// Ensure this file is parsed as a module regardless of dependencies.
export {};
declare module "vue-router" {
  // 自定义元字段声明
  interface RouteMeta {
    // is optional
    isAdmin?: boolean;
    // is optional
    requiresAuth?: boolean;
    // must be declared by every route
    title: string;
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

### 安装 CSS 预处理器

> 请匹配 组件库 的选择

```bash
# .scss and .sass
npm install -D sass
# .less
npm install -D less
# .styl and .stylus
npm install -D stylus
```

#### 替换 moment 为 dayjs

1. `yarn add dayjs` or `npm i dayjs`
2. 在 `vite.config`中添加 alias

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
import dayjs from "dayjs";
import isSameOrBefore from "dayjs/plugin/isSameOrBefore";
import isSameOrAfter from "dayjs/plugin/isSameOrAfter";
import advancedFormat from "dayjs/plugin/advancedFormat";
import customParseFormat from "dayjs/plugin/customParseFormat";
import weekday from "dayjs/plugin/weekday";
import weekOfYear from "dayjs/plugin/weekOfYear";
import isMoment from "dayjs/plugin/isMoment";
import localeData from "dayjs/plugin/localeData";
import localizedFormat from "dayjs/plugin/localizedFormat";
import "dayjs/locale/zh-cn"; // 导入本地化语言

dayjs.extend(isSameOrBefore);
dayjs.extend(isSameOrAfter);
dayjs.extend(advancedFormat);
dayjs.extend(customParseFormat);
dayjs.extend(weekday);
dayjs.extend(weekOfYear);
dayjs.extend(isMoment);
dayjs.extend(localeData);
dayjs.extend(localizedFormat);
dayjs.locale("zh-cn"); // 使用本地化语言
```

## 开发过程中与 vue3 开发文档相关的一些知识点

### 响应式 API

- `reaåctive()`: 返回对象的响应式副本

### SetUp & ref() || unRef() & toRefs()

**_SetUp(props, context)_** 逻辑块组合，可组合:`生命周期函数/watch/computed`，返回**对象**，对象可包含 `data/methods/computed`

#### 生命周期映射关系

- ~~`beforeCreate`~~ -> use `setup()`
- ~~`created`~~ -> use `setup()`
- `beforeMount` -> `onBeforeMount`
- `mounted` -> `onMounted`
- `beforeUpdate` -> `onBeforeUpdate`
- `updated` -> `onUpdated`
- `beforeUnmount` -> `onBeforeUnmount`
- `unmounted` -> `onUnmounted`
- `errorCaptured` -> `onErrorCaptured`
- `renderTracked` -> `onRenderTracked`
- `renderTriggered` -> `onRenderTriggered`

> 由于 `props` 是响应式的，所以不能使用`ES6解构`，因为 **_解构是浅复制，非引用类型属性拷贝后会跟原 prop 属性脱离，是一个新的变量，没有响应性，引用类型的属性（数组，对象，函数）还会保持引用，这 2 种情况下解构后的变量响应性不统一_** 所以会消除 prop 的响应性，如果需要解构 prop,使用 `toRefs`

> 若要获取传递给 setup() 的参数的类型推断，请使用 `defineComponent`

#### 在 Setup 中使用 Router

> 请使用`useRouter()` || `useRoute()`, 模板中可以访问 `$router` 和 `$route`，所以不需要在 setup 中返回 `router` 或 `route`

- `userLink & RouterLink`: 内部行为已经作为一个组合式 API 函数公开

### ts 相关

- `setup()` 函数中，不需要将类型传递给 props 参数，因为它将从 props 组件选项推断类型
- `refs()` 1.初始值推断类型;2.传递一个泛型参数;3. 使用`Ref<T>`替代`ref`
- `readtive()` 使用接口(`Interface`)
- `computed`自动推断

#### 与 Options api 一起使用

- 复杂的类型或接口使用 **as** 强制转换

#### 注解返回类型

- computed 的类型难以推断 需要注解

#### 注解 props

- 使用 PropType 强制转换构造函数

```ts
import { defineComponent, PropType } from "vue";

// 这里是接口，也可以是type
interface Book {
  title: string;
  author: string;
  year: number;
}

const Component = defineComponent({
  props: {
    name: String,
    success: { type: String },
    callback: {
      type: Function as PropType<() => void>,
    },
    book: {
      type: Object as PropType<Book>,
      required: true,
    },
  },
});
```

- 由于 ts 的设计限制：涉及到为了对函数表达式进行类型推理，你必须注意·对象和数组的`validators` 和 `default` 值

```ts
import { defineComponent, PropType } from "vue";

interface Book {
  title: string;
  year?: number;
}

const Component = defineComponent({
  props: {
    bookA: {
      type: Object as PropType<Book>,
      // 请务必使用箭头函数
      default: () => ({
        title: "Arrow Function Expression",
      }),
      validator: (book: Book) => !!book.title,
    },
    bookB: {
      type: Object as PropType<Book>,
      // 或者提供一个明确的 this 参数
      default(this: void) {
        return {
          title: "Function Expression",
        };
      },
      validator(this: void, book: Book) {
        return !!book.title;
      },
    },
  },
});
```

#### 注解 emit

- 为触发的事件注解一个有效载荷。另外，所有未声明的触发事件在调用时都会抛出一个类型错误。

```ts
const Component = defineComponent({
  emits: {
    addBook(payload: { bookName: string }) {
      // perform runtime 验证
      return payload.bookName.length > 0;
    },
  },
  methods: {
    onSubmit() {
      this.$emit("addBook", {
        bookName: 123, // 类型错误！
      });
      this.$emit("non-declared-event"); // 类型错误！
    },
  },
});
```

#### 类型声明 refs

- 有时我们可能需要为 ref 的内部值指定复杂类型。我们可以在调用 ref 重写默认推理时简单地传递一个泛型参数

```ts
const year = ref<string | number>("2020"); // year's type: Ref<string | number>
year.value = 2020; // ok!
// 如果泛型的类型未知，建议将 ref 转换为 Ref<T>
```

#### 类型声明 reactive

> 与 `ref` 的区别： 当变量基础类型时使用`ref`, 引用类型-对象/数组等使用 `reactive`

- 当声明类型 reactive property，我们可以使用接口

```ts
import { defineComponent, reactive } from "vue";

interface Book {
  title: string;
  year?: number;
}

export default defineComponent({
  name: "HelloWorld",
  setup() {
    const book = reactive<Book>({ title: "Vue 3 Guide" });
    // or
    const book: Book = reactive({ title: "Vue 3 Guide" });
    // or
    const book = reactive({ title: "Vue 3 Guide" }) as Book;
  },
});
```

### 开发过程中遇到的问题[TroubleShooting]

#### 注意：vite2 自动构建 typescript 模板项目文件，部分同学会在 build 阶段报错，详情请移步

- [error VueDX/TS: context.imports.add is not a function](https://github.com/znck/vue-developer-experience/issues/208)
- [@vuedx/typecheck vite build error with `const FSP = require('fs/promises');`](https://github.com/znck/vue-developer-experience/issues/144)

> windows 系统的同学可将 build 打包命令由`vuedx-typecheck . && vite build` 改为 `vite build && tsc`,ci 脚本内加一条：`run: npx --no-install vuedx-typecheck`

#### 在`ts`文件内使用 `const xxx = require('xxx')`报错：`require is not defined`

> 已解决: `vite@2.4.1`已修复该错误，升级即可

- [Error: require is mot defined](https://github.com/vitejs/vite/issues/2161)

#### UI 组件的问题

- ant-design-vue 按需加载问题:
  查看[Imports using @ant-design/icons-vue should be importing the ESM version](https://github.com/vueComponent/ant-design-vue/issues/3570)

- and-design-vue 版本问题！！！现发现 `form & input`组件在引入 es 包时一直报错`The requesed module '/node_modules/is-mobile/index.js' is dose not provide an export named 'isMobile'`，遂升级至 2.1.1 版本，同时删除 vite.config 里 optimizeDeps 的 exclude 的配置,才能使用:
  详见[组件库依赖的 ismobilejs 存在兼容性问题](https://github.com/vueComponent/ant-design-vue/issues/1936)

- Element-plus 包问题:
  查看[element plus issues: [Bug Report] error: [ vite:dep-scan ] Failed to resolve entry for package "element-plus". The package may have incorrect main/module/exports specified in its package.json](https://github.com/element-plus/element-plus/issues/1632)

### 依赖强缓存

- 通过浏览器 devtools 的 Network 选项卡暂时禁用缓存；
- 重启 Vite dev server，使用 --force 标志重新打包依赖；
- 重新载入页面。

## esbuild 预构建

## 相关文档链接

- [tslint 配置](https://github.com/typescript-eslint/typescript-eslint/blob/master/docs/getting-started/linting/README.md)
- [esbuild](https://esbuild.github.io/getting-started/#bundling-for-the-browser)
- [vite](https://cn.vitejs.dev/guide/)
