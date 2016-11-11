---
title: webpack+babel加载es6模块
tags: webpack
categories: 前端
toc: true
date: 2016-9-23 12:12:12
---

## 问题出现

最近在学es6，一些新的特性，比如for-of、扩展运算符、生成器、迭代器等是**可以在一些较为流行的浏览器上直接运行的**，比如最新的firefox、chrome等都已经从不同程度上实现了对es6语法的支持。

但是，今天看了[ES6 In Depth: Modules](https://hacks.mozilla.org/2015/08/es6-in-depth-modules/)这篇文章，当我写了一个小Demo并且打算在浏览器上运行看看时，我发现，所有浏览器均不支持`import`和`export`语法。es6提供了这么好的模块化编程语法，不能运行在浏览器上太可惜啦，所以，我开始寻找解决方法。

## 解决过程

既然不能运行es6，那么将es6转化为es5应该可以吧。于是，我想到了[babel](https://babeljs.io/)这个JavaScript编译器，它可以将下一代的JavaScript转化为es5。

然而，只是将es6转化为es5还是不够的。babel提供了一个[在线的js转换器](https://babeljs.io/repl/#?evaluate=true&lineWrap=false&presets=es2015%2Cstage-2&code=import%20%7BprintColor%7D%20from%20'.%2FprintColor.js'%3B)，可以一边写es6，一边看转化后的es5。我在这个在线转换器中，输入了`import`语句，转换结果如下：

{% codeblock lang:javascript %}
// ES6
import {printColors} from './printColor.js';

// 经过babel转换后的ES5语法
'use strict';
var _printColor = require('./printColor.js');
{% endcodeblock %}

并且在页面底部显示：

> require is not supported in the browser, you need a commonjs environment such as node.js/io.js, browserify/webpack etc

意思就是：`require`属于`commonjs`规范的东西，浏览器js是不支持这个语法的，你只能在实现了`commonjs`规范的环境中运行，比如`node.js/io.js`，或者利用`browserify/webpack`等工具。

所以，仅仅使用babel将es6的import语法转化为require语句，还是不能再浏览器端运行。我们还需要实现了commonjs规范，并且能够**解析模块之间依赖关系**的工具。

这种情况下，我想到了[webpack](https://webpack.github.io/)，这个模块打包机（module bundler）。

## webpack基本使用

webpack的快速上手，可以看[其官方教程](http://webpack.github.io/docs/tutorials/getting-started/)&[使用文档](http://webpack.github.io/docs/usage.html)。

### webpack可以干什么

webpack是一个前端**模块化管理和打包工具**。它具有以下几个特点：

---
1. 它可以将许多相互耦合的模块，按照**依赖关系**，打包成符合生产环境部署的前端资源。
2. 通过各种`loader`，它可以**将任何形式的资源都视作模块**，比如commonjs模块、AMD模块、ES6模块、CSS、图片、SASS、JSON等。
3. 它可以将按需加载的模块进行代码分割，等到实际需要的时候再异步加载。
---

webpack的工作方式是：把你的项目当做一个整体，通过**一个给定的主文件**（比如`index.html`），webpack将从这个文件开始找到你项目的**所有依赖文件**，使用**`loaders`处理**他们，并且最终**打包**为浏览器可识别的静态资源。

![](http://7xvlvo.com1.z0.glb.clouddn.com/what-is-webpack.png)

上问题的解决就是使用`wbepack + babel-loader`来解决的。

### 安装

{% codeblock lang:javascript %}
// 方式一：全局安装
$ npm install webpack -g

// 方式二：在项目中局部安装
$ npm init
$ npm install webpack --save-dev
{% endcodeblock %}

我两条命令都执行了，这样webpack可以作为依赖加入到`package.json`文件中。

现在，我的项目文件夹`myApp`下面**有一个文件夹`node__modules`和一个`package.json`文件**。

### 初始化项目结构

现在我们可以将项目文件夹初始化为以下的样子：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.png)

对其中各个文件说明如下：

{% codeblock lang:javascript %}
node_modules                // 通过npm安装的模块，在安装webpack时自动创建
src                         // 存放原始数据和所写的js模块
bin                         // webpack打包后的文件
package.json                // 项目配置文件，在npm init时自动创建
webpack.config.js           // webpack配置文件
{% endcodeblock %}

其中`index.html`文件代码如下：

{% codeblock lang:html %}
<html>
  <head>
    <meta charset="UTF-8">
    <title>webpack基本使用</title>
  </head>
  <body>
    <!--引入webpack打包后的文件-->
    <script src="../bin/bundle.js"></script>
  </body>
</html>
{% endcodeblock %}

`app.js`使用了`import`语法，它作为webpack打包的入口文件：

{% codeblock lang:javascript %}
// 从printColor.js文件中导入一个方法
import {printColors} from './printColor.js';
printColors();
{% endcodeblock %}

`printColor.js`导出了一个方法：

{% codeblock lang:javascript %}
var colors = ['red', 'green', 'yello', 'black'];
export function printColors () {
  for (let color of colors) {
    document.write('<p>' + color +'</p>')
  }
}
{% endcodeblock %}

### 安装bebal-loader

loaders需要单独安装，并且需要在`webpack.config.js`下的`modules`关键字下进行配置。其配置项包括以下几项：

{% codeblock lang:javascript %}
test                  // 一个匹配loaders所处理的文件的扩展名的正则表达式，必须
loader                // loader的名称，必须
include/exclude       // 手动添加必须处理的文件（文件夹）或者屏蔽不需要处理的文件（文件夹），可选
query                 // 为loaders提供额外的设置选项，可选
{% endcodeblock %}

我们通过以下命令来安装[babel-loader](https://github.com/babel/babel-loader):

{% codeblock lang:javascript %}
// 注意加sudo
$ npm install babel-loader babel-core babel-preset-es2015 --save-dev
{% endcodeblock %}

### 配置

完成babel-loader的安装后，我们对`webpack.config.js`进行配置，最终该文件内容如下：

{% codeblock lang:javascript %}
module.exports = {
  entry:__dirname + '/src/app.js',                     // 唯一打包入口文件
  output: {
    path: __dirname + '/bin',          // 打包后文件存放的地方
    filename: 'bundle.js'              // 打包后输出文件的文件名
  },
  module: {
    loaders: [
      {
        test: /\.js$/,                              // 匹配打包文件后缀名的正则
        exclude: /(node_modules|bower_components)/, // 这些文件夹不用打包
        loader: 'babel-loader',
        query: {
          presets: ['es2015']
        }
      }
    ]
  }
}
{% endcodeblock %}

其中`__dirname`是node.js的一个全局变量，它指向当前执行脚本所在的目录。

关于babel-loader更详细的配置，可以看[这里](https://github.com/babel/babel-loader)

### 开始使用

终于完成了大部分工作，现在我们开始运行项目吧！

在命令行输入以下命令：

{% codeblock lang:javascript %}
$ webpack              // webpack将根据webpack.config.js文件中的配置来打包我们的项目。
{% endcodeblock %}

输出结果如下：

{% codeblock lang:javascript %}
Hash: e1abf01f64e61612560f
Version: webpack 1.13.1
Time: 600ms
    Asset     Size  Chunks             Chunk Names
bundle.js  2.42 kB       0  [emitted]  main
    + 2 hidden modules
{% endcodeblock %}

并且发现在`bin`目录下多出了一个`bundle.js`文件，这个文件就是webpack打包后的文件，也是我们在`index.html`中引入的文件。

然后我们在浏览器中打开`index.html`中文件，内容如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/webpackResult.png)

完成！这样我们就可以开心地写es6模块，并且让代码在浏览器中完美运行了！

### 更快捷地打包

以后，我们在修改文件后，只需要在命令行中运行`webpack`这条命令就可以了。但是，对于只局部安装webpack的情况，我们需要执行`node_modules/.bin/webpack`这样一条很繁琐的命令。

还好，npm可以引导任务执行，我们修改`package.json`后，就可以使用比如`npm start`或者`npm run <script>`这样的形式来代替原来的命令，修改后的`package.json`文件内容如下：

{% codeblock lang:json %}
{
  "name": "myapp",
  "version": "1.0.0",
  "description": "a app writed by es6",
  "main": "index.js",
  "scripts": {
    "start": "webpack"            // 相当于把npm的start命令指向webpack命令
  },
  "author": "xwj",
  "license": "ISC",
  "devDependencies": {
    "babel-core": "^6.13.2",
    "babel-loader": "^6.2.4",
    "babel-preset-es2015": "^6.13.2",
    "webpack": "^1.13.1"
  }
}
{% endcodeblock %}

现在，我们就可以运行`npm start`来打包啦~

`npm start`是一个比较特殊的脚本名称，如果对应的脚本不是`start`，想要在命令行中运行时，需要这样用`npm run {script named}`，比如`npm run dev`或者`npm run build`。

## 开发环境

当项目逐渐变大，webpcak编译时间会边长，我们可以通过参数让编译内容带有进度和颜色。

{% codeblock lang:javascript %}
$ webpack --progress --colors
{% endcodeblock %}

如果不想每次修改代码都手动重新打包，可以开启监听模式。开启监听模式后，没有变化的模块会在编译后缓存到内存中，而不会每次都重新编译，所以监听模式的速度是很快的。

{% codeblock lang:javascript %}
$ webpack --progress --colors --watch
{% endcodeblock %}

webpack其实可以使用`webpack-dev-server`基于node.js搭建一个本地服务器，并且会以监听模式自动运行webpack，在浏览器打开`http://127.0.0.1:8080`就可以访问我们的项目资源，并且可以实时监听代码变化并作出改变。

关于webpack-dev-server的详细使用，可以看[这里](http://webpack.github.io/docs/webpack-dev-server.html)。

{% codeblock lang:javascript %}
$ npm install webpack-dev-server -g

$ webpack-dev-server --progress --colors
{% endcodeblock %}
## 相关文档

- [webpack官方文档](http://webpack.github.io/docs/usage.html)
- [webpack极简入门](http://webpack.github.io/docs/tutorials/getting-started/)
- [webpack中文指南](http://zhaoda.net/webpack-handbook/index.html)
- [入门webpack](http://www.tuicool.com/articles/jaAFb22)


