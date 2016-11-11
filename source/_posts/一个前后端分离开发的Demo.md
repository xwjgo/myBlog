title: 一个前后端分离开发的Demo
tags: [node.js, express,vue.js,mongodb]
categories: node.js
toc: true
date: 2016-09-12 10:08:03
---

断断续续折腾了一周多的时间，终于实现了一个前后端分离开发的小项目（没有做UI，所以有点丑），代码可以点击[这里](https://github.com/xwjgo/nodeApp)访问到。

这篇文章就回顾一下整个项目的架构流程以及遇到的技术难点。

## 项目介绍

这是一个简单的购书平台，主要实现了以下功能：

---
1. 注册
2. 登陆/登出
3. 添加书籍
4. 查看书籍
5. 添加购物车并且结算
---

后端基于**node.js**，使用了**express**框架，以及**mongodb、mongoose、express-session**等功能模块。

前端基于**vue.js**，使用了**vue-resource、vue-router**这两个插件。

为了模拟前后端分离开发的环境，后端运行在8888端口，而前端运行在8080端口。

前后端主要通过交换json数据来实现相关功能，总结来说，**后端只负责提供数据（数据库读写），前端负责呈现数据**。

## 后端环境搭建

### 安装express

{% codeblock lang:javascript %}
$ mkdir myapp
$ cd myapp
$ npm init
$ npm install express --save
{% endcodeblock %}

它会为我们将根目录初始化以下：

{% codeblock lang:javascript %}
common           // 文件夹，存放可复用的工具模块
public           // 文件夹，存放前端build的静态资源
routes           // 文件夹，存放路由模块
app.js           // 项目入口文件
package.json     // 项目配置
{% endcodeblock %}

### 启动mongoDB

mongoDB的安装可以参考其官方文档，对于不同的系统环境，有不同的安装方法：

我的系统为**Ubuntu 16.04**，其安装方法可以看[这里](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)。

安装完成之后，可以通过以下命令来启动mongoDB服务器：

{% codeblock lang:javascript %}
$ sudo mongod
{% endcodeblock %}

当然，mongodb自带一个JavaScript shell，常用的命令有以下几个：

{% codeblock lang:javascript %}
$ mongo          // 进入shell
$ show dbs       // 查看当前有有哪些数据库，默认只有一个test
$ db             // db指向当前数据库
$ use <dbname>   // 使db变量指向某个数据库
$ show collections               // 查看当前数据库中的集合
$ db.test.users.find().pretty()  // 查询test数据库中users集合中的所有文档，并且以较为规整的格式输出
{% endcodeblock %}

关于mongoDB更多的操作（**CRUD**），可以参考其[官方教程](https://docs.mongodb.com/manual/introduction/)

### 安装express中间件

express内置的中间件只有一个，那就是`express.static()`，它负责在express应用中托管静态资源。

现在我们在项目中安装我们需要的几个第三方中间件：

{% codeblock lang:javascript %}
$ npm install mongoose --save     // 一个更便捷的操作mongoDB数据的对象模型工具 
$ npm install morgan  --save      // 在控制台打印HTTP请求的记录，用于调试
$ npm install body-parser  --save // 解析http请求体，并且覆盖到原来的req.body属性
$ npm install express-session  --save   // 创建会话，并将会话数据挂载到req.session，保存在服务端（除了connect.id)
{% endcodeblock %}

以上几个中间件，都可以在[这里](http://www.expressjs.com.cn/resources/middleware.html)找到，详细配置可以看其文档。

### app.js

安装完中间件之后，我们就开始写我们的项目入口文件，代码如下：

{% codeblock lang:javascript %}
// 引入第三方中间件
var express = require('express');
var mongoose = require('mongoose');
var morgan = require('morgan');
var bodyParser = require('body-parser');
var session = require('express-session')
// node内置模块
var path = require('path');
// 自定义模块
global.dbHelper = require('./common/dbHelper'); // 用来操作数据库中的集合
var routes = require('./routes/api')            // 用来操作路由

var app = express();

// 调用mongoose，连接数据库test3
mongoose.connect('mongodb://localhost/test3')
// 调用morgan中间件，在控制台打印http请求记录
app.use(morgan('dev'));
// 调用body-parser中间件，将传来的json数据解析为对象，并覆盖到req.body
app.use(bodyParser.json());
// 调用express-session中间件，connect.id过期时间设置为10分钟
app.use(session({
  secret: 'fuckyou',
  resave: false,
  saveUninitialized: true,
  cookie: {
    maxAge: 1000*60*10
  }
}));
// 调用express.static中间件，指定静态资源目录，默认为pubic下的index.html
app.use(express.static(path.join(__dirname, 'public')));

// 调用路由模块，处理不同请求，并且将app作为参数传入
routes(app);

// 监听8888端口
app.listen(8888);
{% endcodeblock %}

### 路由模块

我们将路由模块都放置在`routes`文件夹下，并且由`api.js`统一管理，其代码如下：

{% codeblock lang:javascript %}
module.exports = function (app) {
  // 因为前后端分离开发，需要处理跨域，所以对所有请求均设置响应头
  app.use(function (req, res, next) {
    res.setHeader('Access-Control-Allow-Origin', 'http://192.168.0.101:8080');
    res.setHeader('Access-Control-Allow-Headers', 'content-type');
    res.setHeader('Access-Control-Allow-Credentials', 'true');
    next();
  });
  // 如果是预请求，就直接终结请求-响应循环
  app.options('*', function (req, res, next) {
    res.end();
  })

  // 输出的api有以下5个，为了方便统一管理，我们都统一在这里
  require('./register')(app);        // 处理注册页（/register）发来的请求
  require('./login')(app);           // 处理登录页（/login）发来的请求
  require('./bookslist')(app);       // 处理图书列表页（/home）发来的请求
  require('./cart')(app);            // 处理购物车页（/cart）发来的请求
  require('./logout')(app);          // 处理登出（/logout）请求
};
{% endcodeblock %}

### 数据库设计

数据库我们选用了[MongoDB](http://mongodb.github.io/node-mongodb-native/2.2/quick-start/)，它是一个**开源的NoSQL数据库**，相比于MySQL那样的关系型数据库，它更加适合在**数据规模很大、事务性不强的场合**下使用。同时，它也是一个对象数据库，**没有表、行等概念，只有集合和文档的概念**，数据格式为**JSON**。

[mongoose](http://mongoosejs.com/docs/index.html)是一个针对MongoDB操作的对象模型库，它**封装了MongoDB对文档的一些增删改查**等常用方法。可以点击[这里](http://www.cnblogs.com/aaronjs/p/4489354.html)查看基础示例。

我们将对数据库的Schema定义在`common/models.js`文件中，其代码如下：

{% codeblock lang:javascript %}
// 定义并导出数据库中几个集合的模型（Schema）
// 注意：Schema对应的数据集合会默认加上s，即books，users，carts
module.exports = {
  book: {
    name: String,
    description: String,
    price: Number
  },
  user: {
    username: String,
    password: String
  },
  cart: {
    username: String,
    bookname: String,
    number: Number,
    price: Number,
    status: Boolean
  }
};
{% endcodeblock %}

并且将操作Schma的公共方法封装在了`common/dbHelper.js`文件中，代码如下：

{% codeblock lang:javascript %}
var mongoose = require('mongoose');
var Schema = mongoose.Schema;
var models = require('./models');

// 将models中定义的所有模式（Schema）编译为模型（Model)
for (var m in models) {
  var typeSchema =  new Schema(models[m]);
  mongoose.model(m, typeSchema);
}

// 返回编译后的对应type的模型（该模型可直接对数据库增删改查）
var _getModel = function (type) {
  return mongoose.model(type);
};

// 将getModel方法暴露出去
module.exports = {
  getModel: function (type) {
    return _getModel(type);
  }
};
{% endcodeblock %}

### 添加路由

我们拿`register.js`来举例，它用来专门处理前端注册（register）页面发起的post请求，其代码如下：

{% codeblock lang:javascript %}
module.exports = function (app) {
  // 获取User的Model，用于对集合users的增删改查
  var User = global.dbHelper.getModel('user');

  app.route('/register')
    .post(function (req, res) {
      var postData = req.body;
      // 先从集合users中检索是否存在该用户名
      User.findOne({username: postData.username}, function (err, doc) {
        if (doc) {
          // 如果用户名已经存在
          res.status(500).send('用户名已经存在');
        }else {
          // 如果用户名不存在，那么就该用户数据保存至数据库
          User.create({
            username: postData.username,
            password: postData.password
          }, function (err, doc) {
            if (err) {
              res.status(500).send('服务端保存数据出错');
            } else {
              res.status(200).send('注册成功');
            }
          });
        }
      });
    });
};
{% endcodeblock %}

### 测试

写完路径`/register`对应的路由模块后，我们通过firefox的一款插件`HttpRequester`来模拟一下前端的post请求，模拟结果如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/post%E6%88%90%E5%8A%9F%E5%95%A6.png)

然后，我们再进入mongdb shell，查看是否将user信息存入数据库：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BF%9D%E5%AD%98%E6%88%90%E5%8A%9F.png)

数据保存成功！其他路由模块也是一样的实现原理，这里就不一一写出。

至此，我们已经可以正确地响应前端的各种http请求，后端的开发流程大致就是这样，它主要负责提供api，向前端发送数据。

## 前端环境搭建

前端的搭建基于[vue.js](http://cn.vuejs.org/)，是一个小型的单页应用，并且使用了[vue-resource](https://github.com/vuejs/vue-resource)和[vue-router](https://github.com/vuejs/vue-router)这两个插件。

### vue-cli快速搭建应用

[vue-cli](https://github.com/vuejs/vue-cli)是vue官方提供的一个脚手架工具，可以快速地帮我们构建项目，并且具有**单文件Vue组件、热加载、保存时检查代码，单元测试**等功能。

其使用方式如下：

{% codeblock lang:javascript %}
$ npm install -g vue-cli        // 全局安装vue-cli
$ vue init webpack myProject    // 创建一个基于webpack模板的新项目myProject
$ cd myProject
$ npm install                   // 安装依赖
$ npm run dev                   // 启动项目，可在本地8080端口访问
{% endcodeblock %}

我们可以点击[这里](http://vuejs-templates.github.io/webpack/structure.html)来查看项目的架构。

> 官方的npm可以运行比较慢，建议换成[淘宝的npm镜像](https://npm.taobao.org/)，可以提高模块下载速度。以后就可以用`cnpm`来替代`npm`了（除了publish命令）。

### 插件安装

**vue-resource**可以通过XMLHttpRequest或者JSONP发起请求并且处理响应。
**vue-router**是vue.js的官方路由，可以将路由映射到各个组件。

现在，我们可以在项目中安装并使用它们：

{% codeblock lang:javascript %}
$ cnpm install vue-resource --save    // cnpm是淘宝npm镜像
$ cnpm install vue-router --save
{% endcodeblock %}

然后我们在项目入口文件中使用他们：

{% codeblock lang:javascript %}
// 导入插件
import VueResource from 'vue-resource'
import VueRouter from 'vue-router'
// 全局注入
Vue.use('VueResource')
Vue.use('VueRouter')
{% endcodeblock %}

### vue-resource基本使用

现在我们将实现`Register.vue`组件，其功能就是让用户输入用户名和密码，然后通过发送**POST请求**来实现注册功能。

假设我们通过vue的双向绑定，获取了用户注册信息user，那么我们可以通过vue-resource提交数据，代码实现如下：

{% codeblock lang:javascript %}
// 导入要提交的地址registerApi，这里为http://192.168.0.101:8888/register
import {registerApi} from '../api'

// 要提交的数据
var postData = {username: this.user.username, password: this.user.password}

// 方法一：使用当前实例的$http来提交数据，它返回一个Promise对象
this.$http.post(registerApi, postData).then(successCallBack, errorCallBack)

// 方法二：使用当前实例的$resource来提交数据，同样返回一个Promise对象
var resource = this.$resource(registerApi)
resource.save(registerApi, postData).then(successCallBack, errorCallBack)
{% endcodeblock %}

更加详细的CURD操作，可以看[这里](http://www.cnblogs.com/keepfool/p/5657065.html)。

### vue-router基本使用

vue-router的主要作用就是**将路由映射到各个组件**。

现在假设我们有两个组件：`Register.vue`,`Login.vue`，那么我们可以这样使用vue-router。

在`main.js`中添加如下代码：

{% codeblock lang:javascript %}
// 创建一个路由器实例，可以传入配置参数
var router = new VueRouter()

// 定义路由规则，每条路由规则映射到一个组件
router.map({
  '/register': {
    component: Register
  },
  '/login': {
    component: Login
  }
})

// 启动应用
// 路由器会创建一个App实例，并且挂载到选择符#app的元素上
router.start(App, '#app')
{% endcodeblock %}

完成以上工作后，我们就可以在`App.vue`组件上使用`vue-router`来进行导航：

{% codeblock lang:javascript %}
<div id="app">
  <!-- 使用指令 v-link 进行导航 -->
  <ul>
    <li v-link = "{path: '/register'}">注册</li>
    <li v-link = "{path: '/login'}">登陆</li>
  </ul>
  <!-- 路由外链 -->
  <router-view></router-view>
</div>
{% endcodeblock %}

关于vue-router更详细的使用，可以点击[这里](http://router.vuejs.org/zh-cn/basic.html)。

## 一些难点

在写这个小项目过程中踩过一些**坑**，有的真的是很基础的错误，但是我有时不得不花费大量的时间来发现并改正，所以我将它们总结在这里。

### 解决跨域

完成上面的任务后，我们还不能实现前后端的分离开发，因为还存在跨域问题。

**使用CORS解决跨域问题**在我之前的博客中总结过，可以在[这里](http://xwjgo.github.io/2016/07/29/js_CORS%E8%A7%A3%E5%86%B3%E8%B7%A8%E5%9F%9F%E8%AF%B7%E6%B1%82%E9%97%AE%E9%A2%98/)查看。

这里我们也是通过CORS来解决的跨域，代码之前已经呈现，这里再写出核心代码：

{% codeblock lang:javascript %}
app.use(function (req, res, next) {
  // 之前设置为了星号，但是因为要发送带凭证的请求，必须要指定域名，详见下一个小结
  res.setHeader('Access-Control-Allow-Origin', 'http://192.168.0.101:8080');
  res.setHeader('Access-Control-Allow-Headers', 'content-type');
  res.setHeader('Access-Control-Allow-Credentials', 'true');
  next();
});
app.options('*', function (req, res, next) {
  res.end();
});
{% endcodeblock %}

### vue-resource发送带凭证的请求

后台使用了`express-session`中间件来管理会话，它的工作原理如下：

---

当客户通过浏览器（前端）第一次请求服务器（后端）时，服务器会在http的响应头中，通过`set-cookie`字段来给浏览器端设置一个默认叫做`connect.sid`的cookie，用作该用户的**身份标识**。

此后，浏览器每次对服务器端发起http请求时，都会自动带上这个cookie。服务端在接收到这个`connect.sid`字段后，会判断其是否过期（maxAge)，以及是否正确，并且对客户端做出不同的响应。

---

在这个项目中，当某个客户发来请求时，浏览器端需要根据请求头中是否有`connect.sid`来判定用户登陆与否，所以，我们的**请求必须要带上cookie**。即**要发送带凭证的请求**。

而在vue-resource中，解决方式如下：

{% codeblock lang:javascript %}
// main.js
// 发送带凭证的请求，因为每次请求都要带上cookie中的connect.id
Vue.http.options.credentials = true
{% endcodeblock %}

### vue-router: Cannot find element: #app

在使用vue-router时，遇到这个错误提示，其由`main.js`中的以下代码引起：

{% codeblock lang:javascript %}
router.start(App, '#app')
{% endcodeblock %}

以上代码可以理解为: **将App组件，挂载到#app元素上**。

而错误很明确，即**找不到一个id为`#app`的元素**。

所以，我们需要对vue-cli给我们自动生成的`index.html`文件作出修改：**将`<app></app>`改为`<div id="app"></app>`**。

### mongoose不能重复编译

使用mongoose编写`dbHelper.js`时，出现了错误：**Cannot overwrite 'users' model once compiled.**

谷歌一番之后，发现原因：**重复对某个一模式（Schma）进行了编译**。

[这里](http://stackoverflow.com/questions/19051041/cannot-overwrite-model-once-compiled-mongoose)有一个解决方式，做了很详细的说明。

### 关于跨域的一点思考

在这个demo的开发过程中成功实现了简单的**前后端分离**，那是因为前后端都运行在本地，**只有有端口号不同**。这导致**后端的8888端口是可以通过响应头中的set-cookie字段来向前端的8080端口写入connect.sid这个cookie的**。

但是如果前后端运行在完全不同的两个域，比如`http://www.abc.com`和`http://www.123.com`，那这样的情况下，跨域就有些复杂了。

### 更改vue项目的build目录

由于前后端在同一个主机，所以在没有解决跨域之前，我是通过将前端build目录配置为后端的`public`文件夹下的，这样，我每次修改完前端代码之后，直接`npm run build`，就会直接build到后端，这样，直接直接在后端运行项目了。

那么如何更改bulid的出口呢？

打开`config/index.js`，修改如下两行代码：

{% codeblock lang:javascript %}
index: path.resolve(__dirname, '../../myapp/public/index.html'),
assetsRoot: path.resolve(__dirname, '../../myapp/public'),
{% endcodeblock %}

当然，这样的效率是不高的，如果能解决跨域，不建议这么做。

## 推荐工具

- HttpRequester —— 火狐浏览器的一个插件，可以方便地发起各种http请求
- [showDoc](http://www.showdoc.cc/) —— 一个在线生成api文档的工具
- [rap](http://thx.github.io/RAP/index_zh.html) —— 阿里在用的一个可视化接口管理工具
- [Apizza](http://apizza.cc/) ——  极客专属的api管理工具

以上不免有纰漏之处，仅作参考。
