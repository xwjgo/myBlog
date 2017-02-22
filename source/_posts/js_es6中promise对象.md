title: ES6中promise对象
tags: es6
categories: 前端
toc: true
date: 2016-8-14 12:12:12
---

## 网易互联网笔试

昨天参加了网易互联网前端工程师的笔试，考的很渣-\_-。一共20道选择题，3道编程题，1道问答题。

选择题考的还是挺杂的，不仅仅包括前端，还考到了二叉树的遍历、UDP/TCP等知识点。编程题3道，自己用node.js写了两道，在本地测试了几组用例都能够通过，但是提交到牛客网就出错。问答题没时间了就直接交卷，问题是大致是实现event的on和emit。

选择题中有两道是关于es6中的promise对象的，虽然对promise有所耳闻，但是细节内容却没有掌握，这篇文章就总结了我对promise对象的一些理解。

## 什么是Promise对象

[Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)是异步编程的一种解决方案。

**一个Promise对象代表着一个还未完成，但是预期将来会完成的操作。**

Promise对象是一个返回值的代理，这个返回值在promise对象创建时未必已知。它允许你为异步操作的成功或者失败指定处理方法。

Promise对象拥有以下几种状态：

---
1. pending-----------------------初始状态，既不是fulfilled，也不是rejected。
2. fulfilled(resolved)-----------成功的操作。
3. rejected----------------------失败的操作。
---

## Promise对象的两个特点

Promise对象拥有以下两个特点：

---
1. **对象的状态不受外界影响。**只有异步操作的结果，可以决定当前是哪一种状态，其他任何操作都无法改变这个状态。
2. **一旦状态改变，就不会再变**，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能性，从**pending变为resolved**，或者从**pending变为rejected**。
---

有了Promise对象，我们将可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。

## Promise对象的缺点

---
1. **无法取消Promise**，一旦新建，它就会立即执行，无法中途取消。
2. 如果不设置回调函数，Promise内部抛出的错误，**不会反映到外部**。
3. 当处于pending状态时，**无法得知目前进展到哪一个阶段**（刚刚开始还是即将完成）。
---

## 基本用法

es6规定，Promise对象是一个构造函数，用来生成Promise实例。

下面创建一个Promise实例：

{% codeblock lang:javascript %}
var p1 = new Promise(function (resolve, reject) {
    setTimeout(function (name) {
        resolve(name);
    }, 2000, 'xuwenjiang'); 
});

p1.then(function (msg) {
    console.log(msg);
});
{% endcodeblock %}

**Promise构造函数接受一个函数作为参数**，该函数的两个参数是resolve和reject。它们是两个函数，有JavaScript引擎提供，不用自己部署。

resolve函数的作用是，将Promise对象的状态从**”初始化“**变为**”成功“**，即从**pending**变为**resolved**。在异步操作成功时调用。

reject函数的作用是，将Promise对象的状态从**”初始化“**变为**”失败“**，即从**pending**变为**rejected**。在异步操作失败时调用。

Promise实例生成后，可以用then方法分别指定resolved状态和rejected状态的回调函数。

{% codeblock lang:javascript %}
p1.then(function (value) {
    // success
}, function (error) {
    // failure    可选
});
{% endcodeblock %}

reject方法的参数通常是Error对象的实例，而**resolve方法的参数除了正常值以外，还可能是另一个Promise实例**，比如下面：

{% codeblock lang:javascript %}
var p1 = new Promise(function (reslove, reject) {
    setTimeout(function () {
        reject('p1 reject...');
    }, 2000);
}).then(undefined, function (msg) {
    console.log(msg);
});

var p2 = new Promise(function (reslove, reject) {
    // resolve参数设置为p1
    resolve(p1);
}).then(function () {
    console.log('p2 resolve...')
});
{% endcodeblock %}

控制台在两秒钟之后，同时输入以下两条内容：

{% codeblock lang:javascript %}
p1 reject...
p2 resolve...
{% endcodeblock %}

上面代码中，p1和p2都是Promise的实例，但是p2的resolve方法将p1作为参数。

注意，这时**p1的状态就会传递给p2**，也就是说，**p1的状态决定了p2的状态**。如果p1的状态是pending，那么p2的回调函数就会等待p1的状态改变，如果p1的状态已经为resolved或者rejected，那么p2的回调函数将会立刻执行。

## 几个常用方法

### [Promise.prototype.then()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/then)

then方法返回的是一个新的Promise实例。因此可以采用链式写法，即then方法后面再调用一个then方法。

分为两种情况，一种是**直接返回非Promise对象**的结果，一种是**返回Promise对象**的结果。

---
返回非Promise对象：紧跟着then方法，resolve立即执行。并可使用前一个then方法返回的结果。

{% codeblock lang:javascript %}
var promise = new Promise (function (resolve, reject) {
  setTimeout(function () {
    resolve(1);
  }, 2000);
});

promise.then(function (value) {
  console.log(value);
  // 返回非Promise对象
  return value + 1;
}).then(function (value) {
  console.log(value);
});
{% endcodeblock %}

控制台在两秒钟后，同时输出两行数据：1,2

上面代码中使用then方法，依次指定了两个回调函数。**第一个回调函数完成之后，会将返回结果作为参数，传入第二个回调函数**。

---

返回promise对象

{% codeblock lang:javascript %}
var promise = new Promise (function (resolve, reject) {
  setTimeout(function () {
    resolve(1);
  }, 2000);
});

promise.then(function (value) {
    var p = new Promise (function (reslove, reject) {
      setTimeout(function () {
          reject(value + 1);
      }, 2000);
    });
    // 返回一个promise对象
    return p;
}).then(undefined, function (value) {
    console.log(value);
});
{% endcodeblock %}

控制台在两秒钟后，输出1，又过两秒钟，输出2。

上面代码中，第一个then方法返回了一个Promise对象，这是第二个then方法指定的回调函数，就会等待这个新的Promise对象状态发生变化。

---

### [Promise.prototype.catch()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch)

Promise.prototype.catch方法是**.then(null, rejection)**的别名。

catch()方法只处理**Promise被拒绝**的情况，并返回一个Promise。

**如果异步操作发生错误，状态就会变为rejected**，就会调用catch方法指定的回调函数，处理这个错误。

另外，如果**then方法指定的回调函数，如果运行中抛出错误，也会被catch方法捕获**。

{% codeblock lang:javascript %}
// 写法一

var p1 = new Promise(function(resolve, reject) {
  setTimeout(function() {
    resolve(1);
  }, 2000);
});

p1.then(function (value) {
    console.log(value);    // 1
    throw 'error...';
}).catch(function (e) {
  console.log(e);    // error...
});

// 写法二

var p1 = new Promise(function(resolve, reject) {
  setTimeout(function() {
    reject(1);
  }, 2000);
});

p1.catch(function (e) {
  console.log(e);    // 1
});
{% endcodeblock %}

通过以上可以看出，**reject方法的作用，等同于抛出错误**。

一般来说，**不要在then方法中定义rejected状态的回调函数（即不写第二个参数），总是使用catch方法**。

{% codeblock lang:javascript %}
// bad
promise.then(function (value) {
  // success
}, function (err) {
  // error
});

// good
promise.then(function (data) {
  // success
}).catch(function (err) {
  // error
});
{% endcodeblock %}

### [Promise.all()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)

Promise.all方法将用于多个Promise实例，包装成一个新的Promise实例。

{% codeblock lang:javascript %}
var p = Promise.all([p1, p2, p3]);
{% endcodeblock %}

在上面代码中，Promise.all接收一个数组作为参数，p1，p2，p3都是Promise对象的实例。

p的状态由p1，p2，p3决定，分成两种情况：

---
1. 只有p1，p2，p3的状态**都变为fulfilled，p的状态才会变成fulfilled**。此时，p1，p2，p3的返回值组成一个**数组**，传递给p的回调函数。
2. 只要p1，p2，p3之中**有一个被rejected，p的状态就变成rejected**，此时**第一个被reject的实例的返回值**，会传递给p的回调函数。
---

{% codeblock lang:javascript %}
var p1 = new Promise(function (resolve, reject) {
  setTimeout(function () {
    resolve(1);
  }, 2000);
});

var p2 = new Promise(function (resolve, reject) {
  setTimeout(function () {
    resolve(2);
  }, 2000);
});

// 只有当p1和p2的状态都为fulfilled时，以下函数才能执行

var p = Promise.all([p1,p2]).then(function (value) {
  console.log(value);    // [1,2]
});
{% endcodeblock %}

### [Promise.race()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)

Promise.race方法同样是将多个Promise实例，包装成一个新的Promise实例。

{% codeblock lang:javascript %}
var p = Promise.race([p1, p2, p3]);
{% endcodeblock %}

上面代码中，只要p1，p2，p3之中有一个实例**率先改变状态，p的状态就跟着改变**。那个率先改变Promise实例的返回值，就传递给p的回调函数。

{% codeblock lang:javascript %}
var p1 = new Promise(function (resolve, reject) {
  setTimeout(function () {
    resolve(1);
  }, 2000);
});

var p2 = new Promise(function (resolve, reject) {
  setTimeout(function () {
    resolve(2);
  }, 3000);
});

// p1实例的状态先改变，所以p1实例的返回值，会传递给p的回调函数

var p = Promise.race([p1,p2]).then(function (value) {
  console.log(value);    // 1
});
{% endcodeblock %}

### [Promise.resolve()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve)

如果需要将现有对象转换为Promise对象，Promise.resolve 方法就可以用到。

**静态方法**Promise.resolve返回一个promise对象，这个对象是被解析后（resolve）的。

它的语法有以下四种：

{% codeblock lang:javascript %}
Promise.resolve(value)              // 参数不是thenable，也不是对象
Promise.resolve(promise)            // 参数是一个promise实例
Promise.resolve(thenable)           // thenable是一个具有then方法的对象
Promise.resolve()                   // 可以快速拿到一个promise对象
{% endcodeblock %}

示例代码如下：

{% codeblock lang:javascript %}
// 场景一：Promise.resolve(value)

Promise.resolve('success').then(function (value) {
  console.log(value);    // success
});

Promise.resolve([1,2,3]).then(function (value) {
  console.log(value);    // [1,2,3]
});

// 场景二：Promise.resolve(promise)

var p1 = Promise.resolve(true);
var p2 = Promise.resolve(p1).then(function (value) {
  console.log(value);    // true
});

// 场景三： Promise.resolve(thenable)

var promise = Promise.resolve({
  then: function (onFulfill, onReject) {
    onFulfill('fulfilled...');
  }
}).then(function (value) {
  console.log(value);    // fulfilled...
});

// 场景四：Promise.resolve()

var p = Promise.resolve();  // 可以直接拿到一个promise对象
p.then(function () {
  // ...
});
{% endcodeblock %}

需要注意的是，**立即resolve的promise对象，是在本轮“事件循环”（event loop）的结束时，而不是在下一轮“事件循环”的开始时**。

{% codeblock lang:javascript %}
setTimeout(function () {
  console.log('three');
},0);

Promise.resolve().then(function () {
  console.log('two');
});

console.log('one');

// one
// two
// three
{% endcodeblock %}

上面代码中，setTimeout(fn,0)在下一轮”事件循环“开始时执行，Promise.resolve()在本轮”事件循环“结束时执行，console.log()则是立即执行。

### [Promise.reject()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/reject)

静态方法Promise.reject()方法返回一个用reason拒绝的Promise。

{% codeblock lang:javascript %}
Promise.reject('error...').then(null, function (reason) {
  console.log(reason);        // error...
});

Promise.reject(new Error('error!')).then(null, function (reason) {
  console.log(reason);        // Error:error!
});
{% endcodeblock %}

## 简单应用

以下是最终效果图：

![](http://7xvlvo.com1.z0.glb.clouddn.com/three%20balls.gif)

首先先不用Promise，核心代码如下:

{% codeblock lang:javascript %}
function animate (ball, distance, cb) {
  setTimeout(function () {
    var marginLeft = parseInt(ball.style.marginLeft, 10);
    if (marginLeft === distance) {
      cb && cb();
    } else {
      if (marginLeft < distance) {
        marginLeft ++;
      } else {
        marginLeft --;
      }
      
      ball.style.marginLeft = marginLeft + 'px';
      animate(ball, distance, cb);
    }
    // 一开始把这两行写在这里, 出错了
//  ball.style.marginLeft = marginLeft + 'px';
//  animate(ball, distance, cb);
  }, 13);
};

// 层层回调
animate(ball1, 100, function () {
  animate(ball2, 200, function () {
    animate(ball3, 300, function () {
      animate(ball3, 150, function () {
        animate(ball2, 150, function () {
          animate(ball1, 150, function () {
    
          });
        })
      });
    });
  });
});
{% endcodeblock %}

在使用了promise之后，核心代码如下：

{% codeblock lang:javascript %}
function promiseAnimate (ball, distance) {
  return new Promise (function (resolve, reject) {
    function _animate () {
      setTimeout(function () {
        var marginLeft = parseInt(ball.style.marginLeft, 10);
        if (marginLeft === distance) {
          resolve();
        } else {
          if (marginLeft < distance) {
            marginLeft ++;
          } else {
            marginLeft --;
          }

          ball.style.marginLeft = marginLeft + 'px';
          _animate();
        }
      }, 13);
    }
    
    _animate();
  });
}

promiseAnimate(ball1, 100)
  .then(function () {
    return promiseAnimate(ball2, 200)
  })
  .then(function () {
    return promiseAnimate(ball3, 300)
  })
  .then(function () {
    return promiseAnimate(ball3, 150)
  })
  .then(function () {
    return promiseAnimate(ball2, 150)
  })
  .then(function () {
    return promiseAnimate(ball1, 150)
  });
{% endcodeblock %}

通过对比，可以看出使用Promise使得代码可读性高，并且容易维护。























































