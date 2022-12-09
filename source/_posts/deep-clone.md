---
title: 手写「深拷贝」的心路历程
date: 2021-12-09 02:01:21
tags:
top:
---

跟着思路，从 0 开始，写出一个深拷贝吧！

## 雏形

实现深拷贝，有这个 `深` 字，我们就想到需要使用递归，但是我们先把 `数据类型` 给判断了吧！

写深拷贝，我们必须知道 JS 的数据类型：

### 先明白 JS 类型

- string
- number
- bool
- null
- undefined
- symbol
- bigint
- object

### 判断数据类型

- 这 8 种里有个明显的分割线：`对象` 和 `其他的 7 种` 是不一样的
- 如何判断？如果不考虑 iframe，那就用 `instanceof` 判断
- if(a 是 Object 的`实例`){ 复杂类型`Object` } else { 基本类型 `string` `number` `bool` `null` `undefined` `symbol` `bigint` }
- 判断对象的类：
  - 函数
    - 普通函数
    - 箭头函数
  - 数组
  - 日期
  - 正则
- 最后循环，递归/调用深拷贝（自己调自己）
- 那按照思路开始写吧！

```js
const deepClone = a => {
  if (a instanceof Object) {
    // 判断类型（不考虑 iframe
    let result // 拷贝的结果
    if (a instanceof Function) {
      // 如果是函数，则需要判断普通函数和箭头函数
      if (a.prototype) {
        // 有 prototype 则是普通函数，则返回一个函数
        // 传 this 给我，我就传 this 给 a，传什么参数给我，我就传什么参数给 a，然后把 a 的返回值作为我的返回值
        // 这样我的输入和输出和 a 一模一样，所以我是 a 的深拷贝
        result = function () {
          return a.apply(this, arguments)
        }
      } else {
        // 如果是箭头函数，则返回一个箭头函数，
        // 传给我什么参数，我就把什么参数传给 a，箭头函数没有 this，则返回undefined，把 a 的返回值作为我的返回值
        result = (...args) => {
          return a.call(undefined, ...args)
        }
      }
    } else if (a instanceof Array) {
      // 如果是数组，则返回一个空数组
      // 数组的内容由下面递归拷贝
      result = []
    } else if (a instanceof Date) {
      // 如果是日期，则返回一个Date
      // 日期的内容为传给我的时间戳（把日期减0就会得到数字，这个数字是时间戳）或者 ISO 8601 字符串
      result = new Date(a - 0)
    } else if (a instanceof RegExp) {
      // 如果是正则，则返回一个正则
      // 正则的内容为传给我的 source 和 flags
      result = new RegExp(a.source, a.flags)
    } else {
      // 其他的则是普通对象，则返回一个空对象
      // 对象内容有下面递归拷贝
      result = {}
    }
    for (let key in a) {
      // 递归：把 a 的所有子属性再深拷贝一遍
      result[key] = deepClone(a[key])
    }
    return result
  } else {
    // 不是对象则是：string number bool null undefined symbol bigint
    // 则直接返回 a
    return a
  }
}
```

这样，一个深拷贝就写完了！\
但是，这个代码还有问题\
如果 `a.self = a` 那么就会在递归里出不来\
那么需要检查 `环` - 检查 `循环引用`

## 检查「环」

- 上面说了，可能会在递归里出不来
- 需要检查拷贝的东西是不是曾经拷贝过了，如果拷贝过，我就不再拷贝，直接返回已经被我拷贝过的东西
- 那我们需要一个 Map，为什么是使用 Map 而不是对象呢？
- 因为我们的 key 是一个对象，那么他的 key 就是 String 或者 Symbol
- 所以需要一个 Map 来映射：只要我曾经拷贝过了，我就不会再拷贝第二次
- 开始写吧：

```js
const cache = new Map() // 定义缓存
const deepClone = a => {
  // 在深拷贝的时候检查一下
  if (cache.get(a)) {
    // 如果发现 a 已经存在
    // 就直接返回已经被我拷贝的东西
    return cache.get(a)
  }
  if (a instanceof Object) {
    let result
    if (a instanceof Function) {
      if (a.prototype) {
        result = function () {
          return a.apply(this, arguments)
        }
      } else {
        result = (...args) => {
          return a.call(undefined, ...args)
        }
      }
    } else if (a instanceof Array) {
      result = []
    } else if (a instanceof Date) {
      result = new Date(a - 0)
    } else if (a instanceof RegExp) {
      result = new RegExp(a.source, a.flags)
    } else {
      result = {}
    }
    // 构造出 result 之后往 cache 放东西
    cache.set(a, result)
    for (let key in a) {
      result[key] = deepClone(a[key])
    }
    return result
  } else {
    return a
  }
}
```

写的不错，请问还有什么问题？\
请看下面

## 不拷贝原型

- 我们在遍历 a 的时候，不能什么都遍历，有些属性是继承得到的，如果是继承得到的，我们不应该去深拷贝它
- 需要用 `hasOwnProperty` 来判断是不是自己拥有

```js
const cache = new Map()
const deepClone = a => {
  if (cache.get(a)) {
    return cache.get(a)
  }
  if (a instanceof Object) {
    let result
    if (a instanceof Function) {
      if (a.prototype) {
        result = function () {
          return a.apply(this, arguments)
        }
      } else {
        result = (...args) => {
          return a.call(undefined, ...args)
        }
      }
    } else if (a instanceof Array) {
      result = []
    } else if (a instanceof Date) {
      result = new Date(a - 0)
    } else if (a instanceof RegExp) {
      result = new RegExp(a.source, a.flags)
    } else {
      result = {}
    }
    cache.set(a, result)
    // 细节在这里，我们在遍历 a 的时候，不能什么都遍历，有些属性是继承得到的，如果是继承得到的，我们不应该去深拷贝它
    for (let key in a) {
      // 如果 key 在 a 的身上，我才会拷贝到 result，如果不在，则是在原型，在原型上我就不拷贝
      if (a.hasOwnProperty(key)) {
        result[key] = deepClone(a[key])
      }
    }
    return result
  } else {
    return a
  }
}
```

深拷贝还是很考察基础知识的

感谢阅读，下次见 :)
