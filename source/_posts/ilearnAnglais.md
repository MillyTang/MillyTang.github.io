---
title: 编程与类型系统 - 类型部分&补充知识
date: 2022-01-11 15:40:07
tags: typeScript, javaScript, 编程与类型系统
---
## 第七章 - Subtyping 儿类型关系

类型之间的关系-儿类型关系：1. 在TS中消除类型的歧义；2.安全的反序列化；3.错误情况的值；4.和类型、集合以及函数的类型兼容。

### 在TS中区分相似的类型

回顾磅力秒那个例子，使用了 `unique symbol` 来区分两个相似类型，确保类型检查器不会把`一个类型的值`解释为`另一个类型的值` - 通过`模拟名义Subtype(亚类型、儿类型)`👉见下文

```ts 省略NsType和LbsType的unique symbol唯一属性
declare const NsType: unique symbol;
class Ns {
  value: number;
  // [NsType]: void;
  constructor(value: number) {
    this.value = value
  }
}
declare const LbfsType: unique symbol;
class Lbfs {
  value: number;
  // [LbfsType]: void;
  constructor(value: number) {
    this.value = value
  }
}
// 将2个 NsType & LbsType 省略，将2个对象互传，编辑器不会报错
// 演示过程
function acceptNs(momentum: Ns) {
  console.log(`Momentum: ${Momentum.value}Ns`)
}
acceptNs(new Lbfs(10)) // 实际调用传错参数，不会报错，控制台打印：Momentum: 10
```

> Subtyping：如果在期望类型 **T** 的实例的任何地方，都可以安全地使用类型 **S** 的实例，那么称类型 **S** 是类型 **T** 的Subtype(亚类型、儿类型)。- 里氏(Liskov substitution principle)替换原则中的一种非正式定义

#### Subtyping 的构成

1. `Normal Subtyping`: 显示声明一个类型是另一个类型的Subtype(亚类型、儿类型)，则二者构成Subtyping 关系
2. `Structural Subtyping`: 如果一个类型具有另一个类型的所有成员，并且可能还有其他成员，那么前者是后者的Subtype(亚类型、儿类型) - ***TS使用Structural Subtyping***

#### Structural Subtyping 和 Normal Subtyping 的优缺点

1. Normal Subtyping: 在类型间建立关系，即使类型是外部类型，不在控制范围内；
2. Structural Subtyping: 默认结构相似不会混淆参数类型

#### 在 TS 中模拟 Normal Subtyping

在TS中，`unique symbol` 生成了一个在所有代码中保持唯一的`名称`。用户声明的名称绝对不会匹配生成的名称，使用该名称创建一个属性，需要给这个属性定义一个类型，但是我们不关心它的实际值，只是为了区分类型，所以选择了最适合的单元类型`void`。所以当要创建 `NS`和`Lbfs`的Subtype(亚类型、儿类型)，只能显示地继承。

### Subtype(亚类型、儿类型)的极端情况

1. 把任何类型赋值给它的类型(Assigning anything to)：`any`, `unknown`
2. 给任何东西赋值类型(assigning to anything)：`any`- 从而绕过类型检查

```ts 反序列化any
// JSON.parse()反序列化返回类型是 any
class User {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}
// 只是封装了 JSON.parse() 并返回了 any 类型的一个值
function deserialize(input: string): any {
  return JSON.parse(input)
}
function greet(user: User): void {
  console.log(`hi ${user.name}`)
}
// 反序列化一个有效的 User JSON
greet(deserialize('{"name": "Alice"}'))
// 也可以反序列化一个不是 User 对象的对象
greet('{}') // 输出hi undefined，因为 `any`会绕过类型检查
```

#### User的运行时检查 - isUser()

增加一个属性类型检查函数可以规避类型问题，但是仍然存在不足：没有强制调用 **isUser**，所以依然会存在忘记调用而出现的 `undefined` 的情况

```ts 增加isUser()函数调用前检查类型
class User {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}
function deserialize(input: string): any {
  return JSON.parse(input)
}
function greet(user: User): void {
  console.log(`hi ${user.name}`)
}
// 检查给定实参是否是 User 类型，我们认为，具有 string 类型的 name 属性的变量是 User 类型
function isUser(user: any): user isUser {
  if (!user) {
    return false
  }
  return typeof user.name === 'string'
}
let user: any = deserialize('{"name": 'Alice'}')
// 在使用之前，先检查 user 是否具有 string类型的name属性
if (isUser(user)) {
  greet(user)
}
user = undefined
// 不会执行
if (isUser(user)) {
  greet(user)
}
```

#### 顶层类型(Top Type) - unknown

> 如果能够把任何值赋给一个类型，则称为顶层类型，**和类型**`Object | null | undefined` === `unknown`

一旦忘记调用 `isUser`，代码在编译过程就会报错 `Argument of type 'unknown' is not assignable to parameter of type 'User'`

> unknown 和 any 的区别：unknown 需要存在可转换的类型，而 any 不需要直接可用。

```ts 使用 unknown 的强类型
class User {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}
// 返回 unknown
function deserialize(input: string): unknown {
  return JSON.parse(input)
}
function greet(user: User): void {
  console.log(`hi ${user.name}`)
}
// 检查给定实参是否是 User 类型，我们认为，具有 string 类型的 name 属性的变量是 User 类型
// 实参保持为 any
function isUser(user: any): user isUser {
  if (!user) {
    return false
  }
  return typeof user.name === 'string'
}
// 将变量声明为 unknown
// 这里一旦有值就会有类型从 any 转换为 unknown，向上转化类型是安全的
let user: unknown = deserialize('{"name": 'Alice'}')
// 在使用之前，先检查 user 是否具有 string类型的name属性
if (isUser(user)) {
  greet(user)
}
user = deserialize('null')
// 不会执行
if (isUser(user)) {
  greet(user)
}
```

#### 错误情况的值

一个相反的问题：一种类型可以替代其他类型使用。

```ts TurnDirection到角度的转换
enum TurnDirection {
  Left,
  Right
}
funtion turnAngle(turn: Turndirection): number {
  switch(turn) {
    case TurnDirection.Left: return -90;
    case TurnDirection.Right: return 90;
    default: throw new Error('Unknown TurnDirection')
  }
}
```

```ts 报告错误
funtion fail(message: string): never {
  console.error(message);
  throw new Error(message)
}
funtion turnAngle(turn: Turndirection): number {
  switch(turn) {
    case TurnDirection.Left: return -90;
    case TurnDirection.Right: return 90;
    default: fail('Unknown TurnDirection')
  }
}
// 这段代码可以工作，但是还差一点
// 在严格模式下（使用了 --strict）编辑器会报错
// Function lacks ending return statement and return tyoe does not include "undefined"
// 因为编辑器在 `default` 分支中没有找到 return 语句，所以标记错误
```

```ts 使用fail()并返回一个虚拟的值
funtion turnAngle(turn: Turndirection): number {
  switch(turn) {
    case TurnDirection.Left: return -90;
    case TurnDirection.Right: return 90;
    default: {
      fail('Unknown TurnDirection')
      return -1; // 因为 fail 会抛出错误，所以永远不会返回这个虚拟值
    }
  }
}
```

如果后续改变了 fail 的返回，虚拟值就会直接被抛出，不太好，后面还会面临修改，需要另一种解决方案

```ts 使用fail()并返回其结果的turnAngle()
// 即使返回never也没问题：因为 never 是所有类型的Subtype(亚类型、儿类型)
funtion turnAngle(turn: Turndirection): number {
  switch(turn) {
    case TurnDirection.Left: return -90;
    case TurnDirection.Right: return 90;
    default: {
      return fail('Unknown TurnDirection');
    }
  }
}
```

如果后续需求更新了`fail`，使其在某些情况不再抛出错误，编译器将强制我们修改`fail`的返回类型，如果返回类型是`void`，这时试图将返回的值传递给 string 类型，就不会通过类型检查。

> 如果一个类型是其他类型的 Subtype(亚类型、儿类型)，则为底层类型，底层类型是所有类型的 Subtype(亚类型、儿类型) 则需要具有其他类型的成员。因为其他类型可以有无限个类型和成员，所以底层类型也必须有无限个成员，然而这是不可能的。所以底层类型始终是一个空类型：不能为其创建实际值

### 允许的替换

#### Subtyping 与和类型（Sum Type）

```ts Trangle | Square 作为 Trangle | Square | Circle
declare const TriangleType: unique symbol;
class Trangle {
  [TriangleType]: void;
  // Trangle members
}
declare const SquareType: unique symbol;
class Square {
  [SquareType]: void;
  // Square members
}
declare const CircleType: unique symbol;
class Circle {
  [CircleType]: void;
  // Circle members
}
// 这里忽略函数的具体实现
declare function makeShape(): Trangle | Square;
declare function draw(shape: Trangle | Square | Circle): void;
// draw 接受 Trangle | Square | Circle 这3种类型之一
// 可以运行，makeShape提供的类型少了一个但是是draw可以接受的范围内
draw(makeShape())
```

```ts Trangle | Square | Circle 作为 Trangle | Square
// 这里忽略函数的具体实现
declare function makeShape(): Trangle | Square | Circle;
declare function draw(shape: Trangle | Square):void;
draw(makeShape()) // 无法通过编译，draw 处理不了 Circle
```

当使用**继承**时，得到的 `Subtype(亚类型、儿类型)` 比 `Parent type(双亲类型)` 的属性更多。对于**Sum Type**，则是相反的，`Parent type(双亲类型)` 比 `Subtype(亚类型、儿类型)` 的类型更多。

> Sum Type: `Trangle | Square` 是 `Trangle | Square | Circle` 的 Subtype(亚类型、儿类型)

#### Subtyping and collections(集合)

```ts Triangle[]作为Shape[]
class Shape {/* Shape members */}
declare const TriangleType: unique symbol;
// Triangle 是 Shape 的 Subtype(亚类型、儿类型)
class Triangle extends Shape {
  [TriangleType]: void;
  // Triangle members
}
declare function makeTriangles(): Triangle[];
declare function draw(shapes: Shape[]): void;
draw(makeTriangles())
```

> 结论：数组会保留它们存储的底层类型的Subtyping关系，反过来则不成立。

`Triangle` 作为 `Shape` 的 `Subtype(亚类型、儿类型)`，则 `Triangle[]` 是 `Shape[]`的 `Subtype(亚类型、儿类型)`，如果 `Triangle` 可以用作 `Shape`，那么 `Triangle[]` 也可以用作 `Shape[]`

##### `LinkedList<Triangle>作为LinkedList<Shape>`

```ts
// 一个泛型链表集合
class LinkedList<T> {
  value: T;
  next: LinkedList<T> | undefined = undefined;
  constructor(value: T) {
    this.value = value
  }
  append(value: T): linkedList<T> {
    this.next = new LinkedList(value);
    return this.next
  }
}
declare function makeTriangles(): linkedList<Triangle>;
declare function draw(shapes: linkedList<Shape>): void;
draw(makeTriangles()); // 代码能够编译
```

> 协变(Covariance)：如果一个类型保留其底层类型的 `subtyping` 关系，就成该类型具有协变性。数组具有协变性。因为它保留了 `subtyping` 关系: `Triangle` 是 `Shape` 的 `Subtype(亚类型、儿类型)`，所以 `Triangle[]` 是 `Shape[]` 的 `Subtype(亚类型、儿类型)`

#### Subtyping 和函数的返回类型

##### () => Triangle 作为 () => Shape

```ts
declare function makeTriangle(): Triangle;
declare function makeShape(): Shape();
function useFactory(factory: () => Shape): Shape {
  return factory()
}
let shape1: Shape = useFactory(makeShape);
let shape2: Shape = useFactory(makeTriangle);
// Shape <---- Triangle // subtying 关系
// () => Shape <---- () => Triangle // subtying 关系
```

函数的返回类型具有协变性：如果 `Triangle` 是 `Shape` 的 Subtype(亚类型、儿类型)，则可以使用返回 `Triangle` 替代返回  `Shape`的函数。反过来则不成立。

##### () => Shape 作为 () => Triangle

```ts
declare function makeTriangle(): Triangle;
declare function makeShape(): Shape();
function useFactory(factory: () => Triangle): Triangle {
  return factory()
}
// 代码编译错误❌
let shape1: Shape = useFactory(makeShape);
let shape2: Shape = useFactory(makeTriangle);
```


#### Subtyping 和函数的实参类型

带参数的 `(argument: Shape) => void` 和 `(argument: Triangle) => void` 的关系是什么？

```ts 引入一个render()函数
declare function drawShape(shape: Shape): void;
declare function drawTriangle(triangle: Triangle): void;
function render(
  triangle: Triangle,
  drawFunc: ((argument: Triangle) => void); void,
  drawFunc(triangle);
)
```

> 逆变：如果一个类型颠倒了其底层类型的 subtying 关系，则称该类型具有逆变性。大部分语言中函数的实参是逆变的。

一个接受 Triangle 作为实参的函数, `Shape <---- Triangle` 箭头指向代表 subtying 关系, `(Shape) => void ----> (Triangle) => void`  箭头指向代表subtying 关系
可以被替换成接口 Shape 作为实参的函数，因为总是可以把 Triangle 传递给一个接受 Shape 作为实参的函数。函数之间的关系与其实参类型之间的关系相反。

> TS是例外，因为TS反过来也成立：传入期望接受 Subtype(亚类型、儿类型) 的函数，而不是期望接受 Parent type(双亲类型)，这是故意作出的设计决策，目的是方便实现常见的 JS 编程模式，不过，可能会导致运行时问题

```ts Shape和包含isRightAbgled()方法的Triangle
class Shape {
  // Shape members
}
declare const TriangleType: unique symbol;
class Triangle extends Shape {
  [TriangleType]: void;
  // 判断给定的实例是否描述了一个直角三角形
  isRightAbgled(): boolean {
    let result: boolean = false;
    // determine whether it is a right-angled triangle
    return result
  }
  // more triangle members
}
```

```ts 反转绘制示例:更新后的绘制和渲染函数
declare function drawShape(shape: Shape): void;
declare funtion drawTriangle(triangle: Triangle): void;
// render 期望收到 Shape 和一个接受 Shape 作为实参的函数
function render(
  shape: Shape,
  drawFunc: (argument: Shape) => void
): void {
  drawFunc(shape)
}
```

```ts 试图对Triangle的双亲类型调用isRightAbgled
funtion drawTriangle(triangle: Triangle): void {
  console.log(triangle.isRightAbgled())
}
function render(
  shape: Shape,
  drawFunc: (argument: Shape) => void
): void {
  drawFunc(shape)
}
// 使用 Shape 和 drawTriangle 来绘制
// drawTriangle 的实参变成了 Shape，将对 Shape 调用 isRightAbgled()
// 运行时isRightAbgled肯定会报错❌
// 这是实现TS有意作出的决策
render(new Shape(), drawTriangle)
```

在 TS 中，如果 Triangle 是 Shape 的 subtype，那么函数类型 `(argument: Shape) => void` and `(argument: Triangle) => void` 能够彼此替换，互为subtype，这种属性成为双变性（双向协变）。


## 第八章 - 面向对象变成的元素-应用subtype

> OOP: Object-Oriebted Programming, 面向对象编程。对象可以包含数据和代码，数据是对象的状态，代码是一个或多个方法，也叫「消息」。在面向对象系统中，通过使用其他对象的方法，对象之间可以“对话”或者发送消息。

OOP关键特征：
1. 封装：允许隐藏数据和方法
2. 继承：使用额外的数据和代码扩展一个类型

### 使用接口定义契约

 接口或契约： 描述了实现该接口的任何对象都理解的`一组消息`。消息是方法，包括名称，实参和返回类型。接口没有任何状态，相当于书面协议，规定了实现中将提供什么。

> 抽象类和接口的区别是什么？

1. 抽象类 `ConsoleLogger` 和 `ALogger` 之间的关系是所谓的 **是** 关系，即 `ConsoleLogger` 继承了 `ALogger`， 所以它是一个`ALogger`
2. `ILogger` 定义了一个契约，没有继承，没有创建 **是** 关系，接口可以扩展也可以合并。接口最终让消费者受益，而不是实现接口的类受益。针对接口编程可以降低系统中组件的耦合。


```ts 使用抽象类实现日志系统
// ALogger 是一个抽象类
abstract class ALogger {
  // log 是一个抽象方法，没有实现
  abstract log(line: string): void;
}
// 继承了抽象类，实现了 log方法
class ConsoleLogger extends ALogger {
  log(line: string): void {
    console.log(line)
  }
}
```

```ts 使用接口实现日志系统-这种场景中推荐使用接口
interface ILogger {
  log(line: sring): void;
}
class ConsoleLogger implements ILogger {
  log(line: string): void {
    console.log(line)
  }
}
```

```ts 扩展和合并接口
// 扩展
interface ILogger {
  log(line: sring): void;
}
interface IExtendedLogger extends ILogger {
  warn(line: sring)L void;
  error(line: sring)L void;
}
// 合并
interface ISpeaker {
  playSound(): void;
}
interface IVolumeControl {
  up(): void;
  down(): void;
}
interface ISpeakerWithIVolumeControl extends ISpeaker, IVolumeControl {
  // 合并为一个
}
```

#### 练习

实现一个 `IterableIterator<T>`接口` 

```ts
interface Iterable<T> {
  [Symbol.iterator](): Iterator<T>
}
interface Iterator<T> {
  next(): IteratorResult<T>
}
interface IterableIterator<T> extends Iterable, Iterator<T> {

}
```

### 继承数据和行为

#### 「是一个」经验准则

```ts 不好的继承
class Point {
  x: number;
  y: number;
  constructor(x: number, y: number) {
    this.x = x
    this.y = y
  }
}
class Circle extends Point {
  radius: number;
  constructor(x: number, y: number, radius: number) {
    super(x, y)
    this.radius = radius
  }
}
```

继承和 **是一个** 关系： 继承会在 儿类型和双亲类型之间建立一个 **是一个**，如果基类是 Shape，派生类是 Circle，关系就是 ***Circle是一个Shape***，上面 Circle 明显不是一个 Point。

#### 建模层次

一种场景：当数据模型包含不同层次是，应该考虑继承。

#### 参数化表达式的行为

另一种场景：大部分行为和状态是多个类型共有的，但是又一小部分行为或状态需要随不同实现而有变化。这里的多个类型应该通过**是一个**测试

> 注意⚠️：不要创建出非常深的类层次，否则一个对象的多个状态和方法可能来自层次结构中的不同级别，导致代码难以理解。通常让 subclass 是具体类，让层次结构上方的双亲类是抽象类比较好。 

```ts 表达式层次结构
// IExpression 不需要是类，因为它不保存状态
interface IExpression {
  eval(): number;
}
abstract class BinaryExpression implements  {
  readonly a: number;
  readonly b: number;
  constructor(a: number, b: number) {
    this.a = a;
    this.b = b
  }
  // 抽象方法
  abstract eval(): number
}
class SumExpression extends BinaryExpression {
  eval(): number {
    return this.a + this.b
  }
}
class MyExpression extends BinaryExpression {
  eval(): number {
    return this.a * this.b
  }
}

abstract class UnaryExpression implements  {
  readonly a: number;
  constructor(a: number) {
    this.a = a;
  }
  // 抽象方法
  abstract eval(): number
}
class UnaryMinusExpression extends UnaryExpression {
  eval(): number {
    return -this.a
  }
}
```

### 组合数据和行为

重写或扩展行为，比继承更好 -> 组合

> 只要可能就优先选择**组合**而不是继承

```ts 继承和组合-使用一个Shape
class Shape {
  id: string;
  constructor(id: string) {
    this.id = id
  }
}
class Point {
  x: number;
  y: number;
  constructor(x: number, y: number) {
    this.x = x
    this.y = y
  }
}
class Circle extends Shape {
  center: Point;
  radius: number;
  constructor(id: string, center: Point, radius: number) {
    super(id)
    this.center = center
    this.radius = radius
  }
}
```

#### 「有一个」经验准则

组合和 **有一个** 关系： 组合在容器类型和被包含类型之间建立了一个 **有一个** 的关系。如果类型是 `Circle`， 被包含的类型是 `Point`，那么他们的关系是 `Circle`有一个 `Point`

#### 复合类

可以把复合类的状态的部分声明为私有成员，封装起来，然后使用额外的方法来增强这个复合类，让这些方法来访问实现中的私有成员。

```ts 向CEO提出问题
class CEO {
  isBusy(): boolean {}
  answer(question: string): string {}
}
class Department {}
class Budget {}
class Company {
  private ceo: CEO = new CEO();
  private departments: Department[] = []
  private budget: Budget = new Budget()
  askCEO(question: string): string | undefined {
    if (!this.ceo.isBusy()) {
      return this.ceo.answer(question)
    }
  }
}
```

#### 实现适配器模式

适配器模式能够兼容2个不同的类，且不需要修改其中任何一个类。不把几个组件合并到一起，而是封装一个组件，并提供它需要的胶水代码，使其能够作为另一种类型使用。

```ts 一个外部的几何库，但是不适合我们的对象模型
namespace GeometryLibrary {
  export interface ICircle {
    getCenterX(): number;
    getCenterY(): number;
    getDiameter(): number; // 获取直径
  }
  // operations on ICircle omitted
}
```

```ts CircleAdapter适配器
// 实现了库期望的ICircle接口
class CircleAdapter implements GeometryLibrary.ICircle {
  private circle: Circle;
  constructor(circle: Circle) {
    this.circle = circle
  }
  getCenterX(): number {
    return this.circle.center.x;
  }
  getCenterY(): number {
    return this.circle.center.y;
  }
  getDiameter(): number {
    return this.circle.center.radius * 2;
  }
}
```

### 扩展数据和行为

#### 使用组合扩展行为
#### 使用混入扩展行为

混入能够减少样板代码，允许混入不同的行为来构成一个对象，以及在多个类型中重用共有的行为。最呵呵实现横切关注点(cross-cutting concern): 程序中影响其他关注点单数不容易被分解的方面，例如引用计数，缓存，持久化等。

> 混入：混入通常是使用多重继承实现的（与本章开始的**是一个**经验准则发生了冲突）

从多重继承的角度看：创建一个 **IHunter** 类来实现捕猎行为，并让所有捕食性动物派生自这个类，这样 `Cat`既是`Animal`又是`Hunter`，混入与继承并不相同，可以只创建一个`HunterBehavaior`行为类来实现捕猎行为，并让所有捕食性动物包含这种行为。

> 混入与包含关系：混入在一个类型与其混入类型之间建立一个包含关系，与 **是一个** 关系不同

#### TS 中的混入

使用 `extend()`泛型函数，让该函数接受2个不同类型的实例，并把第二个实例的所有成员复制到第一个实例中。

> `&` 交叉类型，交集

```ts 使用一个实例的成员扩展另一个实例
function extend<First, Second>(first: First, second: Second): First & Second {
  const result: unknown = {}
  // 首先，迭代第一个对象的所有成员，把他们复制到result中
  for (const prop in first) {
    if (first.hasOwnProperty(prop)) {
      (<First>result)[prop] = first[prop]
    }
  }
  // 接下来，对第二个类型的成员做相同的处理
  for(const prop in second) {
    if (second.hasOwnProperty(prop)) {
      (<Second>result)[prop] = second[prop]
    }
  }
  return <First & Second>result
}
```

```ts 混入行为
// 不定义 Cat， 而是定义 喵喵行为 的宠物类型
// 豹子，猫 都可以喵喵
// 由于没有扩展捕猎行为，所以还不是 Cat
class MeowingPet extends Pet {
  meow(): void {}
}
class HunterBehavior {
  track(): void {};
  stalk(): void {};
  pounce(): void {};
}
type Cat = MeowingPet & HunterBehavior
const fluffy: Cat = extend(new MeowingPet(), new HunterBehavior())
```

#### 习题

```ts 跟踪快递单和包裹的状态
class Letter {}
class Package {}
class Tracking {
  private status: string;
  constructot(status: string) {
    this.status = status
  }
  updateStatus(status) {
    this.status = status
  }
}
type TrackingLetter = Letter & Tracking
type TrackingTracking = Package & Tracking
```

### 纯粹面向对象代码的替代方案

#### 1.和类型

如果需要用相同的方式传递不同类型的对象，或者把他们放到一个公共的集合中，他们并非必须实现相同的接口，或者有一个公共的双亲类型。相反，可以利用和类型，就可以不用在类型之间建立任何关系而获得相同的行为。

#### 2.函数式编程

避免了维护状态，直接计算返回，也不需要改变任何状态，(猜：这里我们可以窥见：既不需要维护状态也不需要改变状态，那么就可以实现 `immutable`?，所有实参固定成一个引用类型？)

#### 3.泛型编程

将在第九章，第十章深入介绍。

## 第九章 - 泛型数据结构

要点：分离关注点；为数据布局使用泛型数据结构；遍历任何数据结构；设置数据处理管道；

### 解耦关注点

```ts getNumbers()
type TransformFunction = (value: number) => number;
function getNumbers(transform: TransformFunction): number[] {

}
// 如果不需要转换，使用一个默认的doNothing
// doNothing 是恒等函数
function doNothing(value: number): number {
  return value;
}
function getNumbers(transform: TransformFunction = doNothing): number[] {

}
```

```ts assembleWidgets()
type PluckFunction = (widgets: Widget[]) => Widget[];
function assembleWidgets(pluck: PluckFunction): AssembleWidget[] {

}
// 如果不需要转换，使用一个默认的 pluckAll
// pluckAll 是恒等函数
function pluckAll(widgets: Widget[]): Widget[] {
  return widgets;
}
function assembleWidgets(pluck: PluckFunction = pluckAll): AssembleWidget[] {

}
```

`pluckAll` 和 `doNothing` 非常相似，只是类型不同：都接受一个实参，不做任何处理就返回该实参，2个都是恒等函数。


> 代数中定义的恒等函数: `f(x) = x`

#### 可重用的恒等函数

```ts 简单的可重用的恒等函数
// 简单实用 any 类型
// 当开始使用的时候，会失去类型安全。
function identity(value: any): any {
  return value
}
function square(x: number): number {
  return x * x
}
// 运行时失败❌
square(identity('hello'))
```

> 类型参数：泛型名称标志符

```ts 泛型恒等函数
function identity<T>(value: T): T {
  return value;
}
// 编译器足够只能，不需要显示指定 T 是什么类型
function getNumnbers(
  transform: TransformFunction = identity
): number[] {}

function assembleWidgets(pluck: PluckFunction = identity): AssembleWidget[] {

}
// 编译❌
square(identity('hello'))
```

#### 可选类型

泛型类 `Optional` 类型：当处理没有赋值的情况时，使用的逻辑与该值的实际类型没有关系。`Optional`可以存储其他任何类型。对 `Optional` 的修改不会影响 `T`，而对 `T` 的修改也不会影响 `Optional`。这是 ***泛型编程*** 极为强大的特征。

#### 泛型类型

泛型类型：指参数化一个或多个类型的泛型函数，类、接口等。能让代码的组件化程度更高。


#### 习题

```ts
class Box<T> {
  readonly value: T;
  constructor(value: T) {
    this.value = value
  }
}
function UnBox<T>(boxed: Box<T>): T {
  return boxed.value
}
```

### 泛型数据布局

```ts 数字二叉树&字符串链表
// 数字二叉树
class NumberBinaryTreeNode {
  value: number;
  left: NumberBinaryTreeNode | undefined;
  right: NumberBinaryTreeNode | undefined;
  constructor(value: number) {
    this.value = value
  }
}
// 字符串链表
class StringLinkedListNode {
  value: string;
  next: StringLinkedListNode | undefined;
  constructor(value: string) {
    this.value = value
  }
}
```

#### 泛型数据结构

2个例子可以看出 `NumberBinaryTreeNode` & `StringLinkedListNode`在数据结构和类型上产生了不必要的耦合。当有新的 `StringBinaryTreeNode` 时会有冗余代码。

```ts 泛型二叉树
class BinaryTreeNode<T> {
  value: T;
  left: BinaryTreeNode<T> | undefined;
  right: BinaryTreeNode<T> | undefined;
  constructor(value: T) {
    this.value = value
  }
}
```

```ts 泛型链表
class LinkedListNode<T> {
  value: T;
   next: LinkedListNode<T> | undefined;
  constructor(value: T) {
    this.value = value
  }
}
```

#### 什么是数据结构

##### 包含3个部分：
1. 数据自身，数据结构包含数据；
2. 数据的形状：例如二叉树，以分层方式布局数据，每层最多2个 children node。
3. 一组保留星座的操作：添加或移除元素。

##### 2个关注点：
1. 数据，包括数据的类型，实例保存的实际值
2. 数据的形状和保留形状的操作。

#### 习题

```ts
// 实现栈，后进先出
class Stack<T> {
  private value: T[] = [];
  public push(v: T): T[] {
    this.value.push(v)
  }
  public pop(): T | undefined {
    if (this.value?.length > 0) {
      const p = this.value.pop()
      return p
    }
    return undefined;
  }
  public peek(): T | undefined {
    if (this.value?.length > 0) {
      return this.value[this.value.length - 1]
    }
  }
}
// 实现元组
class Pair<T, U> {
  readonly first: T;
  readonly second: U;
  constructor(f: T, s: U) {
    this.first = f;
    this.second = s;
  }
}
```

### 遍历数据结构

> 中序遍历：递归地找到最深处左侧节点，然后依次遍历双亲节点，右侧节点，最左侧child tree遍历结束后访问该 child tree 的双亲节点，然后访问右侧节点

```ts 中序打印
class BinaryTreeNode<T> {
  value: T;
  left: BinaryTreeNode<T> | undefined;
  right: BinaryTreeNode<T> | undefined;
  constructor(value: T) {
    this.value = value
  }
}
function printInOrder<T>(root: BinaryTreeNode<T>): void {
  if(roor.left) {
    printInOrder(root.left)
  }
  console.log(root.value)
  if (root.right) {
    printInOrder(root.right)
  }
}
let root: BinaryTreeNode<number> = new BinaryTreeNode(1)
root.left = new BinaryTreeNode(2)
root.left.right = new BinaryTreeNode(3)
root.left.right = new BinaryTreeNode(4)
printInOrder(rrot)
// 2 3 1 4
```

#### 使用迭代器

把遍历逻辑移动到自己的组件中。

> 迭代器：能够用来遍历数据结构的一个对象。提供了一个标准结构，将数据结构的实际形状对客户的隐藏起来。

```ts 自定义迭代器
// 定义迭代器结果
type IteratorResult<T> = {
  done: boolean;
  value: T;
}
// 迭代器接口
interface Iterator<T> {
  next(): IteratorResult<T>
}
// 二叉树迭代器
// 这种实现方式并不是最高效的，因为我们需要队列的元素数量和树的节点数量相同，也就是说，如果树的层次很深，会需要占用较大的内存来存储这个队列
// 后面会有更好的实现，先使用这个作为示例
class BinaryTreeIterator<T> implements Iterator<T> {
  private values: T[]; // 值的队列
  private root: BinaryTreeNode<T>;
  constructor(root: BinaryTreeNode<T>) {
    this.values = [];
    this.root = root;
    this.inOrder(root) // 构造函数进行中序遍历来填充值的队列
  }
  next(): IteratorResult<T> {
    // 对 next() 的每次调用将通过调用 shift() 从队列中取出值
    const result: T | undefined = this.values.shift()
    if (!result) {
      return {
        done: true,
        value: result
      }
    }
    return {
      done: false,
      value: result
    }
  }
  private inOrder(node: BinaryTreeNode<T>): void {
    if (node.left) {
      this.inOrder(node.left)
    }
    this.values.push(node.value)
    if (node.right) {
      this.inOrder(node.right)
    }
  }
}
```

```ts 链表迭代器
class LinkedListIterator<T> implements Iterator<T> {
  private head: LinkedListNode<T>;
  private current: LinkedListNode<T> | undefined;
  constructor(head: LinkedListNode<T>) {
    this.head = head
    this.current = head
  }
  next():  IteratorResult<T> {
    if (!this.current) {
      return {
        done: true,
        value: this.head.value
      }
    }
    const result: T = this.current.value
    this.current = this.current.next
    return {
      done: false,
      value: result
    }
  }
}
```

```ts 使用迭代器的print()
// print()是一个泛型函数，接受一个迭代器作为实参
function print<T>(iterator: Iterator<T>): void {
  // 使用 next 初始化，取出第一个值
  let result: IteratorResult<T> = iterator.next()
  while(!result.done) {
    console.log(result.value)
    result = iterator.next()
  }
}
```

```ts 使用迭代器的contains()
// print()是一个泛型函数，接受一个迭代器作为实参
function contains<T>(value: T, iterator: Iterator<T>): boolean {
  // 使用 next 初始化，取出第一个值
  let result: IteratorResult<T> = iterator.next()
  while(!result.done) {
    if (result.value === value) return true
    result = iterator.next()
  }
  return false
}
```

迭代器是把数据结构和算法连接起来的胶水，使得这种解耦能够实现。

#### 流水线化迭代代码

TS中已经预定义了 `IteratorResult<T>` 和 `Iterator<T>` 类型

```ts 可迭代的接口
interface Iterable<T> {
  // [Symbol.iterator] 是 TS 特殊语法，只是代表一个特殊名称。
  // 与 前文中实现 Normal Subtype 技巧类似
  [Symbol.iterator](): Iterator<T>;
}
```

```ts 可迭代的链表
class LinkedListNode<T> implements Iterable<T> {
  value: T;
  next: LinkedListNode<T> | undefined;
  constructor(value: T) {
    this.value = value;
  }
  // 通过在链表中创建 LinkedListIterator 的新实例来实现 Iterable<T> 接口
  [Symbol.iterator](): Iterator<T> {
    return new LinkedListIterator<T>(this)
  }
}
```

> 在TS中，Iterable 数据结构允许使用 `for...of`语法。

```ts '使用Iterator实参的print()和contains()'
function print<T>(iterator: Iterator<T>): void {
  let result: IteratorResult<T> = iterator.next()
  while(!result.done) {
    console.log(result.value)
    result = iterator.next()
  }
}

function contains<T>(value: T, iterator: Iterator<T>): boolean {
  let result: IteratorResult<T> = iterator.next()
  while(!result.done) {
    if (result.value === value) return true
    result = iterator.next()
  }
}
```

```ts '使用 Iterable 实参的print()和contains()'
function print<T>(iterable: Iterable<T>): void {
  for (const item of iterable) {
    console.log(item)
  }
}

function contains<T>(value: T, iterable: Iterable<T>): boolean {
  for (const item of iterable) {
    // return 可以退出 for...of 循环
    if (item === value) return true
  }
  return false
}
```

```ts 使用生成器的二叉树迭代器
function* inOrderIterator<T>(root: BinaryTreeNode<T>):IterableIterator<T> {
    if (root.left) {
      for (const value of inOrderIterator(root.left)) {
        yield value;
      }
    };
    yield root.value
    if (root.right) {
      for (const value of inOrderIterator(root.right)) {
        yield value;
      }
    }
  }
```

```ts 使用生成器的链表迭代器
function* LinkedListIterator<T>(head: LinkedListNode<T>): IterableIterator<T> {
  let current: LinkedListNode<T> | undefined = head;
  while(current) {
    yield current.value;
    current = current.next;
  }
}
```

```ts 使用生成器的可迭代链表
class LinkedListNode<T> implements Iterable<T> {
  value: T;
  next: LinkedListNode<T> | undefined;
  constructor(value: T) {
    this.value = value;
  }
  // [Symbol.iterator]() 只是返回 LinkedListIterator 的结果
  [Symbol.iterator](): Iterator<T> {
    return LinkedListIterator(this)
  }
}
```

#### 习题

```ts 泛型二叉树的先序遍历&顺序遍历数组
// 泛型二叉树结构
class BinaryTreeNode<T> {
  value: T;
  left: BinaryTreeNode<T> | undefined;
  right: BinaryTreeNode<T> | undefined;
  constructor(value: T) {
    this.value = value
  }
}
function* inOrderIteratorFirst<T>(root: BinaryTreeNode<T>):inOrderIteratorFirst<T> {
    yield root.value
    if (root.left) {
      for (const value of inOrderIteratorFirst(root.left)) {
        yield value;
      }
    };
    if (root.right) {
      for (const value of inOrderIteratorFirst(root.right)) {
        yield value;
      }
    }
  }

function* IteratorArr<T>(array: T[]): void {
  for (const item of array) {
    yield item;
  }
}
```

### 数据流

> 迭代器并非是有限的

```ts 无限随机数据流
// 无限随机数据流
function* generateRandomNumbers(): IterableIterator<number> {
  while(true) {
    yield Math.random()
  }
}
// 使用流中的数字
let iter: IterableIterator<number> = generateRandomNumbers()
// 迭代器 的调用方式
console.log(iter.next().value)
console.log(iter.next().value)
console.log(iter.next().value)
console.log(iter.next().value)
```

#### 处理管道

处理管道的组件是一些函数，接受一个迭代器作为实参，返回一个迭代器。这种函数可以链接起来。这种模式是反应式编程的基础。

> 由前文实现 `IterableIterator<T>` 方式可知，`IterableIterator<T>` 是 `Iterable<T>` 类型的 subtype， 函数实参是双向协变的。所以也可以传入 `Iterable<T>`

管道是延迟计算的，可以逐个（按调用顺序）处理值

```ts square & take & 管道
function* square(iter: Iterable<number>): IterableIterator<number> {
  for (const value of iter) {
    yield value ** 2
  }
}
function* take<T>(iter: Iterable<T>, n: number): IterableIterator<T> {
  for (const value of iter) {
    if (n-- <= 0) return
    yield value
  }
}
const values: IterableIterator<number> = take(square(generateRandomNumbers()), 5)
for (const value of values) {
  console.log(value)
}
// 练习 实现 drop 丢弃一个迭代器的前n个元素而返回其余元素
function* drop<T>(iter: Iterable<T>, n: number): IterableIterator<T> {
  for (const [v, k] of iter) {
    if (n-- > 0) continue
    yield value
  }
}
function* count(): IterableIterator<number> {
  let n: number = 0
  while(true) {
    n++
    yield n
  }
}
for (const value of take(drop(count(), 5), 5)) {
  console.log(value)
}
```

## 第十章 - 泛型算法和迭代器

### 更好的 `map()`、`filter()`和`reduce()`

```ts 使用迭代器的 map & filter & reduce
// 可应用到任何 Iterable<T>,而不只是数组
function* map<T, U>(iter: Iterable<T>, func: (item: T) => U):IterableIterator<U> {
  for (const value of iter) {
    yield func(value)
  }
}
// pred=谓词
function* filter<T, U>(iter: Iterable<T>, pred: (item: T) => boolean):IterableIterator<U> {
  for (const value of iter) {
    if (pred(value)) {
      yield func(value)
    }
  }
}
function* reduce<T>(iter: Iterable<T>, init: T, op: (x: T, y: T) => T):T {
  let result: T = init
  for (const value of iter) {
    result=op(result, value)
  }
  return result
}
```

#### `filter()/reduce()管道`

从一个二叉树中取出偶数并对它们求和，使用第九章的 `BinaryTreeNode<T>`，采用中序遍历，并链接到一个偶数过滤函数和使用加法操作的reduce

```ts
let root: BinaryTreeNode<number> = new BinaryTreeNode(1);
root.left = new BinaryTreeNode(2);
root.left.right = new BinaryTreeNode(3)
root.right = new BinaryTreeNode(4)
//       1
//    2     4
//     3
// 中序遍历打印结果：2 3 1 4
const result: number = reduce(filter(inOrderIterator(root), (value) => value % 2 == 0), 0, (x, y) => x + y)
console.log(result) // 6
```

#### 习题

```ts
// 连接所有非空字符串，处理 string 类型的一个可迭代结构
// reduce + filter
function concatStrAndFilterEmpty(iter: Iterable<string>): string {
  return reduce(filter(iter, (value) => value.length > 0), '', (x: string, y: string) => x + y)
}
// 选择所有的奇数并求平方，处理 number 类型的一个可迭代结构
// map + filter
function squareOdds(iter: Iterable<number>): IterableIterator<number> {
  return map(filter(iter, (value) => value % 2 == 1), (x) => x * x)
}
```

### 常用算法

实现流畅管道：封装更好的流畅的可迭代结构

```ts
class FluentIterable<T> {
  iter: Iterable<T>;
  constructor(iter: Iterable<T>) {
    this.iter = iter
  }
  map<U>(func: (item: T) => U): FluentIterable<U> {
    return new FluentIterable(this.mapImpl(func))
  }
  private *mapImpl<U>(func: (item: T) => U): IterableIterator<U> {
    for (const value of this.iter) {
      yield func(value)
    }
  }
  filter<U>(pred: (item: T) => boolean): FluentIterable<T> {
    return new FluentIterable(this.filterImpl(pred))
  }
  private *filterImpl(pred: (item: T) => boolean): IterableIterator<T> {
    for (const value of this.iter) {
      if (pred(value)) {
        yield value
      }
    }
  }
  // 增加take()
  take<T>(n: number): FluentIterable<T> {
    return new FluentIterable(this.takeImpl(n))
  }
  private *takeImpl<T>(n: number): IterableIterator<T> {
    for (const value of this.iter) {
      if (n-- <= 0) return
      yield value
    }
  }
  // 增加drop()
  drop<T>(n: number): FluentIterable<T> {
    return new FluentIterable(this.dropImpl(n))
  }
  private *dropImpl<T>(n: number): IterableIterator<T> {
    for (const [v, k] of this.iter) {
      if (n-- > 0) continue
      yield value
    }
  }
  reduce(init: T, op: (x: T, y: T) => T): T {
    let result: T = init;
    for(const value of this.iter) {
      result = op(result, value)
    }
    return result
  }
}
```

```ts 应用filter/reduce管道
let root: BinaryTreeNode<number> = new BinaryTreeNode(1);
root.left = new BinaryTreeNode(2);
root.left.right = new BinaryTreeNode(3)
root.right = new BinaryTreeNode(4)
//       1
//    2     4
//     3
// 中序遍历打印结果：2 3 1 4
const result: number = new FluentIterable(inOrderIterator(root)).filter(value => value % 2 == 0).reduce(0, (x, y) => x + y)
console.log(result)
```

### 约束类型参数

约束告诉编译器某个类型实参必须具有的能力（该类型独有的方法，属性等）。一旦要求泛型类型要必须有特定成员，就使用约束将允许类型的集合限制为具有必要成员的那些类型

```ts 使用类型约束的 renderAll
interface IRenderable {
  render(): void;
}
function renderAll<T extends IRenderable>(iter: Iterable<T>): void {
  for(const item of iter) {
    item.render()
  }
}
```

#### 具有类型约束的泛型算法

> 以实现一个 `max()` 泛型算法为例

```ts
// Icomparable 接口
enum ComparisonResult {
  LessThan,
  Equal,
  GreeterThan
}
interface Icomparable<T> {
  compareTo(value: T): ComparisonResult;
}
// 迭代器无值的情况下会返回undefined
// 所以不使用 for of循环，而使用 next()手动向前推进迭代器
function max<T extends Icomparable<T>>(iter: Iterable<T>): T | undefined {
  let iterator: Iterator<T> = iter[Symbol.iterator]()
  // 取出第一个值
  let current: IteratorResult<T> = iter.next()
  if (current.done) return undefined
  let result: T = current.value
  while(true) {
    current = iterator.next()
    if (current.done) return result
    if (current.value.compareTo(result) == ComparisonResult.GreeterThan) {
      result = current.value
    }
  }
}
// 也可以接受一个 compare 方法作为实参
function max<T extends Icomparable<T>>(iter: Iterable<T>, compare: (x: T, y: T) => ComparisonResult): T | undefined {
  let iterator: Iterator<T> = iter[Symbol.iterator]()
  // 取出第一个值
  let current: IteratorResult<T> = iter.next()
  if (current.done) return undefined
  let result: T = current.value
  while(true) {
    current = iterator.next()
    if (current.done) return result
    if (compare(current.value, result) === ComparisonResult.GreeterThan) {
      result = current.value
    }
  }
}
```

#### 习题
实现一个泛型函数 `Clamp(value, min, max)`, 如果 `min < value < max ? value : (value <= min ? min : max)

```ts
enum ComparisonResult {
  LessThan,
  Equal,
  GreeterThan
}
interface Icomparable<T> {
  compareTo(value: T): ComparisonResult;
}

function Clamp<T extends Icomparable<T>>(v: T, min: T, max: T): T | undefined {
  if (!value) return undefined
  if (value.compareTo(min) === ComparisonResult.LessThan) return min
  if (value.compareTo(max) === ComparisonResult.GreeterThan) return max
  return value
}
```

### 高效 reverse 和其他使用迭代器的算法

> 使用一个栈存储来实现`reverse`的空间是O(n)，时间也是T(n)，都是线性的；使用数组交换两头来实现`reverse`的空间是O(1)，只用了一个临时变量，时间是T(n/2)

#### 迭代器的基础模块

判断迭代器合适应该停止=捕获2个迭代器相同的时机：重新定义为一组接口，公开一个`get()`方法，用于返回`T`类型的一个值，使用这个方法从迭代器中读取值

```ts 'IReadable<T>, IIncrementable<T>, IInputIterator<T>'
// get() 用于获取迭代器当前的值
interface IReadable<T> {
  get(): T;
}
// increment() 用于将迭代器推进到下一个元素
interface IIncrementable<T> {
  increment(): void;
}
interface IInputIterator<T> extends IReadable<T>, IIncrementable<T> {
  equals(other: IInputIterator): boolean;
}
```

为链表实现一个满足新的 `IInputIterator<T>` 接口的 `LinkedListInputIterator<T>`

> 输入迭代器: 能够遍历序列一次并提供其值的迭代器，不能第二次重放值。并非必须遍历持久性数据结构，也可以从生成器或者其他某种数据源提供值。

```ts 链表输入迭代器
class LinkedListInputIterator<T> implements IInputIterator<T> {
  private node: LinkedListNode<T> | undefined;
  constructor(node: LinkedListNode<T> | undefined) {
    this.node = node
  }
  increment(): void {
    if(!this.node) throw Error();
    this.node = this.node.next
  }
  get(): T {
    if(!this.node) throw Error();
    return this.node.value
  }
  equals(other: IInputIterator<T>): boolean {
    return this.node == (<LinkedListInputIterator<T>>other).node
  }
}
// 使用：链表上的一对迭代器
const head: LinkedListNode<number> = new LinkedListNode(0)
head.next = new LinkedListNode(1)
head.next.next = new LinkedListNode(2)
let begin: IInputIterator<number> = new LinkedListInputIterator(head)
// end 的值是 undefined
let end: IInputIterator<number> = new LinkedListInputIterator(undefined)
```

> 输出迭代器: 能够遍历一个序列并向其卸任值的迭代器，并不需要能够读值。同样的，并非必须遍历持久性数据结构，也可以把值卸任其他输出。

```ts 'IWritable<T> and IOutputIterator<T>'
interface IWritable<T> {
  set(value: T): void;
}
interface IOutputIterator<T> extends IWritable<T>, IIncrementable<T> {
  equals(other: IOutputIterator<T>): boolean;
}
```

实现一个写入到控制台的输出迭代器：写入到输出流。例如：网络连接，标准输出，标准错误等，但是不能读值。

```ts 控制台输出迭代器
class ConsoleOutputIterator<T> implements IOutputIterator<T> {
  set(value: T): void {
    console.log(value)
  }
  // 这个例子没有遍历数据结构，所以可以啥也不做
  increment(): void {}
  equals(other: IOutputIterator<T>): boolean {
    // 始终返回false，因为写入控制台没有可以比较的序列末尾
    return false
  }
}
```

```ts 具有输入和输出迭代器的map
function map<T, U>(
  begin: IInputIterator<T>,
  end: IInputIterator<T>,
  out: IOutputIterator<U>,
  func: (value: T) => U
): void{
  while (!begin.equals(end)) {
    out.set(func(begin.get()))
    begin.increment()
    out.increment()
  }
}
```

#### `有用的find()`

前向迭代器：向前推进，可以读取当前位置的值以及更新该值的迭代器。前向迭代器也可以被克隆，使得推进该迭代器不会影响该迭代器的克隆。跟输入输出迭代器不同，它可以多次遍历一个序列。在推进原迭代器时，副本迭代器不会移动，是独立的。因此它可以多次遍历。

```ts 'IForwardIterator<T> and LinkedListIterator<T> 实现 IForwardIterator<T>'
interface IForwardIterator<T> extends IReadable<T>, IWritable, IIncrementable<T> {
  equals(other: IForwardIterator<T>): boolean;
  clone(): IForwardIterator<T>
}
class LinkedListIterator<T> implements IForwardIterator<T> {
  private node: LinkedListNode<T> | undefined;
  constructor(node: LinkedListNode<T> | undefined) {
    this.node = node
  }
  increment(): void {
    if (!this.node) return
    this.node = this.node.next
  }
  get(): T {
    if(!this.node) throw Error()
    return this.node.value
  }
  // set() 是 IWritable<T> 需要的一个额外的方法，用于封信链表节点的值
  set(value: T): void {
    if(!this.node) throw Error()
    this.node.value = value
  }
  // equals 现在期望收到另外一个 IForwardIterator<T>
  equals(other: IForwardIterator<T>): boolean {
    return this.node = (<LinkedListIterator<T>>other).node
  }
  // 
  //  创建一个新的迭代器，指向与当前迭代器相同的节点
  clone(): IForwardIterator<T> {
    return new LinkedListIterator(this.node)
  }
}
```

```ts 使用前向迭代器的find
function find<T> (
  begin: IForwardIterator<T>,
  end: IForwardIterator<T>,
  pred: (value: T) => booean
): IForwardIterator<T> {
  while(!begin.equals(end)) {
    if (pred(begin.equals(end))) {
      return begin
    }
    begin.increment()
  }
  return end
}
// 应用：将链表中的42替换为0
let head: LinkedListNode<number> = new LinkedListNode(1)
head.next = new LinkedListNode(2)
head.next.next = new LinkedListNode(42)
let begin: IForwardIterator<number> = new IForwardIterator(head)
let end: IForwardIterator<number> = new IForwardIterator(undefined)
let iter: IForwardIterator<number> = find(begin, end, (value: number) => value == 42)
// 调用
if (!iter.equals(end)) {
  iter.set(0)
}
```

#### 高效的`reverse()`

数组实现`reverse()`时是从数组的两端开始互换元素，一直递增前向索引。递减后向索引，直到两者相遇。

> 双向迭代器：不仅具有前向迭代器的所有能力，而且还可以递减。既能前向，又能后向遍历。可以读写当前元素的值，克隆自己，以及向前或向后步进。

链表不支持后向迭代，但是可以使用一个双向链表保存其前导节点和后继节点的引用。

```ts 'IBiddirectionalIterator<T> and ArrayIterator<T>'
interface IBiddirectionalIterator<T> extends IReadable<T>, IWritable<T>, IIncrementable<T> {
  decrement(): void;
  equals(other: IBiddirectionalIterator<T>): boolean;
  clone(): IBiddirectionalIterator<T>
}
class ArrayIterator<T> implements IBiddirectionalIterator<T> {
  private array: T[];
  private index: number;
  constructor(array: T[], index: number) {
    this.array = array;
    this.index = index
  }
  get(): T {
    return this.array(this.index)
  }
  set(value: T): void {
    this.array(this.index) = value
  }
  increment(): void {
    this.index++
  }
  decrement(): void {
    this.index--
  }
  equals(other: IBiddirectionalIterator<T>): boolean {
    return this.index == (<ArrayIterator<T>>other).index
  }
  clone(): IBiddirectionalIterator<T> {
    return new ArrayIterator(this.array, this.index)
  }
}
```

```ts 使用双向迭代器的reverse
function reverse<T>(
  begin: IBiddirectionalIterator<T>,
  end: IBiddirectionalIterator<T>
): void {
  while(!begin.equals(end)) {
    // end 从数组末尾后面的一个元素开始，所以在使用之前需要递减
    end.decrement()
    if (begin.equals(end)) return
    const tmp: T = begin.get()
    begin.set(end.get())
    end.set(temp)
    begin.increment()
  }
}
// 翻转一个数值数组
let array: number[] = [1,2,3,4,5];
let begin: IBiddirectionalIterator<number> = new ArrayIterator(array, 0)
let end: IBiddirectionalIterator<number> = new ArrayIterator(array, array.length) // 将数组的end迭代器初始化为索引长度，所以初始array[end]值为undefined
reverse(begin, end)
console.log(array) // [5,4,3,2,1]
```

#### 高效地获取元素

随机访问迭代器：能够以O(1)时间向前或向后跳过任意多元素。可以读写当前元素的值、克隆自己以及向前或向后移动任意元素。

> 数组可以通过索引快速获取任何元素，但是双向链表/链表均不支持随机访问迭代器。

```ts 'IRandomAccessIterator<T> & 更新后的 ArrayIterator<T>'
interface IRandomAccessIterator<T> extends IReadable<T>, IWritable<T>, IIncrementable<T> {
  decrement(): void;
  equals(other: IRandomAccessIterator<T>): boolean;
  clone(): IRandomAccessIterator<T>;
  move(n: number): void;
  distance(other: IRandomAccessIterator<T>): number;
}
class ArrayIterator<T> implements IRandomAccessIterator<T> {
  // ...
  // 与上面相同的部分
  // 更新
  equals(other: IRandomAccessIterator<T>): boolean {
    return this.index == (<ArrayIterator<T>>other).index
  }
  clone(): IRandomAccessIterator<T> {
    return new ArrayIterator(this.array, this.index)
  }
  // 将迭代器推进 n 个步长（n 可以是负数，代表向后移动）
  move(n: number): void {
    this.index += n
  }
  distance(other: IRandomAccessIterator<T>): number {
    return this.index - (<ArrayIterator<T>>other).index
  }
}
```

实现一个 `elementAt()` 算法，返回只想序列中第n个元素的迭代器。使用一个输入迭代器来实现时：需要递增迭代器 `n` 次此案达到期望的元素，时间复杂度为 O(n)；使用随机访问迭代器时，时间复杂度为 O(1)；

```ts 访问指定位置的元素
function elementAtRandomAccessIterator<T> (
  begin: IRandomAccessIterator<T>,
  end: IRandomAccessIterator<T>,
  n: number
): IRandomAccessIterator<T>{
  // 移动 n 个元素
  begin.move(n)
  if (begin.distance(end) <= 0) return end;
  return begin;
}
```

### 自适应算法

```ts 使用输入迭代器的elementAt
function elementAtForwardIterator<T> (
  begin: IForwardIterator<T>,
  end: IForwardIterator<T>,
  n: number
): IForwardIterator<T>{
  while(!begin.equals(end) && n > 0) {
    begin.increment()
    n--
  }
  return begin
}
```

```ts 自适应的elementAt
function isRandomAccessIterator<T> (
  iter: IForwardIterator<T>
): iter is IRandomAccessIterator<T>{
  return 'distabce' in iter
}
function elementAt<T> (
  begin: IForwardIterator<T>,
  end: IForwardIterator<T>,
  n: number
): IForwardIterator<T> {
  if(isRandomAccessIterator(begin) && isRandomAccessIterator(end)) {
    return elementAtRandomAccessIterator(begin, end, n)
  } else {
    return elementAtForwardIterator(begin, end, n)
  }
}
```

## 补充：

### ES6遍历器 `Iterator` 和 `for...of`

> ES6中的：Array, Map, Set, String, TypedArray, arguments, NodeList都自带遍历器接口**Iterable定义见上**，但是对象没有，原因见下

一种接口，为各种不同的数据结构提供统一的访问机制。

#### 3种作用

1. 为各种不同的数据结构提供统一的访问接口；
2. 使得数据成员能够按照某种次序排列-**对象的属性没有固定顺序，这就是为什么对象类型没有遍历器接口**，不过可以曲线使用 `Object.keys() or Object.values()` 来实现遍历。
3. Iterator接口主要提供给 `for...of` 消费

```ts 模拟 Iterator
let it = makeIterator(['a', 'b'])
it.next() // {value: 'a', done: false}
it.next() // {value: 'b', done: false}
it.next() // {value: undefined, done: true}
function makeIterator(array) {
  let nextIndex = 0
  return {
    next: function () {
      return nextIndex < array.length ?
        // array[nextIndex]
        // nextIndex++
        { value: array[nextIndex++], done: false}
        :
        { value: undefined, done: true}

    }
  }
}
```

#### 默认调用Iterator接口的场合

1. 解构赋值
2. 扩展运算符
3. `yield*` 后面根的是一个可遍历的结构，它会在调用该结构的遍历器接口；
4. 其他场合：使用数组作为参数的场合: `for...of, Array.from(), Map(), Set(), WeakMap(), WeakSet(), Promise.all(), Promise.race()`

### ES6迭代器 `generator`

1. 使用 `generator` 生成遍历器对象；
2. 状态机：保存多个内部状态；
3. 调用方式和前文的 Iterator 一样，不会立即执行，需要反复调用`next，直到遇到 `return` 或者 下一个`yield`；
4. `yield` 后面的表达式是惰性的，只有next指针指向该语句时才会求值。
5. `yield*` 后可跟 `generator` - `generator` 嵌套使用
6. `iter.next()` 迭代器实例的***next是同步的***

#### `generator` 错误捕获

外部执行函数时使用 `try catch` 捕获

### `generator` 的异步应用

#### 协程 `generator` 函数实现

> 最大的特点是可以交出函数的执行权；但是流程管道却不方便。

`iter.next(1)`  函数 `next`可传入参数作为上一步的返回结果，前提是 return 的是 yield

```ts
// 这种写法可以
function* gen(x) {
  const y = yield x + 2
  return y
}
// 这种不行
function* gen(x) {
  yield x + 2
  return 2
}
const g = gen(1)
g.next() // {value: 3, done: false}
// 如果上一步value不为undefined，那么只改变value，done的值由当前调用次数决定
g.next(2) // {value: 2, done: true}
// 已经执行完毕后值不变，就是undefined
g.next(3) // {value: undefined, done: true}
```

#### Trunk 函数

自动执行 Generator 函数的一种方法

参数的求值策略：
1. 传值策略 - JS
2. 传名策略 - 编译器


### async await

1. `forEach`可以传入 `async` 函数实现并发
2. `async` 函数内使用`for` 循环、或 `reduce()`实现继发
3. `map` 内传入异步实现不了并发，只能得到一组异步函数

    ```ts
    const testArr = [1,2,3,4,5]
    testArr.map(async (item) => {
      return await item
    })
    // 等于
    async () => {
      return await 1
    }
    async () => {
      return await 2
    }
    async () => {
      return await 3
    }
    async () => {
      return await 4
    }
    async () => {
      return await 5
    }
    ```
4. `async` 函数可以保留运行堆栈: 而普通函数调用异步函数则没有保留运行堆栈。

### 异步遍历器

```ts 异步遍历器接口定义
interface AsyncIterable {
  [Symbol.asyncIterator](): AsyncIterator;
}
interface AsyncIterator {
  next(): Promise<IteratorResult>
}
interface IteratorResult {
  value: any;
  done: boolean;
}
```

> `next()`返回的 `value`属性是一个 `Promise` 对象，而`done`还是同步的，在 调用 `then()` 或 `await` 后返回真正的值。

```ts
const asyncIterable = creatAsyncIterable(['a', 'b'])
const asyncIterator = asyncIterable[Symbol.asyncIterator]()
// then
asyncIterator.next().then(r1 => {
  console.log(r1) // {value: 'a', done: false}
  return asyncIterator.next()
}).then(r2 => {
  console.log(r2) // {value: 'b', done: false}
  return asyncIterator.next()
})
.then(r3 => {
  console.log(r3) // {value: undefined, done: true}
})
// await
async funcction f() {
  const asyncIterable = creatAsyncIterable(['a', 'b'])
  const asyncIterator = asyncIterable[Symbol.asyncIterator]()
  console.log(await asyncIterator.next()) // {value: 'a', done: false}
  console.log(await asyncIterator.next()) // {value: 'b', done: false}
  console.log(await asyncIterator.next()) // {value: undefined, done: true}
}
// 异步遍历器的 next() 是可以连续调用的
// 1. Promise.all() 将所有的 next 方法放在 Promise.all() 里面
const [{value: v1}, {value: v2}] = await Promise.all([asyncIterator.next(), asyncIterator.next()])
console.log(v1, v2)
// 2. 一次性调用所有的 next 方法，然后 await 最后一步操作
async function runner() {
  const writer = openfile('somefile.txt')
  writer.next('hello')
  writer.next('world')
  await writer.return()
}
runner()
```

#### `for await..of` 遍历异步 Iterator 接口

> 也可以用于同步遍历器

`for..of` 循环自动调用这个对象的异步遍历器 next，会得到一个 Promise 对象， `await` 处理这个 Promise 对象的 `resolve`值，`try catch`来处理这个 Promise 对象的 `reject`值

#### 异步 `Generator` 函数

> 返回一个异步遍历器对象 `AsyncIterator`，跟在 `yield`命令后面的应该是一个 Promise 对象， 如果不是也会被自动包装成一个Promise 对象

```ts
async function* gen() {
  yield 'hello'
}
const genObj = gen()
genObj.next().then(x => console.log(x))
// 如果抛错会被Promise.catch捕获
```
