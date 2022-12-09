---
title: 更好地认识「Promise」
date: 2021-10-21 1:40:36
tags:
top:
---

> 关于 `Promise` 相关的 `使用`，语言尽量`通俗易懂`，`减少废话`，如果想了解更深层次，去看隔壁看的博客/文档，或许会更好

这篇博客将是一个合集开始，会持续以 `通俗易懂` `减少废话` 的形式，来简述 Promise 的 `基本用法`，如果想更 `深层` 地研究 Promise，那么我的博客将不适合你阅览。

---

# Promise

首先要明白：前端向 `服务器发送请求` 会干嘛？\
显然会 `得到响应`\
那么：得到响应是 `需要一定的时间`\
明白之后，在「**怎么做**」部分就能更好的理解

## Promise 是什么

- 解决异步编程的一种方案：`异步 => 同步`，比传统的 `回调函数和事件` 更合理
- 原先于 `1976 年`，由 Daniel P.Friedman 和 David Wise 提出 `Promise 思想`
- 前端结合 Promise 和 JS，制定了 [Promise/A+ 规范](https://promisesaplus.com/)
- `ES6` 将其写进了 `语言标准`

## 为什么要用 Promise

- 因为好用啊，哪还有为什么？
- 因为之前解决异步的方法 `不够规范`、`回调地狱`、`很难进行错误处理`

## 怎么做

具体看 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 或 `阮一峰` 的博客

我这里做一个 `面向现实编程`，跟着我的思路，你就能看懂\
PS：更深的原理，请看 `MDN`

### 动手:「红绿灯」

1. 我们做一个交通信号灯，排除停电，信号灯`始终只处于3种状态其中之一`：红灯、绿灯、黄灯；那再简单点，把黄灯排除掉，现在还有哪些灯？自己用幼儿园数学算算！
2. 那我们就实现这个信号灯：
3. 因为我们需要判断信号灯处于哪个状态，由于只有两种状态：`非绿即红`，那我们声明一个变量 `isGreenLight` 判断是否为 `绿灯`，默认为 `true`（绿灯）
4. 因为 Promise 是一个构造函数，那我们首先需要 `new` 一个 `Promise`，里面执行一个函数，该函数有两个参数，分别为 `resolve (等待 => 成功)` 和 `reject (等待 => 失败)`
5. 接着判断红绿灯的状态，如果为绿灯，那么返回 `resolve` 绿灯，否则返回 `reject` 红灯
6. 调用 `promise`，先设置成功的返回结果，就是 `.then`然后传入一个函数，这个函数接受一个 `参数`，这个 `参数` 就是前面 `resolve` 保留的参数
7. 再 `.catch`，里面也传入一个函数，函数同样接受一个 `参数`，这个 `参数` 就是前面 `reject` 保留的参数
8. 最后 `.finally` 传入一个函数，返回 `最终的结果` (我们这里要研究的最终结果是 `promise 执行了`)

```js
let isGreenLight = true
const promise = new Promise((resolve, reject) => {
  isGreenLight ? resolve('绿灯') : reject('红灯')
})

promise
  .then(light => {
    console.log(`现在交通灯为${light}，你可以大胆往前走了`)
  })
  .catch(light => {
    console.log(`现在交通灯为${light}，你想走？那祝你好运！`)
  })
  .finally(() => {
    console.log('但是最终 promise 还是执行了，灯也亮了')
  })

// 现在交通灯为绿灯，你可以大胆往前走了
// 但是最终 promise 还是执行了，灯也是亮着的
```

### ajax 请求

还有另外一种写法：\
`.then(success, fail)`\
也可以不用 `.catch`，直接 `.then(传入的第一个函数是成功函数，第二个函数是失败函数)`\

我们来执行 `AJAX 请求` (假设已经 `new` 了 `Promise`)

```js
ajax('get'. '/xxx')
  .then((response) => {}, (request) => {})

// .then(success, fail)
// 也可以不用 catch，直接.then(传入的第一个函数是成功函数，第二个函数是失败函数)
```

### 小结

- 第一步
  - `return new Promise((resolve, reject) => { ... })`
  - 成功则调用 `resolve(result)`
  - 失败则调用 `reject(error)`
  - `resolve` 和 `reject` 会再去调用成功和失败函数
- 第二步
  - 使用 `.then(success, fail)` 传入成功和失败函数
- Stop！
  - 还有其他更高级的用法，可能新手会觉得这个 `Promise` 会不好理解，这时候需要停止研究，请先熟悉它！

## 优点

1. `统一` 异步 API
2. 支持 `链式调用`，解决 `回调地狱`
3. 更好的 `错误处理`

## 缺点

1. 无法取消
2. 处于 `Pending` 时，无法得知当时是在哪个阶段

## 如何解决缺点

- 别用，咋想的

如果你很 `硬核` 想研究 `Promise 源码`，那么请 [点击这里](https://github.com/FrankFang/promise-2/blob/master/src/promise.ts)\
注：但这不是给新手看的，我没看，我不敢看

感谢阅读，下次见 :)
