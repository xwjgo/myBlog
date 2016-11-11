---
title: 浅谈Flex布局
tags: css
categories: 前端
toc: true
date: 2016-9-15 10:08:03
---

最近学习Flex布局，写了一个骰子的小案例，运行结果如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E9%AA%B0%E5%AD%90.png)

它的实现代码如下：

<p data-height="600" data-theme-id="0" data-slug-hash="EgKQOp" data-default-tab="css" data-user="xwj_pen" data-embed-version="2" class="codepen">See the Pen <a href="http://codepen.io/xwj_pen/pen/EgKQOp/">EgKQOp</a> by xwj_pen (<a href="http://codepen.io/xwj_pen">@xwj_pen</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

这里就总结下写这个案例中过程中涉及的Flex布局以及其他一些CSS属性：

## 什么是Flex布局

Flex是Flexible Box的缩写，意思是**"弹性布局"**。

无论是块级元素还是行内元素，都可以使用弹性布局：

{% codeblock lang:css %}
display: flex;
{% endcodeblock %}

但是在Webkit内核的浏览器上，比如加上**-webkit**前缀。

注意：设为Flex布局之后，子元素的<span style="color: red;">float、clear和vertical-align属性将失效。</span>

## 基本概念

如下图所示，采用Flex布局的元素，称为Flex容器（flex container），简称**“容器”**。

它的所有子元素自动成为容器成员，称为Flex项目，简称**“项目”**。

![](http://7xvlvo.com1.z0.glb.clouddn.com/flex%E5%AE%B9%E5%99%A8.png)

容器默认存在两根轴：**水平的主轴（main axis）和垂直的交叉轴（cross axis）**。

项目**默认按照主轴排列**，当`flex-direction`属性设置为`column`或者`column-reverse`时，主轴变为垂直方向。

## 容器的6个属性

有以下6个属性可以设置在容器上。

### flex-direction

flex-direction属性决定主轴的方向（即项目的排列方向）

{% codeblock lang:javascript %}
row（默认值）       // 主轴为水平方向，起点为左端
row-reverse         // 主轴为水平方向，起点为右端
cloumn              // 主轴为垂直方向，起点在上沿
column-reverse      // 主轴为垂直方向，起点在下沿
{% endcodeblock %}

### flex-wrap

flex-wrap属性定义，如果一个容器中项目很多，如何换行。

{% codeblock lang:javascript %}
nowrap（默认值）   // 不换行
wrap               // 换行，第一行在最上方（就和普通的word文字排版一样）
wrap-reverse       // 换行，第一行在最下方
{% endcodeblock %}

### flex-flow

flex-flow属性是`flex-direction`属性和`flex-wrap`属性的简写形式，**默认值为row nowrap**。

{% codeblock lang:javascript %}
.box{
  flex-flow: <flex-direction> || <flex-wrap>
}
{% endcodeblock %}

### justify-content

justify-content属性定义了**项目在主轴上的对齐方式**。

{% codeblock lang:javascript %}
flex-start（默认值）      // 沿着主轴，左对齐
flex-end                  // 沿着主轴，右对齐
center                    // 沿着主轴，居中
space-between             // 沿着主轴，两端对齐，项目之间间隔相等
space-around              // 沿着主轴，每个项目两侧间隔相等（类似加上左右margin)
{% endcodeblock %}

### align-items

algin-items属性定义**项目在交叉轴上如何对齐**。

{% codeblock lang:javascript %}
stretch（默认值）         // 如果项目没有设置高度或者设置为auto，将占满整个容器的高度
flex-start                // 交叉轴起点对齐
flex-end                  // 交叉轴终点对齐
center                    // 交叉轴中点对齐
baseline                  // 第一个项目的基线对齐
{% endcodeblock %}

### align-content

align-content属性定义了**多根轴线的对齐方式**。如果容器只有一根轴线，那么该属性不起作用。

{% codeblock lang:javascript %}
stretch（默认值）         // 项目占满整个交叉轴
flex-start                // 交叉轴起点对齐
flex-end                  // 交叉轴终点对齐
center                    // 交叉轴中点对齐
space-between             // 与交叉轴两端对齐，轴线之间间隔相等
space-around              // 每根轴线两侧间隔相等
{% endcodeblock %}

## 项目的6个属性

以下6个属性设置在项目上：

### order

order属性定义项目的排列顺序。**数值越小，排列越靠前，默认为0**。

{% codeblock lang:javascript %}
order: <integer>
{% endcodeblock %}

### flex-grow

flex-grow属性定义项目的**放大比例**。**默认为0，即如果存在剩余空间也不放大**。

{% codeblock lang:javascript %}
flex-grow: <number>
{% endcodeblock %}

如果某个项目的flex-grow属性值为1，则**它将剩余空间全部占满**。

如果所有项目都为1，那所有项目将**均分全部剩余空间**。

如果一个项目的flex-grow属性值为2，其他项目都为1，则前者将占据的剩余空间比其他项多一倍。

### flex-shrink

flex-shrink属性定义了项目的缩小比例，**默认为1，即如果空间不足，该项目将缩小**。

{% codeblock lang:javascript %}
flex-shrink: <number>
{% endcodeblock %}

如果所有项目的flex-shrink属性都为1，当空间不足时，都将等比例缩小。

如果一个项目的felx-shrink属性为0，其他项目都为1，则空间不足时，前者不缩小。

### flex-basis

flex-basis属性定义了**在分配多余空间之前，项目占据的主轴空间**。默认值为auto，即项目本来大小。

它可以**为某个项目分配固定大小的空间**。

{% codeblock lang:javascript %}
flex-basis: <length> | auto;
{% endcodeblock %}

### flex

flex属性是**flex-grow,flex-shrink,flex-basis**的简写，默认值是`0,1,auto`。后两个属性可选。

该属性有两个快捷值：`auto(1,1,auto)`和`none(0,0,auto)`。

### align-self

align-self属性**允许单个项目有与其他项目不一样的对齐方式**，可以覆盖align-items属性。

默认值为auto，表示继承父元素的align-items属性，如果没有父元素，则等同于stretch。

## 其他一些CSS属性

当然在写这个案例的过程中，除了Flex布局，还涉及到了其他一些常用属性。整理如下：

### box-sizing

box-sizing属性用来**改变默认的CSS盒模型对元素宽高的计算方式**。

它的取值主要有以下：

{% codeblock lang:javascript %}
content-box  // 宽度和高度分别应用到元素的内容框，在宽度和高度之外绘制元素的内边距和边框
border-box   // 为元素设定和宽度和高度决定了元素的边框盒，内容框的宽高需要减去边框和内边距
{% endcodeblock %}

### box-shadow

骰子的阴影效果就是由box-shadow设置内阴影来实现的。

box-shadow允许向元素盒子添加阴影，包括**内阴影**和**外阴影**。

其属性如下：

{% codeblock lang:javascript %}
h-shadow       //必需。水平阴影的位置。允许负值
v-shadow       //必需。垂直阴影的位置。允许负值
blur           //可选。模糊的距离
spread         //可选。阴影的尺寸
color          //可选。阴影的颜色
inset          //可选。将外部阴影改为内部阴影。
{% endcodeblock %}

如下代码，较为清晰地展示了内阴影的实现：

{% codeblock lang:css %}
.box {
box-shadow: 10px 0px 2px red inset,
            -10px 0px 2px blue inset,
            0px 10px 2px green inset,
            0px -10px 2px orange inset;
}
{% endcodeblock %}

其实现效果为；

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E8%AE%BE%E7%BD%AE%E5%86%85%E9%98%B4%E5%BD%B1.png)

### 背景渐变

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E8%83%8C%E6%99%AF%E6%B8%90%E5%8F%98.png)

线性渐变使用**linear-gradient**函数，而径向渐变使用**radial-gradient**函数，重复的渐变可以使用**repeating-linear-gradient**和**repeating-radial-gradient**函数。

关于渐变更加详细的使用，请移步[这里](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Using_CSS_gradients)。

## 参考资料

[Flex布局教程：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)
[Flex布局教程：实例篇](http://www.ruanyifeng.com/blog/2015/07/flex-examples.html)
