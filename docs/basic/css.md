---
toc: content
order: 12
---

# CSS 基础

## CSS 盒模型

CSS 盒子组成由 4 个区域组成，从内到外依次为：内容区域、内边距区域、边框区域和外边距区域。

![](https://t1.picb.cc/uploads/2019/09/09/gXP6y7.jpg)

标准盒模型（content-box）

对于现代浏览器来说，元素默认应用标准盒模型。当然你也可以像下面这样做显式的设置。

```css
.box {
  box-sizing: content-box;
}
```

怪异盒模型（border-box）

IE 浏览器的早期版本没有遵循 CSS 标准，width 和 height 包含content padding border，而不是 content 的宽高，导致不同浏览器的表现不同，毫无疑问是个浏览器 bug。

后来 CSS3 引入了 box-sizing，让开发者可以选择使用哪种盒模型，提供更好的灵活性。通过下面的设置，我们可以将元素的盒模型设置为怪异盒模型。

```css
.box {
  box-sizing: border-box;
}

```

## 什么是 BFC

BFC 全称为`块级格式化上下文` (Block Formatting Context) 。**它决定了元素如何对其内容进行定位以及与其他元素的关系和相互作用**，当涉及到可视化布局的时候，BFC 提供了一个环境，HTML 元素在这个环境中按照一定规则进行布局。

### 从一个现象说起

- 一个盒子不设置 height，当内容子元素都浮动时，无法撑起自身（height = 0）
- 这个盒子没有形成 BFC （代码如下：）

```js
// css
.son {
    width: 300px;
    height: 300px;
    background: blue;
    float: left;
}

<div class="father">
    <div class="son"></div>
    <div class="son"></div>
    <div class="son"></div>
</div>


// 解决方案之一
.father {
    float: left;
}
```

触发 BFC 的条件：

-   float 的值不为 none；
-   position 的值不为 static 或 relative；
-   display 的值是 inline-block、flex、或者 inline-flex；
-   overflow 的值为 hidden；

BFC 解决的问题

-   清除浮动：父元素设置 overflow: hidden 触发 BFC 实现清除浮动，`防止父元素高度塌陷`；
<!-- -   `消除相邻元素垂直方向的边距重叠`：第二个子元素套一层，并设置 overflow: hidden，构建 BFC 使其不影响外部元素； -->
-   `消除父子元素边距重叠`，父元素设置 overflow: hidden；

<!-- ## 层叠上下文

元素提升为一个比较特殊的图层，在三维空间中 (z轴) 高出普通元素一等。

1、触发条件

* position
* css3属性：flex，transform，opacity

2、层叠等级：层叠上下文在z轴上的排序

* 在同一层叠上下文中，层叠等级才有意义
* z-index的优先级最高 -->

<!-- ![](https://t1.picb.cc/uploads/2019/09/09/gXPgxK.jpg) -->

## 清除浮动方法

`去除浮动影响，防止父级高度塌陷。`

第一种方法

在浮动元素后面添加 class 为 clear 的 `空 div` 元素，给这个 div 设置样式 `clear:both`。

```js
<div class="container">
    <div class="left"></div>
    <div class="right"></div>
    <div class="clear"></div>
</div>

.clear{
    clear:both;
}
```

第二种方法

给`父容器`添加 `overflow:hidden` 或者 auto 样式。

```js
<div class="container">
    <div class="left"></div>
    <div class="right"></div>
</div>

.container{
    width: 300px;
    border: 1px solid #000;
    background-color: #aaa;
    overflow: hidden;
}
```

第三种方法

给`父容器`添加 clearfix 的 class ，用 `伪元素` clearfix::after 来设置样式，清除浮动。

```js
<div class="container clearfix">
    <div class="left"></div>
    <div class="right"></div>
</div>

.clearfix{
    zoom: 1;         /*IE6*/
}
.clearfix::after{
    content: ".";
    height: 0;
    clear: both;
    display: block;
    visibility: hidden;
}
```

## CSS 预/后处理器

### CSS 预处理器

包括：`Sass`、`Less`、`Stylus`。用来预编译 `Sass` 或 `Less`，增强了 css 代码的`复用性`。CSS 预处理器的原理: `是将类 CSS 语言通过 Webpack 编译 转成浏览器可读的真正 CSS`。在这层编译之上，便可以赋予 CSS 更多更强大的功能，常用功能:

-   嵌套
-   变量
-   循环语句
-   条件语句
-   自动前缀
-   mixin 复用

### CSS 后处理器

> PostCSS: 利用从 Can I Use 网站获取的数据为 CSS 规则添加特定厂商的前缀。`Autoprefixer` 自动获取浏览器的流行度和能够支持的属性，并根据这些数据帮你自动为 CSS 规则添加前缀。

例如：`PostCSS`，在完成的样式表中根据 CSS 规范处理 CSS，让其更有效；目前最常做的是给 CSS 属性`添加浏览器私有前缀，实现跨浏览器兼容性的问题`。

## 伪类和伪元素

`伪类`：用于当`已有元素处于的某个状态时`，为其`添加对应的样式`。比如，当用户悬停在指定的元素时，我们可以通过 `:hover` 来描述这个元素的状态。

`伪元素`：用于创建一些`不在文档树中的元素`，并`为其添加样式`。比如说，我们可以通过 `::after` 来在一个元素后增加一些文本，并为这些文本添加样式。虽然用户可以看到这些文本，但是这些文本实际上`不在文档树中`。

CSS3 规范中的要求：

-   使用单冒号表示`伪类` 如：`:hover` 和 `:active`
-   使用双冒号表示`伪元素` 如：`::before` 和 `::after`

## CSS3 之动画

transition（过渡）

什么叫过渡？字面意思上来讲，就是元素从这个属性(color)的某个值(red)过渡到这个属性(color)的另外一个值(green)，
这是一个状态的转变，需要一种条件来触发这种转变，比如我们平时用到的:hover、:focus、:checked、媒体查询或者 JavaScript。

```js
<div>transition</div>

div {
    height: 100px;
    width: 100px;
    background: red;
    transition: transform 1s ease-in-out .1s;
}
div:hover {
    transform: rotate(180deg) scale(.5, .5);
}
```

| 值                  | 描述                                                                 |
| :------------------ | :------------------------------------------------------------------- |
| transition-property | 指定 CSS 属性的 name，transition 效果                                |
| transition-duration | transition 效果需要指定多少秒或毫秒才能完成                          |
| t-timing-function   | transition 效果的转速曲线：linear,ease,ease-in, ease-out,ease-in-out |
| transition-delay    | 定义 transition 效果开始的时候                                       |

<!-- [在线体验 CSS3 transition](https://codepen.io/zhangfangchao/pen/oNvpypJ) -->

animation（动画）

```js
animation:
    name || duration || timing-function || delay || iteration-count || direction
```

-   animation 主要包含六个属性，具体含义如下：

| 属性名                    | 属性值                               | 描述                  |
| :------------------------ | :----------------------------------- | :-------------------- |
| animation-name            | 规定需要绑定到选择器的 keyframe 名称 | 自定义                |
| animation-duration        | 规定完成动画所花费的时间             | 以秒或毫秒计          |
| animation-timing-function | 规定动画的速度曲线                   | linear,ease-in-out... |
| animation-delay           | 规定在动画开始之前的延迟             | 默认值为 0            |
| animation-iteration-count | 规定动画应该播放的次数               | 默认值为 1            |
| animation-direction       | 规定是否应该轮流反向播放动画         | 默认值是正向          |

-   在介绍 animation 具体使用之前，要先介绍 keyframe

@keyframes 让开发者通过指定动画中特定时间点必须展现的关键帧样式（或者说停留点）来控制 CSS 动画的中间环节。这让开发者能够控制动画中的更多细节而不是全部让浏览器自动处理。

要使用关键帧, 先创建一个带名称的@keyframes 规则，以便后续使用 animation-name 这个属性来调用指定的@keyframes. 每个@keyframes 规则包含多个关键帧，也就是一段样式块语句，每个关键帧有一个百分比值作为名称，代表在动画进行中，在哪个阶段触发这个帧所包含的样式。
关键帧的编写顺序没有要求，最后只会根据百分比按由小到大的顺序触发。

```js
@keyframes animation-name {
    keyframes-selector {
        css-styles;
    }
}
```

| 值                 | 描述                                                | 是否必填 |
| :----------------- | :-------------------------------------------------- | :------- |
| animation-name     | 定义动画的名称                                      | 必需     |
| keyframes-selector | 动画时长的百分比。合法的值：0-100% from 0 - to 100% | 必需     |
| css-styles         | 一个或多个合法的 CSS 样式属性                       | 必需     |

案例：

```js
<div>福</div>

div {
    width: 100px;
    height: 100px;
    font-size: 80px;
    text-align: center;
    line-height: 100px;
    margin: 20px auto;
    background: red;
    animation: name 6s linear infinite;
}
@keyframes name {
    0% {
        transform: rotate(0deg);
    }
    100%{
        transform: rotate(360deg);
    }
}
```

<!-- [在线体验 CSS3 animation](https://codepen.io/zhangfangchao/pen/XWrVVpX) -->

## transition 和 animation 的区别

Animation 和 transition 大部分属性是相同的，他们都是随时间改变元素的属性值，他们的主要区别是 transition 需要触发一个事件才能改变属性，
而 animation 不需要触发任何事件的情况下才会随时间改变属性值，并且 transition 为 2 帧，从 from .... to，而 animation 可以一帧一帧的。

transform（转变）

translate 平移

```js
// 2D
transform: translate(100px, 200px);
transform: translate(-50px, 200px);
// 3D
transform: translateX(100px);
transform: translateY(100px);
```

rotate 旋转

```js
// 2D
transform: rotate(30deg);
transform: rotate(-30deg);
// 3D
transform: rotateX(30deg);
transform: rotateY(30deg);
transform: rotateZ(30deg);
```

scale 缩放

```js
// 2D
transform: scale(0.2);
transform: scale(5);
// 3D
transform: scaleX(0.2);
transform: scaleY(0.2);
```

skew 2D 倾斜（无 3d）

```js
transform: skewX(30deg);
transform: skewY(30deg)
// 即
transform: skew(30deg, 40deg);
```

matrix 方法和 2D 变换方法合并成一个

```js
// matrix 方法有六个参数，包含旋转，缩放，移动（平移）和倾斜功能
// 利用matrix()方法旋转div元素30
transform: matrix(0.866, 0.5, -0.5, 0.866, 0, 0);
```

<!-- [在线预览 CSS3 transform](https://codepen.io/zhangfangchao/pen/ZEzvJZB) -->

<!-- ## 水平居中方案

<h3>text-align: center</h3>

``` js
<div class="box1">
    <span>内敛元素水平居中</span>
    <div>块元素水平居中</div>
</div>

.box1 {
    text-align: center;
}
```

<h3>flex 布局 </h3>

``` js
<div class="box2">
    <div>flex 布局测试</div>
</div>

.box2 {
    display: flex;
    justify-content: center;
}
```
<h3>绝对定位和transform</h3>

``` js
<div class="box3">
    <div class="box3-center">使用绝对定位和CSS3新增的属性transform</div>
</div>

.box3 {
    position: relative;
}
.box3-center {
    position: absolute;
    left: 50%;
    transform: translate(-50%,0);
}
```

<h3>绝对定位和margin-left（元素定宽）</h3>

``` js
<div class="box4">
    <div class="box4-center">使用绝对定位和margin-left（元素定宽）</div>
</div>

.box4 {
    width: 100%;
    position: relative;
}
.box4-center {
    width: 300px;
    position: absolute;
    left: 50%;
    margin-left: -150px;
}
```
[在线预览 水平居中](https://codepen.io/zhangfangchao/pen/BaBmXLv) -->

<!-- ## 垂直居中

### line-height
``` js
<div class="box1">
    <span>内敛元素垂直居中</span>
    <div>块元素垂直居中</div>
</div>

.box1 {
    height: 50px;
}
.box1 > span {
    line-height: 50px;
}
.box1 > div {
    line-height: 50px;
}
```

### flex 垂直居中
``` js
<div class="box2">
    <div>flex 布局测试</div>
</div>

.box2 {
    display: flex;
    height: 50px;
}
.box2 > div {
    margin: auto 0;
}
```

### 垂直居中元素高度不定

``` js
<div class="box3">
    <div>垂直居中元素高度不定</div>
</div>

.box3 {
    height: 50px;
    display: table;
}
.box3 >div{
    display: table-cell;
    vertical-align: middle;
}
```

### 垂直居中元素高度固定
``` js
<div class="box4">
    <div>垂直居中元素高度固定</div>
</div>

.box4 {
    height: 50px;
    position: relative;
}
.box4 > div {
    position: absolute;
    top: 0;
    bottom: 0;
    margin: auto 0;
    height: 25px;
}
```
[在线预览 垂直居中](https://codepen.io/zhangfangchao/pen/zYOPgWZ)
-->

## 水平垂直居中

<h3>绝对定位 + auto</h3>

```js
<div class="box1">
    <div>垂直居中布局</div>
</div>

.box1 {
    height: 100px;
    background-color: pink;

    position: relative;
}
.box1 > div {
    width: 100px;
    height: 25px;
    background-color: greenyellow;

    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    top: 0;
    margin: auto;
}
```

<!-- <h3>绝对定位 + 负margin</h3>

``` js
<div class="box2">
    <div>垂直居中布局</div>
</div>

.box2  {
    width: 100%;
    height: 100px;
    position: relative;
    background-color: pink;
}
.box2 > div {
    width: 100px;
    height: 20px;
    position: absolute;
    top: 50%;
    left: 50%;
    margin-left: -50px;
    margin-top: -10px;
    background: greenyellow;
}
``` -->

<h3>纯 Flex</h3>

```js
<div class="box4">
    <div>纯 Flex 水平垂直居中</div>
</div>

.box4 {
    height: 100px;
    background: pink;

    display: flex;
    justify-content: center;
    align-items: center;
}
.box4 > div {
    background: greenyellow;
}
```

<h3>Flex + margin</h3>

```js
<div class="box3">
    <div>垂直居中布局</div>
</div>

.box3 {
    height: 100px;
    background: pink;

    display: flex;
}

.box3 > div{
    margin: auto;
    background: greenyellow;
}
```

<h3>网格布局</h3>

```js
<div class="box5">
    <div>网格布局</div>
</div>

.box5 {
    width: 100%;
    height: 100px;
    background: pink;

    display: grid;
}
.box5 > div {
    height: 20px;
    background: yellowgreen;

    justify-self: center; // 水平居中
    align-self: center; // 垂直居中
}
```

### 绝对定位 + transform

```js
.father {
    position: relative;
}
.son {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
```

[在线预览 水平垂直居中](https://codepen.io/zhangfangchao/pen/wvwpBrZ)

<h3>优缺点汇总</h3>

| 布局方式        |     优点     |                       缺点                        |
| --------------- | :----------: | :-----------------------------------------------: |
| 绝对定位 + auto |   兼容性好   |               需要知道子元素的宽高                |
| 纯 flex 布局    | \*简单、方便 | 存在兼容性问题，其实对比过利弊之后，还是推荐 flex |
| flex + margin   |  简单、方便  |                  存在兼容性问题                   |
| 网格 布局       |   代码量少   |                  兼容性不如 flex                  |
| 绝对定位 + transform       |   css3 新属性   |                  兼容性不是很好                  |

## 左右两栏布局

<h3>浮动布局</h3>

```js
<div class="box">
    <div class="box-left">left box</div>
    <div class="box-right">right box</div>
</div>
```

CSS：

```js
.box {
   height: 200px;
}

.box > div {
   height: 100%;
}

.box-left {
   width: 200px;
   float: left;
   background-color: green;
}

.box-right {
   width: 100%;
   background-color: red;
}
```

[在线预览 浮动布局](https://codepen.io/dmljc/pen/aboVJNJ)

<h3>定位布局</h3>

```js
.box {
    height: 200px;
    position: relative;
}

.box > div {
    height: 100%;
}

.box-left {
    width: 200px;
    background-color: green;
    position: absolute;
    top: 0;
    left: 0;
}

.box-right {
    width: calc(100% - 200px);
    background-color: red;
    position: absolute;
    top: 0;
    right: 0;
}
```

[在线预览 定位布局](https://codepen.io/zhangfangchao/pen/jONaLqN)

<h3>calc 计算宽度</h3>

```js
.box {
    height: 200px;
}

.box > div {
    height: 100%;
}

.box-left {
    width: 200px;
    float: left;
    background-color: green;
}

.box-right {
    width: calc(100% - 200px);
    float: right;
    background-color: red;
}
```

[在线预览 calc 计算宽度](https://codepen.io/zhangfangchao/pen/OJLOgaE)

<h3>flex 布局</h3>

```js
.box {
    height: 200px;
    display: flex;
}

.box > div {
    height: 100%;
}

.box-left {
    width: 200px;
    background-color: green;
}

.box-right {
    flex: 1;
    overflow: hidden;
    background-color: red;
}
```

[在线预览 flex 布局](https://codepen.io/zhangfangchao/pen/BaBmdBG)

<h3>表格布局</h3>

```js
.box {
    height: 200px;
    display: table;
    width: 100%;
}

.box > div {
    height: 100%;
    display: table-cell; /*  display: table-cell 属性指让标签元素以表格单元格的形式呈现，类似于td标签 */
}

.box-left {
    width: 200px;
    background-color: green;
}

.box-right {
    background-color: red;
}
```

[在线预览 表格布局](https://codepen.io/zhangfangchao/pen/YzKExGL)

<h3>网格布局</h3>

```js
.box {
    height: 200px;
    width: 100%;
    display: grid;
    grid-template-rows: 200px;
    grid-template-columns: 200px auto;
}

.box > div {
    height: 100%;
}

.box-left {
    width: 200px;
    background-color: green;

}

.box-right {
    background-color: red;
}
```

[在线预览 网格布局](https://codepen.io/zhangfangchao/pen/ZEzaJvB)

<h3>优缺点汇总</h3>

| 布局方式  |        优点         |                         缺点                         |
| --------- | :-----------------: | :--------------------------------------------------: |
| 浮动 布局 |   简单，兼容性好    |          浮动后脱离文档流，必要时需清除浮动          |
| 定位 布局 |        快捷         | 布局已经脱离文档流，意味着下面的子元素也要脱离文档流 |
| calc 布局 |   CSS3 新增，简单   |                    存在兼容性问题                    |
| flex 布局 | \* 简单、方便、快速 |                    存在兼容性问题                    |
| 表格 布局 |      兼容性好       |  会阻挡浏览器渲染引擎的渲染顺序，延迟页面的生成速度  |
| 网格 布局 |      代码量少       |                    存在兼容性问题                    |

<!-- 参考文章：https://www.cnblogs.com/chengzp/p/layout.html -->

<!-- https://juejin.im/post/5c64d15d6fb9a049d37f9c20#heading-11 -->
