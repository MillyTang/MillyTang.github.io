---
title: TypeScript 类型体操练习
date: 2021-09-12 19:00:21
tags:
---

> 挑战类型体操答案合集，一部分是自己做的，一部分是自己抄的

## 热身运动

```ts
// 期望是一个 string 类型 👉 so eazy
// any -> string
type HelloWorld = string
// 你需要使得如下这行不会抛出异常
type test = Expect<Equal<HelloWorld, string>>
```

## 部分前置知识: keyof, typeof, extends 等，具体请移步到官方文档查阅详情

```ts
// typeof Type Operator
typeof 'hello' // string
function f(a?: string) {
  return {x: 10, y: 1, ...a} // 返回具体值，而不是类型
}
type F0 = ReturnType<f> // ❌ 'f' refers to a value, but is being used as a type here, you should use 'typeof f'
type F1 = ReturnType<typeof f> // {x: number; y:number; a?: string}
// typeof 后面不能跟执行函数
// meant to use = ReturnType<typeof f>
let shouldcontinue: typeof f('aaaa') // ❌

// keyof Type Operator
type Point = {a: string, b: string}
type P0 = keyof Point // type P0 = 'a' | 'b'

// as 类型断言：key Remapping via as
type NewKeyType<P> = Capitalize<string & P>
type MappedTypeWithNewProperties<T> = {
  [P in keyof T as NewKeyType<P>]: T[P]
}
// 条件类型通过never来过滤key
type RemoveKindField<T> = {
  [P in keyof T as Exclude<P, 'kind'>]: T[P]
}
interface Circle {
  kind: 'circle';
  radius: number
}
type kindlessCircle = RemoveKindField<T> // type kindlessCircle = {radius: number}

// extends 继承，扩展，衍生，Class继承，非实例，继承单个Class，衍生多个interface，类型约束 K extends keyof T，K 是 T的子类型（类似于Class A extends Class B, A 是 B的子类一个意思）
// 类继承👉协变&逆变
// infer 待推断的类型

// InstanceType<Type> 提取构造函数的实例函数类型：Class
class C {
  x = 0
}
// 类 & 枚举 横跨 类型空间 & 值空间，所以 C 也算一种类型，但是 `typeof C !== C`, 因为 `typeof C` 并非 `C` 的类型，而`InstanceType<typeof C>`才是, 而 `C` 是 构造函数 `constructor`
type C0 = InstanceType<typeof C> // type C0 = C
```

## 实现 `Pick<T, Keys>`

```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

// Pick<Type, Keys> 👉 MyPick<Type, Keys> 使用方法一致

type MyPick<T, K extends keyof T> = {
  [P in K]: T[P] // P in K, K = 'title' | 'description' | 'completed', `in` means for P in K
}

type TodoPreview = MyPick<Todo, 'title' | 'completed'>

const todo: TodoPreview = {
    title: 'Clean room',
    completed: false,
    description: 'xxxx' // ❌，description 多余属性
}
```

## `Readonly<T>`

```ts
interface Todo {
  title: string
  description: string
}
// ReadOnly<Type> 👉 MyReadOnly<Type>

type MyReadOnly<T> = {
  readonly [P in keyof T]: T[P]
}

const todo: MyReadonly<Todo> = {
  title: "Hey",
  description: "foobar"
}

todo.title = "Hello" // Error: cannot reassign a readonly property
todo.description = "barFoo" // Error: cannot reassign a readonly property
```

## `DeepReadonly<T>` 🌟

```ts
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends {} ? DeepReadonly<T[P]> : T[P]
}
```

## `ReadonlyAndPick<T, Keys>` 🌟

```ts
type ReadonlyAndPick<T, K extends keyof T> = {readonly [P in keyof T as P extends K ? P : never]: T[P]} & {[P in keyof T as P extends K ? never : P]:[P]}
// 使用 Exclude 替代 P extends K ? never : P && 使用 Pick 替代  P extends K ? P : never
type ReadonlyAndPick<T, K extends keyof T> = {readonly [P in keyof Pick<T, K>]: T[P]} & {[P in keyof T as Exclude<P, K>]: T[P]}
// or
type ReadonlyAndPick<T, K extends keyof T> = {readonly [P in keyof Pick<T, K>]: T[P]} & {[P in Exclude<keyof T, K>]: T[P]}
// 进一步 替换 readonly
type ReadonlyAndPick<T, K extends keyof T> = Readonly<Pick<T, K>> & {[P in Exclude<keyof T, K>]: T[P]}

```

## `Omit<T, Keys>`

```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

// Omit<Type, Keys> 👉 MyOmit<Type, Keys>
type MyOmit<T, K extends keyof T> = {
  [P in keyof T as P extends K ? never: P]: T[P]
}
// 跟上面用 never 过滤属性的方式一致，所以还可以这么写
type MyOmit1<T, K extends keyof T> = {
  [P in keyof T as Exclude<P, K>]: T[P];
}

type TodoPreview = MyOmit<Todo, 'description' | 'title'>

const todo: TodoPreview = {
  completed: false,
}
```

## `Exclude<T, U>`

```ts
// 从T中排除可分配给U的类型
// T, U 都是字符串类型
// MyExclude<'a' | 'b' | 'c', 'a'> // 输出 'b' | 'c'
type MyExclude<T, U> = T extends U ? never : T
```

## `ReturnType<T>` 提取函数的返回类型

```ts
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any
```

## `Parameters<T>` 提取函数的参数类型

```ts
type Parameters<T> = T extends (...args: infer P) => any ? P : T
```
## `First<T>` of Array 实现一个泛型类型, 使得可以返回数组类型的第一项

```ts
type arr1 = ['a', 'b', 'c']
type arr2 = [1, 2, 3]
type head1 = First<arr1> // 'a'
type head2 = First<arr2> // 1

type First<T> = T extends any[] ? never : T[0]
```
## `Length<T>` of Tuple

```ts
type tesla = ['tesla', 'model3', 'model X', 'model Y']
type Length<T> = T extends any[] ? T['length'] : never
type teslaLength = Length<tesla> // 4
```
## `Last<T>` of Array 实现一个泛型类型, 使得可以返回数组类型的最后一项

```ts
type arr1 = ['a', 'b', 'c']
type arr2 = [1, 2, 3]
type tail1 = Last<arr1> // 'a'
type tail2 = Last<arr2> // 1
// F 只是一个变量，代表最后一项之前的所有项
type Last<T extends any[]> = T extends [...infer F, infer E]? E: never;
```

## `Concat<T, P>`: 实现一个Array.concat 方法功能相同的类型


```ts
type Concat<T, P> =T extends any[] ? P extends any[] ? [...T, ...P] : never : never;
// or type Concat<T extends any[], P extends any[]> = [...T, ...P]
type Result = Concat<[1], [2]> // [1, 2]
```

## `Push<T, P>`: Array.push 类型方法

```ts
type Push<T extends any[], P extends any> = [...T, P]
type Result = Push<[1, 2], '3'> // [1, 2, '3']
```

## `Unshift<T, P>`: Array.unshift 类型方法

```ts
type Unshift<T extends any[], P extends any> = [P, ...T]
type Result = Unshift<[1, 2], '3'> // ['3', 1, 2]
```


## `Pop<T>` 🌟

```ts
type Pop<T> = T extends [...infer P, infer E] ? P : never;  
```

## `Shift<T>` 🌟

```ts
type Shift<T> = T extends [infer E, ...infer P] ? P : never;  
```

## `Promise.all` 🌟🌟

```ts
type inferTupleT<T extends any[]> = {
  [P in keyof T]: T[P] extends Promise<infer S> ? S : T[P]
}
declare function PromiseAll<T extends any[]>(values: readonly [...T]): Promise<inferTupleT<T>>
const promise1 = Promise.resolve(3);
const promise2 = 42;
const promise3 = new Promise<string>((resolve, reject) => {
  setTimeout(resolve, 100, 'foo');
});
const p = PromiseAll([promise1, promise2, promise3] as const) // Promise<[number, 42, string]>
```

## `If<truthy, T, F>` 🌟🌟

> `If<true, T, F> => T, If<false, T, F> => F`

> 这里还需要考虑 `any`, `never`, `boolean `被视为 `union type`, 同时 never是union运算的幺元【可在 category-theory-for-programmers 这本书上找到相关解释】, 但是 `unknown` 却没有

```ts
type If<C, T, F> = C extends true ? T : C extends false ? F : never; 
type A = If<true, 'a', 'b'> // 'a'
type B = If<false, 'a', 'b'> // 'b'
type C = If<'c', 'a', 'b'> // never
// 考虑输入  `any` && `boolean`
type D = If<boolean, 'a', 'b'> // 'a' | 'b', boolean 是 union type => distributive conditional types
type E = If<any, 'a', 'b'> // 'a' | 'b', any 是 union type => distributive conditional types
// 第一步去除 T 的 naked type 特点，怎么办呢？和下面 IsNever 处理一致, 加入 array, record, function 结构
type If<C, T, F> = [C] extends [true] ? T : [C] extends [false] ? F : never; 
// 此时 boolean 可以了，但是 any 还不行
type D = If<boolean, 'a', 'b'> // never
type E = If<any, 'a', 'b'> // 'a'
// any 还是不对，还需要破坏类型检查，处理方式就是加入 加入破坏条件: 破坏 T 作为 checked type 的条件 即可
// any extends C ? never : T
type If<C, T, F> = [C] extends [true] ? (any extends C ? never : T) : [C] extends [false] ? (any extends C ? never : F) : never;
// 此时
type D = If<boolean, 'a', 'b'> // never
type E = If<any, 'a', 'b'> // never
```

## 插播：conditional type & distributive conditional types

> distributive conditional types 的3个前提条件：1. T = naked type; 2. T = checked type; 3. T 实例化为 union type;

```ts conditional type
// T: checkedType 被检查的类型
// U: extendsType 判断条件
// 下面这个表达式的意思是：如果T能够assignable（这里不是subtype的意思）给U那么结果为X，否则结果为Y
// 根据官方文档的 assignable 图：
type checkTtype = T extends U ? X : Y
// 如果 T 是 naked type: 详见本文末尾解释
// 则表达式就是 distributive conditional types
// T = A | B | C // A,B,C是naked type
type unionTypes = string | number
type checkTtypeUnion<T> = T extends any ? T[] : T
type checkTtypeUnionI1 = checkTtypeUnion<unionTypes> //  string[] | number[]
// 如何消除 distributive conditional types
// 破坏上述3个条件即可，然后用 never 过滤掉该分支
```

## `TupleToUnion<T>` 元组转联合类型

```ts
type TupleToUnion<T extends any[]> = T[number]
```

## `TupleToObject<T>` 元组转对象

```ts
type tuple = ['tesla', 'model 3', 'model X', 'model Y']
// or
// const tuple = ['tesla', 'model 3', 'model X', 'model Y'] as const
type TupleToObject<T extends readonly any[]> = {
  [P in T[number]]: P
}
type result = TupleToObject<tuple> // {tesla: 'tesla', ...}
// when tuple is const
// type result = TupleToObject<typeof tuple>
```

## `IsNever<T>` 🌟

> conditional type 判断 IsNever 在某种情况下有问题：如果这么写 `type IsNever<T> = T extends never ? true : false` , 当 `type A = IsNever<never>` 时 `type A = never` 而不是 true or false, 查阅[怎么理解 conditional type 中的联合类型与 never](https://zhuanlan.zhihu.com/p/47590228)

```ts
// IsNever
type IsNever<T> = [T] extends [never] ? true : false
// 将输入变成 不可能是空列表的状态即可，[T] 🉑️， {a: T} 也 🉑️
```

## `Flatten<T>`

```ts
type Flatten<T> = T extends Array<infer I> ? I : T;
```


> [类型体操链接](https://github.com/type-challenges/type-challenges/)

## TroubleShooting

1. naked type: The type parameter is present without being wrapped in another type, (ie, an array, or a tuple, or a function, or a promise or any other generic type)



