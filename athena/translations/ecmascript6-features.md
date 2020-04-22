# ECMAScript6 Features

<!-- TOC -->

- [ECMAScript6 Features](#ecmascript6-features)
    - [Introduction](#introduction)
        - [Arrows](#arrows)
        - [Classes](#classes)
        - [Enhanced Object Literals](#enhanced-object-literals)
        - [Template Strings](#template-strings)
        - [Destructuring](#destructuring)
        - [Default + Rest + Spread](#default--rest--spread)
        - [Let + Const](#let--const)
        - [Iterators + For…Of](#iterators--forof)
        - [Generators](#generators)
        - [Unicode](#unicode)
        - [Modules](#modules)
        - [Module Loaders](#module-loaders)
        - [Map + Set + WeakMap + WeakSet](#map--set--weakmap--weakset)
        - [Proxies](#proxies)
        - [Symbols](#symbols)
        - [Subclassable Built-ins](#subclassable-built-ins)
        - [Math + Number + String + Array + Object APIs](#math--number--string--array--object-apis)
        - [Binary and Octal Literals](#binary-and-octal-literals)
        - [Promises](#promises)
        - [Reflect API](#reflect-api)
        - [Tail Calls](#tail-calls)

<!-- /TOC -->

## Introduction

> 原文: [ECMAScript 6 Features](https://github.com/lukehoban/es6features/blob/master/README.md)

本文是对 ECMAScript 6 新标准的简要介绍。

ECMAScript 6，也称 ECMAScript 2015，是 ECMAScript 的最新标准。ES6 是对 JavaScript 的一次重大更新，也是自2009年的 ES5 标准后的第一次更新。
主流的 JavaScript 引擎都逐渐对这些新标准提供支持。

### Arrows

箭头函数是简写的函数表达式，可以被记为 `=>` 这样的语法形式。语法上和 C#， Java 8 以及 CoffeeScript 有点相似。
他们都支持语句块形式和表达式形式，并且表达式形式会返回表达式计算的返回值。和普通函数有所区别的是，箭头函数会捕获当前所在上下文的 this 来作为该函数的 this 值(而不像普通函数一样每个定义的函数都有自己的 this 值)。

```js
// Expression bodies
var odds = evens.map(v => v + 1)
var nums = evens.map((v, i) => v + i)
var pairs = evens.map(v => ({ even: v, odd: v+1 }))
// Statement bodies
nums.forEach(v => {
  if (v % 5 === 0) {
    fives.push(v)
  }
})
// Lexical `this`
var bob = {
  _name: 'Bob',
  _friends: [],
  printFriends() {
    this._friends.forEach(f => {
      console.log(this._name + " knows " + f)
    })
  }
}
```

☞ 更多信息: [MDN Arrow Functions](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

### Classes

ES6 的 classes 是 JavaScript 现有的面向对象-原型继承(prototype-based)的语法糖。单一声明链使得 JavaScript 的类更易于使用以及相互协作。
`classes` 支持原型继承、调用父类的`super`、创建实例、静态方法以及构造函数等等…

```js
class SkinnedMesh extends THREE.Mesh {
  constructor(geometry, materials) {
    super(geometry, materials)
    this.idMatrix = SkinnedMesh.defaultMatrix()
    this.bones = []
    this.boneMatrices = []
    // ...
  }
  update(camera) {
    //...
    super.update()
  }
  
  get boneCount() {
    return this.bones.length
  }
  set matrixType(matrixType) {
    this.idMatrix = SkinnedMesh[matrixType]()
  }
  static defaultMatrix() {
    return new THREE.Matrix4()
  }
}
```

☞ 更多信息: [MDN Classes](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes)

### Enhanced Object Literals

对象字面量(封闭在对象花括号`{}`内键值对)扩展是为了支持构造函数、类似`foo: foo`这类语法的简写、定义方法、调用父类方法、通过表达式计算属性值等作用。这些功能让对象字面量更加靠近声明类，也让对象继承更加易于使用。

```js
var obj = {
  // 原型内部属性
  __proto__: theProtoObj,
  // `handler: handler` 语法的简写
  handler,
  // 定义方法
  toString() {
    // 调用父类方法
    return 'd ' + super.toString()
  },
  // 动态计算属性值
  [ 'prop_' + (() => 42)() ]: 42
}
```

☞ 更多信息: [MDN Grammar and types: Object literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types#Object_literals)

### Template Strings

模板字符串提供了创建字符串的语法糖。这种功能和Perl、Python有点相似。增加一个’标签’可以创建一个普通的字符串，同时避免注入攻击或无意从字符串中创建了更高级的数据结构。

```js
// 基本创建用法
`In JavaScript '\n' is a line-feed.`
// 多行形式的字符串
`In JavaScript this is
not legal.`
// 字符串模板
var name = 'Bob', time = 'tody'
`Hello ${name}, how are you ${time}?`
// 通过新特性(构造、替换)来说明如何创建一个 HTTP 请求前缀
POST`http://foo.org/bar?a=${a}&b=${b}
     Content-Type: application/json
     { "foo": ${foo},
       "bar": {$bar} }`(myOnReadyStateChangeHandler)
```

☞ 更多信息: [MDN Template Strings](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/template_strings)

### Destructuring

‘解构’特性允许通过模式匹配来赋值，支持数组和对象。和对象相似，如果 `foo['bar']` 找不到结果会返回 `undefined`，解构匹配赋值没有找到结果也返回 `undefined`。

```js
// 数组匹配
var [a, ,b] = [1, 2, 3]
// 对象匹配
var {
  op: a,
  lhs: {
    op: b
  },
  rhs: c
} = getASTNode()
// 对象属性简写语法糖赋值
var { op, lhs, rhs } = getASTNode()
// 传参形式
function g({ name: x }) {
  console.log(x)
}
g({ name: 5 })
// Fail Soft(意即使出错也保持程序能够继续运行)
var [a] = []
a === undefined // true
// Fail Soft (带默认值)
var [a = 1] = []
a === 1 // true
```

☞ 更多信息: [MDN Destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

### Default + Rest + Spread

类似 Java 中的可变参数，将参数变成一个数组传入，来替代先前的零个或多个参数。

```js
function f(x, y=12) {
  // 如果没有传入 y 或 y 等于 undefined, 给 y 一个默认值12
  return x + y
}
f(3) === 15 // true
function f(x, ...y) {
  // y 是一个数组类型
  return x * y.length
}
f(3, 'hello', true) === 6
function f(x, y, z) {
  return x + y + z
}
// 多个参数打包成数组传入，并自动解构参数
f(...[1, 2, 3]) === 6
```

☞ 更多信息: [Default parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters), [Rest parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters), [Spread Operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator)

### Let + Const

只在块级域中生效的变量声明。`const` 用来声明一个只读的常量，一经声明就无法修改值。

```js
function f() {
  {
    let x
    {
      const x = 'sneaky' // 起作用，因为是块级域的声明
      x = 'foo' // 赋值失败，x 是只读的
    }
    let x = 'inner' // 错误，`let` 无法重复声明
  }
}
```

☞ 更多信息: [let statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let), [const statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)

### Iterators + For…Of

迭代器对象的作用类似 [CLR](https://msdn.microsoft.com/zh-sg/library/ms131103.aspx) 的 `IEumberable` 和 Java 中的 `Iterable`。

```js
let fibonacci = {
  [Symbol.iterator]() {
    let pre = 0, cur = 1
    return {
      next() {
        [pre, cur] = [cur, pre + cur]
        return {
          done: false,
          value: cur
        }
      }
    }
  }
}
for (let n of fibonacci) {
  if (n > 1000) {
    break
  }
  console.log(n)
}
```

☞ 更多信息:[MDN for…of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of)

### Generators

生成器类似迭代器， 通过 `function*` 和 `yield` 关键词可以达到同样的作用。如果一个函数以 `function*` 声明那么将返回一个生成器实例。生成器是迭代器的子类型，所以它也有 `next` 和 `throw`。使用关键词 `yield` 能返回一个值(或抛出错误)。

☞ 同样也能使用 `await` 完成异步操作。

```js
let fibonacci = {
  [Symbol.iterator]: function*() {
    let pre = 0, cur = 1
    for (;;) {
      let temp = pre
      pre = cur
      cur += temp
      yield cur
    }
  }
}
for (let n of fibonacci) {
  if (n > 1000) {
    break
  }
  console.log(n)
}
```

☞ 更多信息: [MDN Iteration protocols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols)

### Unicode

增强对 Unicode 的支持，包括扩展字符串、正则以及 `codePointAt()`(返回一个字符的码点,方便处理4个字节存储的字符)

```js
// 同 ES5.1 结果相同
'𠮷'.length === 2
// 可以用正则匹配 `u` (unicode)
'𠮷'.match(/./u)[0].length === 2
// 字符的 Unicode 表示法 `\uxxxx`(`\u0000 - \uFFFF) or `\u{xxx}`
'\u{20BB7}' === '𠮷' === '\uD842\uDFB7'
// JavaScript 字符以 UTF-16 存储，每个字符固定2个字节
// 使用 codePointAt 方法可以正确处理以4个字节存储的字符
'𠮷'.codePonintAt(0) == 0x20BB7
for (let c of '𠮷') {
  console.log(c)
}
```

☞ 更多信息: [MDN RegExp.prototype.unicode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/unicode)

### Modules

增加了模块化支持，原先的解决方案主要有 AMD, CommonJS 两种模块加载方案。ES6 提出的模块化希望尽可能地静态化，在编译时就确定模块的依赖关系。当然也支持异步加载模块，在模块没有被加载完毕前就不会执行指定代码。

```js
// lib/math.js
export function sum(x, y) {
  return x + y
}
export const PI = 3.141593
```

```js
// app.js
import * as math from 'lib/math'
alert('2π = ' + math.sum(math.PI, math.PI))
```

```js
// otherApp.js
import { sum, PI } from 'lib/math'
alert('2π = ' + sum(PI,PI))
```

支持 `export default` 和 `export *`:

```js
// lib/mattplusplus.js
export * from 'lib/math'
export const E = 2.71828182846
export default function(x) {
  return Math.log(x)
}
```

```js
// app.js
import ln, { pi, E } from 'lib/mathplusplus'
alert('2π = ' + ln(E) * PI * 2)
```

☞ 更多信息: [import statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import), [export statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export)

### Module Loaders

模块加载器支持:

* 动态加载
* 隔离状态(变量、方法…)
* 隔离全局域空间
* 钩子
* 嵌套使用

可以配置默认的模块加载器:

```js
// 动态加载 - `System` 是默认的加载器
System.import('lib/math').then(function(m) {
  alert('2π = ' + m.sum(m.PI, m.PI))
})
// 给新的加载器创建沙盒
var loader = new Loader({
  global: fixup(window) // 替代 `console.log`
})
loader.eval('console.log("hello world!")')
// 直接设置模块缓存
System.get('jquery')
System.set('jquery', Module({$: $})) // 尚未定稿
```

### Map + Set + WeakMap + WeakSet

提供更高效的数据结构。WeakMap/WeakSet 提供弱引用的对象键值对。

```js
// Sets
var s = new Set()
s.add('hello').add('goodbye').add('hello')
s.size === 2
s.has('hello') === true
// Maps
var m = new Map()
m.set('hello', 42)
m.set(s, 34)
m.get(s) === 34
// Weak Maps
var wm = new WeakMap()
wm.set(s, { extra: 42 })
wm.size === undefined
// Weak Sets
// 因为 WeakSet 中的对象都是弱引用，如果其他对象不再引用该对象，就被垃圾回收了，该对象就在 Set 中消失了。
var ws = new WeakSet()
ws.add({ data: 42 })
```

☞ 更多信息: [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map), [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set), [WeakMap](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap), [WeakSet](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakSet)

### Proxies

Proxy 相当于在目标对象之外设置一层拦截，让 Proxy 来额外执行某些操作，可以理解为对象的代理器。

```js
// 代理一个普通对象
var target = {}
var handler = {
  get: function(receiver, name) {
    return `Hello, ${name}`
  }
}
var p = new Proxy(target, handler)
p.world === 'Hello, world'
// 代码一个函数对象
var target = function() {
  return 'I am a the target'
}
var handler = {
  apply: function(receiver, ...args) {
    return 'I am the proxy'
  }
}
var p = new Proxy(target, handler)
p() === 'I am the proxy'
// Proxy 支持的拦截操作一览:
var handler = {
  get: ...,
  set: ...,
  has: ...,
  deleteProperty: ...,
  apply: ...,
  construct: ...,
  getOwnPropertyDescriptor: ...,
  defineProperty: ...,
  getPrototypeOf: ...,
  setPrototypeOf: ...,
  enumerate: ...,
  ownKeys: ...,
  preventExtensions: ...,
  isExtensible: ...
}
```

☞ 更多信息: [MDN Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)

☞ 更多信息: [MDN Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)

### Symbols

使用 Symbol 来保证对象的每个属性的名字都是独一无二的，能够控制访问对象的状态。Symbols 允许属性的键可以是 `string` 或 `symbol`。Symbols 是新增加的原始数据类型(Undefined, Null, Boolean, String, Number, Object)。可选参数 `description` 可以用来 debug。Symbols 是独一无二的，可以通过 `Object.getOwnPropertySymbols` 获取指定对象的所有 Symbols 属性名。

```js
var MyClass = (function() {
  // 模块域的 symbol
  var key = Symbol('key')
  function MyClass(privateData) {
    this[key] = privateData
  }
  MyClass.prototype = {
    doStuff: function() {
      // ... this[key] ...
    }
  }
  return MyClass
})
var c = new MyClass('hello')
c['key'] === undefined
```

☞ 更多信息: [MDN Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)

### Subclassable Built-ins

在 ES6 中， 可以继承类似 `Array`, `Date` 以及 DOM 的 `Element` 等内建类型的 built-ins。

对象的构造如今分两个阶段执行:

* 调用 @@create 分配对象空间，执行一些特定行为
* 调用 constructor 创建一个新的实例

可以通过 `Symbol.create` 来调用 `@@create`。

```js
// 假冒的 Array
class Array {
  constructor(...args) {
    /* ... */
  }
  static [Symbol.create]() {
    // 可定义一些属性
  }
}
class MyArray extends Array {
  constructor(...args) {
    super(...args)
  }
}
// 创建一个 MyArray 有两个阶段:
// 1) 调用 `@@create` 来分配一个对象空间
// 2) 调用 `constructor` 来创建一个新的实例
var arr = new MyArray()
arr[1] = 12
arr.length === 2
```

### Math + Number + String + Array + Object APIs

扩展了许多库，包括核心的 Math 库，数组转换，字符串扩展，以及对象扩展。

```js
Number.EPSILON
Number.isInteger(Infinity) // false
Number.isNaN('NaN') // false
Math.acosh(3) // 1.762747174039086
Math.hypot(3, 4) // 5
Math.imul(Math.pow(2, 32) - 1, Math.pow(2, 32) - 2) // 2
'abcde'.includes('cd') // true
'abc'.repeat(3) // 'abcabcabc'
Array.from(document.querySelectorAll('*')) // 返回一个数组
Array.of(1, 2, 3) // 类似创建一个新的数组，不同的是 Array(2) 是创建一个长度为2的数组`[,,]` 而不是 [2]
[0, 0, 0].fill(7, 1) // [0, 7, 7]
[1, 2, 3].find(x => x === 3) // 3
[1, 2, 3].findIndex(x => x === 2) // 1
[1, 2, 3, 4, 5].copyWithin(3, 0) // [1, 2, 3, 1, 2]
['a', 'b', 'c'].entries() // iterator [0, 'a'], [1, 'b'], [2, 'c']
['a', 'b', 'c'].keys() // iterator 0, 1, 2
['a', 'b', 'c'].values() // iterator 'a', 'b', 'c'
Object.assign(Point, { origin: new Point(0, 0) }) // 对象合并
var target = { a: 1 }
var source = { b: 2 }
Object.assign(target, source) // {a:1, b:2}
```

☞ 更多信息: [Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number), [Math](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math), [Array.from](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from), [Array.of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/of), [Array.prototype.copyWithin](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/copyWithin), [Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)

### Binary and Octal Literals

对二进制和八进制的扩展

```js
0b111110111 === 503 // true
0o767 === 503 // true
```

### Promises

Promise 对象用来进行异步编程，替代以往的回调函数和事件。

```js
function timeout(duration = 0) {
  // pending 进行中
  // resolved 已完成
  // reject 已失败
  return new Promise((resolve, reject) => {
    setTimeout(resolve, duration)
  })
}
var p = timeout(1000).then(() => {
  return timeout(2000)
}) .then(() => {
  throw new Error('hmm')
}).catch(err => {
  return Promise.all([timeout(100), timeout(200)])
})
```

☞ 更多信息: [MDN Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

### Reflect API

Reflect 对象类似 Proxy 对象，暴露一些运行时级别的对象操作方法。允许调用 Proxy 调用的那些操作。可以用来修改一些 Object 方法的返回结果。

```js
// 修改返回结果
try {
  Object.defineProperty(target, property, attributes)
  // success
} catch (e) {
  // failure
}
if (Reflect.defineProperty(target, property, attributes)) {
  // success
} else {
  // failure
}
// 让命令式操作变成函数式操作
'assign' in Object // true
Reflect.has(Object, 'assign') // true
```

☞ 更多信息: [MDN Reflect](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect)

### Tail Calls

尾调用保证不会产生栈溢出，确保递归调用的安全性。

```js
function factorial(n, acc=1) {
  'use strict'
  if (n <= 1) return acc
  return factorial(n - 1, n * acc)
}
factorial(10000)
```

---
参考:

- [Fail-Soft System](http://www.computerhope.com/jargon/f/failsoft.htm)
- [https://github.com/lukehoban/es6features/](https://github.com/lukehoban/es6features/)
- [http://es6.ruanyifeng.com/](http://es6.ruanyifeng.com/)

