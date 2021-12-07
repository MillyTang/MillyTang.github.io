---
title: 复习前端基础知识
date: 2021-11-04 16:10:13
tags: js, ts, react, vue3
---

## ts

1. `class` & `enum` 横跨 类型空间 & 值空间
2. 泛型 可以约束 参数和返回值，而泛型可以用 extends 来约束
3. keyof 在类型空间使用（基本），返回的是集合（并集）
4. -? 操作符筛出 undefined 

    ```ts
    interface Shoes {
      sizes: number[]
      types: string
      colors: string[]
    }
    type ShoesKey = keyof Shoes
    ShoesKey = 'sizes' // 👌
    ShoesKey = 'types' // 👌
    ShoesKey = 'cost' // 🙅‍♂️
    // T = sizes | types | cost, 这里泛型T的类型已经被约束了
    function buyAShoe<T extends ShoesKey>(a: T): T {
      return a
    }
    ```
5. 函数重载解决以下情况

```ts
// 多参数函数，函数返回值与单个参数类型相关，参数之间可能有某种关联关系
function 
```

## es6 ~ 10
## vue3
## react
## node
## 几种通用模式
1. 单例模式
2. 观察者模式
3. 代理模式/装饰器模式
## 作用域

```js
// 《JS二十年》
// ES2015

//  1. 三表达式内的作用域
// for (const p in x) {body} 的去糖后近似表示
// 循环中的每次迭代使用一个绑定，并在迭代之间传递值
// 对于 const p 来说，每次使用循环的唯一绑定就足够了，因此， p 不能被 for 头部或者循环体内的表达式修改 
{ let $next;
  for ($next in x) {
    const p = $next;
    {body}
  }
}
// 2. 严格模式下禁止块级函数声明
```
