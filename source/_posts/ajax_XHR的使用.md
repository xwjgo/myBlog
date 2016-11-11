title: ajax中XHR对象的使用
tags: ajax
categories: 前端
toc: true
date: 2016-7-25 12:12:12
---

关于ajax中XHR对象的使用，以后就写在这里，慢慢完善。

## 什么是ajax

ajax，是**Asynchronous JavaScript + XML**的简写，这种技术使得无需刷新页面即可从服务器取得数据，可以带来更好的用户体验。但是，ajax通信与数据格式无关，也就是说，通信数据不一定是XML。

## XMLHttpRequest对象

一个完整的ajax请求过程如下：

{% codeblock lang:javascript %}
// 创建一个请求对象

var xhr = new XMLHttpRequest();

// 监听请求状态
// 监听函数必须在调用open()方法之前，能够确保跨浏览器兼容性

xhr.onreadystatechange = function () {
    if (xhr.readyState === 4) {
        if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304) {

            // 无论响应内容什么格式，都会保存到responseText中
            // 如果响应数据不是XML，则responseXML为null

            console.log(xhr.responseText);
        } else {
            console.log("Request was unsuccessful:" + xhr.readyState);
        }
    }
}

// 启动请求，以备发送

xhr.open('GET', 'test.txt', true);

// 发送请求，send()方法要接收一个参数作为发送数据
// 如果不需要发送数据，则必须传入null，因为这个参数对某些浏览器是必须的

xhr.send(null);
{% endcodeblock %}


## 程序说明

open()方法接收三个参数：请求类型、请求的url、是否异步发送请求（true表示是）。

send()方法接收一个参数，即要作为请求主体发送的数据。如果不需要发送数据，则传入null，以保证浏览器兼容性。

客户端在收到响应后，响应数据会填充到XHR对象的属性中，比如：

{% codeblock lang:html %}
resopnseText         // 作为响应主体被返回的数据
responseXML         // 如果响应数据为XML，则保存在此属性。否则，将为null
status              // 响应的HTTP状态码
{% endcodeblock %}

一般来说，HTTP返回的状态码为200,即status属性值为**200**时，表明响应已经成功返回。此外，状态码为**304**表示请求的资源并没有被修改，可以直接使用浏览器中缓存的版本。

HTTP状态码由3位数字构成，其中首位数字定义了状态码的类型。关于HTTP状态码说明如下：

{% codeblock lang:html %}
1XX     // 信息类，表示收到浏览器请求，服务器正在进一步处理
2XX     // 成功，表示用户请求被正确接收，理解和处理
3XX     // 重定向，表示请求没有成功，客户必须采取进一步动作
4XX     // 客户端错误，表示客户端提交信息有误。比如404 NOT Found表示请求的文件不存在
5XX     // 服务器错误，表示服务器不能完成对请求的处理
{% endcodeblock %}

## readyState

我们发送的是异步请求，所以JavaScript可以继续执行而不必等待回应。

此时，可以检测XHR对象的**readyState**属性，该属性表示请求/响应过程的当前活动阶段。这个属性取值如下:

{% codeblock lang:html %}
0       // 未初始化。尚未调用open()方法
1       // 已经启动。已经调用open()方法，但是未调用send()方法
2       // 已经发送。已经调用send()方法，但是还没收到响应
3       // 已经接收到部分响应数据
4       // 完成。已经接收到全部响应数据
{% endcodeblock %}

只要readyState属性由一个值变成另外一个值，就会触发**readystatechange**事件。

我们可以通过以下代码来验证：

{% codeblock lang:html %}
xhr.onreadystatechange = function () {
 console.log(xhr.readyState);
}
{% endcodeblock %}

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20160721214829.png)

## 进度事件

xhr对象还可以监听响应的进度，共有以下几个事件：

{% codeblock lang:javascript %}
loadstart               // 在接受到响应数据的第一个字节时触发
progress                // 在接受响应期间持续不断地触发
error                   // 在请求发生错误时触发
abort                   // 因为调用abort()而终止连接时触发
load                    // 在接收到完整的响应数据时触发
{% endcodeblock %}

## 超时设定

xhr对象的**timeout属性**，可以设置请求在等待响应多少毫秒之后就终止。

{% codeblock lang:javascript %}
xhr.timeout = 1000;
xhr.ontimeout = function () {
    console.log('timout...');
};
{% endcodeblock %}



