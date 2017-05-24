---
layout:     post
title:      "CSS居中布局"
date:       2017-05-13 15:16:45
author:     "Joan"
tags:		    ["css"]
---
> 参考链接：https://css-tricks.com/centering-css-complete-guide/

## 一、水平居中

### 1.inline 元素

```
.center-children {
  text-align: center;
}

```
`text-align` 属性规定元素中的文本的水平对齐方式。该属性需要添加在 inline 元素的父元素上。

### 2.block 块级元素

```
.center-me {
  margin: 0 auto;
}
```
使用这种方法的时候元素 width 不能为 auto。当 width 没有设置的时候 block 元素将占满整行。

### 3.多个 block 元素

#### 方法1：inline 方法

```
.inline-block-center {
  text-align: center;
}

.inline-block-center div {
  display: inline-block;
  text-align: left;
}
```
第一种方法将多个块的属性设置为 `inline-block`，然后使用 inline 元素水平居中的方法。

#### 方法2：flex 布局

```
.flex-center {
  display: flex;
  justify-content: center;
}
```
第二种方法，直接使用 `flex` 布局。这样的好处在于不用将 block 元素设置为 inline 元素。

简单介绍一下 `flex` 布局：`flex-direction` 属性决定了主轴的方向，`row` 水平方向（默认）或者 `column` 垂直方向。而 `justify-content` 属性定义了项目在主轴上的对齐方式。

## 二、垂直居中

### 1.inline 元素

#### 单行 inline 元素

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

#### 多行 inline 元素

```
.flex-center-vertically {
  display: flex;
  justify-content: center;
  flex-direction: column;
  height: 400px;
}
```
使用 `flex` 布局，设置 `flex-direction` 排列方向为垂直。这个方法的一个很重要的问题就是：需要知道父级元素的高度。如果不定义父级元素的高度，那么父级元素会根据内部元素自动调整自身高度，那么也就不会有居中的问题了。

**PS：在很多情况下，这种自适应还是很有用处的。**但是主轴不同时，情况有可能会有所不同。当主轴方向为水平方向的时候，每一行子元素的高度将是一致的，也就是说子元素很可能会被拉长，除非固定每个子元素的高度。父级元素没有指定高度与宽度的话，高度将会自动包裹住所有行，宽度则是100%。因此，**当每一行的高度都需要根据子元素内部包含的东西自适应的时候，需要将主轴设置为垂直方向。**

### 2.block 块级元素

#### 知道元素高度—— margin 负值

```
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  height: 100px;
  margin-top: -50px; 
}
```

首先把元素放置到父元素一半的高度上，即 top:50%。然后再把块元素向上移动高度的一半。最终元素就垂直居中了。

将 margin 负值设置为总高度的一半（如果设置了 padding 或者 border，那么要记得计算进去），达到向上移动一半高度的效果。

#### 不知道元素高度—— transform

```
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
}
```

CSS3 中的新属性 `transform`，这个属性翻译过来就是变形、转换，它可以帮助我们对元素进行 2D 或者 3D 转换。`translate` 值定义了平移量，其参数的百分比相对的是元素本身而不是父元素。

PS：translate 常译为“翻译”，但在这里它的意思是“平移”。

#### 布局方法—— flex

```
.parent {
  display: flex;
  flex-direction: column;
  justify-content: center;
}
```

## 三、水平垂直居中

其实与水平居中的方法差不多，有三种分别是：`margin` 负值； `transform` 属性平移；以及 `flex` 布局。代码如下：

```
.parent {
  position: relative;
}

.child {
  width: 300px;
  height: 100px;
  padding: 20px;

  position: absolute;
  top: 50%;
  left: 50%;

  margin: -70px 0 0 -170px;
}
```

```
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

```
.parent {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

## 总结
当需要居中的元素是 `inline` 内联元素的时候，比如居中一些文字，那么则需要用到一些文字属性，比如 `text-align` 以及 `inline-height`。

当需要居中的属性为 `block` 块级元素的时候，使用 `transform` 或者 `flex` 会更方便一些。如果内部涉及的元素比较多，使用 `flex` 会更好。


