---
title: React@17 & Router@16 项目总结
date: 2022-01-04 10:25:57
tags: React@17, Router@16
---

## 重表单功能项目

整个项目80%都是填写表单，一开始没防备，生写新建和编辑，痛苦不堪，后来上了 antd 的 Form 表单组件好多了，但是时间紧迫改的又急，业务代码变的很混乱，先记录一下重构思路，声明：代码均已脱敏。

### 示例组件： 1个parent组件+3个child组件，其中，某个child还嵌套了2层，层级较深

```ts
// parent.tsx
// childA.tsx
// childB.tsx
// childC.tsx
```

#### 页面卡顿，function 函数组件的UI更新问题

```tsx
// parent.tsx
import React, { useState, useEffect } from 'react'
export const Parent = () => {
  const [count, setCount] = useState(0)
  return <ChildA count={count} onchange={(c) => setCount(c)} />
}
// ChildA.tsx
export const ChildA = ({count, onChange}: {count: number; onChange: (c: number) => void;}) => {
  useEffect(() => {
    
  }, [count])
  return (
    <div onClick={() => }>You click it {count} times</div>
  )
}
```
#### 页面卡顿，children 组件被重复渲染: 使用 useMemo & useCallback & useReducer 替换 useState
#### 页面卡顿，child 的 useEffect 狂刷页面，因为有依赖是多层嵌套引用对象类型
#### useEffect 内部接口请求的处理 + 外部请求 = 出现2个相同定义函数 = 冗余代码

#### 组件组织结构优化

有2层嵌套的map渲染组件，原来的思路是`EitherBox` 内直接 `map => AndBox`，而 `AndBox` 内也 `map =>(<div>xxx</div>)`

```tsx 原思路
export const EitherBox = ({children: any}: props) => {
 const style = {
    ors: data.length
  }
  return <div className={style.ors}>{children.length > 1 && (<div>or</div>)}{children}</div>
}
export const AndBox = ({children: any}: props) => {
  return <div>{children.length > 1 && (<div>and</div>)}{children}</div>
}

export default function pageA () => {
  const data = getSomeData<Record<string, Record<string, unknown>[]>[]>() ?? []
  return (
    <EitherBox>
      {data.length === 0 && (<div>add</div>)}
      {data.length > 0 && data?.map(item => (
        <AndBox>
          {item.length === 0 && (<div>none</div>)}
          {item.length > 0 && item?.map(v => (
            <div onClick={}>{v}</div>
          ))}
        </AndBox>
      ))}
    </EitherBox>
  )
}
```

```tsx 优化后
// 改下名称
export const EitherUnit = ({data: Record<string, Record<string, unknown>[]>[], children: TSX.element}: props) => {
  const style = {
    ors: data.length
  }
  return <div className={style.ors}>{children.length > 1 && (<div>or</div>)}{children}</div>
}
export const AndUnit = ({children: TSX.element}: props) => {
  return <div>{children.length > 1 && (<div>and</div>)}{children}</div>
}
export const EitherMap = ({data: Record<string, Record<string, unknown>[]>[], children: TSX.element}: props) => {
  if (!Array.isArray(data)) {
    return <div>暂无</div>
  }
  if (data.length === 0) {
    return <div>add</div>
  }
  return <EitherUnit data={data}>{children}</EitherUnit>
}
export const AndMap = ({data, children, ...rest}: props) => {
  if (!Array.isArray(data)) {
    return <div>input Error</div>
  }
  if (data.length === 0) {
    return <div>none</div>
  }

  return (
    <AndUnit>
    {data.map(v => (
      <div>{v}</div>
    ))}
    </AndUnit>
  )
}
export default function pageA () => {
  const data = getSomeData<Record<string, Record<string, unknown>[]>[]>() ?? []
  return (
    <EitherMap data={data}>
      {data?.map(item => (
        <AndBox data={item} />
      ))}
    </EitherMap>
  )
}
```

#### 使用 immer / immutable 


## 类型复用逻辑

> type && interface 的使用，考虑项目中类型，简短的可做基础类型的优先考虑 type， 浏览器请求接口的参数和返回值的类型使用 interface，函数&其他变量，集合类型 优先使用 type

> type和interface 区别在于：inteface无法应用于union type | intersection type | conditional type | tuple，但是可以 augumented 而type不行

> 把类型当作值的集合，理解 类型中的 union type - 交集 和 intersection type -并集

```ts
// type
type AB = 'A' | 'B' // union type 并集
type NamedVariable = {age: number} & { name: string} // intersection type 交集
type mytuple = [number, string] // tuple元组
// inteface
// a.ts
export interface BaseInfo {
  name :string;
  age: number;
}
// b.ts
import { BaseInfo } from 'path/to/a.ts'
interface BaseInfo {
  phoneNum: string
}
const base: BaseInfo = {
  name: 'xx',
  age: 21,
  phoneNum: '19xxxxxxxxx'
}
// 把类型当作值的集合
type AA = {aa: 'A'}
type BB = {bb: 'B'}
type ABor = AA | BB
type ABand = AA & BB
type AAkeys = keyof AA // 'aa'
type BBkeys = keyof BB // 'bb'
type ABandKeys = keyof ABand // 'aa' | 'bb', keyof ABand === (keyof AAkeys) | (keyof BBkeys)
type ABorKeys = keyof ABor // never, keyof ABor === (keyof AAkeys) & (keyof BBkeys)

const aaa: ABand = {
  aa: 'A',
  bb: 'B'
}
```

## useEffect: 基于 useMemo & useCallback 导致的更新问题

写Vue3写多了，初次写React项目很不适应，首先面临的一个大问题就是 useEffect 更新 child 组件，数次进入无限循环，把浏览器搞崩了。由于我本人首次写 React 上了强 debuff, 导致首次写的 react 项目业务代码一坨屎，本人痛定思痛，决定对业务代码脱敏后重构一次，以彰显写更棒的代码的决心


## 改进： 考虑使用 immer.js

## 改进后： 思考 Class 组件 🆚 函数组件 的便利性

> Class & enum 横跨值&类型

## 其他： 
1. 使用 Vue3 实现相似功能