---
title: ts-in-vue3
date: 2021-09-12 18:46:11
tags:
---

> 特别说明：本项目只关注如何写可用 ts 类型，其他诸如：根路径配置类型文件、ts 类型查找顺序、ts 类型体操等均不涉及

## 项目中需要使用 ts 类型的文件及其结构

```bash
# 文件名及文件层级
|---- server # 请求
|      |--- index.ts # 请求配置
|      |--- api.ts # 具体api
|---- store # vuex
|------|--- modules
|------|--- index.ts
|---- composables # 逻辑复用组件
|------|--- *.ts # echarts.ts / 变量声明，使用，值更新
|---- router # vue-router
|------|--- index.ts
|---- directives # 自定义全局指令
|------|--- index.ts
|---- pages # 页面
|---- components # 组件（非全局）
|---- my.t.ts # 共用类型定义文件
```

## 先看项目中出现过的几个 ts 典型写法

### `*.vue` 文件内

```ts
// some-detail.vue script.lang = ts
import { defineComponent, reactive, ref, toRefs } from 'vue'
import { useStore } from 'vuex'
import { key } from 'path/to/store'

export default defineComponent({
  props: {
    list: Array, // round 0: 不够准确
    default() {
      return []
    }
  },
  // setup 内部
  // round 1
  setup: (props: any) =>  {
  // round 2: ?
    const leftType: any = reactive({
      rightType: 0,
      list: []
    })
    // round 3: ?
    const newArray = leftType.list.map(v:any => {
      return {
        ...v,
        changeSomePropName: v.somePropName
      }
    })
    // round 4: ?
    const textFunc: void = () => {
      // do something but do not return anything
    }
    // round 5: ?
    const hasTrue = ref<boolean>(false)
    const isStr = ref<string>('initStr')

    // round 6: ?
    const { list } = toRefs(props) // list: any
    // round 7: ?
    const store: any = useStore(key)
    // round 8: ?
    const getData = getDataByAxiosApi().then(res => {
      const ret: any = res
    })
  })
}
```

### 总结

#### 1. 频繁使用 ts 类型的几个地方

1. `server/api.ts` 接口定义，接口通用 axios 实例类型返回动态类型
2. `store/modules` **state** 定义，融合多个模块的类型
3. `router/mpdules` **routes** 定义，扩展字段类型
4. 全局类型定义：扩展原模块类型
5. `*.vue`文件内 `setup` & `composables/*.ts` 逻辑组件， 内部：函数声明，变量定义，赋值操作等

#### 2. 文件内频繁写错方向的类型：使用推导类型

1. 凡是赋值操作，赋值右边带有类型推导的，类型一律由右边决定，不必在左边写 `any`
2. 凡是简单类型赋值操作，ref内部的，都不必声明类型 `const a = ref('some str'); // a 一定会被推导为 Ref<string>`
3. 函数操作（useStore/useRoute/数组操作（map,forEach,for(of,in)）/自定义函数，此处只是执行操作的函数），类型由函数定义时返回的参数类型决定

```ts
// 以上写法可以改成？
```

#### 3. 定义函数/变量时如何返回需要的类型

1. 扩展全局模块类型： router/store/axios

    > 衍生问题，如何知道 router 上有 RouteMeta 类型？（或者说如何知道 Router 有哪些类型）

    ```ts
    // 扩展全局模块类型router/store/axios，以router为例
    /**
    * 模块导出的2种方式
    */
    // 方法一：全局导出声明
    // 确保是模块
    // export {}

    // declare module 'vue-router' {
    //   // 自定义元字段声明
    //   interface RouteMeta {
    //     // is optional
    //     isAdmin?: boolean
    //     requiresAuth?: boolean,
    //     // must be declared by every route
    //     title?: string
    //   }
    // }

    // 方法二：模块导出声明
    declare interface RouteMeta {
      // is optional
      isAdmin?: boolean
      requiresAuth?: boolean
      // must be declared by every route
      title: string
    }
    ```

2. composables/directives/server

> Method 是什么类型？
> 如何修改 request 的定义，使得可以在使用 `getData` 是返回与后端确定好的结构？在修改过程中碰到哪些问题？请举例

```ts
// server/index
import axios, { AxiosRequestConfig, Method } from 'axios'
// 这种写法会导致什么问题？
function request(method: Method, url: string, config: AxiosRequestConfig) {
  return new Promise((resolve, reject) => {
    instance
      .request({
        method: method,
        url: url,
        ...config
      })
      .then((res) => {
        resolve(res)
      })
      .catch((err) => {
        reject(err)
      })
  })
}
// 定义 get 请求方法
export function get(url: string, params?: any, responseType?: resType) {
  return request('GET', url, { params, responseType })
}
// api.ts
const getData = async (data) => {
  try {
    const res = await get('path/to/get/some/data', data)
    return res
  } catch(err) {
    console.error('getData error', error)
  }
 
}
// *.vue 使用getData函数时能否拿到返回的res 类型？？
import { getData } from 'path/to/api.ts'
const res = getData()
```

## 如何在快速迭代的项目中书写正确可用的类型

### 1. 渐进式类型更新：比 any 更好一点的写法

#### 引用类型reactive

```ts
// 一开始可能只知道某个变量的几个属性，大致结构，不确定后续属性的新增/减少，可以这样写
// 第一步
// 类型文件B.ts
export type typeAfromB = {
  [k: string]: unknown
  // [k: string]: any
} // unknown & any 的区别： unknown 类型的变量赋值可以赋值给 any 和 unknown， 有类型检查更安全， any 可以赋值给任意类型(包括null & undefined)，没有类型检查

// 第二步
// 组件A.vue
import { reactive, ref } from 'vue'
import { typeAfromB } from 'path/to/B.ts'
const someTypeInSometimes = reactive<typeAfromB>({
  propA: '123',
  propB: 456,
  propC: {},
  propD: [],
  propE: () => {}
})
```

#### 原始类型：联合类型、交叉类型、字面量类型

```ts
// 字面量类型
export type typeCfromB = 'GET' | 'POST' | 'DELETE' | 'PUT'
// 联合类型
export type typeDfromB = string | number
// 交叉类型
export type typeEfromB = string & number // 此处的 typeEfromB 是什么?考虑一种类型既是 `string` 又是`number`的类型 
export type typeEEfromB = string[] & number // 此处的 typeEfromB 是什么?考虑一种类型既是 `string[]` 又是`number`的类型 

// 组件B.vue: 使用以上定义的类型
const cb = ref<typeCfromB>('oh my god!') // 这里正确吗？
const db = ref<typeDfromB>(124)
db.value = '124' // 这样写ok吗？
const copyDB = db.includes('1') // 这样行不行？
const eb = ref<typeEfromB>(['125', 123]) // 有没有问题？
const copyEB = ref<typeEEfromB>()
const eeb = copyEB?.value.length // 这样能行吗？eeb 的类型是什么，
```

> 把类型当作值的集合，理解 类型中的 union type - 交集 和 intersection type -并集

```ts
type AA = {aa: 'A'}
type BB = {bb: 'B'}
type ABor = AA | BB
type ABand = AA & BB
type AAkeys = keyof AA
type BBkeys = keyof BB
type ABandKeys = keyof ABand // 'aa' | 'bb'
type ABorKeys = keyof ABor // never

const aaa: ABand = {
  aa: 'A',
  bb: 'B'
}
```

### 2. 项目中需要的类型基本上可由：联合类型（|）、交叉类型（&）、 字面量类型、泛型 以及枚举覆盖，不需要类型体操

> 从以上例子中考虑：`联合类型` 和 `交叉类型`的区别? 可以得出交叉类型的应用场景是什么？

实际上，项目中使用最常见的还是联合类型和单一的type/interface, 使用 交叉类型 较多的场景是 `store`，为什么？

#### 项目中常见的需要利用泛型的几种类型：`new Promise`, `new Map`, `new Set`等

```ts
new Promise<T>((resolve, reject) => {
  // do something
})
const a = ref<Map<tring, string[]>>(new Map())
```

## 进阶：类型越来越多，每新增一个文件、每新增一个api、每新增一个函数都会面临类型不够用的情况

### 1. 梳理项目中的类型分类，文件结构

> 考虑众多的类型定义，引用，需要对所有类型进行划分，文件结构梳理，有哪几种方式？考虑以上涉及到的文件和使用地方，类型导出声明的几种方式，类型分布项目中的文件结构可能有哪几种？

### 2. 先提炼类型最多，最频繁使用的文件：`server/api.ts`, `store/modules`, `components/*.vue`

1. 提炼相近的结构

    ```ts
    // 以api.ts为例：前置条件为 基础方法都已经封装完毕，直接使用即可：get, post, put等
    import { post, get } from 'path/to/server/index'
    // 管理后台项目考虑众多的list接口返回的结构基本都是一致的，所以以下泛型其实并不需要
    const getList = <T>(data: {idList: string[]}) => {
      return get<T>('/path/to/get/list', data)
    }
    // 修改如下
    import { ListReturnType } from '/path/to/interfaceAPI'
    const getList = <ListReturnType>(data: {idList: string[]}) => {
      return get<ListReturnType>('/path/to/get/list', data)
    }
    // interfaceAPI.ts
    export interface ListReturnType {
      records: string[] | number[] | Record<string, unknown>[]
      page: number
      total: number
    }
    ```

2. 使用泛型，先不考虑合理的问题

    ```ts
    // 考虑一种情况，返回类型由参数类型决定，此时最简单的就是泛型
    import { post, get } from 'path/to/server/index'
    const getListTypeByParamsType = <T>(data: T) => {
      return get<T[]>('/path/to/get/list', data)
    }
    // 在组件A.vue中使用：一个异步函数中获取返回值的类型，以便后续的操作
    const getListFinallyA = async () => {
      // getListTypeByParamsType: number[]
      const getNumberList = await getListTypeByParamsType<number>()
      const handleNumList = getNumberList.filter(v => v > 1000)
    }
    // 在组件B.vue中使用
    const getListFinallyB = async () => {
      // getStringList: string[]
      const getStringList = await getListTypeByParamsType<string>()
      const handleStrList = getStringList.map(v => v.length)
    }
    ```

3. 将所用类型分类拆分，大量使用联合类型、交叉类型

    ```ts
    // 考虑一种实际业务情况，一位客户的公开信息由：基础信息 & 会员信息 & 订单信息组成
    // 对应的接口在不同页面或不同条件下返回的可能会只有其中1种或多种，而且只能在页面使用时才能确定类型，那么此时的接口返回类型怎么写比较好呢？
    // interfaceAPI.ts
    export type GenderType = 'f' | 'm' | 1 | -1 | 2; // 考虑兼容情况
    export interface GoodsType {
      name: string
      amount: number
      id: string
      type: string
    }
    export interface BaseInfo {
      nickName: string
      anyAge: number
      gender: GenderType
      children?: BaseInfo[]
    }
    export interface MemberInfo {
      level: 1 | 2 | 3 | 4 | 5
      numMember: string | number
      bonusPoint: number
      timesConnected: number
    }

    export interface OrderInfo {
      channels: 'jd' | 'tmall' | 'pdd' | 'amazon'
      totalAmount: number
      purchasedGoods: GoodsType[]
    }
    // api.ts
    import { get } from 'path/to/server/index'
    const getCustomerInfo = <T>(data: { id: string; type?: number }) => {
      return get<T>('/path/to/get/list', data)
    }

    const getCustomerInfoA = (id: string) => {
      return getCustomerInfo<BaseInfo>({id, type: 0})
    }

    const getCustomerInfoB = (id: string) => {
      return getCustomerInfo<BaseInfo & MemberInfo>({id, type: 1})
    }

    const getCustomerInfoC = (id: string) => {
      return getCustomerInfo<BaseInfo & MemberInfo & OrderInfo>({id, type: 2})
    }

    const resA = getCustomerInfoA('123').then((res) => {
      const { nickName } = res
      console.log(nickName)
    })

    const resB = getCustomerInfoB('123').then((res) => {
      const { nickName, level } = res
      console.log(nickName, level)
    })

    const resC = getCustomerInfoC('123').then((res) => {
      const { nickName, level, purchasedGoods } = res
      console.log(nickName, level, purchasedGoods)
    })
    ```

4. 使用部分联合类型时，考虑用 `Pick` `Omit` 等方法 

```ts
// 可选
// Partial<Type>
interface Todo {
  title: string
  dess: string
  children: string[]
}
type PartialTodo = Todo = Partial<Todo>
// Equal to
type PartialTodo = {
  title?: string
  desc?: string
  children?: string[]
}
// 必选
// Required<Type>
type RequiredPartialTodo = Required<PartialTodo>
// Equal to Todo

// 只读
// Readonly<Type>
function Freeze<T>(obj: T): Readonly<T>

// Pick<Type, Keys> 只使用部分属性 - Keys = Type里的字符串或者字符串集合（并集）：'name', 'name' | 'title' | 'desc' 这2种形式
// Omit<Type, Keys> 忽略部分属性 - Keys = Type里的字符串或者字符串集合（并集）：'name', 'name' | 'title' | 'desc' 与 Pick<Type, Keys> 正好相反
type partProps = 'title' | 'desc'
type TodoPreview = Pick<Todo, partProps>
const todoPreview: TodoPreview = {
  titel: 'xxx',
  desc: '555'
}

// Exclude<Type, ExcludeUnion> 排除联合类型内的某个或者某些类型
type T0 = Exclude<'a' | 'b' | 'c', 'a'> // type T0 = 'a' | 'b'
type T1 = Exclude<'a' | 'b' | 'c', 'a' | 'c'> // type T0 = 'b'

// Extract<Type, Union> 提取 Type 和 Union 的交集
type T2 = Extract<'a' | 'b' | 'c', 'a' | 'f'> // type T2 = 'a'

// 与函数相关的实用方法
// Parameters<Type> 提取函数参数类型
// ConstructorParameters<Type> 提取构造函数参数类型：ErrorConstructor, FunctionConstructor, RegExpConstructor 等
// ReturenType<Type> 提取函数返回类型


// 不常用到的实用方法
// ThisParameterType<Type> 提取`有this`的函数参数类型，或者提不到就是 unknown
// ThisType<Type> 不返回转换类型，只作为一个上下文相关的 this 类型 的标记，必须`tsconfig.ts`开启 `noImplicitThis`

// 内置字符串操作类型
// Uppercase<StringType> 全部大写
// Lowercase<StringType> 全部小写
// Capitalize<StringType> 首字母大写
// Uncapitalize<StringType> 非首字母大写
```


## 部分答案，仅供参考，如有错误，概不负责，请自行甄别

```ts
// some-detail.vue script.lang = ts
import { defineComponent, reactive, ref, toRefs, PropType } from 'vue'
import { useStore } from 'vuex'
import { key } from 'path/to/store'

export default defineComponent({
  props: {
    list: Array as PropType<string[]>, // round 0: props 使用 PropType 强制转换类型为具体类型
    default() {
      return []
    }
  },
  // setup 内部
  // round 1: 给props 提供具体的类型 { list: string[] }
  setup: (props: {list: string[]}) =>  {
  // round 2: 
    const leftType = reactive<{
      rightType: number
      list: {label: string;value: number | string; extraProps?: string}[]
    }>({
      rightType: 0,
      list: []
    })
    // round 3: v: {label: '', value: 0, extraProps?: ''}
    const newArray = leftType.list.map(v => {
      return {
        ...v,
        extraProps: v.label
      }
    })
    // round 4: 只要没有明确 return 都是void, 特殊情况为 never
    const textFunc = () => {
      // do something but do not return anything
    }
    // round 5: 简单类型可以自动推导
    const hasTrue = ref(false)
    const isStr = ref('initStr')

    // round 6: props在setup引入的时候就应该定义list类型，此处是 string[]
    const { list } = toRefs(props) // list: string[]
    // round 7: 在createStore 就已经定义完成，不必在左边加 any
    export const key: InjectionKey<Store<LoginState & PermissionState & UserState>> = Symbol('index')
    export const store = createStore<LoginState & UserState & PermissionState>({
      modules
    })
    // store 类型即推导出的类型
    const store = useStore(key)
    // round 8: 参照 api.ts 的修改
    const getData = getDataByAxiosApi().then(res => {
      const ret = res
    })
  })
}
```
