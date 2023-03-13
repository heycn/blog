---
title: 用人话阐述 JS 中的「闭包」
date: 2022-03-02 13:49:19
tags:
top:
---

> 网上许多文章解释得太官方了，很简单的东西讲得很难理解
>
> 当做唠嗑吧

适合初学者阅读。

---

先明白这么个东西：对于一个函数来说，变量分为「全部变量」、「局部变量/本地变量」、「自由变量」

## 是什么

闭包是 JS 的一种`「语法特性」`

> 闭包 = 函数 + 自由变量

或者：\
一个函数`访问了函数外的变量`，就形成了闭包

JS 所有函数都支持闭包

## 怎么做

### 答案：

- 声明一个变量等于一个立即执行函数
- 然后立即执行函数里面声明一个变量 count
- 再声明一个函数对外面的 count 进行操作

```js
const add1 = function () {
  var count
  return function add2() { // 访问了外部变量的函数
    count += 1
  }
}
```

或者这么理解：

```js
let count
function add() { // 访问了外部变量的函数
  count += 1
}
```

把上面代码放在「非全局环境」里，就是闭包

闭包不是 count，也不是 add，而是 count + add 组成的整体

---

### 思路（可忽略）：

```js
var count
function add() {
  count += 1
}
// 这个 count 是全局变量，那函数本来就可以访问全部变量的呀？
// 所以这并不是一个闭包，这是全局变量的特性
```

那怎么做到是一个自由变量呢？\
那就让它不是全局变量就行了呗！

```js
{
  let count
  function add() {
    count += 1
  }
}
// 现在 count 它是一个局部变量，不是全局变量了
// 所以这才是真正的闭包
```

再说说什么是自由变量：

- 看以上代码
- count 既不是全局变量，也不是 add 函数 的局部变量
- 那么 count 就是 add 函数 的自由变量

那么这个时候 add 再去访问 count，那就形成了闭包

## 解决了什么问题

那这只是个语法特性啊，有什么用呢？\
光语法特性当然没有什么用，重点看在什么场景去用它

### 举个例子：

- 假设我们现在要做一个游戏，叫魂斗罗
- 那么魂斗罗每人有 3 条命，那声明一个全部变量(3 条命)

```js
window.lives = 3
/*
这里写游戏代码
*/
```

- 那这个游戏代码还能去写吗？不可以
- 如果你不小心写了 `window.lives = -100`，那就完了？你的游戏就有 BUG 了！
- 所以你把 lives 作为一个全部变量是很危险的，所以我们需要把它 '藏' 在一个地方，然后只让他加一条命或减一条命，这样会更安全一点
- 那要怎么做？那就搞一个局部变量呗。怎么做呢？

```js
{
  let lives = 3
}
/*
这里是游戏代码
lives = -100
*/
```

- 这样你就访问不到 lives 了，你游戏代码改成 -100，没用！
- 但这我也访问不了，我想知道我现在有几条命，也不知道呀，怎么办？
- 给几个接口嘛，我写几个接口：获取生命，减少生命，增加生命

```js
{
  let lives = 3
  window.getLives = function () { return lives } // 获取生命
  window.die = () => { lives -= 1 } //减少生命
  window.award = () => { lives += 1 } // 增加生命
}
/*
这里是游戏代码
*/
// 诶？这不就是形成闭包了嘛？
// 但是重点我们不是这个闭包，而是获取这些接口，那怎么获取呢？看下面

window.getLives()

// 所以实现了什么效果：你不能直接访问 lives，但可以使用我给你提供的接口来访问 lives
```

至此，我们就实现了一个完整的「闭包引用」，也是闭包的作用

### 解决了什么问题：

1.  避免污染全局环境/隐藏变量（因为我用的是局部变量，用词真有意思...）
1.  提供对局部变量的间接访问/暴露访问器（因为上面代码只能 count += 1，不能 count -= 1）
1.  维持变量，使其不被垃圾回收（不被回收不是挺正常的事吗...）

## 优点

简单，好用\
不需要学习新知识，新语法

## 缺点

闭包 `使用不当` 可能造成内存泄漏\
绝对不是：闭包会造成内存泄漏，而是「使用不当」会造成内存泄漏

「闭包造成内存泄漏」这句话好多文章都有说过，这是曾经旧版本 IE 的 bug 导致的问题。。。

### 举例说明：

```js
function test() {
  var x = { name: 'x' }
  var y = { name: 'y', content: '...很长很长的内容...' }
  return function fn() {
    return x
  }
}

const myFn = test() // myFn 就是 fn 了
const myX = myFn() // myX 就是 x 了
// 请问，y 会消失吗？
```

对于一个正常的浏览器来说，y 会在**一段时间后**自动消失（被垃圾回收器给回收掉）\
但旧版本的 IE 并不是正常的浏览器，所以是 IE 的问题

只是有些浏览器对闭包的支持不够好，但是绝对不是「闭包造成内存泄漏」，好吧？

## 怎么解决缺点

慎用，少用，不用（既然有些浏览器对闭包支持不够友好，那我就少用呗）\
但我觉得闭包是没有缺点的，如果有，那就是浏览器的问题

最后推荐一篇方大的文章：\
[《JS 中的闭包是什么？》—— 方应杭](https://zhuanlan.zhihu.com/p/22486908)

完。

感谢阅读，下次见 :)