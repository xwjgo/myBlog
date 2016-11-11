title: CORS解决跨域请求问题
tags: javascript
categories: 前端
toc: true
date: 2016-8-07 12:12:12
---
最近学习node，写了一个简单的demo：后端使用node的http模块来监听http请求，主要负责数据输出，而前端则负责请求后台api得到数据，最终渲染到页面。

所以当浏览器加载一个页面时,会有两次http请求，一次请求页面，即index.html，另外一次为请求api数据，数据为一个json对象。最终通过原生js将数据渲染到页面中。

## 问题出现

看着非常简单的一个案例，却发生了一个开发过程中非常经典的问题，即**浏览器的同源策略阻止了跨域请求资源**。

服务器运行在`http://121.249.216.190:8030`，而前端js运行在`http://127.0.0.1:8020`，所以当在前端js使用ajax方法来请求服务器数据时，浏览器就会抛出阻止跨源请求的错误。

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E7%BB%84%E7%BB%87%E8%B7%A8%E6%BA%90%E8%AF%B7%E6%B1%82.png)

解决跨域问题的方式有很多种，比如架设代理服务器、jsonp等。这次使用了CORS来解决这个问题。最终demo的服务器代码在我的github上，点击[这里](https://github.com/xwjgo/nodeServer)可以访问到。

## 什么是跨域

一张图来解释什么是跨域：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E8%B7%A8%E5%9F%9F%E6%A6%82%E5%BF%B5.png)

当**协议、子域名、主域名、端口号**中任意一个不相同时，都算作不同域。

不同域间相互请求资源，就算做**跨域**。

## [同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)

出于安全考虑，浏览器会限制脚本中发起的跨域请求。比如，使用**XMLHttpRequest**对象发起http请求就必须遵守同源策略。

注意：浏览器并非限制了http请求的发起，跨域请求可以正常发起，但是返回结果会被浏览器拦截。

## [CORS](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)

CORS，**跨域资源共享**，即**Cross-Origin Resource Sharing**，是W3C推荐的一种新的机制来让Web应用服务器能支持跨域访问控制，它的侧重点在服务器，换言之，CORS的核心就是**让服务器来确定是否允许跨域访问**。

跨源资源共享标准使得以下应用场景可以使用跨域http请求：

---
1. 使用XMLHttpRequest发起的跨域http请求
2. Web字体（CSS中通过@font-face使用跨域字体资源）
---

## 典型访问控制场景

### 简单请求

所谓的简单请求，需要同时满足以下两个条件：

---
1. 只使用**GET、HEAD或者POST**请求方法。如果使用POST向服务器传送数据，则数据类型（Content-Type）只能是**application/x-www-form-urlencoded、multipart/form-data或者text/plain**一种。
2. 不使用自定义请求头（比如X-TOKEN)。
---
Server代码如下：

{% codeblock lang:javascript %}
'use strict';

var http = require('http');

var xwj = {
  name: 'xwj',
  age: 21,
  school: 'OUC',
  love: 'play basketball'
};

http.createServer (function (request, response) {
  // 不加Access-Control-Allow-Origin响应头就无法实现跨域，*表示所有域都可以请求
  response.setHeader('Access-Control-Allow-Origin', '*');
  response.write(JSON.stringify(xwj));
  response.end();
}).listen(8030, function () {
  console.log('Server has stared...')
});
{% endcodeblock %}

Clinet代码：

{% codeblock lang:javascript %}
var xhr = new XMLHttpRequest();

xhr.onreadystatechange = function () {
  if (xhr.readState === 4) {
    if ((xhr.status >= 200 && xhr.status < 300) || xhr.status === 304) {
      console.log(responseText);
    }
  }
}

// 场景1：GET请求，不需要header，允许跨域

xhr.open('GET', 'http://121.249.216.190:8030', true);
xhr.send(null);

// 场景2：POST请求，如果设置header，则必须为指定类型，否则无法跨域

xhr.open('POST', 'http://121.249.190:8030', true);
// 此处Content-Type的值必须为text/plain、application/x-www-form-urlencoded或者multipart/form-data，其他不行
// 也可以不设置Content-Type
xhr.setRequestHeader('Content-Type', 'text/plain')
xhr.send(null);

// 场景3: DELETE请求，不允许跨域

xhr.open('DELETE', 'http://121.249.216.190:8030', true);
xhr.send(null);

// 场景4：POST请求，有自定义header，不允许跨域

xhr.open('POST', 'http://121.249.216.190:8030', true);
xhr.setRequestHeader('X-TOKEN', 'abc');
xhr.send(null);

{% endcodeblock %}

### 预请求

下面我们来探究场景3和场景4跨域请求失败的原因，比如场景4，因为我们加了一个自定义的请求头，所以导致跨域请求失败，而浏览器也告知了我们原因。

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E8%87%AA%E5%AE%9A%E4%B9%89%E8%AF%B7%E6%B1%82%E5%A4%B4.png)

但我们打开网络面板时，可以看到多了一次**OPTIONS请求**。

![](http://7xvlvo.com1.z0.glb.clouddn.com/option%E8%AF%B7%E6%B1%82.png)

OPTIONS请求是HTTP/1.1中的方法，用来获取更多的服务器信息，是一个不应该对服务器数据造成影响的方法。

并且在OPTIONS请求的请求头中，有以下两个字段：

{% codeblock lang:javascript %}
Access-Control-Request-Method: POST           // 提醒服务器跨域请求使用POST方法
Access-Control-Request-Headers: X-TOKEN       // 告知服务器跨域请求将携带一个自定义请求头X-TOKEN
{% endcodeblock %}

OPTIONS请求就是**预请求**发送的。

不同于上面的简单请求，预请求要求必须先发送一个OPTIONS请求给目的站点，来查明这个跨域请求对于目的站点是否安全可接受。

当具备以下任何一个条件，这个请求就会被当做预请求处理：

---
1. 请求以GET、HEAD或者POST以外的方式发起请求，比如**DELETE**。
2. 使用POST，但是请求数据类型为 **application/x-www-form-urlencoded，multipart/form-data或者text/plain**以外的数据类型，比如**application/json**。
3. 使用自定以请求头，比如X-TOKEN。
---

所以，场景4中的请求由于加了自定义请求头，所有该请求是一个预请求形式的跨域请求。我们可以改动服务器端的代码，使其允许这次跨域请求。

{% codeblock lang:javascript %}
http.createServer (function (request, response) {
  response.setHeader('Access-Control-Allow-Origin', '*');
  // 允许GET、POST、DELETE请求
  response.setHeader(('Access-Control-Allow-Methods:', 'GET, POST, DELETE');
  // 允许请求携带请求头X-TOKEN
  response.setHeader(('Access-Control-Allow-Headers:', 'X-TOKEN');
  response.write(JSON.stringify(xwj));
  response.end();
}).listen(8030, function () {
  console.log('Server has stared...')
});
{% endcodeblock %}

改动之后，场景3和场景4都可以完成跨域请求。

并且我们可以看到这次OPTIONS请求的响应头中包含以下字段：

{% codeblock lang:javascript %}
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET,POST,DELETE
Access-Control-Allow-Headers: X-TOKEN
{% endcodeblock %}

### 附带凭证信息的请求

一般而言，对于跨域请求，浏览器是不会发送凭证信息（HTTP Cookie和验证信息）的。

但如果将XMLHttpRequest的一个特殊标志位`withCredentials`设置为true，浏览器就将允许该请求的发送。

在带凭证的请求中，后端的响应头必须包含`Access-Control-Allow-Credentials`，并且`Access-control-Allow-Origin`不能再用\*匹配，而必须明确指明域名。

我们对clinet.js做出以下修改：

{% codeblock lang:javascript %}
// 写入cookie，用于测试
document.cookie = 'name=xwj';
document.cookie = 'age=20';

var xhr = new XMLHttpRequest();

xhr.onreadystatechange = function () {
  if (xhr.readState === 4) {
    if ((xhr.status >= 200 && xhr.status < 300) || xhr.status === 304) {
      console.log(responseText);
    }
  }
}

// 场景5：带凭证的请求

xhr.open('GET', 'http://121.249.216.190:8030', true);
// 表示发送带凭证的请求
xhr.withCredentials = true;
xhr.send(null);

{% endcodeblock %}

现在浏览器就会发送带凭证的请求，假如我们只是将`server.js`中`Access-control-Allow-Origin`的值改为`http://127.0.0.1:8020`，那么会出现以下错误。

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E5%B8%A6%E9%AA%8C%E8%AF%81%E7%9A%84%E8%AF%B7%E6%B1%82%E5%8F%91%E7%94%9F%E9%94%99%E8%AF%AF.png)

我们还需对server.js做出以下修改：

{% codeblock lang:javascript %}
http.createServer (function (request, response) {
  // 这里不能再使用*,而应该指明具体的域名
  response.setHeader('Access-Control-Allow-Origin', 'http://127.0.0.1:8020');
  response.setHeader(('Access-Control-Allow-Methods:', 'GET, POST, DELETE');
  response.setHeader(('Access-Control-Allow-Headers:', 'X-TOKEN');
  // 设置了该Header，才能接受带凭证的请求
  response.setHeader('Access-Control-Allow-Credentials', true);

  response.write(JSON.stringify(xwj));
  response.end();
}).listen(8030, function () {
  console.log('Server has stared...')
});

{% endcodeblock %}

## http响应头

这些为服务器响应客户端http请求时候的响应头：

{% codeblock lang:javascript %}
Access-control-Allow-Origin              // 允许跨域访问的域名，*表示全部
Access-Control-Allow-Headers             // 允许的白名单Header，多个用逗号分开
Access-Control-Allow-Credentials         // 是否允许带凭证的请求，true代表是
Access-Control-Allow-Methods             // 允许的请求类型，多个用逗号分隔开
Access-Control-Allow-Headers             // 允许的自定义Header，多个用逗号分开
{% endcodeblock %}

## http请求头

以下为发送http请求时的请求头：

{% codeblock lang:javascript %}
origin                                  // 告诉服务器，请求来自哪个域
Access-Control-Request-Method           // 预请求时，告诉服务器的请求方式
Access-Control-Request-Headers          // 预请求时，实际请求所携带的自定义Header

{% endcodeblock %}

## 参考资料

参考资料主要有以下：

- [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)
- [http://www.ruanyifeng.com/blog/2016/04/cors.html](http://www.ruanyifeng.com/blog/2016/04/cors.html)

---

以上就是对CORS解决跨域问题一些总结。
