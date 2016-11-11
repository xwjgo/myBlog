---
title: line-height和vertical-align
tags: css
categories: 前端
toc: true
date: 2016-7-28 12:12:12
---

虽然写过不少css代码，可是在实战中关于css的问题依旧很多，比如有时想利用`line-height`来实现垂直居中的效果，但是结果却经常令我费解。

这篇文章就整理一下css中关于`line-height`和`vertical-align`的概念和知识。

## 几个基本概念

在进入正题之前，我们需要先了解几个概念：

### 顶线、中线、基线、底线

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E5%9F%BA%E7%BA%BF.png)

上图标识出了顶线、中线、基线、底线的位置，而他们正好对应`vertical-align`中的`top`、`middle`、`baseline`、`bottom`这几个属性属性值。

其中，font-size即为顶线到底线之间的垂直距离。

### 行高、行距、半行距

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E8%A1%8C%E8%B7%9D%E8%A1%8C%E9%AB%98.png)

其中行高（line-height）为文本行基线间的垂直距离。

行距为上一行的底线和下一行的顶线之间的距离，而半行距为行距的一半。

### inline box、line box

inline box（行内框）：每个行内元素会生成一个行内框。行内框是浏览器渲染模型中的一个概念，无法现实出来。**行内框的高度等于font-size**。设置line-height时，行内框的高度不会变，改变的是行距。

line-box（行框）：行框指本行中一个虚拟的矩形框，由该行中行内框组成。**行框高度取决于本行中所有行内框高度的最大值以及line-height**。当由多行内容时，每一行都有自己的行框。

![](http://7xvlvo.com1.z0.glb.clouddn.com/inline%20box.png)

### 替代元素和非替代元素

替代元素就是指**浏览器根据元素的属性来判断具体要显示什么内容**，比如`img`标签，浏览器是根据`src`的属性值来显示图片内容的。

常见的替代元素还有`input`、`select`、`video`等。这些元素，浏览器都不直接显示其内容，而是根据某个属性值来显示对应的内容，比如根据`input`元素的`type`属性值来决定显示哪一种类型的的input。

---

而对于非替代元素，**浏览器则直接显示元素所包含的内容**，比如`p`、`label`等。

## line-height

下面我们来进入正题，来聊一聊`line-height`这个属性。

### 定义

[MDN line-height](https://developer.mozilla.org/zh-CN/docs/Web/CSS/line-height)给出了以下定义：

---
1. 对于块级元素，line-height指定了元素内部的line boxes的最小高度。
2. 对于非替代行内元素（span、a等），line-height用于计算line box的高度。
3. 对于替代行内元素（img，input等），line-height没有影响。
---

总结来说，**line-height设置行框（line-box）的高度**，如果一个标签没有设置height属性，那么其最终表现的高度一定是由line-height起作用。

### 取值

| 取值             | 说明                                                       | 继承            |
| ---              | ---                                                        | ---         |
| **normal**       | 取决与用户代理，桌面浏览器默认约为1.2                      | 继承1.2，然后乘以本身font-size   |
| **[ number ]**     | line-height值为font-size乘以[ number ]               | 继承[ number ]，然后乘以本身font-size|
| **[ length ]**     | length为具体长度，比如20px或者50rem                     | 继承20px                          |
| **[ percentage ]** | 百分比，比如200%，若font-size为100px，则line-height为200px | 继承计算后的200px          |

而最为推荐的方式是，使用`[ number ]`来为line-height赋值。

## vertical-align

[vertical-align](http://www.w3school.com.cn/cssref/pr_pos_vertical-align.asp)属性用来**设置行内元素的垂直对齐方式**。

该属性定义**行内元素的基线**相对于**该元素所在行的基线**的垂直对齐。

它可能的取值有以下：

{% codeblock lang:css %}
baseline（默认）         // 元素放置在父元素的基线上。
sub                      // 垂直对齐文本的下标
super                    // 垂直对齐文本的上标
top                      // 把元素的顶端和行中最高元素的顶端对齐
text-top                 // 把元素的顶端和父元素字体的顶端对齐
middle（常用）           // 把此元素放置在父元素的中部
bottom                   // 把元素与父元素底端对齐
length                   
%（与line-height有关）   // 使用line-height乘以百分比来排列此元素
inherit                  // 从父元素继承
{% endcodeblock %}

比如我们可以使用vertical-align来将一个块级元素中，所有内联元素垂直居中：

刚开始是这样的：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E6%9C%AA%E5%9E%82%E7%9B%B4%E5%B1%85%E4%B8%AD.png)

当我们加入以下样式之后：

{% codeblock lang:css %}
main *{
    vertical-align: middle;
}
{% endcodeblock %}

这样就实现了垂直居中。

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E5%9E%82%E7%9B%B4%E5%B1%85%E4%B8%AD.png)

## 应用

假设我们在一个div中添加了一个图片，并给这个div设置了一个背景色。HTML如下：

{% codeblock lang:html %}
<div>
  <img src="img/2.jpg" height="200px"/>xwj
</div>
{% endcodeblock %}

然后，我们在浏览器中运行，发现结果如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E5%A4%9A%E5%87%BA%E6%9D%A5%E4%B8%80%E9%83%A8%E5%88%86.png)

其中图片的高度为200px，但是div的高度为什么却是203px？div的高度**为什么会多出来几个像素**呢？

上面这个令人费解的表现，其实就可以使用`line-height`和`vertical-align`来解释。

### 幽灵空白节点

这个名词取自[这篇文章](http://www.zhangxinxu.com/wordpress/2015/08/css-deep-understand-vertical-align-and-line-height/)，它表达的意思为；

> 在HTML5文档声明下，**块状元素内部的内联元素行为表现，就好像块状元素内部还有一个看不见的宽度为0的空白节点**，这个空白的节点的表现就像一个**匿名的inline-box**一样。

假如我们在img标签后面加上几个字符，并且把这几个字符的`font-size`属性调为200px，那么表现会更加明显：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E6%9B%B4%E6%98%8E%E6%98%BE%E7%9A%84%E8%A1%A8%E7%8E%B0.png)

如果我们再给图像加上100px的`margin-bottom`，那么会变成以下的样子：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E5%8A%A0%E4%B8%8Amargin-bottom.png)

从上面可以看出，vertical-align默认值是baseline，也就是基线对齐。

### 解决问题

既然知道了以上问题是由于基线对齐引起的，那么我们就可以对症下药，来把图片下面多的几个像素去掉吧。

有以下几种解决方案：

{% codeblock lang:css %}
/*方法一:让 vertical-align失效，因为其只对行内元素起作用*/
img{
  display: block;
}

/*方法二:使用其他vertical-align值,比如bottom/middle/top等.*/
img{
  vertical-align: bottom;
}

/*方法三:直接修改line-height值*/
div{
  line-height: 1px;
}

/*方法四:line-height为相对单位,font-size间接控制，本质为修改line-height*/
div{
  font-size: 0px;  
}
{% endcodeblock %}

以上几种方法，都能去掉图片下面多出来的几像素，其中方法一迫使后面的字符换行，方法四会让后面文字消失，方法二和方法三则会影响后面文字的表现：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E6%96%B9%E6%B3%95%E4%BA%8C.png)
![](http://7xvlvo.com1.z0.glb.clouddn.com/%E6%96%B9%E6%B3%953.png)

## 个人总结

以下是我个人的总结，正确性有待考究：

**对于块级元素，如果里面有行内元素（可能是img，也可能是inline-block的span等），则第一个元素的基线就是父元素的基线，其后的行内元素都会依照此基线对齐**。

**如果第一个元素内部有inline boxes，比如一些字符，那么该元素基线为字母`x`的下边缘，如果第一个元素没有inline boxes，比如一个空的span，那么该元素的基线就是margin的底边缘**。

## 最后一个例子

最后通过一个例子来说明以上总结：

html如下，两个span，一个包含字符，一个没有包含；

{% codeblock lang:html %}
<div id="main">
  <span>xwj</span>
  <span></span>
</div>
{% endcodeblock %}

css如下：

{% codeblock lang:css %}
#main{
  background-color: #0bc;
  font-size: 100px;
}
#main span{
  display: inline-block;
  border: solid 1px #0AA284;
  background-color: #BBBBBB;
}
#main span:first-child{
  width: 200px;
  height: 200px;
}
#main span:last-child{
  width: 300px;
  height: 300px;
}
{% endcodeblock %}

最终的结果如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E6%9C%89%E5%AD%97%E7%AC%A6%E5%9C%A8%E5%89%8D.png)

如果我们作出一点点改变呢？比如，将两个span标签的顺序调换一下，那么将结果如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E6%9C%89%E5%AD%97%E7%AC%A6%E5%9C%A8%E5%90%8E.png)

哈哈，是不是很有趣。

那么，这并非我们的期望，我们如何是他们表现正常呢？答案当然还是`line-height`和`vertical-align`，大家自己考虑一下吧~

完结。























