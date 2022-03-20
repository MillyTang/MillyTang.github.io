---
title: 复习前端基础知识
date: 2021-11-04 16:10:13
tags: js, ts, react, vue3
---

## es6 ~ 10
<!-- ## vue3 -->
<!-- ## react -->
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

## 其他示例代码

1. reTryRequest: 接口轮询，一分钟内轮询直到成功，超时取消
2. generator or async/await 实现排序动画
3. vue3 composed 组件示例 + vue3 实现 formWrapper 类型标注
4. 设计一个 `generator` 状态机，和一个栈保留10个不同时间发出的异步任务，取消栈内除了栈顶的所有异步任务

## debug


## 如何上报准确错误信息

### 不同类型的错误捕获
### 错误上报方式
### 错误处理