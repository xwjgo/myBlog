title: Generators
tags: javascript
categories: 前端
toc: true
date: 2016-8-01 12:12:12
---

## 什么是生成器函数

生成器函数是一个状态机，**封装了多个内部状态**。

生成器函数可以**生成（返回）生成器对象，生成器对象同时也是迭代器对象**。

生成器函数就是一个普通函数，不过它具有以下三个特点：

---
1. 使用`function*`来声明。
2. 当调用一个生成器函数时，函数主体**并不马上执行**，而是**返回一个生成器对象**，这个**生成器对象同时也是迭代器对象**。
3. 每次当我们调用这个生成器对象的`next()`方法时，生成器函数会**被执行到第一个`yield`表达式**。
---

下面是一个简单的生成器：

{% codeblock lang:javascript %}
function* sayNum () {
  console.log('begin...');
  yield 1,
  yield 2,
  yield 3
}

// 返回一个生成器对象，同时也是迭代器对象
var iter = sayNum();    

console.log(iter.next());     // begin... && Object{value=1, done=false}
console.log(iter.next());     // Object{value=2, done=false} 
console.log(iter.next());     // Object{value=3, done=false}
console.log(iter.next());     // Object{value=undefined, done=true}
{% endcodeblock %}

## yield和yield\*

### yield

[yield](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/yield)关键字用来**使生成器函数暂停执行**，并且**返回跟在它后面的表达式的值**。

yield关键字**实际返回一个对象**，包含`value`和`done`两个属性。

yield语句**不能用在普通函数中**，否则会报错。

### yield\*

[yield\*](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/yield*)用来**在一个生成器函数中执行另外一个生成器函数**。比如以下代码：

{% codeblock lang:javascript %}
function* one () {
  yield 1,
  yield* two(),          // 返回另一个生成器函数的执行结果
  yield 4
}

function* two () {
  yield 2,
  yield 3
}

console.log(...one());  // 1 2 3 4
{% endcodeblock %}

其实，`yield*`后面**可以跟任何一个迭代器对象**。

{% codeblock lang:javascript %}
function* one () {
  yield 1,
  yield* [2,3],   //  后面可以跟任何一个迭代器对象
  yield 4
}

function* two () {
  yield 2,
  yield 3
}
console.log(...one());  // 1 2 3 4
{% endcodeblock %}

如果**被代理的生成器函数有return语句**，就可以向代理它的生成器函数**返回数据**。

{% codeblock lang:javascript %}
function* one () {
  yield 1;
  var a = yield* two(); // a = 3
  yield (a + 1);
}

function* two () {
  yield 2;
  return 3;
}

console.log(...one());   //1 2 4
{% endcodeblock %}

## 生成器对象就是迭代器对象

**调用生成器函数能生成一个迭代器对象**。

调用任意一个可迭代对象的`[Symbol.iterator]()`函数，就会返回一个迭代器对象。

所以，可以**将生成器函数赋值给`[Symbol.iterator]`属性**，就可以使对象具有迭代器方法。

{% codeblock lang:javascript %}
var myIterator = {};

// 将生成器函数的调用结果 赋值给 自定义可迭代对象中迭代器方法的返回值
myIterator[Symbol.iterator] = () => range(5, 8);

function* range (begin, end) {
  for (let i = begin; i <= end; i ++) {
    yield i;
  }
}

// 想在自定义对象可以迭代了
console.log(...myIterator);   // 5 6 7 8
{% endcodeblock %}

## next方法的参数

yield语句本身**没有返回值**，但是，next()方法可以带一个参数，作为**上一个yield语句的返回值**。

{% codeblock lang:javascript %}
function* printNums () {
  var a = 2 * (yield 1);
  var b = a + (yield 2);
  var c = (yield 3);
  return (a + b + c);   // 此时done为true，后面代码不会执行，并且for-of遍历不到该值
  yield 4;              // 不会执行
}

var itNums = printNums();
console.log(itNums.next().value);        // 1
console.log(itNums.next(10).value);      // 2, 但是(yield 1)的返回值为 10，所以a = 20
console.log(itNums.next(20).value);      // 3, 但是 (yield 2)的返回值为20，所以b = 40
console.log(itNums.next(30).value);      // 90, (yield 3)的返回值为30，所以c = 30,最终返回a+b+c=90

console.log(...printNums());  // 1,2,3
{% endcodeblock %}

由于**return语句执行时，返回对象的done为true**，所以后面代码不会执行，并且for-of或者扩展运算符遍历被返回的迭代器对象时，也**不会遍历done为true的对象的值**。

## [Generator.prototype.throw()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Generator/throw)

生成器函数返回的迭代器对象，都有一个`throw`方法，可以**在函数体外抛出错误**，下面为简单应用：

{% codeblock lang:javascript %}
function* gen () {
  try {
    yield 20;
  } catch (e) {
    console.log('Error caugth: ' + e);
  }
}

var iter = gen();
iter.next();    // {value: 20, done: false}
iter.throw(new Error('something wrong...'));    // Error caught: Error: something wrong...
{% endcodeblock %}

## [Generator.prototype.return()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Generator/return)

生成器函数返回的迭代器对象，还有一个`return`方法，可以返回给定的值，并且**终结生成器函数的继续执行**。

{% codeblock lang:javascript %}
function* gen () {
  yield 1;
  yield 2;
  yield 3;
}

var iter = gen();

iter.next();        // {value: 1, done: false}
iter.return('xwj');     // {value: "xwj", done: true}
iter.next();            // {value: undefined, done: true}
{% endcodeblock %}

## 作为对象属性的Generator

{% codeblock lang:javascript %}
var xwj = {
  // 表示一
  * one () {
    //...
  },
  // 表示二
  two: function* () {
    //...
  }
};
{% endcodeblock %}
## 简单应用

生成器函数可以**暂停函数执行**，并且**返回任意表达式的值**。这种特点使得Generator有多种应用场景：

### 异步操作的同步化表达

生成器的暂停执行的效果，意味着可以**把异步操作写在yield语句里面**，等到调用next()方法再往下执行。这实际上等同于**不需要写回调函数**了，因为异步操作的后续操作可以放在yield语句下面。

yield语句是同步执行的。

下面是一个简单的例子：

{% codeblock lang:javascript %}
// 通过ajax异步获取数据
var getData = () => {
  var xhr = new XMLHttpRequest();
  xhr.onreadystatechange = () => {
    if (xhr.readyState === 4) {
      if ((xhr.status >= 200 && xhr.status < 300) || xhr.status === 304) {
        // 可以执行yield后面的操作了，参数为异步操作的结果
        iter.next(JSON.parse(xhr.responseText));
      } else {
        console.log('error: ' + xhr.status);
      }
    }
  };
  xhr.open('GET', 'http://192.168.0.101:8030', true);
  xhr.send(null);
};

// 将数据渲染到DOM
var render = (data) => {
  var items = document.getElementById('main').getElementsByTagName('p');
  for (let item of items) {
    item.innerHTML = data[item.classList[0]];
  }
};

// 生成器函数
function* gen () {
  var result = yield getData();
  render(result);
}

var iter = gen();
iter.next();
{% endcodeblock %}

### 部署迭代器接口

利用生成器函数，我们可以在任意对象上部署迭代器接口。

因为生成器函数返回一个迭代器对象，只需要将讲这个迭代器对象，作为`[Symbol.iterator]()`的返回值即可。

## 参考资料

- [ES6 In Depth: Generator](https://hacks.mozilla.org/2015/05/es6-in-depth-generators/)
- [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Generator](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Generator)














