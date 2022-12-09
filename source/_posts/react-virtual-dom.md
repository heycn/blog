---
title: 浅析「React 虚拟 Dom」
date: 2022-3-09 19:50:41
tags:
top:
---

> 本篇博客用来记录我的学习成果\
> 防止遗忘，

## 是什么

虚拟 DOM 就是虚拟节点，React 用 **JS 对象**来**模拟 DOM 节点**，然后将其**渲染成真实的 DOM 节点**，它**本质上是一个 JS 对象**。

## 怎么做

### 1. 模拟一个节点

使用 JSX 语法写出来的 div 其实就是一个 **虚拟节点**

```js
<div id='x'>
  <span class='red'> hi </span>
</div>
```

上面代码会得到这样一个对象：

```js
{
  tag: 'div',
  props: {
    id: 'x'
  },
  children: [
    {
      tag: 'span'
      props: {
        className: 'red'
      },
      children: [
        'hi'
      ]
    }
  ]
}
```

这是因为，JSX 语法会被转译为 createElement 函数的调用（也叫 h 函数），如下：

这些代码是使用 babel 或者 webpack 的一些 loader 实现的

实际上，我们是调用了 createElement 来得到的一个虚拟 DOM，只不过这个 createElement 可以嵌套

```js
React.createElement('div', { id: 'x' }, Create.createElement('span', { class: 'red' }, 'hi'))
```

### 2. 将虚拟节点渲染为正式的节点

```js
function render() {
  // 如果是字符串或者数字，那么就创建文本节点
  if (typeof vdom === 'string' || typeof vdom === 'number') {
    return document.createTextNode(vdom)
  }
  const { tag, props, children } = vdom
  // 创建真实节点
  const element = document.createElement(tag)
  // 设置属性
  setProps(element, props)
  // 遍历子节点，并获取创建真实 DOM，插入到当前节点
  children.map(render).forEach(element.appendChild.bind(element))

  // 虚拟 DOM 中缓存真实 DOM 节点
  vdom.dom = element
}
```

## 优点

1.  让 DOM 操作的性能更好：通过 DOM diff 算法，减少不必要的 DOM 操作
1.  解决了 DOM 操作不方便问题：以前需要记很多 DOM API，现在只有 setState
1.  为 React 带来了跨平台能力，因为虚拟节点除了渲染真实节点，还可以渲染为其他东西

## 缺点

1.  性能要求极高的地方，还是需要用真实 DOM 操作（但是目前没有这种需求）
1.  React 为虚拟 DOM 创造了 **合成事件**，跟原生 DOM 事件不太一样，工作中要额外注意
1.  1.  所有 React 事件都绑定到根元素，自动实现事件委托
    1.  如果混用合成事件和原生 DOM 事件，有可能会出现 BUG

## 如何解决缺点

。。。用 Vue3

感谢阅读，下次见 :)
