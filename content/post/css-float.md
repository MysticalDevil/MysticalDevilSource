---
title: "CSS 学习笔记 -- 浮动"
date: 2023-02-14T21:59:40+08:00
categories : [
"CSS",
]
tags : [
"CSS",
"Notes",
"Programming",
]
---

## 简介

通过浮动可以使一个元素向其父元素的左侧或右侧移动，使用 `float` 来设置元素的浮动，可选值如下

- `none` 默认值，元素不浮动
- `left` 元素像左浮动
- `right` 元素向右浮动

设置浮动后，元素的水平布局等式便不需要强制成立

浮动在目前的主要目的是用来制作一些**水平方向**的布局

## 特点

- 浮动元素会完全脱离文档流，不再占据文档流的位置

- 设置浮动后，元素会向父元素的左/右移动

- 浮动元素默认不会从父元素中移出

- 浮动元素向左/右移动时，不会超过它前边的其他元素

- 浮动元素不会超过它上边的浮动的兄弟元素，最多就是和他一样高

- 浮动不会盖住文字，文字会自动环绕到浮动元素的周围，可以使用浮动来设置文字环绕图片的效果

- 元素设置浮动后，由于从文档流中脱离，元素的一些特点也会发生变化

  元素脱离文档流的特点

  1. 块元素不再独占页面的一行
  2. 块元素的宽高被内容撑开
  3. 行内元素会变为块元素

  脱离文档流后，不需要再区分行内元素和块元素

## 缺点

### 高度塌陷

```css
.outter {
    border: 10px black solid;
}
.inner {
    width: 100px;
    height: 100px;
    background: cyan;
    float: left;
}
```

在浮动布局中，父元素的高度是默认被子元素撑开的，当子元素浮动后，其会完全脱离文档流，子元素从文档流中脱离，将会无法撑起父元素，导致父元素的高度丢失。 父元素高度丢失后，其下的元素会自动上移，导致页面的布局混乱。所以高度塌陷是浮动布局中比较常见的一个问题，它必须要被处理。	

## 解决方式 

### BFC（Block Formatting Context）

BFC 是 CSS 中的一个隐含的属性，可以为一个元素开启 BFC ，开启 BFC 后该元素会变成一个独立的布局区域。

元素开启 BFC 后的特点

- 不会被浮动元素所覆盖
- 子元素和父元素的外边距不会重叠
- 可以包含浮动的子元素

可以通过特殊方式开启 BFC

1. 可以设置元素浮动 （不推荐

2. 将元素设为 `inline-block`（不推荐

3. 将元素的 `overflow` 设置为非 `visible`。常用的方式：为元素设置 `overflow: hidden`开启 BFC 以使其可以包含浮动元素 （副作用最小）

### Clear 属性

如果不希望某个元素因为其他元素浮动的影响而改变位置，可以通过 `clear` 属性来清除浮动元素所产生的影响，可选值如下

- `left` 清除左侧浮动元素对当前元素的影响
- `right` 清除右侧浮动元素对当前元素的影响
- `both` 清除两侧中影响最大的一侧

设置清除浮动后，浏览器会的自动为元素添加一个上外边距，以使其不受其他元素的影响

### 使用  after 伪元素

可以使用 `after` 伪元素来解决高度塌陷，解决方式如下

```css
.box1 {
    border: 10px green solid;
}
.box2 {
    width: 100px;
    height: 100px;
    background: cyan;
    float: left;
}
/* 伪元素默认是行内元素 */
.box1::after {
    content: "";
    clear: both;
    display: block;
}
```

### clearfix

使用 `before` 伪元素来解决外边距折叠，如下

```css
.box1 {
    width: 200px;
    height: 200px;
    background: cyan;
}
.box2 {
    width: 100px;
    height: 100px;
    background: orange;
    margin-top: 100px;
}
.box1::before {
    content: "";
    display: table;
}
```

`clearfix` 该样式可以同时解决高度塌陷和外边距重叠的问题

```css
.clearfix::after,
.clearfix::before {
    content: "";
    disaply: table;
    clear: both;
}
```

