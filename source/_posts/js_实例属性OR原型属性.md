title: 实例属性OR原型属性
tags: javascript
categories: 前端
toc: true
date: 2016-8-02 12:12:12
---

这里总结了区分实例属性和原型属性的几个方法。

## [Object.prototype.hasOwnProperty()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty)

所有继承了Object.prototype的对象都会从原型链上继承到hasOwnProperty()方法，这个方法可以用来检测对象是否含有特定的自身属性，和in运算符不同，该方法会自动忽略那些从原型链上继承到的属性和方法。

{% codeblock lang:javascript %}
o = new Object();
o.name = 'xwj';
o.hasOwnProperty('name');             // true    实例属性
o.hasOwnProperty('toString');         // false   原型属性
o.hasOwnProperty('hasOwnProperty');   // false   原型属性
{% endcodeblock %}

## [in操作符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/in)

无论是实例属性还是原型属性，只要实例对象能够访问的到，就返回true。

{% codeblock lang:javascript %}
// 数组
var myArr = ['apple', 'orange', 'banana'];
console.log(0 in myArr);           // true
console.log(2 in myArr);           // true
console.log('orange' in myArr);    // false    必须使用索引号，而不是数组元素的值
console.log("length" in myArr);    // true     length是一个数组属性

// 内置对象
console.log(PI in Math);    // true

// 自定义对象
var myObj = new Object();
myObj.name = 'xwj';
myObj.prototype.sayName = function () {
   console.log(this.name);
}

console.log('name' in myObj);       // true   实例属性
console.log('sayName' in myObj);    // true   原型属性
{% endcodeblock %}

[for-in循环](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...in)，返回的是所有能够通过实例对象访问的、**可枚举**的属性，也包括实例属性和原型属性。

## 获取所有原型属性

{% codeblock lang:javascript %}
function getPrototypeProperty (Object, name) {
    return !Object.hasOwnProperty(name) && (name in object);
}
{% endcodeblock %}

## 获取所有可以枚举的实例属性

使用[Object.keys()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)方法即可获取所有可以枚举的实例属性。

该方法接收一个对象作为参数，返回一个包含所有枚举属性的字符串数组。

{% codeblock lang:javascript %}
console.log(Object.keys(myObj));
{% endcodeblock %}

## 获取所有实例属性

[Object.getOwnPropertyNames()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyNames)方法可以获取对象所有的实例属性，无论这个属性能否枚举。

{% codeblock lang:javascript %}
console.log(Object.getOwnPropertyNames(myObj));
{% endcodeblock %}



