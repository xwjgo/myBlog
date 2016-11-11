title: 数据属性和访问器属性
tags: javascript
categories: 前端
toc: true
date: 2016-7-23 12:12:12
---

ECMAScript中定义了两种属性：**数据属性**和**访问器属性**。这里对其定义做一个简单整理。

## 数据属性

数据属性包含一个数据值的位置，在这个位置可以读取和写入值。

数据属性有4个描述其行为的特性：

---
1. **configurable**    表示能否通过delete删除属性，能否修改属性的特性，能否把属性修改为访问器属性。
2. **enumerable**    表示能否通过 for-in循环返回属性。
3. **writable**    表示能否修改属性的值。
4. **value**    读取属性值的时候，从这个位置读;写入属性值的时候，把新值保存在这个位置。
---

比如我们创建了一个对象，并且增加了name属性：

{% codeblock lang:javascript %}
var person = {
    name: 'xwj'
}
{% endcodeblock %}

则person对象的name就是一个数据属性，它的`configurable`,`enumerable`,`writable`,`value`都为默认的true，而`value`的值为'xwj'。

## [Object.defineProperty()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

如果我们想修改属性name的特性，则我们需要用到**Object.defineProperty()**方法。这个方法接收三个参数：属性所在对象，要修改或者定义的属性名称，描述符对象。

{% codeblock lang:javascript %}
'use strict'
var person = {
    name: 'xwj'
};

// 修改name属性的特性
Object.defineProperty(person, 'name', {
    configurable: true,    // 如果是false，以后将不能用defineProperty()方法配置此属性，也不能delete属性。
    enumerable: fasle,     // 表示不能通过for-in来返回属性
    writable: false,       // 表示只读，不可修改属性
    value: 'newXwj'        // 属性的值
});

delete person.name;            // Error，不可删除
for (prop in person) {
    console.log(prop);         // 打印不出name，因为name不可枚举
}
person.name = 'anotherName'    // Error, 只读，不可修改
console.log(person.name);      // 'newXwj'
{% endcodeblock %}

## 访问器属性

访问器属性不包含值，它们包含一对儿函数：getter和setter。不过这两个函数都不是必须的。

在读取访问器属性时，会调用getter函数，这个函数负责返回有效的值;在写入访问器属性时，会调用setter函数并传入新值，这个函数负责决定如何处理数据。

同数据属性一样，访问器属性包含以下四个特性：

---
1. **configurable**    表示能否通过delete删除属性，能否修改属性的特性，能否把属性修改为访问器属性。
2. **enumerable**    表示能否通过 for-in循环返回属性。
3. **get**    在读取属性时调用的函数，默认为undefined。
4. **set**    在写入属性时调用的函数，默认为undefined。
---

## 定义访问器属性

访问器属性不能够直接定义，而必须使用Object.defineProperty()来定义。请看下面例子：

{% codeblock lang:javascript %}
var book = {
    _year: 2004,
    edition: 1
}

// 定义访问器属性year
Object.defineProperty(book, 'year', {
    get: function () {
        return this._year;
    },
    set: function (newValue) {
        if (newValue > 2004) {
            this._year = newValue;
            this.edition += newValue - 2004;
        }
    }
});

// year属性值改变，会执行set函数
book.year = 2005;
console.log(book.edition);        // 2
{% endcodeblock %}

**\_year**前面的下划线是一种常用的记号，用于表示只能通过对象方法来访问的属性。

## [Object.defineProperties()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties)

**Object.defineProperty()**方法可以为对象同时定义多个属性。

这个方法接收两个对象参数：第一个对象是要添加和修改的属性对象，第二个对象的属性与第一个对象中要添加或修改的属性一一对应。

{% codeblock lang:javascript %}
var book = {};

Object.defineProperties(book, {

    //定义两个数据属性
    _year: {
        value: 2004
    },
    edition: {
        value: 1
    },

    // 定义一个访问器属性
    year: {
        get: function () {
            return this._year;
        },
        set: function () {
            if (newValue > 2004) {
                this._year = newValue;
                this.edition += newValue - 2004;
            }
        }
    }
});
{% endcodeblock %}

## [Object.getOwnPropertyDescriptor()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor)

**Object.getOwnPropertyDescriptor()**方法可以读取实例属性的特性，但是不可以用来读取原型属性的特性。

{% codeblock lang:javascript %}
//  读取访问器属性的特性
Object.getOwnPropertyDescriptor(person, 'age');
{% endcodeblock %}

该函数返回一个对象，属性包括 configurable、enumerable、get、set、四项。

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E8%AF%BB%E5%8F%96%E8%AE%BF%E9%97%AE%E5%99%A8%E5%B1%9E%E6%80%A7%E7%89%B9%E6%80%A7.png)

{% codeblock lang:javascript %}
// 读取数据属性的特性
Object.getOwnPropertyDescriptor(person, '_age');
{% endcodeblock %}

读取数据属性返回的对象，属性包括 configurable、enumerable、writable、value、四项。

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E8%AF%BB%E5%8F%96%E6%95%B0%E6%8D%AE%E5%B1%9E%E6%80%A7%E7%89%B9%E6%80%A7.png)

完。






