---
title: 编程与类型系统 - 编程部分
date: 2022-01-11 15:39:42
tags: typeScript, javaScript, 编程与类型系统
---

## 第二章：基本类型

### never 与 自定义 never 类型

1. 无法实例化的类
2. 没有元素的枚举

```ts
export {}
declare const EmptyType: unique symbol; // symbols 的子类， 用于常量和属性名，必须用const不能用let
// 私有构造函数确保其他代码不能实例化这个类型
// 因此，也无法用 InstanceType 获取实例 类型
class Empty {
  [EmptyType]: void;
  private constructor() {}
}

// Empty 与 never 作用相当
// 因为不能创建 Empty 实例，所以根本不能添加 return 语句
function raise(msg: string): Empty {
  console.log(`Error "${msg}" raised at ${new Date()}`)
  throw new Error(msg)
}
// 没有值的枚举
enum EmptyType {}
```

### 单元类型

单元类型只允许有一个值，JS & TS 中都是 **void**，

> 利用函数的 **副作用**而不是返回值, React & Vue3: useEffect, useEffective -> 考虑这俩的返回值是什么？获得其真正的功能。函数接受任意数量的实参却不返回任意有意义的值，这种函数称为 **动作 or 消费者**；

1. `void`
2. 自制单元类型：只有一个值的枚举，或者没有状态的单例

```ts
// 只有一个值的枚举
enum UnitTypeSingle {
  value = 'void' // 值是什么并不重要，满足枚举特性即可
}
// 没有状态的单例
// 与上面的 Empty 类似，只不过有一个值，可以 return
declare const UnitType: unique symbol;
// 私有构造函数确保其他代码不能实例化这个类型
// 因此，也无法用 InstanceType 获取实例 类型
// 仅作为类型使用，无法实例化
class Unit {
  [EmptyType]: void;
  static readonly value: Unit = new Unit(); // 单个实例，静态只读属性
  private constructor() {}; // 不能实例化
}
function greet(): Unit {
  return Unit.value
}
```

### 数值类型常见陷阱

1. 整数类型与溢出

> 处理`上溢`和`下溢`的3种主要方式：环绕（简单丢弃不合适的位），饱和（停在最大值，算数运算失去结合性），报错

```ts
//  4位无符号编码表示：0~15，即 `b^N-1b^N-2....b1b0` 转为 10进制 `b^N-1 x 2^N-1 + ....b^0 x 2^0`
// 4位带符号编码表示：-8~7，首位1即负数，0即正数，负数编码是 2^N - (绝对值)
-8 = 2^4 - 8 // 
// 算数上溢 - 数超出表达范围
// 下溢 - 数太小无法给定 位数 表示

// 上溢 - 环绕： 1111(15) + 1 => 10000 超出4位，丢弃1位（从右到左）1 => 0000，即环绕回0
```

2. 浮点类型和圆整

> TS & JS: 使用 binary64 编码将数字表示为 64 位浮点数，0.10 的近似数为 `0.100000000000000005551115123216`，处理 0.1 时 近似数被圆整为 0.1，相加 3次后的和被圆整成`0.10000000000000004`，所以 0.30 !== 0.1 + 0.1 + 0.1

```ts
// 防止圆整
Number.isSafeInteger() // 一个整数值能否在不被圆整的情况下表示出来
```

3. 比较浮点数

> 确保2浮点数差值在给定阈值内，Js内的给定阈值是 `Number.epsilon`

4. 任意大整数JS:  `BigInt`

 JS 原生提供BigInt函数，可以用它生成 BigInt 类型的数值。转换规则基本与Number()一致，将其他类型的值转为 BigInt

```ts
typeof 123n // 'bigint'
typeof 123 // number
-42n // 正确
+42n // 报错 - BigInt 可以使用负数 - ，不能使用 +，与 asm.js 语法冲突
BigInt(123) // 123n
BigInt('123') // 123n
```

### 编码

> UTF-32 是固定编码，一个字符由4个字节组成，UTF-8是变长编码，字节数依赖字符数

### 使用数组和引用构建数据结构

固定大小数组（具有极快的读取/更新能力，利于表示稠密数据）：连续内存区域，不能在原内存位置增长或者缩减。如果要 push 一个值，则改变了 `固定`，此时则需要分配一个新数组，将之前的5个值复制过来

> 考虑 React 中业务对数组的处理：使用 slice & concat 而不是 push & pop，对 `不可变` 的处理

### 高效列表

1. 使用 `链表` 实现 列表数据结构: 访问某个元素开销较大 `T(N)`，append 则很容易
2. 使用 `数组` 实现 列表数据结构: 访问某个元素很容易，append 开销较大 `T(N)`

> 大部分语言都使用 具有额外容量的基于数组的列表实现

```ts
class NumberList {
  private numbers: number[] = new Array(1);
  private length: number = 0;
  private capacity: number = 1;
  at(index: number): number {
    if (index >= this.length) {
      throw new RangeError()
    }
    return this.numbers[index];
  }
  append(value: number) {
    if (this.length < this.capacity) {
      this.numbers[this.length] = value
      this.length++
      return
    }
    // 扩容
    this.capacity =  this.capacity * 2
    let newNumbers: number[] = new Array(this.capacity)
    // 复制原数组值到新扩容后的数组
    for (let i = 0; i< this.length; i++>) {
      newNumbers[i] = this.numbers[i]
    }
    // append value
    newNumbers[this.length] = value
    // 迁移数组
    this.numbers = newNumbers
    this.length++
  }
}
```



### 二叉树

1. 基于数组的二叉树实现 - 稀疏二叉树浪费空间

  ```ts
  // 树的下标从1开始
  class Tree {
    nodes: (number | undefined)[] = []
    // 给定上层节点的索引时，计算左侧节点位置
    left_node_index(index: number): number {
      return index * 2
    }
    // 给定上层节点的索引时，计算右侧节点位置
    right_node_index(index: number): number {
      return index * 2 + 1
    }
    add_level() {
      // 数组扩容
      let newNodes: (number | undefined)[] = new Array(this.nodes.length * 2 + 1)
      // 扩容后复制原数组值
      for (let i = 0; i < this.nodes.length; i++) {
        newNodes[i] = this.nodes[i]
      }
      // 迁移数组
      this.nodes = newNodes
    }
  }
  ```

2. 对树的根节点的引用来表现树 - 紧凑二叉树实现

  ```ts
  class TreeNode {
    value: number;
    left: TreeNode | undefined; // 引用其他节点
    right: TreeNode | undefined;// 引用其他节点
    constructor(value: number) { // 初始化节点
      this.value = value
      this.left = undefined
      this.right = undefined
    }
  }
  ```

### 关联数组 - 字典/哈希表（JS/TS）

> 固定大小数组+引用（引用的追加效果更好，利于表示稀疏数据），引用指向一个列表（参考上述高效列表）

## 第三章：组合

### 复合类型

1. 元组 `Tuple` - 固定长度数组（一组不同或相同类型），按分量值位置访问值，容易出错
2. 记录 `Record` - 通过属性名称访问值

  ```ts
  // 自定义元组
  class Pair<T1, T2> {
    m0: T1;
    m1: T2;
    constructor(m0: T1, m1: T2) {
      this.m0 = m0
      this.m1 = m1
    }
  }
  type Point2D = Pair<number, number>
  // Record
  class Point<T1, T2> {
    x: T1;
    y: T2;
    constructor(x: T1, y: T2) {
      this.x = x
      this.y = y
    }
  }
  // point1 只能有 x, y2个属性
  const point1: Point<number, number> = {
    x: 1,
    y: 4
  }
  ```
3. 维护不变量：class 创建的变量（object）可以有关联的方法
  - 不需要保证不变量，属性公有即可
  - 属性只读，也不需要维护不变量，在构造函数中验证初始值即可

> 确保值的格式正确的规则称为不变量：以JS/TS 为例，通过让属性私有，并提供更新私有属性方法，可以确保维持不变量的稳定

### 使用`类型`表达多选一

1. 使用枚举替代常量

  ```ts
  enum DayofWeek {
    Sunday,
    Monday,
    Tuesday,
    Webnesday,
    Thursday,
    Friday,
    Saturday
  }
  // 自定义可选类型
  class Optional<T> {
    private value: T | undefined;
    private assigned: boolean;
    constructor(value?: T) {
      this.value = value ? value : undefined
      this.assigned = value ? true : false
    }
    hasValue():boolean {
      return this.assigned
    }
    getValue(): T {
      if(!this.assigned) throw Error();
      return this.value as T
    }
  }
  ```

2. 结果或错误：返回一个结果或者返回错误

  ```ts
  enum InputError {
    NoInput,
    Invaild
  }
  class Result {
    error: InputError | undefined;
    value: DayofWeek | undefined;
    constructor(error?: InputError, value?: DayofWeek) {
      this.error = error || undefined
      this.value = value || undefined
    }
  }
  function parseDayofWeek(input: string): Result {
    if (!input) {
      return new Result(InputError.NoInput)
    }
    switch(input.toLowerCase()) {
      // sunday ~ monday
      // new Result(undefained, input)
      default:
        return new Result(InputError.Invaild);
    }
  }
  ```

3. 自制 Either 类型

  考虑上述的 `Result` 并不通用，进一步提取 Either，使其可以应用到其他地方

  > Either 类型 包含2个类型， TLeft & TRight, 约定 TLeft 存储错误类型， TRight存储有效值类型

  ```ts
  class Either<TL, TR> {
    private readonly value: TL | TR;
    private readonly left: boolean
    // 使用私有构造函数，因为需要确保 value 和 left 标志是同步的
    private constructor(value: TL | TR, left: boolean) {
      this.value = value
      this.left = left
    }
    isLeft(): boolean {
      return this.left
    }
    getLeft(): TL {
      if(!this.isLeft()) throw new Error()
      return <TL>this.value
    }
    isRight(): boolean {
      return !this.left
    }
    getRight(): TR {
      if(!this.isRight()) throw new Error()
      return <TR>this.value
    }
    static makeLeft<TL, TR>(value: TL) {
      return new Either<TL, TR>(value, true)
    }
    static makeRight<TL, TR>(value: TR) {
      return new Either<TL, TR>(value, false)
    }
  }
  // 使用 Either 改造上述 返回
  enum InputError {
    NoInput,
    Invaild
  }
  type Result = Either<InputError, DayofWeek>
  function parseDayofWeek(input: string): Result {
    if (!input) {
      return Either.makeLeft(InputError.NoInput)
    }
    switch(input.toLowerCase()) {
      case 'sunday':
        return Either.makeRight(DayofWeek.Sunday)
      // 其他同理
      // ...
      default:
        return Either.makeLeft(InputError.Invaild);
    }
  }
  ```

#### 变体类型：标签联合类型 (ts tagged union ?)

  > 变体类型包含任意数量的基本类型的值，标签指的是 即使基本类型有重合的值，仍然能够准确说明该值来自哪个类型

1. 先来欣赏一段体操

  ```ts
  // 将 tagged union 类型转为 union type
  interface Example {
    a: string;
    b: number;
  }
  type SingleProp<T, K extends keyof T> = {[P in keyof T]?: P extends K ? T[P] : never}
  type oneOf<T> = {[K in keyof T]: Pick<T, K> & SingleProp<T, K>}[keyof T]
  type ExamplePropUnion = oneOf<Example>
  ```

2. 几何形状集合：添加一个 `kind` 属性标签，方便使用时判断或者强制转换形状

  ```ts
  class Point {
    readonly kind: string = 'Point'
    x: number = 0;
    y: number = 0;
  }
  class Circle {
    readonly kind: string = 'Circle'
    x: number = 0;
    y: number = 0;
    radius: number = 0
  }
  type Shape = Point | Circle
  ```

3. 实现一个通用的变体：而不需要类型自身存储一个标签

  ```ts
  // 自制变体
  class Variant<T1, T2, T3> {
    readonly value: T1 | T2 | T3;
    readonly index: number;
    private constructor(value: T1 | T2 | T3, index: number) {
      this.value = value;
      this.index = index;
    }
    static make1<T1, T2, T3>(value: T1): Variant<T1, T2, T3> {
      return new Variant<T1, T2, T3>(value, 0);
    }
    static make2<T1, T2, T3>(value: T2): Variant<T1, T2, T3> {
      return new Variant<T1, T2, T3>(value, 1);
    }
    static make3<T1, T2, T3>(value: T3): Variant<T1, T2, T3> {
      return new Variant<T1, T2, T3>(value, 2);
    }
  }
  // 更新 几何形状集合
  class Point {
    x: number = 0;
    y: number = 0;
  }
  class Circle {
    x: number = 0;
    y: number = 0;
    radius: number = 0
  }
  class Rectangle {
    x: number = 0;
    y: number = 0;
    width: number = 0;
    height: number = 0;
  }
  type Shape = Variant<Point, Circle, Rectangle>;
  let shapes: Shape[] = [
    Variant.make2(new Circle()),
    Variant.make3(new Rectangle()),
  ]
  for (let shape of shapes) {
    switch (shape.index) {
      case 0:
        let point: Point = <Point>shape.value;
        console.log(`Point ${JSON.stringify(point)}`)
        break;
      case 1:
        let cicle: Circle = <Circle>shape.value;
        console.log(`Circle ${JSON.stringify(cicle)}`)
        break;
      case 2:
        let rectangle: Rectangle = <Rectangle>shape.value;
        console.log(`Point ${JSON.stringify(rectangle)}`)
        break;
      default:
        throw new Error()
    }
  }
  ```

### 访问者模式

#### 角度1: 面向对象

  实现一个文档，该文档包含3个项目：段落、图片和表格，在屏幕中渲染或者为有视障人群阅读的功能;
  可见 `IDocItem` 存储了描述文档的信息，不应该负责其他工作，如 渲染和阅读，如果添加新功能，就需要更新 `IDocItem`接口以及所有实现类：`Paragraph`，`Picture`，`Table`，来实现这个新功能。

  1. 不使用访问者模式

  ```ts
  // 不使用访问者模式
  class Renderer {/* Rendering methods */}
  class ScreenReader {/* Screen reading methods */}
  interface IDocItem {
    render(renderer: renderer): void;
    read(ScreenReader: ScreenReader): void;
  }
  // 实现段落
  class Paragraph implements IDocItem {
    // Paragraph members omitted
    render(renderer: renderer) {
      // draw
    }
    read(ScreenReader: ScreenReader) {
      // read
    }
  }
  // 图片
  class Picture implements IDocItem {
    // Picture members omitted
    render(renderer: renderer) {
      // draw
    }
    read(ScreenReader: ScreenReader) {
      // read
    }
  }
  // 表格
  class Table implements IDocItem {
    // Table members omitted
    render(renderer: renderer) {
      // draw
    }
    read(ScreenReader: ScreenReader) {
      // read
    }
  }
  let doc: IDocItem = [new Paragraph(), new Table()]
  let renderer: Renderer = new Renderer()
  for (let v of doc) {
    v.render(renderer)
  }
  ```

  2. 使用访问者模式: 在一个对象结构的元素上执行的操作，这种模式允许在定义新操作时，不改变其操作元素的类

  > 使用 `双分派` 机制，让文档接受访问者，然后把自己传递给访问者来实现任务

  ```ts
  interface IVistor {
    visitParagraph(paragraph: Paragraph): void;
    visitPicture(picture: Picture): void;
    visitTable(table: Table): void;
  }
  
  class Randerer implements IVistor {
    // Randerer methods ommited
    visitParagraph(paragraph: Paragraph) {/* do something */};
    visitPicture(picture: Picture){/* do something */};
    visitTable(table: Table) {/* do something */};
  }
  class ScreenReader implements IVistor {
    // Randerer methods ommited
    visitParagraph(paragraph: Paragraph) {/* do something */};
    visitPicture(picture: Picture) {/* do something */};
    visitTable(table: Table) {/* do something */};
  }
  interface IDocItem {
    accept(visitor: IVistor, printer: Printer<>): void;
  }
  // 实现段落
  class Paragraph implements IDocItem {
    // Paragraph members omitted
    accept(visitor: IVistor) {
      visitor.visitParagraph(this)
    }
  }
  // 图片
  class Picture implements IDocItem {
    // Picture members omitted
    accept(visitor: IVistor) {
      visitor.visitPicture(this)
    }
  }
  // 表格
  class Table implements IDocItem {
    // Table members omitted
    accept(visitor: IVistor) {
      visitor.visitTable(this)
    }
  }
  let doc: IDocItem = [new Paragraph(), new Table()]
  let renderer: IVistor = new Renderer()
  for (let v of doc) {
    v.accept(renderer)
  }
  ```

#### 角度2: 泛型标签联合类型

  1. 访问变体

  > 使用变体访问者进行处理: 将域对象与访问者完全分离开，甚至不需要在存储文档类内实现 `accept` 方法

  ```ts
  // 变体访问者
  function visit<T1, T2, T3>(
    variant: Variant<T1, T2, T3>,
    func1: (value: T1) => void;
    func2: (value: T2) => void;
    func3: (value: T3) => void;
  ): void {
    switch (variant.index) {
      case 0:
        func1(<T1>variant.value);
        break;
      case 1:
        func2(<T2>variant.value);
        break;
      case 2:
        func3(<T3>variant.value);
        break;
      default:
        throw new Error();
    }
  }
  // 使用变体访问者进行处理
  class Randerer {
    // Randerer methods ommited
    visitParagraph(paragraph: Paragraph) {/* do something */};
    visitPicture(picture: Picture) {/* do something */};
    visitTable(table: Table) {/* do something */};
  }
  class ScreenReader {
    // Randerer methods ommited
    visitParagraph(paragraph: Paragraph) {/* do something */};
    visitPicture(picture: Picture) {/* do something */};
    visitTable(table: Table) {/* do something */};
  }
  // 文档不再需要一个公共接口
  // 实现段落
  class Paragraph {
    // Paragraph members omitted
  }
  // 图片
  class Picture {
    // Picture members omitted
  }
  // 表格
  class Table {
    // Table members omitted
  }
  // 将文档存储到变体中
  let doc: Variant<Paragraph, Picture, Table>[] = [
    Variant.make(new Paragraph(), 0),
    Variant.make(new Table(), 2),
  ]
  let renderer = new Renderer()
  for (let v of doc) {
    visit(
      v,
      (paragraph: Paragraph) => renderer.visitParagraph(paragraph),
      (picture: Picture) => renderer.visitPicture(picture)
      (table: Table) => renderer.visitTable(table)
    )
  }
  ```

  2. 习题：实现 `visit()` 在给定 `Variant<T1, T2, T3>` 返回 `Variant<U1, U2, U3>`

  ```ts
  function visit<T1, T2, T3, U1, U2, U3> (
  variant: Variant<T1, T2, T3>,
  func1: (value: T1) => U1,
  func2: (value: T2) => U2,
  func3: (value: T3) => U3
  ): Variant<U1, U2, U3> {
    switch (variant.index) {
      case 0:
        return Variant.make1(func1(<T1>variant.value));
      case 1:
        return Variant.make2(func2(<T2>variant.value));
      case 2:
        return Variant.make3(func3(<T3>variant.value));
      default:
        throw new Error();
    }
  }
  ```

  ### 代数数据类型 AlgeBraic Data Type, ADT

  > 在类型系统中组合类型的方式

  #### 乘积类型 => 复合类型（元组 & 记录）

  ```ts
  enum A {
    a1,
    a2
  }
  enum B {
    b1,
    b2
  }
  // 2 X 2 = 4
  type AB = [A, B]; // [A.a1, B.b1], [A.a1, B.b2], [A.a2, B.b1], [A.a2, B.b2]
  type AB1 = Record<A, B>; // {0: B; 1: B}
  ```

  #### 和类型 -> 多选一类型 & 变体类型

  > TS 提供的 `|` 操作， 前面的 `Optional`, `Either`, `Variant`

  ```ts
  type AB = A | B // A.a1 | A.a2 | B.b1 | B.b2, 2 + 2 = 4种
  ```

## 第四章 - 类型安全

> 如果2种类型具有相同形状，那么TS会认为类型兼容，如果使用 `unique symbol` ，就不能将一种隐式类型解释为另一种类型

### 避免基本类型偏执来防止错误解释

> 基本类型偏执反模式: 指依赖基本类型来表示所有的内容，因为类型系统没有显示捕捉到值的意义，所以很容易出错

1. 不兼容组件

```ts
// lbfs(磅力秒)，Ns(牛顿秒)
function trajectoryCorrection(momenttum: number) {
  if (momenttum < 2 /* Ns */) {
    disintegrate()
  }
}
function provideMomenttum() {
  trajectoryCorrection(1.5 /* lbfs */)
}
```

2. 增加 磅力秒 & 牛顿秒 类型

```ts
declare const NsType: unique symbol;
class Ns {
  readonly value: number;
  [NsType]: void;
  constructor(value: number) {
    this.value = value
  }
}
declare const LbfsType: unique symbol;
class Lbfs {
  readonly value: number;
  [LbfsType]: void;
  constructor(value: number) {
    this.value = value
  }
}
// 转化单位：从 lbfs => Ns
function lbfsToNs(lbfs: Lbfs): Ns {
  return new Ns(lbfs.value * 4.448222)
}
function trajectoryCorrection(momenttum: Ns) {
  if (momenttum < new Ns(2) /* Ns */) {
    disintegrate()
  }
}
function provideMomenttum() {
  trajectoryCorrection(lbfsToNs(new Lbfs(1.5)))
}
```

### 实施约束

#### 使用构造函数实施约束

```ts
declare const celsiusType: unique symbol;
class Celsius {
  readonly value: number;
  [celsiusType]: void;
  constructor(value: number) {
    if (value < -273.15) throw new Error(); // 试图创建一个无效的温度，构造函数会抛出异常
    // 或者：强制修改无效值
    // if (value < -273.15) value = -273.15;
    this.value = value
  }
}
```

#### 使用工厂实施约束

不想抛出异常，而是希望返回 `undefined` 或者其他某个不是温度而是表示失败的值以创建一个有效的实例时，使用工厂函数；如果构造函数和验证对象的逻辑很复杂，则应该在外部实现逻辑更加合理，构造函数不应该做太复杂的工作，而应该只做初始化对象。

```ts
declare const celsiusType: unique symbol;
class Celsius {
  readonly value: number;
  [celsiusType]: void;
  // private 声明构造函数，只有工厂函数才能调用
  private constructor(value: number) {
    this.value = value
  }
  static makeCelsius(value: number): Celsius | undefined {
    if (value < -273.15) return undefined;
    return new Celsius(value)
  }
}
```

#### 习题：实现一个 `Percentage` 类型来表示 0～100之间的值，小于 0 置0，大于 100 置 100

```ts
declare const percentageType: unique symbol;
class Percentage {
  readonly value: number;
  [percentageType]: void;
  // private 声明构造函数，只有工厂函数才能调用
  private constructor(value: number) {
    this.value = value
  }
  static makePercentage(value: number): Percentage | undefined {
    if (value < 0) value = 0;
    if (value > 100) value = 100;
    return new Percentage(value)
  }
}
```



### 添加类型信息

#### 类型转换

```ts
// 类型强制转换导致运行时错误
class Bike {
  ride(): void {}
}
class SportsCar {
  drive(): void {}
}
let myBike: Bike = new Bike()
Bike.ride()
// 先把 myBike 转成 unknown 再转成 SportsCar
let myPretendSportsCar: SportsCar = <SportsCar><unknown>myBike;
// 或者写成：
// let myPretendSportsCar: SportsCar = myBike as unknown as SportsCar;
myPretendSportsCar.drive() // 运行时错误
```

#### 常见类型转换

1. 向上转换 - 这个是安全的：将派生类型的对象解释为基类型，可自动*隐式转换*
2. 向下转换 - 这个不是安全的：将基类型转化成派生类型，不会自动完成向下转换，因为编译器无法自动确定某个值是多个派生类的哪一个。
3. 拓宽转换：一种常见的*隐式转换*-从固定位数的整数类型（8位无符号整数）转换为另一个位数更多的整数类型（16位无符号整数）。
4. 缩窄转换：位数更多的整数转换成位数更少的无符号整数，只能用小类型可以表示的值。***必须显示指定这种转换***

### 隐藏和恢复类型信息

1. 同构集合：包含相同类型项的集合
2. 异构集合：包含不同类型项的集合

#### 异构集合

```ts
// 层次结构
interface IDocItem {
  /* 段落，图片和表格 公共部分 */
}
class Paragraph implements IDocItem {
    /* 段落独有部分 */
}
class Picture implements IDocItem {
    /* 图片独有部分 */
}
class Table implements IDocItem {
    /* 表格独有部分 */
}
class MyDoc {
  items: IDocItem[]
}
// 和类型
class Paragraph {
    /* 段落 */
}
class Picture {
    /* 图片 */
}
class Table {
    /* 表格 */
}
class MyDoc1 {
  items: (Paragraph | Picture | Table)[]
}
// unknown 类型
class MyDoc2 {
  items: unknown[];
}
```

#### 异构类型优缺点

| 类型 |优点 | 缺点 |
| ----- | ----- | ----- |
| 层次结构 | 能够轻松地使用基础类型的任何属性和方法，不需要转换 | 集合中的类型必须通过基础类型或者实现的接口彼此相关 |
| 和类型 | 不要求类型彼此相关 | 如果没有 `Variant` 的 `visit()`, 就需要转换为实际的类型来驱动 |
| unknown类型 | 可以存储任何内容 | 需要跟踪实际类型并转换为对应的类型才能使用 |


#### 序列化

> JSON.stringify() & JSON.parse()

1. 序列化及跟踪类型

```ts
class Cat {
  meow() { /* how about bark ? */ }
}
class Dog {
  bark() { /* how about meow ? */ }
}
function serializeCat(cat: Cat): string {
  return 'c' + JSON.stringify(cat)
}
function serializeDog(dog: Dog): string {
  return 'd' + JSON.stringify(dog)
}
function tryDeserializeCat(from: string): Cat | undefined {
  if (from[0] !== 'c') return undefined
  return <Cat>Object.assign(new Cat(), JSON.parse(from .substr(1)))
}
```

2. 带有跟踪类型的反序列化

```ts
let catString: string = serializeCat(new Cat())
let dogString: string = serializeDog(new Dog())
let maybeCat: Cat | undefined = tryDeserializeCat(catString)
if (maybeCat) {
  let cat: Cat = <Cat>maybeCat
  cat.meow()
}
maybeCat = tryDeserializeCat(dogString) // undefined
```

## 第五章 - 函数类型

### 一个简单的策略模式

> 一等函数：当语言看待函数的方式与看待其他任何值相同时，则该语言支持一等函数。

```ts
class Car {
  // Represents a car
}
type WashingStrategy = (car: Car) => void;
function standardWash(car: Car): void {
  // perform standard wash
}
function premiumWash(car: Car): void {
  // perform premium wash
}
class CarWash {
  service(car: Car, premium: boolean) {
    let washStrategy: WashingStrategy
    if (premium) {
      washStrategy = premiumWash
    } else {
      washStrategy = standardWash
    }
    washStrategy(car)
  }
}
```

### 不使用 switch 语句的状态机

1. 将状态集合定义为一个枚举，跟踪当前的状态

> 缺点：状态和相应的逻辑没有联系在一起，同时还需要单独维护枚举和case，会有值和逻辑不同步的风险

```ts
// 状态机用一个枚举表示
enum TextProcessingMode {
  Text,
  Marker,
  Code
}
class TextProcessor {
  private mode: TextProcessingMode = TextProcessingMode.Text;
  private result: string[] = [];
  private codeSample: string[] = [];
  processText(lines: string[]): string[] {
    this.result = []
    this.mode = TextProcessingMode.Text
    // 处理文本文档意味着处理每行文本，并返回结果字符串数组
    for (let line of lines) {
      this.processLine(line)
    }
    return this.result
  }
  private processLine(line: string): void {
    switch(this.mode) {
      case TextProcessingMode.Text:
        this.processTextLine(line);
        break;
      case TextProcessingMode.Marker:
        this.processMarkerLine(line);
        break;
      case TextProcessingMode.Code:
        this.processCodeLine(line)
        break;
    }
  }
  // 处理一行文本，如果本行以 `<!--`开头，则加载代码示例，并转移到下一个状态
  private processTextLine(line: string): void {
    this.result.push(line)
    if (line.startsWith('<!--')) {
      this.loadCodeSample(line);
      this.mode = TextProcessingMode.Marker
    }
  }
  private processMarkerLine(line: string): void {
    this.result.push(line)
    if (line.startsWith('```ts')) {
      this.result = this.result.concat(this.codeSample)
      this.mode = TextProcessingMode.code
    }
  }
  private processCodeLine(line: string): void {
    if (line.startsWith('```')) {
      this.result.push(line)
      this.mode = TextProcessingMode.Text
    }
  }
  private loadCodeSample(line: string) {
    // load sample based on marker, store in this codeSample
  }
}
```

2. 另一种完全由函数实现的状态机，不需要在内部跟踪状态也不需要枚举

> 缺点：1. 需要为每种状态关联更多信息；2. 可能更需要显示声明的状态和转移

```ts
class TextProcessor {
  private result: string[] = []
  private processLine: (line: string) => void = this.processTextLine
  private codeSample: string[] = []
  processText(lines: string[]): string[] {
    this.result = []
    this.processLine = this.processTextLine
    for (let line of lines) {
      this.processLine(line)
    }
    return this.result
  }
  private processTextLine(line: string): void {
    this.result.push(line)
    if(line.startsWith('<!--')) {
      this.loadCodeSample(line)
      this.processLine = this.processMarkerLine; // 通过把 this.processLine更新为合适的方法来实现状态转移：一种策略模式？
    }
  }
  private processMarkerLine(line: string): void {
    this.result.push(line)
    if(line.startsWith('```ts')) {
      this.processLine = this.processCodeLine; // 通过把 this.processLine更新为合适的方法来实现状态转移：一种策略模式？
    }
  }
  private processCodeLine(line: string): void {
    if(line.startsWith('```ts')) {
    this.result.push(line)
      this.processLine = this.processTextLine; // 通过把 this.processLine更新为合适的方法来实现状态转移：一种策略模式？
    }
  }
  private loadCodeSample(line: string) {
    // load sample based on marker, store in this codeSample
  }
}
```

3. 使用和类型的状态机

> 缺点：代码量较多

#### 习题

```ts
// 1. 一个具有open, close2种状态简单连接为一个状态机，使用 connect 打开，使用 disconnect关闭
enum State {
  open = 'OPEN',
  closed = 'CLOSED'
}
class Connection {
  private state: string = State.open
  private processState: () => void = this.connect
  private connect() {
    this.state = State.open
    this.processState = this.disconnect
  }
  private disconnect() {
    this.state = State.closed
    this.processState = this.connect
  }
}
// 2. 使用 process方法将前面的连接实现为一个函数状态机，process 函数扭转状态

declare function read(): string;
class Connection {
  private processState: () => void = this.disconnect
  public process() {
    this.processState()
  }
  private connect() {
    const value: string = read()
    if (!value) {
      this.processState = this.disconnect
    } else {
      console.log(value)
    }
    this.processState = this.disconnect
  }
  private disconnect() {
    this.processState = this.connect
  }
}
```

### 使用延迟值避免高开销的计算

1. 立即创建

```ts
class Bike {}
class Car {} // 假设创建 Car 开销较大
function chooseMyRide(bike: Bike, car: Car) Car | Bike {
  if (isItRaining()) {
    return car
  } else {
    return bike
  }
}
// 调用时就创建Car
chososeMyRide(new Bike(), new Car())
```

2. 延迟生成 car

```ts
class Bike {}
class Car {} // 假设创建 Car 开销较大
// 参数改为一个返回 Car 的函数
function chooseMyRide(bike: Bike, car: () => Car) Car | Bike {
  if (isItRaining()) {
    return car()
  } else {
    return bike
  }
}
function makeCar(): Car {
  return new Car()
}
// 调用时就创建Car
chososeMyRide(new Bike(), makeCar)
```

#### lambda 匿名函数

```ts
// 使用匿名函数
class Bike {}
class Car {} // 假设创建 Car 开销较大
// 参数改为一个返回 Car 的函数
function chooseMyRide(bike: Bike, car: () => Car) Car | Bike {
  if (isItRaining()) {
    return car()
  } else {
    return bike
  }
}
// 调用时就创建Car
chososeMyRide(new Bike(), () => new Car())
```

### 使用 map,filter,reduce

#### map()

对每个值调用一个函数，返回结果集合

```ts
// 自制 map()
function map<T, U>(items: T[], func: (item: T) => U) : U[] {
  let result: U[] = []
  for (const item of items) {
    result.push(func(item))
  }
  return result
}
```

#### filter()

丢弃某些元素，接受一个实参并返回一个 `boolean` 的函数也称为 `谓词`

```ts
// 自制 filter()
function filter<T>(items: T[], pred: (item: T) => boolean): T[] {
  let result: T[] = []
  for (const item of items) {
    if (pred(item)) {
      result.push(item)
    }
  }
  return result
}
```

#### reduce()

将所有集合项合并为一个值

```ts
// 自制 reduce()
function reduce<T>(items: T[], op: (x: T, y: T) => T, init: T):T {
  let result: T = init
  for (const item of items) {
    result = op(result, item)
  }
  return result
}
```

#### 幺半群

抽象代数处理集合以及集合上的操作。 如果类型 `T` 的一个操作接受 2个 `T` 作为实参，并返回另一个 `T`， 即 `(T, T) => T`，则可以把该操作解释为 `T` 值集合上的一个操作。例如：`number 集合 +`，即 `(x, y) => x + y`, 就构成了一个代数结构。
相关性：表明对元素序列应用操作的顺序并不重要，因为最终结果都是相同的。

> 如果集合 `T` 上的操作有一个操作元，并且具有相关性，那么得到的代数结构称为 幺半群

#### 习题

```ts
// 实现一个 first 函数，返回数组的第一个满足 谓词的元素，如果数组为空，则返回undefined, 功能与 find 相似
function first<T>(items: T[], pred: (item: T) => boolean): T | undefined {
  if (!items || items.length) return undefined
  for (const item of items) {
    if (pred(item)) {
      return item
    }
  }
  return undefined
}
// all<T>(items: T[], pred: (item: T) => boolean): boolean
function  all<T>(items: T[], pred: (item: T) => boolean): boolean {
  if (!items || items.length) return undefined
  for (const item of items) {
    if (!pred(item)) {
      return false
    }
  }
  return true
}
```

## 第六章 - 函数类型的高级应用

### 一个简单的装饰器模式

可扩展对象的 `行为`，而不必修改对象的类

```ts
class Widget {}
// 接口
interface IWidgetFactory {
  makeWidget(): Widget;
}
// 实现接口
// 如果想获取多个实例，可以直接使用 WidgetFactory
class WidgetFactory implements IWidgetFactory {
  public makeWidget(): Widget {
    return new Widget() // WidgetFactory 只是创建一个新的 Widget
  }
}
// 装饰器，负责单例行为
// 如果想获取单个实例，使用SingletonDecorator
class SingletonDecorator implements IWidgetFactory {
  private factory: IWidgetFactory
  private instance: Widget | undefined = undefined
  constructor(factory: IWidgetFactory) {
    this.factory = factory
  }
  public makeWidget(): Widget {
    if (!this.instance) {
      this.instance = this.factory.makeWidget() // makeWidget() 实现了单例逻辑，并确保只会创建一个 Widget
    }
  }
}
```

#### 函数装饰器

```ts
class Widget {}
type WidgetFactory = () => Widget;
function makeWidget(): Widget {
  return new Widget()
}
function singletonDecorator(factory: WidgetFactory): WidgetFactory {
  let instance: Widget | undefined = undefined;
  // singletonDecorator() 返回一个 lambda, 该 lambda 实现了单例行为，并使用给定工厂创建一个 Widget
  return (): Widget => {
    if (!instance ) {
      instance = factory()
    }
    return instance
  }
}
function use10Widgets(factory: WidgetFactory) {
  for(let i = 0;i<10;i++) {
    let widget = factory()
  }
}
use10Widgets(singletonDecorator(makeWidget))
```

#### 闭包

> `lambda捕获`：`singletonDecorator()` 尽管返回一个 `lambda`，但该  `lambda`引用了  `factory` 实参和 `instance` 变量，而该对象是 `singletonDecorator()`函数的局部变量

比较对象和闭包：
对象：对象代表一组方法的某个状态；闭包：代表捕获到某个状态的函数

#### 习题

```ts
class Widget {}
type WidgetFactory = () => Widget;
function factory() {
  return new Widget()
}
function loggingDecorator(factory: WidgetFactory): WidgetFactory {
  // 返回一个 lambda 延迟调用
  return () => {
    console.log('Widget created')
    return factory()
  }
}
```

### 实现一个计数器

#### 一个面向对象的计数器

```ts
class Counter {
  private n: number = 1; // 禁止被外部访问修改
  next(): number {
    return this.n++
  }
}
```

#### 使用闭包实现的计数器

```ts
type Counter = () => number;
function makeCounter(): Counter {
  let n: number = 1
  return () => n++
}
```

#### 可恢复的计数器

> 跟踪自己状态的函数，调用时，不会从头开始运行，而是从上一次返回时的状态恢复执行；在 TS 中不使用 `return` 关键字来退出函数，而是使用 `yield`: 2个限制：1. 必须将函数声明为一个生成器； 2. 并且其返回类型必须是可叠戴的迭代器；

```ts
function* counter(): IterableIterator<number> {
  let n: number = 1;
  while(true) {
    yield n++
  }
}
let counter1: IterableIterator<number> = counter()
counter1.next()
```

#### 习题

```ts
// 使用 闭包创建函数 返回斐波那契数列中的下一个数字
function fib1(): () => number {
  let a: number = 0;
  let b: number = 0;
  return () => {
    let next: number = a
    a = b
    b = b + next
    return next
  }
}
// 使用生成器创建函数 返回斐波那契数列中的下一个数字
function* fib2(): IterableIterator<number> {
  let a: number = 0;
  let b: number = 0;
  while(true) {
    let next: number = a
    a = b
    b = b + next
    return next
  }
}
```

### 异步执行的运行时间长的操作

#### 同步执行

```ts
function greet(): void {
  const readlineSync = require('readline-sync');
  let name: string = readlineSync.question('what is your name')
  console.log(`Hi ${name}`)
}
function weather(): void {
  const open = require('open')
  open('heeps://weather.com/')
}
greet(); // 先调用 greet
weather(); // 等待 greet 执行完毕
```

#### 异步执行：回调

```ts
function greet(): void {
  // 使用异步 readline
  const readline = require('readline')
  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
  })
  rl.question('what is your name', (name: string) => {
    console.log(`Hi ${name}`)
    rl.close()
  })
}
function weather(): void {
  const open = require('open')
  open('heeps://weather.com/')
}
greet(); //  rl.question 会直接跳过先执行 剩下的 同步函数 weather，等回调通知
weather(); // 不必等待 greet 执行完毕
```

#### 异步执行模型

1. 在事件循环中倒数

> 优点：不需要同步；缺点：在 I/O 操作等待数据时排队效果很好，但是 CPU密集操作（复杂计算）仍然会造成阻塞。因为这种操作没有等待数据，而是需要CPU周期，对于这种任务线程的效果更好。

```ts
type AsyncFunction = () => void;
let queue: AsyncFunction[] = []; // 队列将是一个函数数组
function countDown(counterId: string, fro: number): void {
  console.log(`${counterId}: ${from}`)
  if (from > 0) {
    queue.push(() => countDown(counterId, from - 1))
  }
}
queue.push(() => countDown('counter1', 4)) // 启动倒数1
queue.push(() => countDown('counter2', 4)) // 启动倒数2
while(queue.length > 0) {
  let func: AsyncFunction = <AsyncFunction>queue.shift()
  func()
}
// 输出
// counter1: 4
// counter2: 2
// counter1: 3
// counter2: 1
// counter1: 2
// counter2: 0
// counter1: 1
// counter1: 0

```

### 简单异步代码

链式回调很难阅读，嵌套太深

#### 链接 promise

> Promise: 将来某个时刻可用值的一个代理。Continuation: 在promise的结果可用后调用的函数称为 continuation;

```ts
declare function getUserName(): Promise<string>;
declare function getUserBirthday(name: string): Promise<Date>;
declare function getUserEmail(birthday: Date): Promise<string>;
getUserName().then((name: string) => {
  console.log(`hi ${name}!`)
  return getUserBirthday(name)
}).then((birthday: Date) => {
  const today: Date = new Date()
  if (birthday.getMonth() === today.getMonth() && birthday.getDay() === today.getDay()) {
    console.log('Happy birthday!')
    return getUserEmail(birthday)
  }
}).then((emain: string) => {
  // 处理邮件
})
```

#### async/await

> `async` 不会改变函数类型，只会将一个函数标记成异步函数，并允许在其中调用 `await`

```ts
// 使用async/await
async function getUserData(): Promise<void> {
  let name: string = await getUserName();
  console.log(`hi ${name}!`);
  let birthday: Date = await getUserBirthday(name)
  const today: Date = new Date()
  if (birthday.getMonth() === today.getMonth() && birthday.getDay() === today.getDay()) {
    console.log('Happy birthday!')
    let email: string = getUserEmail(birthday)
  }
}

try {
  getUserData()
} catch {
  // 异常将从 await 调用抛出，在 try/catch 语句中捕获错误
}
```




