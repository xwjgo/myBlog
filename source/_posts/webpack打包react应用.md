title: webpack打包react应用
tags: [webpack, react]
categories: 前端
toc: true
date: 2017-3-1 12:12:12
---

前段时间学习redux写了一个简单的react单页应用，使用了jsx，es6，less等语法，并且引用了一些第三方的npm模块，比如`redux`，`react-redux`，`lodash`，`moment`等，构建工具选择的是`webpack2`，这里就来整理一下其中关于webpack打包的相关知识。

源代码可以点击[这里](https://github.com/xwjgo/redux-learning)访问。

## 使用loaders

每引入一种loader，就相当于告诉了webpack：当遇到某种类型的文件时，就使用该loader来解析。

### less

项目中使用了less，解析less文件需要使用3个loader:`style-loader`，`css-loader`，以及`less-loader`。

{% codeblock lang:javascript %}
$ npm install style-loader css-loader less-loader less --save-dev
{% endcodeblock %}

其中less-loader用来将less解析为css，css-loader能够解析`import`以及`url()`的语法，而style-loader可以将生成的css插入到HTML文档中（注意最后只生成一个bundle.js）。

所以`webpack.config.js`中解析less的配置如下：

{% codeblock lang:javascript %}
// options.module
module: {
    rules: [
        {
            test: '/\.less$/', // 要解析的文件类型
            use: ['style-loader', 'css-loader', 'less-loader'] // 要使用的loaders，注意顺序
        }
    ]
}
{% endcodeblock %}

### jsx以及es6

因为webpack默认是可以解析`commonJS`，`AMD`以及`es6`的模块语法的，所以即使不使用任何的laoder，`import`以及`require`语法也可以直接使用。但是除了`import/export`这些模块语法，webpack是不会动你其他代码的。所以，如果我们在项目中使用es6、JSX等语法，还需要添加`babel-loader`。

{% codeblock lang:javascript %}
$ npm install babel-core babel-loader babel-preset-es2015 babel-preset-react --save-dev
{% endcodeblock %}

webpack.config.js中解析jsx以及es6的配置如下：

{% codeblock lang:javascript %}
// options.module
module: {
    rules: [
        test: /\.(js|jsx)$/,
        exclude: /node_modules/, // 这里面的不解析
        loader: 'babel-loader',
        query: [  // 相当于babel的options
            presets: ['react', 'es2015']
        ]
    ]
}
{% endcodeblock %}

## 提取css

通过上面的配置，我们可以愉快地在项目中使用es6以及jsx语法了，并且最终会打包出唯一的一个`bundle.js`文件，然后再html中引用bundle.js即可。

但是，还有一点不足。项目中所有的样式都被打包到了bundle.js文件中，所以浏览器必须等待bundle.js完全记载完毕之后，才可以给文档加上样式，这样一来，我们就没办法利用浏览器的**异步加载css**以及**并行加载**的优势了。

### extract-text-webpack-plugin

所以，我们可以把所有的css打包到一个单独的样式文件中。这就需要一个webpack插件，叫做`extract-text-webpack-plugin`。

{% codeblock lang:javascript %}
$ npm install extract-text-webpack-plugin
{% endcodeblock %}

在修改我们的webpack.config.js如下：

{% codeblock lang:javascript %}
const ExtractTextPlugin = require('extract-text-webpack-plugin');
module.exports = {
    entry: './index.js',    // 项目入口文件
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'public'),
    },
    module: {
        rules: [
               {
                    test: /\.less$/,
                    use: ExtractTextPlugin.extract({
                         fallback: 'style-loader',
                         use: ['css-loader', 'less-loader']
                    })
               }
        ]
    },
    plugins: [
        new ExtractTextPlugin('index.css')  // 最终会在public目录下生成index.css
    ]
};
{% endcodeblock %}

现在在控制台运行`webpack --config webpack.config.js`，我们会发现生成了一个bundle.js，还有一个index.css，可以将它们直接引入html中去。

### 一点副作用

extract-text-webpack-plugin可能会让css无法实现热重载（后面会讲到），所以我们最好只在**生产环境**中使用这个插件。

详细信息参见[这里](https://github.com/css-modules/webpack-demo/issues/8)。

## 提取第三方js模块

上面我们把css单独提取出来了，但是`bundle.js`依旧可能是一个很大的文件，因为我们将一些第三方库打包了进去，比如`lodash`和`react`等。

这样做依旧有一个缺点： 每次修改了项目代码，哪怕只是一点点，整个bundle.js必须重新打包，浏览器也必须重新从服务器拉取新的bundle.js，这样就**无法利用浏览器缓存静态资源的优势**。

假如我们将不经常变化的第三方js库和我们的项目代码分开打包为`vendor.js`和`app.js`，那么当我们修改了项目代码并重新打包，只有`app.js`发生了变化，这样浏览器就只需要重新从服务器拉取新的app.js就可以了。

接下来我们就来这样做。

### 多个入口

修改`options.entry`以及`options.output`如下：

{% codeblock lang:javascript %}
entry: {
    app: './index.js',    // 项目代码入口
    vendor: ['react', 'react-dom', 'redux', 'react-redux'] // 第三方js库
}
output: {
    filename: '[name].[chunkHash]js',  // 对于多入口，这里必须使用占位符
    path: path.join(__dirname, 'public'),
}
{% endcodeblock %}

再次打包，在public目录下生成了两个js文件（当然还有index.css）：`vendor.[hash]>js`以及`app.[hash].js`。文件名中的`[hash]`代表的是根据文件的内容所生成的hash值。

### CommonsChunkPlugin

但是我们发现，**第三方的js库竟然同时打包进了这两个js文件中**。这是因为webpack将从每个入口文件开始分析依赖，这将得到多颗独立的依赖树，并且分别打包，互不干扰。如果我们指向将第三方模块打包到vendor.js中，我们还需要使用`CommonsChunkPlugin`这个插件。

{% codeblock lang:javascript %}
const webpack = require('webpack');

// options.plugins
plugins: [
    new webpack.optimize.CommonsChunkPlugin({
        name: 'vendor'    // 指定模块的名称
    })
]
{% endcodeblock %}

运行webpack，发现公共的js库只打包在了`vendor.[hash].js`中，而app.[hash].js减小了很多。

### manifest

现在终于可以安心的该项目代码了，可是，当我们改了一点项目代码，重新使用webpack来打包时，发现public目录下竟然又多出了两个文件`app.[hash].js`以及`vendor.[hash].js`。这与我们预想的不同，vendor应该保持不变才对，为什么又重新打包了一份呢？这也意味着我们没办法享受到浏览器缓存带来的好处，因为vendor的hash变了，我们每build一次，浏览器必须要重新加载vendor。

翻阅wepack的官方文档，对这个问题解释如下：

每次build，webpack都生成一些**webpack运行时代码**来帮助webpack完成它的工作。当只打包出一个文件时，这些运行时代码自然就在这个文件中，如果我们像上面那样打包出两个文件，那么运行时代码将被打包到公共模块中，正是vendor.js中。

为了解决这个问题，我们需要把这些运行时代码单独提到一个manifest.js中。尽管我们又多打包了一个文件，但是我们的vendor.js再也不会发生变化了，这使得我们可以享受浏览器缓存带来的性能提升。

修改配置文件如下：

{% codeblock lang:javascript %}
// options.plugins
plugins: [
    new webpack.optimize.CommonsChunkPlugin({
        names: ['vendor', 'manifest']    // 指明公共模块的名称
    })
]
{% endcodeblock %}

## 按需加载

通过以上配置，我们已经可以使用webpack来完成最基本的打包工作了。但是，浏览器加载的资源总比它实际用得到的多，如果我们能在应用中实现**按需加载（懒加载）**，那么，我们应用的性能可能再进一步提升。

我们有两种方式可以实现按需加载，下面将一一介绍，不过在此之前，可以先看一下这篇[文章](https://dzone.com/articles/lazy-loading-es2015-modules-in-the-browser)。

### import()

import()方法是es6中关于模块加载的一个规范，用于在**运行时动态加载模块**。该函数将一个模块名当作参数，并且返回一个Promise对象，这意味着当模块加载没有成功时，我们可以作出一些处理。

webpack把`import()`当作一个代码分离点，并且把通过`import`引入的模块，单独打包到一个chunk。

下面看import()在项目代码中的使用：

{% codeblock lang:javascript %}
// 只有点击提交按钮之后才会触发，如果不点击，我们希望里面的js模块不用不加载
handleSubmit () {
    import('moment')    // 代码分离点，将单独打包到一个chunk中
        .then(moment => {
            console.log(moment().format('MMMM Do YYYY, h:mm:ss a'));
        })
        .catch(err => {
            console.log('模块加载失败！')
        });
}
{% endcodeblock %}

现在我们来配置webpack，让其支持这种动态加载模块语法。首先，需要安装`bebel-plugin-syntax-dynamic-import`插件：

{% codeblock lang:javascript %}
// 注意，我们之前已经安装过bable-core babel-loader等插件
$ npm install babel-plugin-syntax-dynamic-import --save-dev
// 同时安装moment.js
$ npm install moment --save
{% endcodeblock %}

然后配置webpack如下：

{% codeblock lang:javascript %}
// options.module.rules
{
    test: /\.(js|jsx)$/,
    exclude: /node_modules/,
    use: [{
        loader: 'babel-loader',
        query: {
            presets: ['react', 'es2015'],
            plugins: ['syntax-dynamic-import']    // 注意这里的变化
        }
    }]
}
{% endcodeblock %}

运行wepack，发现除了`manifest.js`，`vendor.js`，`app.js`，`index.css`以外，还生成了`0.js`，打开0.js可以发现，其中包含了moment模块的所有代码，这样，在应用初次加载的时候，是不需要加载`0.js`的，只有当用户点击了提交按钮之后，0.js才会从浏览器拉取并执行。

此外我们还可以给这些按需加载的代码块规定名称；

{% codeblock lang:javascript %}
// options.output
output: {
    filename: [name].js,
    path: path.join(__dirname, 'public'),
    chunkFileName: '[id].chunk.js'    // 这样生成的代码块名称将是 0.chunk.js
}
{% endcodeblock %}

### require.ensure()

除了es6的提案中的`import()`，webpack还为我们提供了`require.ensure()`的方式，并且这种方式不需要额外安装插件，还可以指定打包的chunk名称。

其api如下：

{% codeblock lang:javascript %}
require.ensure(dependencies: String[], callback: function(require), chunkeName: string);
// 一个require.ensure指定一个代码分离点
// dependencies 依赖模块列表，只会被加载，加载完毕之后，才可以在callback中执行
// callback 所有dependencies加载完毕之后，callback函数会立即执行
// chunkName 对应webpack配置中output.chunkFilename中的[name]占位符，并且如果require.ensure指定同一个chunkName，那么将合并为一个chunk
{% endcodeblock %}

我们要明白的一点是：**require.ensure()只会加载dependencies中指定的依赖模块，但不执行。如果想执行，可以在callback中调用require**。

现在来看一个案例，假设在`testModules`目录下有3js文件，名称和内容如下：

{% codeblock lang:javascript %}
// a.js
console.log('aaaaaaaaaaaaaaaaaaaa');
// b.js
console.log('bbbbbbbbbbbbbbbbbbbb');
// c.js
console.log('ccccccccccccccccccc');
{% endcodeblock %}

下面我们再另外一个js模块中的`handleSubmit()`中按需加载这3个模块：

{% codeblock lang:javascript %}
// 场景1，dependencies为空数组
// 将a和b两个模块单独打包到1.test.chunk.js中
// a和b模块都执行了
require.ensure([], require => {
    require('../testModules/a');
    require('../testModules/b');
}, 'test'); // 'test'对应chunkFilename中的[name]

// 场景2，a和b两个模块作为dependencies
// 将a,b,c模块打包到1.test.chunk.js文件中
// 但是只有b和c模块执行了，a模块没有执行
// a模块如果想执行，必须require()
require.ensure(['../testModules/a', '../testModules/b'], require => {
    require('../testModules/b');
    require('../testModules/c');
})
{% endcodeblock %}

以上还需要在实践中多多体验。

## 更愉快的开发

到目前位置，我们可以使用es6，less，jsx等语法以及按需加载的功能了，但是对于开发环境来讲，这种体验还不是很好。因为我们每次修改完代码，都必须手动执行`webpack --config webpack.config.js`，等待webpack打包出新文件之后，我们再去浏览器刷新页面，才能看到效果。

我们想要的效果应该是这个样子：

1. 每次修改完less，按下`ctrl+s`，浏览器不会整页刷新，而是直接把新样式运用到页面中
2. 每次修改完js，保存后，浏览器自动整页刷新
3. 有source-map，方便排除bug

如果是纯前端项目，使用`webpack-dev-server`就可以实现热重载了，不过现在我们要把它整合到一个后台使用`express`的项目中。

下面我们来一一实现，先介绍热重载依靠的两个插件[webpack-dev-middleware](https://github.com/webpack/webpack-dev-middleware)和[webpack-hot-middleware](https://github.com/glenjamin/webpack-hot-middleware)。

### webpack-dev-middleware

这是一个只应该在开发环境中使用的webpack插件，它可以监听项目代码的变化，自动打包文件到内存中，并且server这些文件。
当自动编译还未完成的时候，如果浏览器已经发来了请求，那么这个请求将会被阻塞，直到文件编译完成。这看起来就像网速很慢，卡住了一样。

{% codeblock lang:javascript %}
$ npm install webpack-dev-middleware --save-dev
{% endcodeblock %}

配置webpack.dev.config.js如下；

{% codeblock lang:javascript %}
// options.output
output: {
    filename: [name].js,
    path: path.join(__dirname, 'public'),
    chunkFilename: '[id].[name].chunk.js',
    publicPath: '/assets/'    // 服务器server文件的地址，/assets可能并不实际存在磁盘中
}
{% endcodeblock %}

上面配置中的[publicPath](https://webpack.js.org/configuration/output/#output-publicpath)字段比较难理解，它指明了浏览器按需加载或者加载外部资源（图片，文件）时候的路径。在这里我们配置它，是为指明了webpack-dev-middleware serve 内存文件的路径（继续往下看）。

现在我们可以在html中这样引用资源了，并且按需加载文件的url也会指向`/assets/1.test.chunk.js`这样的。

{% codeblock lang:html %}
<script src="/assert/manifest.js"></script>
<script src="/assert/vendor.js"></script>
<script src="/assert/app.js"></script>
{% endcodeblock %}

最后，我们来正式使用`webpack-dev-middleware`:

{% codeblock lang:javascript %}
const path = requrie('path');
const express = rquire('express');
const webpack = require('webpack');
const webpackDevMiddleware = rquire('webpack-dev-middleware');
const webpackCofig = require('./webpack.dev.config.js');
const app = express();

const compiler = webpack(webpackConfig);
app.use(webpackDevMiddleware({
    publicPath: webpackConfig.output.publicPath    // 该参数必须指定，它
}));

app.get('/', function (req, res) {
    res.sendFile(path.join(__dirname, 'index.html'));
});

app.listen(3000, function () {
    console.log('listening on port 3000');
});
{% endcodeblock %}

现在我们可以在3000端口访问应用了，并且每修改完代码，按下`ctrl+s`或者直接刷新浏览器，webpack都会自动打包文件到内存中。
这样，就不用每次都运行`webpack --config webpaack.dev.config.js`了，打包速度也快了很多。

### webpack-hot-middleware

美中不足的是我们还需要手动刷新浏览器，现在我们通过`wepack-hot-middleware`来实现[HMR](http://webpack.js.org/concepts/hot-module-replacement/)。

webpack-hot-middleware只关心将一个客户端和一个服务器连接起来，并且订阅服务端资源的更新，并且将更新应用与客户端（通过webpack的HMR api）。

{% codeblock lang:javascript %}
$ npm install --save-dev webpack-hot-middleware
{% endcodeblock %}

更改webpack的配置文件:

{% codeblock lang:javascript %}
const HotMiddlewareScript = 'webpack-hot-middleware/client?reload=true';
// options.entry，每个进入点之后，都加HotMiddlewareScript
entry: {
    app: ['index.js', HotMiddlewareScript],
    vendor: ['react', 'react-dom', 'redux', 'react-redux', HotMiddlewareScript]
}
// options.plugins
plugins: [
    new webpack.HotModuleReplacementPlugin()
]
{% endcodeblock %}

配置app.js:

{% codeblock lang:javascript %}
const webpackHotMiddleware = rquire('webpack-hot-middleware');
app.use(webpackHotMIddleware(compiler));
{% endcodeblock %}

现在重启app.js，并在浏览器中访问3000端口，发现控制台出现了`[HMR] connected`字样。并且我们修改js，发现webpack会自动打包，并通知浏览器刷新。

但是，当我们**修改了less文件之后，却发现webpack只是自动打包了，新样式并没有被应用到浏览器中**。

这是为什么呢？其实前面在讲`extract-text-webpack-plugin`的时候，我们已经提到过了：extract-text-webpack-plugin并不适合在开发环境中只用，它会导致css的热重载失效。

所以，我们把webpack.dev.config.js中关于extarct-text-webpack-plugin的部分去掉之后，发现一切跟我们预想的一样！

### react-hot-loader

除此之外，针对react，还有一个可以实现热重载的工具，即`react-hot-loader`，这里就不详细介绍了，到[官网](http://gaearon.github.io/react-hot-loader/)看去吧！

### source-map

source-map可以快速帮助我们定位bug，对于提升开发效率有很好的提升。

通过webpack来实现source-map非常简单，只需要添加一行配置：

{% codeblock lang:javascript %}
// options.devtool
devtool: 'cheap-eval-source-map'
{% endcodeblock %}

devtool的值有8中，它们的特性以及适用环境都有很多不同。详细信息就看[这里](https://webpack.js.org/configuration/devtool/)。

## 最后

webpack的基本应用到这里就先告一段落，还有很多高级的特性没有涉及，以后还需要在实践中慢慢摸索。

最后，放上工作成果图：

![](http://7xvlvo.com1.z0.glb.clouddn.com/webpack.gif)


