---
title: 如何优化你的 JS 代码？
date: 2023-03-13 22:53:27
tags:
top:
---

我对代码有洁癖，喜欢代码简写优化 (在保证可读性和可维护性的前提下)

正好最近看到一篇文章，觉得对新手很有帮助，就翻译给下

希望能对大家有所帮助 ｜ [原文](https://www.freecodecamp.org/news/simplify-javascript-code/)

---

作为一个程序员，写出简洁和可维护的代码是我们的目标。但有时候，当我们在一个复杂、难以管理的大型项目时，便难以实现

`简化代码` 是避免这种情况的方法之一，这个方法可以来帮助提高代码的可读性、效率和可维护性

这篇文章将讨论简化 `JS` 代码的 10 种方法，使其变得更具可读性和可维护性，让我想现在开始吧！

## 一、使用箭头函数

箭头函数是 JavaScript 中创建函数的一种简单方法，他们通过减少定义一个函数所需的模版来简化你的代码

例如，使用 `function` 关键字来定义函数是这样的：

```js
function sayHello(name) {
  console.log(`Hello, ${name}!`)
}
```

使用箭头函数，你可以这样写：

```js
const sayHello = name => console.log(`Hello, ${name}!`)
```

除了箭头函数具有更简短的语法外，它可以使你的代码更简洁、更具可读性和减少出错。这使它成为比使用 `function` 关键字更好的选择

## 二、使用具有描述性的变量名

使用具有描述性的变量名可以使你的代码更具可读性，更容易理解。这比使用单字母的缩写要好得多，因为其他人在阅读你的代码时可能不会立即明白这些变量的含义。

不使用：

```js
const x = 10
```

使用：

```js
const numberOfItems = 10
```

`numberOfItems` 比 `x` 更有描述性，可以帮助你（或其他查看你代码的开发者）理解它在做什么。

## 三、使用函数式编程

函数式编程优先考虑使用纯函数和不可变的数据结构。使用函数式编程技术可以帮助大大简化你的代码，减少 bug 和副作用的风险。

例如，不要在原地修改一个数组：

```js
const numbers = [1, 2, 3]
arr.push(4)
```

你可以使用扩展运算符来创建一个新数组：

```js
const numbers = [1, 2, 3]
const newNumbers = [...numbers, 4]
```

使用扩展运算符可以帮助你防止意外的副作用，使你的代码更好预测和推理。

当你在原地修改函数时，你改变了原来的数组或对象，如果你代码的另一部分依赖于该数组或对象，那么就会导致 bug 和意想不到的行为。

另一方面，使用扩展运算符会创建一个新的数组或对象，而保留原有的数组或对象。这使得你的代码更容易预测和推理。

## 四、避免嵌套代码

嵌套代码会使其难以阅读和理解，一个更好的方法是尽可能地使你的代码扁平化。你可以通过使用提前 `return`、三元运算符和函数组合来做到这一点。

例如，不要 if 嵌套：

```js
if (condition1) {
  if (condition2) {
    // code
  }
}
```

使用提前 `return`：

```js
if (!condition1) return
if (!condition2) return
// code
```

在这里使用提前 `return` 使我们的代码更容易阅读和理解，因为它将每个条件分解成一个单独的 if 语句，并在任何条件不成立时提前返回。

早期返回也可以通过防止不必要的计算来提高你的代码的效率。

## 五、使用默认参数

使用默认参数允许你为一个函数参数指定一个默认值。这可以通过减少你需要编写的条件语句的数量来简化你的代码。

例如，不要使用条件逻辑来设置一个默认值：

```js
const greet = name => {
  if (!name) {
    name = 'World'
  }
  console.log(`Hello, ${name}!`)
}
```

你可以使用默认参数来设置一个默认值：

```js
const greet = (name = 'World') => console.log(`Hello, ${name}!`)
```

默认参数为你提供了一个简单的方法来设置参数的默认值。但不仅如此，它使你的代码更加灵活，不容易出错，也使你的代码更容易理解。

## 六、使用解构赋值

解构允许你从数组和对象中提取值并将其分配给变量。这样做可以使你的代码更简明，更容易阅读。

例如，不要像这样直接访问对象属性：

```js
const person = { name: '张三', age: 18 }
const name = person.name
const age = person.age
```

使用解构赋值：

```js
const { name, age } = { name: '张三', age: 18 }
```

使用解构赋值会比访问对象属性好得多，因为它可以帮助你快速了解代码的目的，特别是在处理复杂的数据结构时。它还有助于减少你需要编写的代码量，提供灵活性，使代码更简洁，也有助于避免命名冲突

## 七、使用 Promise

`Promise` 允许你以一种更可读和可预测的方式编写异步代码。它们通过避免回调和使你能够将异步操作连在一起来简化你的代码。

例如，不要使用回调的嵌套：

```js
const getUserData = (userId, callback) => {
  getUser(userId, user => {
    getPosts(user, posts => {
      getComments(posts, comments => {
        callback(comments)
      })
    })
  })
}
```

你可以使用 `Promise`：

```js
const getUserData = userId =>
  getUser(userId)
    .then(user => getPosts(user))
    .then(posts => getComments(posts))
```

使用 `Promise` 而不是回调嵌套可以使代码更简洁，更容易阅读，特别是在处理复杂的异步操作时。

## 八、使用数组方法

`JavaScript`有许多操作数组的内置方法，如 `map`、`filter`、`reduce` 和 `forEach`。使用这些方法可以使你的代码更简明，更容易阅读。

例如，不要使用 `for` 来遍历一个数组：

```js
const numbers = [1, 2, 3]
for (let i = 0; i < numbers.length; i++) {
  console.log(numbers[i])
}
```

你可以使用 `forEach`：

```js
const numbers = [1, 2, 3]
numbers.forEach(number => console.log(number))
```

使用数组方法而不是传统的 `for循环` 可以使你的代码更加简洁、可读和模块化，同时还可以提供更好的错误处理并支持函数式编程。

## 九、使用对象方法

`JavaScript` 对象提供了各种内置方法，如 `Object.key`、`Object.values` 和 `Object.entry`。这些方法可以减少对循环和条件的需要，使你的代码更简洁。

例如，不要用 `for` 来循环一个对象：

```js
const person = { name: '张三', age: 18 }
for (const key in person) {
  console.log(`${key}: ${person[key]}`)
}
```

你可以使用 `Object.entries` 方法：

```js
const person = { name: 'John', age: 30 }
Object.entries(person).forEach(([key, value]) => console.log(`${key}: ${value}`))
```

和数组方法一样，使用对象方法可以使你的代码更加简洁、可读和模块化。

## 十、使用库和框架

`JavaScript` 有各种各样的模块和框架，可以帮助你减少创建模版文件，写更简单的代码。

例如，用于构建用户界面的 `Vue、React`，实用工具类的 `Lodash`，以及用于处理日期和时间的 `day.js`。

你可以考虑在以下情况下使用一个框架/库：

- 当你想建立一个复杂的应用时，其功能可以通过库或框架来实现。
- 当你有一个紧张的时间进度，需要快速交付你的项目时候。
- 当你想提高你的代码质量并减少长期的维护成本时。

另一方面，你以下情况下尽可能避免使用框架/库：

- 当你的项目要求很简单，不需要外部工具。
- 当你想控制你的代码并避免对外部工具的依赖时。

## 结束语

使用简化代码的优化方式可以使你的代码可读性和可维护性更高。通过遵循这些建议，你可以写出更好理解和更高效的代码。

你还有什么建议？如果你觉得有帮助，别忘了分享。

---

感谢阅读，下次见 :)
