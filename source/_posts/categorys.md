---
title: 部分范畴论(category-theory-for-programmers) & 高阶类型及其他
date: 2022-01-25 15:47:17
tags: category-theory-for-programmers
---

## 高阶类型及其他

### 更加通用的 map

1. TS 目前支持[函数重载](https://www.typescriptlang.org/docs/handbook/2/functions.html#function-overloads)

> `Optional<T>`见前文[编程与类型系统 - 编程部分:第三章组合-使用类型表达多选一](https://millytang.github.io/2022/01/11/ilearnTS/)

```ts 'Optional map() & 和类型 map()'
  function map<T, U>(optional: Optional<T>, func: (value: T) => U):Optional<U>;
  function map<T, U>(value: T | undefined, func: (value: T) => U): U | undefined;
  function map<T, U>(a: Optional<T> | T | undefined, func: (value: T) => U) {
    if (!a) return undefined
  if ('getValue' in a) {
    if (a.hasValue()) {
      return new Optional<U>(func(a.getValue()))
    } else {
      return new Optional<U>()
    }
  } else {
    return func(a)
  }
}
```

在一个泛型类型上映射函数，map提供了另外一种方式来将存储数据的类型与操作该数据的函数解耦

```ts '将Box<T>的值取出，应用函数，然后放回一个Box'
class Box<T> {
  value: T;
  constructor(value: T) {
    this.value = value
  }
}
function map<T, U>(
  box: Box<T>,
  func: (value: T) => U
): Box<U> {
  return new Box<U>(func(box.value))
}
```

#### 处理结果或传播错误

```ts 使用process传播错误
// process不对undefine做任何有用的处理
// process有逻辑处理分支
function process(): string | undefined {
  let value: number | undefined = readNumber()
  if (!value) return undefined
  return stringify(square(value))
}
```

```ts 使用 map 处理错误
// 和类型map
function map<T, U> (
  value: T | undefined,
  func: (value: T) => U
): U | undefined {
  if (!value) return undefined
  return func(value)
}
// process 实现没有分支，将错误传播的工作委托给map
function process(): string | undefined {
  let value: number | undefined = readNumber()
  // let squareValue: number | undefined = map(value, square)
  // return map(squareValue, stringify)
  // 直接使用lambda处理
  return map(value, (value: number) => stringify(square(value)))
}
```

#### 混搭函数的应用- map的另外一种应用

> 函子(functor): 执行映射操作函数的推广。对于任何泛型类型，以 `Box<T>` 为例，如果 `map()` 操作接受一个 `Box<T>` 和一个从 T 到 U 的函数作为实参，并得到一个 `Box<U>`，那么该 `map()` 就是一个函子

函数的函子：给定一个有任意数量的实参且返回类型 `T` 的值的一个函数，映射一个函数，使其接受 `T` 宗纬实参返回一个 `U`，从而使得到的函数接受与原函数相同的输入，但是返回类型 `U` 的一个值。

```ts 函数map和在一个函数上应用map
function map<T, U>(
  f: (arg1: T, arg2: T) => T,
  func: (value: T) => U
): (arg1: T, arg2: T) => U{
  return (arg1: T, arg2: T) => func(f(arg1, arg2))
}
// 应用map
function add(x: number, y: number) {
  return x + y
}
function stringify(value: number): string {
  return value.toString()
}
const result: string = map(add, stringify)(40, 2)
```

#### 习题
有一个接口 `IReader<T>`，定义了一个单独的方法 `read(): T`，实现一个函子：在 `IReader<T>` 上映射函数 `(value: T) => U`返回一个 `IReader<T>`

```ts
interface IReader<T> {
  read(): T
}
class MappedReader<T, U> implements IReader<T> {
  reader: IReader<T>;
  func: (value: T) => U;
  constructor(reader: IReader<T>, func: (value: T) => U) {
    this.reader = reader;
    this.func = func
  }
  read(): U {
    return this.func(this.reader.read())
  }
}
function map<T, U>(
  f: IReader<T>,
  func: (value: T) => U
): IReader<U>{
  return new MappedReader(reader, func)
}
```

### 单子

> Either 函数见 [编程与类型系统 - 编程部分:第三章组合-复合类型-自制Either](https://millytang.github.io/2022/01/11/ilearnTS/)

对于一个数据处理管道在链接了多个处理函数的情况来看：一旦出现错误情况的处理函子只会在管道中传播**最初的错误**，而不会进行处理。如果管道中的每个步骤都可能失败，那么函子就无法工作。由于函子规定前一个实参的返回值的类型是后一个处理函数的实参类型，一旦管道中的某个环节出错，返回的是一个错误类型，函子的第2个参数类型将出现不兼容的情况。

>  改造map函数，使其从 `T` 得到 `Either<Error, U>`，如下：

```ts 'Either bind()'
function bind<TLeft, TRight, URight> (
  value: Either<TLeft, TRight>,
  func: (value: TRight) => Either<TLeft, URight> // func的返回类型与map中的类型不同
): Either<TLeft, URight> {
  if (value.isLeft()) return Either.makeLeft(value.getLeft())
  // 简单返回对其应用 func 结果，而不是 Either.makeRight<func(value.getRight())>，只要保证 func 返回类型是 Either 即可
  return func(value.getRight())
}
```

使用 `bind()` 实现 无分支的 `readCatFromFile()`

```ts '无分支的 readCatFromFile()'
declare function openFile(path: string): Either<Error, FileHandle>;
declare function readFile(handle: FileHandle): Either<Error, string>;
declare function deserializeCat(serializeCat: string): Either<Error, Cat>;
function readCatFromFile(path: string): Either<Error, Cat> {
  let handle: Either<Error, FileHandle> = openFile(path)
  let content: Either<Error, string> = Either.bind(handle, readFile)
  return Either.bind(content, deserializeCat)
}
```

#### `map() 与 bind() 的区别`

以 `Box<T>`为例，关注在泛型上下文中，`map() 和 bind()` 如何处理 `T`类型， `U`类型（`Box<T> & Box<U>, T[] & U[], Optional<T> & Optional<U>, Either<T> &  Either<U>`）

> 对于 `Box<T>`，函子（`map()`）接受一个`Box<T>`和一个从 `T` 到 `U` 的函数，返回一个 `Box<U>`；在一些场景中函数直接从 `T` 得到 `Box<U>`，`bind()`接受一个 `Box<T>` 和一个从 `T` 到 `Box<U>`的函数；

区别就是第二个参数 `func` 的参数到返回

#### 单子模式

单子由 bind()` 和另一个更加简单的函数组成。另外的这个函数接受一个类型 `T`，并将其封装到泛型类型中，如 `Box<T>`、`Optional<T>`、`Either<T>`、`T[]`。该函数通常叫做 `return()` 或 `unit()`

> 单子模式：单子是一个泛型类型 `H<T>`。对于该类型，有一个 `unit()` 可接受类型 `T` 的一个值并返回类型 `H<T>` 的一个值。还有一个函数 `bind()` 可接受`H<T>` 的一个值和一个从 `T` 到 `H<U>` 的函数，并返回类型 `H<U>` 的一个值。

#### continuation 单子

看以下代码：`then()`只是 `Promise<T>` 对 `bind()` 的一种称呼；而 `Promise.resolve()` 是 `Promise<T>` 对 `unit()` 的一种称呼

```ts 链接promise
declare function getLocation(): Promise<Location>
declare function hailRideshare(location: Location): Promise<Car>;
let car: Promise<Car> = getLocation().then(hailRideshare)
```

#### 列表单子

除数的例子

```ts 除数&所有除数
// 除数
function divisors(n: number): number[] {
  let result: number[] = []
  for(let i = 2; i< n/2;i++) {
    if (n % i == 0) {
      result.push(i)
    }
  }
  return result
}
// 所有除数
function allDivisors(ns: number[]): number[] {
  let result: number[] = []
  for(const n of ns) {
    result = result.concat(divisors(n))
  }
  return result
}
```

```ts 所有变位词
declare function angram(input: string): string[]
function allAngrams(inputs: string[]: string[]) {
  let result = string[] = []
  for(const input of inputs) {
    result = result.concat(angram(input))
  }
  return result
}
```

将`allDivisors()`和`allAngrams()`替换为一个泛型函数。该函数将接受一个 `T`数组 和从`T`到 `U` 数组的一个函数，并返回一个`U` 数组

```ts '列表bind()'
function bind<T, U>(inputs: T[], func: (value: T) => U[]): U[] {
  let result: U[] = []
  for (const input of inputs) {
    result = result.concat(func(input))
  }
  return result
}
function allDivisors(ns: number[]): number[] {
  return bind(ns, divisors)
}
function allAngrams(inputs: string[]): number[] {
  return bind(inputs, angram)
}
```

#### 其他单子

1. 状态单子：封装一个状态，并把该状态与值一起传递。在给定当前状态时，它将生成一个值和一个更新后的状态。使用 `bind()` 将这些函数链接起来在管道中传播和更新状态，而不需要显示的在一个变量中存储状态，从而能够使用纯粹的函数代码来处理和更新状态。

2. IO单子：封装了副作用。允许实现能够读取用户输入或写入文件或终端的纯粹函数，也因为不纯粹的行为从函数中移除了出来封装到了 **IO单子** 中。

#### 习题

```ts 'Lazy(): T'
type Lazy<T> = () => T;
function lazyUnit(val: T): Lazy<T> {
  return () => val
}
function lazyMap<T, U>(f: Lazy<T>,  func: (val: T) => U): Lazy<U> {
  return () => func(f())
}
function lazyBind<T, U>(f: Lazy<T>, func: (val: T) => Lazy<U>): Lazy<U> {
  return func(f())
}
```

### 继续学习

## 补充：范畴论部分内容

> 对《编程与类型系统》部分章节内容的补充，不代表本人对本书有深刻理解
### 第五章-Products and Coproducts

### 第六章-simply ADT(AlgeBraic Data Type) 简单代数数据类型

#### Product Type 乘积
#### Records 记录
#### Sum Type 和类型
#### Algebra of types

### 第九章-Function Types

#### Currying柯里化

### 第十三章-Free Monoids 自由单子

## Learn you a Haskell 的代码对比补充

### 函子

### 单子
