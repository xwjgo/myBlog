---
title: express的中间件
tags: express
categories: node.js
toc: true
date: 2016-9-23 12:12:12
---

最近在开发一个简单的购书平台，后端采用node.js\+express\+mongoDB\+mogoose来向前端输出数据（api），而前端则采用vue.js以及相关vue插件（比如vue-resource和vue-router等）来实现逻辑和UI。

等这个小Demo完成之后，我会再从头捋一遍，并且整理下这个过程中遇到的技术难点。而今天，就简单总结以下对**express框架中间件**的学习和理解。

## express简介

### 什么是express

[express](http://www.expressjs.com.cn/)是基于node.js的一个web开发框架，它提供了丰富的http方法和任意排列组合的中间件，让我们可以方便地创建健壮、友好的API。

### express安装

我们可以通过以下两条命令，在我们的项目中安装express：

{% codeblock lang:javascript %}
$ npm init
$ npm install express --save       //安装express并且将依赖写入package.json
{% endcodeblock %}

这样我们只是局部安装了express，项目的目录结构还未能初始化，我们需要手动建立。

现在，我们有了**express-generator**这个工具，它可以帮我们快速**创建一个应用的骨架**：

{% codeblock lang:javascript %}
// 全局安装express-generator
$ npm install express-generator -g
// 创建一个命名为myapp的应用
$ express myapp
$ cd myapp
// 安装所有依赖
$ npm install
// 启动项目，supervisor会监听项目代码变化，并且自动帮我们重启项目
$ supervisor app.js
{% endcodeblock %}

## express中间件

从本质上来说，Express就是在调用各种中间件。

### 什么是中间件

[中间件](http://www.expressjs.com.cn/guide/using-middleware.html)（Middleware）是**一个函数**，它可以访问**请求对象（req）、响应对象（res）**，和**web应用中处于请求-响应循环流程中的中间件**，一般被命名为**next**变量。

中间件函数的形式如下；

{% codeblock lang:javascript %}
function (req, res, next)
{% endcodeblock %}

给某个路径应用中间件函数的典型场景如下：

{% codeblock lang:javascript %}
app.use('/api', function (req, res, next)) {
  console.log(req.baseUrl);
  next();
});
{% endcodeblock %}

### 中间件的功能

---
1. 执行任何代码
2. 修改请求和响应对象
3. 终结请求-响应循环
4. 调用堆栈中的下一个中间件
---

### 应用级中间件

应用级中间件绑定到**app对象**上，使用**app.use()**和**app.METHOD()**来加载。其中，METHOD是需要处理的**http请求**的方法，如GET、POST、PUT、DELETE等，**全部小写**。

---

我们先来看app.use()。

express中通过[use方法](http://www.expressjs.com.cn/4x/api.html#app.use)来注册中间件。

而use方法允许**将请求网址写在第一个参数**。这代表，之后**请求路径匹配这个参数，后面的中间件才会生效**。

当然也可以不写网址，那么只要**该中间件在请求-响应流程中**，那么该中间件**总会被执行**。

---

再来看一下app.METHOD()形式的路由方法。

它的基本使用模式如下：

{% codeblock lang:javascript %}
// 其中callback的行为类似中间件
// 其唯一的区别就是这些回调函数可以调用next('route')方法而略过其他路由回调函数
app.METHOD(path, [callback...], callback);
{% endcodeblock %}

**app.all()**是一个特殊的路由方法，没有任何HTTP方法与其对应，它的作用是**对于一个路径上的所有请求加载中间件**。

{% codeblock lang:javascript %}
// 只要请求路径为/secret，无论是GET、POST或是其他请求，都执行中间件函数
app.all('/secret', function (req, res, next) {
  console.log('secret...');
  next(); 
});
{% endcodeblock %}

---

下面是一个基本的使用场景：

{% codeblock lang:javascript %}
var app = express();

// 没有挂载路径的中间件，应用的每个请求都会执行该中间件
app.use(function (req, res, next) {
  console.log('Request Type:', req.method);
  next();
});

// 挂载至/user/:id的中间件，任何指向/user/:id的请求都会执行它
app.use('/user/:id', function (req, res, next) {
  res.send('hello...');      // 此次请求-响应的循环结束，后面的中间件不会执行
});

// 一个中间件栈，对指向/home的GET请求作出处理
app.get('/home', function(req, res, next) {
  console.log('Request URL:', req.originalUrl);
  next();                   // 调用中间件栈中的下一个中间件
}, function (req, res, next) {
  res.end('end...');        // 此次请求-响应的循环结束，后面中间件不会执行
});

// 只有上面路径没有匹配到，才执行这个中间件
// 参数中的*表示对所有路径有效
app.get('*', function(req, res, next) {
  response.writeHead(404, { "Content-Type": "text/plain" });
  response.end("404 error!");
}); 
{% endcodeblock %}

### 路由级中间件

路由级中间件和应用级中间件一样，只是它绑定的对象为**express.Router()**。

{% codeblock lang:javascript %}
var router = express.Router();
{% endcodeblock %}

路由级中间件使用**router.use()**或者**router.VERB()**加载。

上面的应用场景可以改写为以下的样子：

{% codeblock lang:javascript %}
var app = express();
var router = express.Router();

// 没有挂载路径的中间件，通过该路由的所有请求都会执行该中间件
router.use(function (req, res, next) {
  console.log('Request Type:', req.method);
  next();
});

// 挂载至/user/:id的中间件，任何指向/user/:id的请求都会执行它
router.use('/user/:id', function (req, res, next) {
  console.log('id:' + req.params.id);
  res.send('hello...');      // 此次请求-响应的循环结束，后面的中间件不会执行
});

// 一个中间件栈，对指向/home的GET请求作出处理
router.get('/home', function(req, res, next) {
  console.log('Request URL:', req.originalUrl);
  next();                   // 调用中间件栈中的下一个中间件
}, function (req, res, next) {
  res.end('end...');        // 此次请求-响应的循环结束，后面中间件不会执行
});

// 只有上面路径没有匹配到，才执行这个中间件
router.get('*', function(req, res, next) {
  response.writeHead(404, { "Content-Type": "text/plain" });
  response.end("404 error!");
}); 

// 将路由挂载至应用
// 相当于为/api/home和/api/user/:id这两个请求路径，指定了回调函数
app.use('/api', router)

{% endcodeblock %}

另外，**router.route()**方法可以接受访问路径作为参数，它等同于**app.route()**，它其实是express.Router()的缩写形式。

通过**router.route()或者app.route()**，我们可以**链式地处理对一个路径的不同请求**。

{% codeblock lang:javascript %}
app.route('/login')
  .get(function(req, res) {
    res.send('get login success...');
  })
  .post(function(req, res) {
    res.send('post login success...');
  });
{% endcodeblock %}

### 错误处理中间件

错误处理中间件和其他中间件类似，不过必须使用4个参数，而不是3个。其使用场景如下：

{% codeblock lang:javascript %}
app.use(function(err, req, res, next) {
  console.error(err.stack);
  res.status(500).send('error...');
});
{% endcodeblock %}

### 内置中间件

[express static()]()是Express唯一内置的中间件。它主要负责在Express应用中托管静态资源。

{% codeblock lang:javascript %}
express.static(root, [options]);
{% endcodeblock %}

其中参数root指**提供静态资源的根目录**。
options为可选参数，详细可以参考[这里](http://www.expressjs.com.cn/4x/api.html#express)。

### 第三方中间件

第三方中间件需要我们手动安装对应的node模块，并且在应用中加载。
比如cookie-parser这个中间件：

{% codeblock lang:javascript %}
$ npm install cookie-parser --save
{% endcodeblock %}

然后在应用中使用：

{% codeblock lang:javascript %}
var express = require('express');
var cookieParser = require('cookie-parser');

// 加载用于解析cookie的中间件
var app = express();
app.use(cookieParser());
{% endcodeblock %}

[这里](http://www.expressjs.com.cn/resources/middleware.html)可以查看经常用到的第三方中间件。

## 跳过中间件

如果想要在中间件栈中跳过某个中间件，我们可以调用`next('route')`方法将控制权交给下一个路由。

不过要注意的是，next('route')**只对使用app.VERB()或者router.VERB()加载的中间件有效。**

{% codeblock lang:javascript %}
// 一个中间件栈，处理指向 /user/:id 的 GET 请求
app.get('/user/:id', function (req, res, next) {
  // 如果 user id 为 0, 跳到下一个路由
  if (req.params.id == 0) next('route');
  // 否则将控制权交给栈中下一个中间件
  else next(); //
}, function (req, res, next) {
  // 渲染常规页面
  res.render('regular');
});

// 处理 /user/:id， 渲染一个特殊页面
app.get('/user/:id', function (req, res, next) {
  res.render('special');
});
{% endcodeblock %}

完。






