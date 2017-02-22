title: ES6箭头函数
tags: es6
categories: 前端
toc: true
date: 2016-8-15 12:12:12
---
[箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)就是个简写形式的函数表达式，并且**它拥有继承自外部作用域的this值**。此外，**箭头函数总是匿名的**。
## 基本用法

箭头函数引入了一种新的编写函数的语法：

{% codeblock lang:javascript %}
// ES5
var showname = function (name) {
    return name;
};

// ES6
var showName = name => name;
{% endcodeblock %}

这使得当我们声明只有一个参数的简单函数时，可以使用新标准中的`=>`表达式，我们无需输入`function`和`return`。

## 多个参数

如果我们的函数具有多个参数，或者没有参数，我们需要**使用小括号包裹起来参数列表**。

{% codeblock lang:javascript %}
// ES5
var tatal = values.reduce(function (a, b) {
    return a + b;
}, 0);

// ES6
var total = values.reduce((a, b) => a + b, 0);
{% endcodeblock %}

## 语句块

箭头函数不仅仅能包含一个表达式，还可以包含一个语句块，语句块用`{}`来包裹起来。

但是，如果函数体为语句块，那么**函数将不会自动添加return语句**。因此，这时候我们如果想返回什么东西，必须显式声明return语句。

{% codeblock lang:javascript %}
var showNum = num => {
    num ++;
    return num;
};
{% endcodeblock %}

## 返回普通对象

假如我们需要用箭头函数来返回一个普通对象时，我们需要**将对象包裹在小括号里**。

{% codeblock lang:javascript %}
var myObj = (name, age) => {};    // bad
var myObje = (name, age) => ({})  // good
{% endcodeblock %}

因为一个普通对象和语句块都是使用`{}`来包裹的，看起来很像。而ES6规定，总是将`{`看作一个语句块的开始，而非一个对象的开始。因此`name => {}` 被默认解释为一个什么都没有做，并且返回undefined的箭头函数。

## this

在严格模式下，全局作用域下定义的函数中，**this指向undefined**。
在非严格模式下，全局作用域下定义的函数中，this指向window。

而在箭头函数中，**this没有值，箭头函数内部的this值继承自外围作用域**。

我们先来回顾一下平时使用this的场景：

{% codeblock lang:javascript %}
'use strict';

var person = {
  name: 'xuwenjiang',
  love: ['aaa', 'bbb', 'ccc'],
  show: function () {
    console.log(this);        // person 对象
    this.love.forEach(function (item) {
      console.log(this);      // 3个undefined
    });
  }
};

person.show();
{% endcodeblock %}

以上代码，在内部函数中，没有继承外部函数的this指向，所以在严格模式下，this为undefined。

为了让内部函数的this也指向person对象，我们经常这么干：

{% codeblock lang:javascript %}
'use strict';

var person = {
  name: 'xuwenjiang',
  love: ['aaa', 'bbb', 'ccc'],
  show: function () {
    // 利用闭包，将外部函数的this导入内部函数
    var self = this;          
    console.log(this);        // person 对象
    this.love.forEach(function (item) {
      // 注意这里用的 self
      console.log(self);    // 3个person对象
    });
  }
};

person.show();
{% endcodeblock %}

这样可以解决问题，但是这样麻烦不？很显然，麻烦。

ES6中，我们可以更好地解决这个问题，但是我们需要遵循以下规则：

---
1. **对于object.method()形式的函数，使用非箭头函数语法**。这些函数在运行时将会接受到一个有意义的this值，this指向他们的调用者。
2. **其他函数统一使用箭头函数**。
---

所以，以上代码可以利用箭头函数重写：

{% codeblock lang:javascript %}
'use strict';

var person = {
  name: 'xuwenjiang',
  love: ['aaa', 'bbb', 'ccc'],
  // obj.method(),使用非箭头函数      
  show: function () {    
    console.log(this);        // person 对象
    // 箭头函数，this继承自外围作用域
    this.love.forEach((item) => {
      console.log(this);    // 3个person对象
    });
  }
};

person.show();
{% endcodeblock %}

## 更简洁的方式

超赞的是，在ES6中，我们可以使用更加简洁的方式编写对象字面量中的方法，以上代码可以简化成：

{% codeblock lang:javascript %}
'use strict';

var person = {
  name: 'xuwenjiang',
  love: ['aaa', 'bbb', 'ccc'],
  // 注意这里的简化 ，不用写function   
  show () {    
    console.log(this);        // person 对象
    this.love.forEach((item) => {
      console.log(this);    // 3个person对象
    });
  }
};

person.show();
{% endcodeblock %}

## 没有arguments对象

箭头函数和非箭头函数还有第二个小小的区别：**箭头函数也没有他们自己的arguments对象**。

但是，在es6中，我们可以使用[剩余参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Rest_parameters)来代替，案例如下；

{% codeblock lang:javascript %}
// ES5
var getSum = function () {
  var sum = 0;
  for (var i = 0; i < arguments.length; i ++) {
    sum += arguments[i];
  }
  return sum;
};

// ES6
var getSum = (...rest) => {
  var sum = 0;
  for (let i = 0; i < rest.length; i ++) {
    sum += rest[i];
  }
  return sum;
}

console.log(getSum(1,2,3,4,5));    // 15
{% endcodeblock %}

这里也简单总结下剩余参数和arguments对象的区别：

---
1. 剩余参数只包含那些**没有对应形参的实参**，而arguments对象包含了**传给函数的所有实参**。
2. arguments对象**不是一个真实的数组了，而剩余参数是一个**真实的Array实例，也就是说，你可以在它上面直接使用所有的数组方法。
3. arguments对象还有一些附加的属性，比如callee。
---

## 参考资料

- [https://hacks.mozilla.org/2015/06/es6-in-depth-arrow-functions/](https://hacks.mozilla.org/2015/06/es6-in-depth-arrow-functions/)
- [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)


