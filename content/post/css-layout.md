---
title: "CSS 学习笔记 -- 盒模型"
date: 2023-02-10T19:50:59+08:00
categories : [
"CSS",
]
tags : [
"CSS",
"Notes",
"Programming",
]
---

## CSS 盒模型

### 文档流（normal flow）

网页是一个多层的结构，一层摞一层的，可以通过 CSS 分别为每一层来设置样式，作为用户来讲只能看到最上面的一层，这些层中的，最底下的一层被称作文档流。我们所创建的元素，默认都在文档流中

元素主要有两个状态：

- 在文档流中
- 不在文档流中（脱离文档流）

#### 元素在文档流中的特点

##### 块元素

- 块元素会在页面中独占一行
- 默认宽度撑满父元素
- 默认高度被子元素撑开

##### 行内元素

- 行内元素不会独占一行，自左向右水平排列
- 一行中不能容纳下所有行内元素，则会换行
- 默认宽度和高度都是被内容撑开

### 盒模型

CSS 将页面中的所有元素都设为了一个矩形的盒子。将元素设置为矩形后，对页面的布局就变为将不同的盒子摆放到不同的位置

每一个盒子都由以下部分组成：

- 内容区（content），元素中的所有子元素和文本内容都在内容区中排列，其大小由 width 和 height 两个属性来决定，width 和 height 定义的是内容区的大小

- 边框（border），边框属于盒子边缘，出了边框属于盒子外部。设置边框至少需要三个样式：宽度、颜色、风格

  边框大小会影响盒子大小

  `border-width` 一般情况下默认是 3 px，四个值的顺序为上右下左，三个值的顺序为上 左右 下，两个值为上下 左右

  `border-color` 和 `border-style` 规则同上

  `border-color` 默认值为 `color` 的值

  其上属性也可以用 `border` 或者 `border-xxx` 直接定义，两者为简写属性

- 内边距（padding），内容区和边框的距离

  内边距设置忽影响盒子大小，背景颜色会延伸到内边距上

  **一个盒子的可见框大小 = 内容区 + 内边距 + 边框**

- 外边距（margin），盒子与其他盒子的间距

  不会影响盒子可见框的大小，但是会影响盒子位置和实际占用空间大小

  默认情况下，设置了左边和上边的外边距会移动元素自身，设置下边和右边会移动其他元素

  `margin-bottom` 设置正值，其下元素会向下移动

  `margin-right` 默认情况下，不会产生任何效果

  `margin` 可以设置负值，元素会向相反方向移动

![box model](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/The_box_model/box-model.png)

#### 元素水平方向布局

元素在其父元素中水平方向位置由以下几个属性共同决定

- `margin-left/right`
- `border-left/right`
- `padding-left/right`
- `width`

一个元素在其父元素中，水平布局必须满足：子元素的左右外边距、边框、内边距的和 = 父元素内容区的宽度

以上等式必须满足，如果相加结果使等式不成立，则称为过度约束，浏览器会自动调整等

这些值中有三个值可以设置为 auto：`width` `margin-left` `margin-right`，`width` 的值默认为 auto

- **如果所有值中没有 auto 的情况，则浏览器会调整 `margin-right`的值以使等式满足**

- **如果某个值为 auto 则会自动调整设置为 auto 的值，来使等式成立**

- **如果将 `width` 和 一个 `margin` 设置为 auto，则宽度会调整到最大，设置为 auto 的外边距会自动为 0**

- **如果将三个值都设置为 auto，则宽度会调整为最大，外边距会设置为 0**

- **如果将两个外边距设置为 auto，宽度固定，则会将外边距设置为相同的值**。经常利用该特点来使子元素在父元素中水平居中

  ```css
  .box {
      width: 200px;
      height: 200px;
      margin: 0 auto;
  }
  ```

#### 元素垂直方向布局

默认情况下，父元素的高度被内容撑开

子元素在父元素内容区排列。如果子元素的大小超过父元素，则子元素会从父元素中溢出

使用 `overflow` 属性来设置父元素来处理溢出的子元素，可选值如下（`overflow-x`，`overflow-y`）

- `visiable` 默认值，子元素溢出，会在父元素外部显示
- `hidden` 溢出元素将会被裁剪，不会显示
- `scroll` 生成两个滚动条来查看完整内容
- `auto` 根据需要生成滚动条

#### 外边距折叠

有如下两个样式

```css
.box1 {
    background: cyan;
    margin-bottom: 100px;
}
.box2 {
    background: orange;
    margin-top: 100px;
}
```

理论上两个`div`的间距应该为 200px，但实际上为 100px

该问题被叫做外边距折叠，即垂直方向的相邻的外边距会发生折叠（重叠）现象

- **兄弟元素**间的相邻垂直外边距，两者为正值会取两者之间的较大值；一正一负取两者的和；全负值两者之间取绝对值较大的

  兄弟元素间的外边距折叠对开发是有利的，一般不需要处理

- **父子元素**中，如果子元素设置了`margin-top`，子元素和父元素位置会同时改变，不会单独改变子元素位置；即子元素的（上外边距）会传递给父元素

  父子元素的外边距折叠会影响到页面的整体布局

  临时解决方式：不用外边距或者不让两者相邻

#### 行内元素的盒模型

- 不支持设置宽度和高度，内容区大小由内容决定
- 支持设置 `padding`，垂直方向的 `padding` 不会影响页面的布局
- `border` ，`margin`同上

可以使用 `display` 属性来设置元素显示的类型，**display** 的可选值如下

- `inline` 将元素设置为行内元素
- `block` 设置为块元素
- `inline-block`设置为行内块元素，既可以设置宽度高度，又不会独占一行
- `table` 设置为表格
- `none` 元素不在页面中显示（隐藏元素）
- 还有其他可选值，之后会学习

`visibility` 用来设置元素的显示状态，**visibility** 可选值如下

- `visible` 默认值，元素在页面正常显示
- `hidden` 元素在页面隐藏，但是依然占据页面位置

#### 浏览器的默认样式

通常情况下，浏览器都会为元素设置一些默认样式

默认样式会影响到页面的布局。开发时会去除掉浏览器的默认样式（PC 端的页面）

```css
* {
    margin: 0;
    padding: 0;
}
ul {
    /*去除项目符号*/
    list-style: none;
}
```

可以使用 [`normalize.css`](https://necolas.github.io/normalize.css/8.0.1/normalize.css) 和 [`reset.css`](https://meyerweb.com/eric/tools/css/reset/) 来清除默认样式

`normalize.css` 对默认样式进行了统一

`reset.css` 直接去除了所有默认样式

### 练习小记

- 要让文字能够在父元素中垂直居中，只需将父元素的`line-height`设置为一个和父元素`height`相等的值

### 盒子大小

默认情况下，盒子的**可见框**的大小由内容区、内边距和边框共同决定

在盒子模型中可以使用`box-sizing`来设置盒模型的计算方式，可选值如下

- `content-box` 默认值，宽度和高度用来设置内容区的大小
- `border-box` 高度和宽度用来设置整个盒子的可见框的大小

### 轮廓 阴影 圆角

轮廓 `outline` 用来设置盒模型的轮廓线，用法和 `border` 相同，但是不会影响盒模型可见框的大小

阴影 `box-shadow` 用来设置元素的阴影效果，不会影响页面布局；默认在元素的正下方，使用时需要添加偏移量(x, y)，还有第三个值为模糊半径

```css
.box {
    width: 200px;
    height: 200px;
    background: cyan;
    border: 5px solid red;
    border-radius: 20px;
    box-shadow: black 10px 10px;
}
```

圆角 `border-radius` 用来设置盒子的角的半径大小，可以使用相对值和绝对值

- `border-top/bottom-left-right` 元素各个角，可以接受单个值，此时为圆形圆角；也可以接受两个值，此时为椭圆圆角

- `background-clip` 

- `border-radius` 可以使用 `\` 来设置椭圆圆角

  ```css
  {
      /* The syntax of the first radius allows one to four values */
      /* Radius is set for all 4 sides */
      border-radius: 10px;
  
      /* top-left-and-bottom-right | top-right-and-bottom-left */
      border-radius: 10px 5%;
  
      /* top-left | top-right-and-bottom-left | bottom-right */
      border-radius: 2px 4px 2px;
  
      /* top-left | top-right | bottom-right | bottom-left */
      border-radius: 1px 0 3px 4px;
  
      /* The syntax of the second radius allows one to four values */
      /* (first radius values) / radius */
      border-radius: 10px / 20px;
  
      /* (first radius values) / top-left-and-bottom-right | top-right-and-bottom-left */
      border-radius: 10px 5% / 20px 30px;
  
      /* (first radius values) / top-left | top-right-and-bottom-left | bottom-right */
      border-radius: 10px 5px 2em / 20px 25px 30%;
  
      /* (first radius values) / top-left | top-right | bottom-right | bottom-left */
      border-radius: 10px 5% / 20px 25em 30px 35em;
  
      /* Global values */
      border-radius: inherit;
      border-radius: initial;
      border-radius: revert;
      border-radius: revert-layer;
      border-radius: unset;
  }
  ```
