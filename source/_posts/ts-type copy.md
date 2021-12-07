---
title: TypeScript 类型体操练习
date: 2021-09-12 19:00:21
tags:
---

> 挑战类型体操答案合集，一部分是自己做的，一部分是自己抄的

## 热身运动

```ts
// 期望是一个 string 类型
// any -> string
type HelloWorld = string
// 你需要使得如下这行不会抛出异常
type test = Expect<Equal<HelloWorld, string>>
```

## 实现Pick

```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

type MyPick<T, K extends keyof T> = {
  [P in K]: T[P] 
}

type TodoPreview = MyPick<Todo, 'title' | 'completed'>

const todo: TodoPreview = {
    title: 'Clean room',
    completed: false,
}
```

## Readonly

```ts
interface Todo {
  title: string
  description: string
}

type MyReadonly<T> = {
  readonly [P in keyof T]: T[P]
}

const todo: MyReadonly<Todo> = {
  title: "Hey",
  description: "foobar"
}

todo.title = "Hello" // Error: cannot reassign a readonly property
todo.description = "barFoo" // Error: cannot reassign a readonly property
```

## Omit

```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

type MyOmit<T, K extends keyof T> = {
  [P in keyof T as P extends K ? never: P]: T[P]
}

type TodoPreview = MyOmit<Todo, 'description' | 'title'>

const todo: TodoPreview = {
  completed: false,
}
```

## Exclude

```ts
// 从T中排除可分配给U的类型
// T, U 都是类型
// MyExclude<'a' | 'b' | 'c', 'a'> // 输出 'b' | 'c'
type MyExclude<T, U> = T extends U ? never : T
```

```ts
// PromiseAll
type PromiseAll<>
const promise1 = Promise.resolve(3);
const promise2 = 42;
const promise3 = new Promise<string>((resolve, reject) => {
  setTimeout(resolve, 100, 'foo');
});

// expected to be `Promise<[number, number, string]>`
const p = Promise.all([promise1, promise2, promise3] as const) -->

```

> [类型体操链接](https://github.com/type-challenges/type-challenges/)
