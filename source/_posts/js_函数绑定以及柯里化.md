title: 函数柯里化
tags: javascript
categories: 前端
toc: true
date: 2016-7-1 12:12:12
---

## 什么是函数的柯里化？

**将一个函数与其参数的子集绑定到一个闭包中，并且返回这个闭包。**

通俗地说，**只传递给函数一部分参数来调用它，让它返回一个函数去处理剩下的参数**。

假设你有这样一个拼接字符串的函数，代码如下：

{% codeblock lang:javascript %}
'use strict';

var getUrl = function (protocal, domain, path) {
  return protocal + '://' + domain + '/' + path;
};
{% endcodeblock %}

这是一个再普通不过的函数，它需要我们提供3个参数，并且返回最终拼接的url。

但是结合实际来看，我们通常不需要每次都设置**protocal**和**domain**参数，因为他们不是经常变化的。怎么办呢？也许我们会想把该函数改成单参的不就行了吗？

{% codeblock lang:javascript %}
var getUrl = function (path) {
  return 'http://abc.com/' + path;
};

getUrl('http', 'www.xwjgo.com', 'index.html');  //http://www.xwjgo.com/index.html
{% endcodeblock %}

这样改存在一些问题，比如我们要给站点加上SSL，我们总不能把第一个参数再放回去。所以，这个问题正确解决方式应该是[柯里化](http://www.tuicool.com/articles/QzqmqeM);

## 柯里化的实现

柯里化可以使用[Function.prototype.bind()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)来实现，通常我们不需要重复造轮子，不过为了加深对柯里化的了解，这里将通过自己的函数来实现柯里化。

{% codeblock lang:javascript %}
// ES5

'use strict';

// 将要被柯里化的函数
var getUrl = function (protocal, domain, path) {
  return protocal + '://' + domain + '/' + path;
};

var currying = function (fn) {
  // 获取currying中除了fn的其他参数
  var args = Array.prototype.slice.call(arguments, 1);
  // 返回一个闭包，用于再次接收可变参数
  return function () {
    // 获取第二次调用函数的参数
    var innerArgs = Array.prototype.slice.call(arguments);
    // 将两次调用函数的参数合并到一起
    var allArgs = args.concat(innerArgs);
    // 返回fn函数的执行结果
    return fn.apply(null, allArgs);
  };
};

var getUrlByPath = currying(getUrl, 'https', 'www.abc.com');
var getUrlByDomainAndPath = currying(getUrl, 'http');

getUrlByPath('index.html');                        // https://www.abc.com/index.html
getUrlByDomainAndPath('www.123.com', 'abc.html');  // http://www.123.com/abc.html
{% endcodeblock %}

以上代码在ES6中将会更加简介：

{% codeblock lang:javascript %}
// ES6

'use strict';

var getUrl = (protocal, domain, path) => protocal + '://' + domain + '/' + path;

var currying = (fn, ...rest) => {
  // 返回一个函数，用于再次接收可变参数
  return (...innerRest) => {
    // 将两次调用函数的参数合并到一起
    var allArgs = rest.concat(innerRest);
    // 传入所有参数到fn，并返回fn执行结果
    return fn(...allArgs);
  };
};

var getUrlByPath = currying(getUrl, 'https', 'www.abc.com');
var getUrlByDomainAndPath = currying(getUrl, 'http');

getUrlByPath('index.html');                        // https://www.abc.com/index.html
getUrlByDomainAndPath('www.123.com', 'abc.html');  // http://www.123.com/abc.html
{% endcodeblock %}

## [Function.prototype.bind()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)实现

{% codeblock lang:javascript %}
'use strict';

var getUrl = (protocal, domain, path) => protocal + '://' + domain + '/' + path;
      
var getUrlByPath = getUrl.bind(null, 'https', 'www.hyy.com');
var getUrlByDomainAndPath = getUrl.bind(null, 'http');

getUrlByPath('docs/currying.html');                  // https://www.hyy.com/docs/currying.html
getUrlByDomainAndPath('www.abc.com', 'index.html');  // http://www.abc.com/index.html
{% endcodeblock %}

bind函数会**创建一个新函数**，称为绑定函数。新函数与被调函数**具有相同的函数体**。

当新函数被调用时，其中的**this指向bind函数的第一个参数**，而后续的可选参数会**被插入到新函数参数列表的开始位置**。

## 柯里化的作用

上面说明了柯里化的实现和原理。那么，柯里化有什么作用呢？它的作用主要体现在以下三个方面：

---
1. 参数复用
2. 延迟执行
3. 使程序更加纯粹
---

其中参数复用，通过上面的代码已经可以解释。

而延迟执行是因为，currying函数返回一个**待执行的函数，而不是函数的执行结果**。

第三点可以参考[这篇文章](http://www.tuicool.com/articles/MZ3Yrma)。









