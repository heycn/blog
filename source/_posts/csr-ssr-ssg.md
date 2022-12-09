---
title: 浅析「CSR、SSR、SSG 渲染原理」
date: 2022-11-15 22:59:10
tags:
top:
---

## **CSR** 原理及问题

#### 是什么

全称：Client Side Render（客户端渲染）

特点：没有完整的 HTML 内容，依靠 JS 来完成整个页面的渲染

在 Vue 或 React 作为目前前端热门框架的环境下，大部分的页面都是采用 CSR 的渲染模式

例如，我们使用 `vite` 创建的 `React` 脚手架，html 文件的 body 里只有以下内容

```html
<body>
  <div id="root"></div>
  <script type="module" src="/src/main.js"></script>
</body>
```

> HTML 里面只有一个  `id="root"` 的 `div`，里面并没有具体的页面内容，那页面是如何渲染出来的呢？

答案是通过执行 JS 代码：\
平时我们在用 React 开发页面的时候，实际上是在写 React 组件，并不是最后的 HTML 内容\
在 js 执行的时候，React 框架底层会接收我们写的组件内容，然后再进行一系列的真实的 DOM 操作\
只不过，这些操作，被 React `"屏蔽"` 掉了，但归根结底，还是需要依赖 js 的执行，来完成页面的渲染\
这也是 CSR 的核心特征

#### 存在的缺点

- **首屏加载慢**
  - 页面需要 js 向服务端发送 HTTP 请求来获取数据，会带来 `网络 IO 的开销`
  - 页面又需要 `依赖 js 来渲染`，这又是一部分运行时开销
- **对 SEO 不友好**
  - 因为搜索引擎的爬虫不会等待 js 执行完之后再去抓取页面的内容，一开始发现页面上没有有效的内容，那么就很难做 SEO 相关的优化了

#### 如何解决缺点

SSR 就是为了解决 CSR 所带来这一系列的问题

## **SSR**

#### 是什么

全称：Server Side Render（服务端渲染）
特点：服务端返回完整的 HTML 内容

也就是说，浏览器一开始拿到的，就是完整的 HTML，不需要 js 去调用 DOMApi 来去完成渲染

#### SSR 与 CSR 对比

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ae7a8590214491bb912b64c4134352a~tplv-k3u1fbpfcp-watermark.image?)

###### CSR 流程

1. 首先下载 HTML 文本
2. 下载 JavaScript 脚本
3. 执行 JavaScript 脚本（包括数据获取，页面渲染比较重要的逻辑）
4. 页面准备就绪

###### SSR 流程

服务端渲染里，浏览器首先会拿到 `完整的 HTML` 内容，也就是说：`组件的代码在服务端已经被渲染成了完整的 HTML 字符串`，浏览器就能直接完整地渲染这些内容了

#### 存在的缺点

SSR 无法交互

SSR 是直接产出 HTML 的代码，DOM 元素事件绑定的逻辑仍然需要 JS 才能够完成，因此 `页面不可以交互`

#### 如何解决缺点

SSR 页面引入 `CSR的脚本`（同构）

在实际的场景中，会在 SSR 页面中加入 CSR 的脚本，完成 DOM 的事件绑定

这个完成事件绑定的过程，也被称为 `Hydration`

## **SSG** 原理

#### 是什么

全称：Static Site Generation（静态站点生成）

特点：构建阶段的 SSR，build 过程中产出完整 HTML

SSG 在构建的过程当中，也就是当执行 `npm build` 的时候，就可以产出完整的 HTML 内容，构建完成之后进行 HTML 部署，在生产环境下就不需要服务器的开发、运维相关的工作，研发和运维的成本会比 SSR 低一些

#### 优劣势分析

###### 优点

1. 服务器压力小
2. 继承 SSR `首屏性能` 以及 `SEO` 的优势

###### 局限性

不适用于 `数据经常变化` 的场景

###### 使用场景

适用于 `数据变化频率较低` 的站点

如：文档站、官网站点、博客等

业界比较知名的 [VuePress](https://vuepress.vuejs.org/zh/)、[Gatsby](https://www.gatsbyjs.com/) ... 等框架都是比较典型的 `SSG 方案`
