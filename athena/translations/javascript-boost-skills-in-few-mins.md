# JavaScript Boost Skills in Few Minutes

<!-- TOC -->

- [JavaScript Boost Skills in Few Minutes](#javascript-boost-skills-in-few-minutes)
    - [Spread operator](#spread-operator)
    - [for...of iterator](#forof-iterator)
    - [Includes() method](#includes-method)
    - [Some() method](#some-method)
    - [Every() method](#every-method)
    - [Filter() method](#filter-method)
    - [Map() method](#map-method)
    - [Reduce() method](#reduce-method)

<!-- /TOC -->

> 原文：[These JavaScript methods will boost your skills in just a few minutes](https://medium.freecodecamp.org/7-javascript-methods-that-will-boost-your-skills-in-less-than-8-minutes-4cc4c3dca03f)

我们现如今的应用程序会用上很多数据结构，处理这些数据结构也是我们的日常操作之一。暂时忘了传统的
`for-loop`(`let i=0; i < value.length; i++`) 处理方式吧。

> 在 `for-loop`循环中使用 `const` 关键字会出错，原因是这个关键字声明的变量不能重新赋值，
> 那么当变量 `i` 进行 `i++` 操作时就会出错。所以当你纠结是用 `let` 还是 `const` 声明变量时，
> 先思考该变量值是否会重新赋值？如果会，那么应该用 `let`，如果不会，那么应该用 `const`。
> ([查看更多](https://stackoverflow.com/questions/41067790/why-does-const-work-in-some-for-loops-in-javascript))

假设我们想要获取 products/categorize 的内容，或者过滤、查询、修改、更新一个集合，又或者想要
计算一个集合比如求和或求积，那么最优的解决方法是什么呢？

可能你不太喜欢箭头函数(arrow functions)，可能你不喜欢花太多时间学习新东西，又或者他们不太适合。
不用担心，下面会用 ES5(functional deceleration) 和 ES6(arrow functions)各自实现。

注意：箭头函数和普通函数并不等价，而且某些情况下不能随意替换([相关](https://stackoverflow.com/questions/34361379/arrow-function-vs-function-declaration-expressions-are-they-equivalent-exch?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa))。
要注意 `this` 的区别。

我们下面会涉及到如下方法：

1. Spread operator
2. for...of iterator
3. includes()
4. some()
5. every()
6. filter()
7. map()
8. reduce()

## Spread operator

扩展运算符 `...` 可以将数组分解成一个个元素，也能操作对象。

**为什么要用它？**

- 可以既简单又快速地获取数组元素
- 数组和对象都能用
- 传递参数时非常快速又方便
- 只需要三个点...

**例子：**

假设我们不借助循环来获取一系列最喜爱的食物，通过扩展运算符看起来是这样的：

``` js
const favoriteFood = ['Pizza', 'Fries', 'Swedish-meatballs'];

console.log(...favoriteFood);
// Pizza Fries Swedish-meatballs
```

## for...of iterator

`for...of` 语句可以循环一个集合，并能够修改集合元素。有时可以替代 `for-loop`。

**为什么要用它？**

- 增加或修改元素很简单
- 计算操作（求和、求积等）
- 当使用条件语句时（`if`, `while`, `switch`...）
- 更简洁、易阅读

**例子：**

假设你有一个工具箱，你想要展示你所拥有的工具，使用 `for...of` 非常简单：

``` js
const toolBox = ['Hammer', 'Screwdriver', 'Ruler'];

for (const item of toolBox) {
  console.log(item);
}

// Hammer
// Screwdriver
// Ruler
```

## Includes() method

`includes()` 方法用来确认一个集合是否包含指定字符串，返回结果为 `true` 或 `false`。
这个方法是大小写敏感的，如果你查询的元素为 `school`，而集合包含的元素为 `SCHOOL`，那么会返回 `false`。

**为什么要用它？**

- 更简单的查找功能
- 如果想要判断一个字符串是否存在那么它很合适
- 它采用条件语句来修改过滤
- 更易阅读

**例子：**

假设你出于某种原因忘记你车里库有什么汽车，并且你需要一个系统来帮你确认是否拥有指定的汽车，那么就是它了：

``` js
const garage = ['BMW', 'AUDI', 'VOLVO'];

const findCar = garage.includes('BMW');

console.log(findCar);
// true
```

## Some() method

`some()` 方法用来确认一个数组是否包含某些元素，返回结果为 `true` 或 `false`。
它和 `includes` 有点相似，不同点在于 `some()` 的参数为一个函数，而 `includes()` 的参数为一个字符串：

**为什么要用它？**

- 确保某些(**Some**)元素符合条件
- 通过传递函数来条件过滤
- 更语义化
- 一些(**Some**)就足够好

**例子：**

假设你是一个俱乐部的负责人，你并不关心都有谁进到你的俱乐部，但是有一部分人是你不允许的，因为他们经常喝多了:)。
下面是两种不同的实现方式：

ES5:

``` js
const age = [16, 14, 18];

age.some(function (person) {
  return person >= 18;
});

// true
```

ES6:

``` js
const age = [16, 14, 18];

age.some((person) => person >= 18);

// true
```

## Every() method

`every()` 方法循环一个数组，并且检查每一个数组元素，返回结果为 `true` 或 `false`。
部分内容和 `some()` 类似，不同的是 `every()` 函数要确保每一个元素都符合条件，否则就返回 `false`。

**为什么要用它？**

- 确保每一个(**Every**)元素都符合条件
- 通过传递函数来条件过滤
- 更语义化

**例子：**

假设上一次你允许了一些未成年人进入你的俱乐部，结果有人把你给举报了。为了不让这事再次发生，
你决定必须每个人都通过你限制的年龄条件才行：

ES5:

``` js
const age = [15, 20, 19];

age.every(function (person) {
  return person >= 18;
});

// false
```

ES6:

``` js
const age = [15, 20, 19];

age.every((person) => person >= 18);

// false
```

## Filter() method

`filter()` 函数会筛选出所有符合条件的元素，并返回一个新的数组。

**为什么要用它？**

- 避免修改原数组
- 过滤掉你不需要的元素
- 更易阅读

**例子：**

假设你只想要获取高于30的价格，过滤掉其他不符合条件的价格：

ES5:

``` js
const prices = [25, 30, 15, 55, 40, 10];

prices.filter(function (price) {
  return price >= 30;
});

// [ 30, 55, 40 ]
```

ES6:

``` js
const prices = [25, 30, 15, 55, 40, 10];

prices.filter((price) => price >= 30);

// [ 30, 55, 40 ]
```

## Map() method

`map()` 函数的功能有点类似 `filter()` 函数，都返回一个新的数组。不同之处在于，`map()` 函数用来修改元素。

**为什么要用它？**

- 避免修改原数组
- 你可以修改元素
- 更易阅读

**例子：**

假设你有一堆产品的价格，你想要获取加上25%税的价格：

ES5:

``` js
const productPriceList = [200, 350, 1500, 5000];

productPriceList.map(function (item) {
  return item * 0.75;
});

// [ 150, 262.5, 1125, 3750 ]
```

ES6:

``` js
const productPriceList = [200, 350, 1500, 5000];

productPriceList.map((item) => item * 0.75);

// [ 150, 262.5, 1125, 3750 ]
```

## Reduce() method

`reduce()` 函数可以将数组元素转为整数、对象或链式 promise。一个简单的例子就是一系列整数求和，
整个数组会被 `reduce` 为一个总和。

**为什么要用它？**

- 计算操作
- 计算一个值
- 重复利用统计值
- 根据对象属性分类
- 链式执行
- 计算操作很快...

**例子：**

假设你想要统计你一周的开支，`reduce()` 函数就是来干这事的：

ES5:

``` js
const weeklyExpenses = [200, 350, 1500, 5000, 450, 680, 350];

weeklyExpenses.reduce(function (first, last) {
  return first + last;
});

// 8530
```

ES6:

``` js
const weeklyExpenses = [200, 350, 1500, 5000, 450, 680, 350];

weeklyExpenses.reduce((first, last) => first + last);

// 8530
```
