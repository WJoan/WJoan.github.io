---
layout:     post
title:      "CSS居中布局"
date:       2017-05-13 15:16:45
author:     "Joan"
tags:		["css"]
---

## 水平居中

### inline 元素

```
.center-children {
	text-align: center;
}

```
`text-align` 属性规定元素中的文本的水平对齐方式。该属性需要添加在 inline 元素的父元素上。

### block 块级元素

```
.center-me {
  margin: 0 auto;
}
```
使用这种方法的时候元素 width 不能为 auto。

### 多个 block 元素

```
.inline-block-center {
  text-align: center;
}

.inline-block-center div {
  display: inline-block;
  text-align: left;
}
```
第一种方法将多个块的属性设置为 inline-block，然后使用 inline 元素水平居中的方法。

```
.flex-center {
  display: flex;
  justify-content: center;
}
```
第二种方法，直接使用 `flex` 布局。这样的好处在于不用将 block 元素设置为 inline 元素。

## 垂直居中

### inline 元素

#### 单行

```
.link {
  padding-top: 30px;
  padding-bottom: 30px;
}
```
最简单的方法就是让上下内边距相等，但如果内边距不确定的情况下就无法使用这种方法了。

```
.center-text-trick {
  height: 100px;
  line-height: 100px;
  white-space: nowrap;
}
```
这种方法适用于只有一行文字的时候。使 `line-height` 行高与内联元素的高度相等。`white-space` 属性指定元素内的空白怎样处理，`nowrap` 表示不会换行。

#### 多行

```
.flex-center-vertically {
  display: flex;
  justify-content: center;
  flex-direction: column;
  height: 400px;
}
```
继续 `flex` 大法好，设置 `flex-direction` 排列方向为垂直。
[参考链接](https://css-tricks.com/centering-css-complete-guide/)