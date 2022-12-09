---
title: 坑爹的JS：立即执行函数
date: 2022-1-16 00:00:21
tags:
top:
---

> 想一个东西：在 ES6 之前，想创建一个 `局部变量` 要怎么创建？\
> 诶？好像不行哦？ \
> 聪明的程序员就出现了，所以有了立即执行函数

---

## 是什么

声明一个匿名函数，然后立即执行\
就种做法，就是立即执行函数

注意：立即执行函数，指的不是这个函数，而是声明一个函数在执行的这个过程动作叫立即执行函数

> 有趣的过程（可忽略）

```js
function a() {
  console.log('hi')
}
a() // 这其实就是立即执行函数，诶？那这跟普通的声明一个函数然后调用它有啥区别吗？没区别

// 那我变形，把名字给去掉：
function () {
  console.log('hi')
}() // 但是会报错

// 语法有问题，那我这么写就不报错了，这就是匿名执行函数
(
  function () {
    console.log('hi')
  }()
)

// 但是会有 bug，比如在后面加 map，会报错
(
  function () {
    console.log('hi')
  }()
)
[1, 2, 3].map(item => console.log(item))

// 因为 函数里没返回值，返回值是 return undefined
// 浏览器就会看成是：
undefined[1, 2, 3].map(item => console.log(item))

// 当我们有多个值并列的时候，逗号表达式的值就是最后一个东西的值，所以是 3，所以上面代码会被浏览器理解为：
undefined[3].map(item => console.log(item))

// 所以，终极方法：在立即执行函数后面加分号
(
  function () {
    console.log('hi')
  }()
);
[1, 2, 3].map(item => console.log(item))
// 这样就不会往下看了
// 写立即执行函数最安全的方法：加括号再加分号
```

如果上面写法丑，那可以在函数前面加感叹号，结尾加分号：\
`其实前面加：+ - ~ 都行，是用来取反，(return 别让它是 undefined 嘛...)， ~ 是二进制的取反`

```js
!(function () {
  console.log('hi')
})()
;[1, 2, 3].map(item => console.log(item))
```

## 怎么做

```
(function(){console.log('我是匿名函数')} ())	// 用括号把整个表达式包起来
(function(){console.log('我是匿名函数')}) ()	// 用括号把函数包起来
!function(){console.log('我是匿名函数')} ()	// 取反，我不在意值是多少，我只想通过语法检查
+function(){console.log('我是匿名函数')} ()
-function(){console.log('我是匿名函数')} ()
~function(){console.log('我是匿名函数')} ()
void function(){console.log('我是匿名函数')} ()
new function(){console.log('我是匿名函数')} ()
var a = function(){console.log('我是匿名函数')} ()
```

## 解决了什么问题

为了在 ES6 之前，创建一个`「局部作用域」` / `「局部变量」`\
ES6 可以这样创建局部变量：

```js
{
  let a
}
```

## 优点

兼容性好 (ES3 都能支持)

## 缺点

丑。\
我只想创建一个局部变量还要花里胡哨写这么一堆东西

## 怎么解决

使用 block + let 语法

```js
{
  lat a = '我是局部变量'
  console.log(a)	// 能读取 a
}
console.log(a)	// 不能读取 a
```

说实话，这个立即执行函数也挺无语的\
挺坑爹的吧：想正常创建一个局部变量都不行

感谢阅读，下次见 :)
