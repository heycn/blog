---
title: 玩 MOBA 游戏，理解「防抖、节流」
date: 2021-11-18 01:21:23
tags:
top:
---

吐槽：防抖节流这两个名字起的很不好，一点都不形象

英雄联盟玩过吧？王者荣耀玩过吧？\
那开始吧：

## 节流「技能冷却中」

闪现用过吧？\
我点击闪现后，是不是会进入一段 CD？\
CD 期间再点击闪现，是不是没有反应？

那用代码实现这个功能吧：

```js
const sx = () => {
  console.log('闪现')
}

let 冷却中 = false // 默认没进入冷却
let timer = null // 定时器默认为空

function onClickSx() {
  // 点击闪现
  if (冷却中) {
    return
  } // 如果闪现在冷却，就返回，没有在冷却则执行 闪现 并进入冷却
  sx()
  冷却中 = true // 闪现后进入冷却，此时点击闪现没有反应
  timer = setTimeout(() => {
    // 120 秒之后冷却结束
    冷却中 = false
  }, 120 * 1000)
}
```

这，就是`节流`\
那如何写出符合任何需求的节流呢？

写一个函数，实现任何技能都可以进入冷却

```js
const sx = distance => {
  // 闪现接收一个距离 distance
  console.log('闪现')
}

const throttle = (skill, time) => {
  let timer = null // 是否冷却中
  return (...args) => {
    if (timer) {
      return
    } // 有在冷却 就返回
    skill.call(undefined, ...args) // 没有 this 的情况
    timer = setTimeout(() => {
      timer = null // 冷却时间到后，再把时间置为空
    }, time)
  }
}

const sx2 = throttle(sx, 120 * 1000)
```

再修修改改，变成了标准写法的节流

标准写法：

```js
const throttle = (fn, time) => {
  let timer = null
  return (...args) => {
    if (timer) {
      return
    }
    fn.call(undefined, ...args)
    timer = setTimeout(() => {
      timer = null
    }, time)
  }
}
```

## 防抖「回城被打断」

我点击回城，但是被小兵攻击了，所以我回城被打断，需要重新回城

```js
const f = () => {
  console.log('回城成功')
}

let timer = null // 回城计时，默认为空

function x() {
  if (timer) {
    // 如果正在回城，打断回城将会重新计时（不为 null 时，则 timer 存在）
    clearTimeout(timer)
  }
  timer = setTimeout(() => {
    fn()
    timer = null // 回城成功后，再置为空
  }, 3000)
}
// 你只要打断我，我就重新回城；3秒钟之内你又一次打断我，我又重新回城
```

那写一个满足防抖需求的：写一个通用的函数，任何功能都可以实现打断了之后再重新计时

标准写法：

```js
const debounce = (fn, time) => {
  let timer = null
  return (...args) => {
    if (timer != null) {
      clearTimeout(timer)
    }
    timer = setTimeout(() => {
      fn.call(undefined, ...args)
      timer = null
    }, time)
  }
}
```

## 使用场景

### 节流

> 我不希望用户频繁点击按钮的时候使用

### 防抖

> 当用户进行一个频繁拖动操作的时候，我希望在拖动停止之后再去实现一个效果

感谢阅读，下次见 :)
