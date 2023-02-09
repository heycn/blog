---
title: 「编程技巧」—— 链式编程
date: 2023-02-09 17:57:40
tags:
top:
---

分享几个 JavaScript 编程技巧，包含 `函数式编程范式` 和 `链式编程`

需求：\
输入 `['avatar.jpg ', '.gitignore', '    ', 'index.js']`\
输出 `['~/app/avatar.jpg', '~/app/.gitignore', '~/app/index.js']`

请看以下 3 种实现方式

[代码仓库](https://github.com/heycn/javascript-hotpot/blob/master/methodChaining/index.js)

## 一、**for of**

1. 使用 `for of` 获取数组中的字符串，去除空格
2. 判断去除空格之后是不是为空字符串
3. 如果不是为空字符串，就将这些字符串进行拼接并 `push` 进一个数组
4. `return` 这个数组

```js
const files = ['avatar.jpg ', '.gitignore', '    ', 'index.js']

const forLoops = files => {
  const result = []
  for (const file of files) {
    const fileName = file.trim()
    if (fileName) {
      const filePath = `~/app/${fileName}`
      result.push(filePath)
    }
  }
  return result
}

console.log(forLoops(files)) // [ '~/app/avatar.jpg', '~/app/.gitignore', '~/app/index.js' ]
```

这种 `for` 循环的方式对于现在 JS 编程来讲其实不是太流行，现在更多流行的是利用 `函数式` 的这种方式去解决

比如下面的 `reduce`

## 二、**reduce**

1.  `reduce` 

```js
const files = ['avatar.jpg ', '.gitignore', '    ', 'index.js']

const reduceWay = files => {
  return (
    files.reduce((result, file) => {
      const fileName = file.trim()
      if (fileName) {
        const filePath = `~/app/${fileName}`
        result.push(filePath)
      }
      return result
    }, [])
  )
}

console.log(reduceWay(files)) // [ '~/app/avatar.jpg', '~/app/.gitignore', '~/app/index.js' ]
```

除了用 `reduce`，还可以利用方法调用链这种方式来进行实现

## 三、**链式编程**

```js
const chain = files => (
  files
    .map(file => file.trim())
    .filter(Boolean)
    .map(fileName => `~/app/${fileName}`)
)

console.log(chain(files)) // [ '~/app/avatar.jpg', '~/app/.gitignore', '~/app/index.js' ]
```

## 四、**总结**

- `链式调用` 的方式更符合 `声明式` 的编程方式，更易读、更容易扩展、每个方法只注重一件事
- `reduce` 更符合 `函数式` 的编程范式
- 传统 `for` 循环已经慢慢退出 `JS 编程` 最佳范式了，所以现在更加推荐使用 `reduce` 和 `方法链调用`

大家更喜欢哪种呢？


感谢阅读，下次见 :)
