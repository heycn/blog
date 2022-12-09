---
title: JS 函数的执行时机
date: 2021-10-09 1:52:23
tags:
top:
---

> 本篇博客是作为个人自学记录，如有不足之处，请批评指正。

## 为什么如下代码会打印 6 个 6

```JavaScript
let i = 0
for(i = 0; i<6; i++){
  setTimeout(()=>{
    console.log(i)
  },0)
}
```

因为 setTimeout 在这个代码中是循环先执行完再去执行 setTimeout 里的 console.log(i)，这个时候 i=6，所以打印出的就是 6 个 6

## 怎样可以打印出 0,1,2,3,4,5

```JavaScript
for(let i = 0; i<6; i++){
  setTimeout(()=>{
    console.log(i)
  },0)
}
```

感谢阅读，下次见 :)
