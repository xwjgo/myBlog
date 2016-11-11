title: 观察者模式
tags: javascript
categories: 前端
toc: true
---

观察者模式又称**发布/订阅**模式，它定义了一种对象间**一对多的依赖关系**。

当发布者状态改变时，依赖其的所有订阅者都将得到通知并且自动更新自己的状态。

这种一对多的关系，就好比**很多人都会订阅同一个微信公众号**，当这个微信公众号有新消息时，所有订阅者都会收到。

这种设计模式，主要有以下两种角色组成：

---
1. 发布者：作为一对多关系中的一，可以**管理订阅者的增加和删除，以及发布新数据**。
2. 订阅者：一旦发布者发布新数据，将**自动作出相应的反应**。
---

## 基本实例

{% codeblock lang:javascript %}
'use strict';

// 发布者
function Publisher () {
  this.subscribers = [];
}

Publisher.prototype = {
  constructor: Publisher,
  
  // 增加订阅者
  addSubscriber: function (fn) {
    this.subscribers.forEach(function (item) {
      if (fn === item) {
        return;
      }
    });
    this.subscribers.push(fn);
  },
  
  // 发布新数据
  publish: function (data) {
    this.subscribers.forEach(function (item) {
      item(data);
    });
  },
  
  // 删除订阅者
  removeSubscriber: function (fn) {
    this.subscribers = this.subscribers.filter(function (item) {
      return (item !== fn);
    });
  }
};

// 订阅者A
function subscriberA (data) {
  console.log('A has received:' + data);
}
// 订阅者B
function subscriberB (data) {
  console.log('B has received:' + data);
}

// 实例化一个发布者对象
var publisher = new Publisher();

// 增加订阅者A和B
publisher.addSubscriber(subscriberA);
publisher.addSubscriber(subscriberB);

// 发布者发布新数据
publisher.publish('Welcome to subscribe to me!');
{% endcodeblock %}

运行以上代码，输出结果为：

{% codeblock lang:javascript %}
A has received:Welcome to subscribe to me!
B has received:Welcome to subscribe to me!
{% endcodeblock %}

我们也可以删除订阅者：

{% codeblock lang:javascript %}
// 删除订阅者
publisher.removeSubscriber(subscriberA);
{% endcodeblock %}

以上代码中，当发布者发布新数据时，所有的订阅者都会自动作出相应的反应，当然，我们**也可以扩展订阅者的功能，比如只将数据推送给特性的订阅者**。

## 应用场景

### 场景1

我们在Web开发中，在通过Ajax请求到数据后，需要将数据填充到页面的不同部分，通常我们可以直接在Ajax的回调函数中执行，但是j假如需要更新的位置很多，我们就要去不断地修改回调函数，这种体验无疑是非常糟糕的。

而以上的发布/订阅模式则可以较好地解决这个问题，我们只需要在**回调函数中将Ajax请求来的数据发布出去**就行。

### 场景2

js中的事件监听其实也是观察者模式的实现，比如当我我们点击一个按钮，便会触发事件监听函数。这种情况下，**DOM元素就是发布者，事件处理程序便是订阅者**。

## 发布者的继承

因为发布者对象是封装在一个自定义类型中，所以**其他对象也可以继承Publisher类型并且获得这个行为**。

以下代码，使用了寄生组合式的继承方式来实现：

{% codeblock lang:javascript %}
// 寄生组合式的继承
// 理念：将基类型的原型副本赋值给子类型的原型

function inheritPrototype (subType, superType) {
  var prototype = Object.create(superType.prototype);
  prototype.constructor = subType;
  subType.prototype = prototype;
}

// 定义子类型
function SubPublisher () {
  // 继承基类型的实例属性
  Publisher.call(this);
}

// 继承基类型的原型方法
inheritPrototype(SubPublisher, Publisher);

// 订阅者A
function subscriberA (data) {
  console.log('A has received:' + data);
}
// 订阅者B
function subscriberB (data) {
  console.log('B has received:' + data);
}

// 实例化SubPublisher对象
var subPublisher = new SubPublisher();

// 增加订阅者A和B
subPublisher.addSubscriber(subscriberA);
subPublisher.addSubscriber(subscriberB);

// 子类型也可以发布数据了
subPublisher.publish('subPublisher...');
{% endcodeblock %}

## 总结

观察者模式，或者订阅/发布模式的使用场景为：**当一个对象的改变需要同时改变其他对象，并且不知道具体有多少对象需要改变的时候，就应该考虑使用这种模式**。

关于**Object.create()**的具体使用，你可以点击[这里](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)来查看。

当它只传入第一个参数时，相当于以下：

{% codeblock lang:javascript %}
function object(o) {
   function F () {};
   F.prototype = o;
   return new F();
}
{% endcodeblock %}
