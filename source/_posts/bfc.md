---
title: BFC 其实很简单
date: 2021-10-01 22:54:00
tags:
top:
---

这是一个概念

## 是什么

BFC 是 `块级格式化上下文` —— `Block Formatting Context`

## 触发条件

- 浮动元素（元素的 float 不是 none）
- 绝对定位的元素（元素的 position 为 absolute 或 fixed）
- 行内块元素 `inline-block`
- `overflow: hidden` overflow 的值不为 visible 的块元素，只要不是默认值（visible）
- 弹性元素（display 为 flex 或 inline-flex 元素的直接子元素）

## 解决了什么问题

- 清除浮动
- 房子 margin 合并（两个垂直方向的 div 的 margin 是会合并的，但是其中一个 div 编程 BFC，就不会合并）

感谢阅读，下次见 :)
