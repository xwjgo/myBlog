title: 原生js实现双向绑定
tags: javascript
categories: 前端
toc: true
date: 2016-9-08 12:12:12
---

双向绑定是现在很多MVVM框架都要实现的功能，比如[Vue.js](http://cn.vuejs.org/guide/overview.html)。

vue的核心就是一个**响应的数据绑定系统**：当数据改变时，DOM自动改变，DOM内容改变，底层数据也会相应的更新。

本文就借助[Object.defineProperty()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)来定义了两个指令`x-value`和`x-text`，以简单模拟Vue中使用指令实现双向绑定的场景。

最终效果如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E5%8F%8C%E5%90%91%E7%BB%91%E5%AE%9A.gif)

## 实现原理

从`model --> view`的绑定，借助了**Object.defineProperty()**方法，它可以定义访问器属性，并且**通过getter和setter函数对数据的读写进行监听**。

从`view --> model`的绑定，实质是通过**监听DOM的keyup、change等事件，通过事件处理函数来更新底层数据**。

## 代码

下面来看代码实现：

### HTML部分

{% codeblock lang:html %}
<!--其中使用了两个指令，x-value 和 x-text-->
<form autocomplete="off">
      <span>姓名：</span><input type="text" x-value="name"/>
      <p x-text="name"></p>
      <span>年龄：</span><input type="number" x-value="age"/>
      <p x-text="age"></p>
      <span>学校：</span><input type="text" x-value="school"/>
      <p x-text="school"></p> 
</form>
{% endcodeblock %}

### js部分

{% codeblock lang:javascript %}
'use strict';

// 基本数据对象
var data = {
  _name: '',
  _age: 0,
  _school: ''
};

/*
 * model --> view 的绑定
 * 基本思想：使用defineProperty()的getter和setter
 * 当model发生改变时，在setter函数中触发更新DOM的函数
 */

// 将对象的某个属性封装为访问器属性
var defineGetAndSet = (obj, prop) => {
  Object.defineProperty(obj, prop, {
    get: () => {
      return obj['_' + prop];
    },
    set: (newValue) => {
      obj['_' + prop] = newValue;
      // 触发DOM更新
      render(prop, newValue);
    }
  });
};

// 针对不同指令，进行不同DOM的更新操作
var directives = {
  'x-value': function (newValue) {
    this.setAttribute('value', newValue);
  },
  'x-text': function (newValue) {
    this.innerHTML = newValue;
  }
};

// 依据新数据，来触发DOM更新
var els = [document.querySelectorAll('[x-value]'), document.querySelectorAll('[x-text]')];
var render = (prop, newValue) => {
  els.forEach((item) => {
    for (let i = 0, len1 = item.length; i < len1; i ++) {
      let attrs = item[i].attributes;
      for (let j = 0, len2 = attrs.length; j < len2; j ++) {
        if (attrs[j].nodeName.indexOf('x-') !== -1 && attrs[j].nodeValue === prop) {
          directives[attrs[j].nodeName].call(item[i], newValue);
        }
      }
    }
  });
};

// 定义三个访问器属性
defineGetAndSet(data, 'name');
defineGetAndSet(data, 'age');
defineGetAndSet(data, 'school');

// 初始化属性值
data.name = 'xwj';
data.age = 20;
data.school = 'ouc';

/*
 * view --> model的绑定
 * 基本思想：监听表单的keyup、change等事件
 * 在事件处理函数中将值传给model
 */

// 定义事件处理函数
var updateData = (event) => {
  if (event.target.hasAttribute('x-value')) {
    var prop = event.target.getAttribute('x-value');
    data[prop] = event.target.value;
  }
}

// 添加事件监听
document.addEventListener('keyup', updateData);
document.addEventListener('change', updateData);
{% endcodeblock %}

## 一点改进

以上代码，虽然实现了基本的双向绑定，但是，还有一些改进之处。

我们可以把**Object.defineProperty()** 和[观察者模式](http://192.168.0.101:4000/2016/08/06/js_%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F/)结合起来，当**访问器属性中的stter监听到数据变化时，就通过发布者将新的数据发布出去，而DOM作为订阅者，就可以直接作出相应的改变**。












