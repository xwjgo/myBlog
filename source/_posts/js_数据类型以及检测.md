title: js中的数据类型以及检测
tags: javascript
categories: 前端
toc: true
date: 2016-07-21
---

一个完整的JavaScript实现应该有以下三个不同的部分组成：

1. 核心（ECMAScript）
2. 文档对象模型（DOM）
3. 浏览器对象模型（BOM）

## 数据类型

ECMAScript中有5种简单的数据类型： **Undefined**、**Null**、**Boolean**、**Number**和**String**。

还有一种复杂的数据类型： **Object**。

[**关于Object**](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)

在javaScript中，所有的对象都是基于Object的，Object是所有实例的基础。

所有的对象都继承了Object中prototype的属性和方法，但是他们可以被覆盖，比如[Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/toString)就重写了Object.prototype.toString()方法。

## 对象类型检测

### [**typeof**](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof)

要检测一个变量是哪种基本数据类型，typeof 操作符是最佳的工具。

typeof操作符能够确定一个变量是否是字符串、数值、布尔值、还是undefined。

如果变量是一个对象，则typeof操作符会返回object。

{% codeblock lang:javascript %}
typeof "xuwenjiang"     // string
typeof 25               // number
typeof true             // boolean
typeof undefined        // undefined
typeof null             // object
typeof new Object()     // object
{% endcodeblock %}

### [**instanceof**](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof)

虽然在检测基本数据类型时，typeof是非常得力的助手，但在检测引用数据类型时，这个操作符用处不大。

通常我们不想直到某个值是不是对象，而是想知道它是什么类型的对象。这时候，instanceof就派上用场了。

{% codeblock lang:javascript %}
[1,2,3] instanceof Array            // true
function(){} instanceof Function    // true
new Object instanceof Object        // true
/\w/ instanceof RegExp              // true

[1,2,3] instanceof Object           // true
[1,2,3] instanceof Function         // false
null instanceof Object              // false
{% endcodeblock %}

如果使用instanceof操作符检验基本类型，则该操作符始终会返回false，因为基本类型不是对象。

### [**Object.prptotype.toString()**](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/toString)

该方法返回一个代表该对象的字符串，比如`object Object`。

{% codeblock lang:javascript %}
'use strict'
var toString = Object.prototype.toString();
toString.call([1,2,3]);       // [object Array]
toString.call(undefined);     // [object Undefined]
toString.call(null);          // [object Null]
toString.call(new Date());    // [object Date]
toString.call(25);            // [object Number]
{% endcodeblock %}


### [**Object.prptotype.constructor**](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/toString)

该属性返回创建了该对象原型的函数引用，对于原始值（比如1，true或者'test'），该属性为只读。

{% codeblock lang:javascript %}
function Person(name, age){
   this.name = name;
   this.age = age;
}
var xwj = new Person('xuwenjiang', 21);

// 判断引用类型(可以被修改，不可靠）

console.log(xwj.constructor);            // Person(name,age)
console.log([1,2,3].constructor);        // Array()
console.log(/\w/.constructor);           // RegExp()
console.log(function(){}.constructor);   // Function()
console.log(new Date().constructor);     // Date()
console.log({}.constructor);             // Object()

// 判断基本类型(可靠）

console.log((2).constructor);            // Number()  只读
console.log(('xwj').constructor);        // String()  只读
console.log((true).constructor);         // Boolean() 只读

// 除了Number、String、Boolean类型以外，其他对象的constructor可以被修改

xwj.constructor = Array;
console.log(xwj.constructor)    // Array()
{% endcodeblock %}


