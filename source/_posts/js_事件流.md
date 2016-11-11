title: js中的事件流
tags: javascript
categories: 前端
toc: true
date: 2016-8-05 12:12:12
---

## 什么是事件流

事件流是描述页面上接收事件的顺序。
事件流包括两种：**捕获**和**冒泡**。
捕获的过程是文档内容由大到小的过程；冒泡的过程则是相反，是由小到大的过程。

![](http://i4.buimg.com/f29c3f9d48ac63ad.png)

任何事件产生时，如点击一个按钮，将从最顶端的容器开始（一般是html节点）。浏览器会向下遍历DOM树找到触发事件的元素。一旦浏览器找到该元素，事件流就进入事件目标阶段，该阶段完成后，浏览器就会沿着DOM树向上冒泡直到最顶层的容器，看看是否有其他元素需要同一个事件。

现在浏览器默认采用的是事件冒泡，及从下到上。

![](http://i4.buimg.com/f9b9101b9267333c.png)

## 一个简单的例子

> html代码

{% codeblock lang:html %}
<html>
  <head>
    <meta charset = "UTF-8">
    <title>js事件流</title>
  </head>
  <body>
    <ul>
      <li>js事件流</li>
    </ul>
  </body>
</html>
{% endcodeblock %}

> js代码

{% codeblock lang:javascript %}
const $ = (el) => document.querySelector(el);
$('html').addEventListener("click", () => {
  console.log("html");
},true);
$('body').addEventListener("click", () => {
  console.log("body");
},true);
$('ul').addEventListener("click", () => {
  console.log("ul");
},true);
$('li').addEventListener("click", () => {
  console.log("li");
},true);
{% endcodeblock %}

> 在浏览器中点击 li 标签，结果结果为：

![](http://i4.buimg.com/c03918fc1fb73523.png)

> 如果把 addEventListener 中第三个参数**去掉**或者改为**false**，则结果为：

![](http://i4.buimg.com/f188242af49d12ec.png)

## 关于addEventListener

addEventListener 函数接收3个参数，第一个是事件类型，第二个是事件处理函数，第三个是指**是否在事件捕获阶段调用事件处理方法。**

true：是。false：否。

默认为false，表示只在事件冒泡阶段调用事件处理函数。

## 事件流的应用

事件流常应用于事件委托。
