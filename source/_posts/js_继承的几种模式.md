title: 继承的几种模式
tags: javascript
categories: 前端
toc: true
date: 2016-7-18 12:12:12
---

ECMAScript中的继承重要是依靠**原型链**来实现的。

这里就简单分析一下js中实现继承的几种方式：包括**原型链**、**借用构造函数**、**组合继承**、**原型链继承**、**寄生组合式继承**。

## 原型链

### 原型链的概念

简单回顾以下构造函数，原型，实例的关系：**每个构造函数都有一个prototype属性指向一个原型对象，原型对象都包含一个指向构造函数的指针constructor，而实例都包含一个指向原型对象的内部指针\_\_proto\_\_**。

那么，假如让**原型对象等于另一个类型的实例**，会怎么样呢？显然，此时的原型对象包含一个指向另一个原型对象的指针，相应地，另一个原型中也包含着一个指向另一个构造函数的指针。

加入另一个原型又是另一个类型的实例，那么上述关系依然成立，如此层层递进，就构成了实例于原型的链条。这就是原型链的概念。

### 原型链实现继承

其基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。

我们来看以下代码：

{% codeblock lang:javascript %}
// 构造函数——实例属性——每个实例独占一份
function SuperType () {
  this.name = 'parent';
}

// 原型对象——原型属性——所有实例共享
SuperType.prototype.sayName = function () {
  console.log(this.name);
};

// 子类型构造函数
function SubType () {
  this.age = 20;
}

// 继承 SuperType
// 实质为重写SubType的原型——所有实例共享
SubType.prototype = new SuperType();

// 弥补因重新SubType的原型而丢失的constructor属性
SubType.prototype.constructor = SubType;

// 子类原型对象
SubType.prototype.sayAge = function () {
  console.log(this.age);
}

var instance = new SubType();

instance.sayName();        // 'parent'
{% endcodeblock %}

构造函数、原型、实例之间的关系如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E5%8E%9F%E5%9E%8B%E7%BB%A7%E6%89%BF.png)

其实，SuperType还继承自Obejct类型，因为所有类型都默认继承了Object，图中没有展示出来。

一句话来概括，就是SubType继承了SuperType，而SuperType继承了Object。**当调用instance.toString()时，实际上是调用保存在Objet.prototype中的那个方法**。

上图中的红线表示出了原型链的搜索路径，其实SuperType.prototype也有一个\_\_proto\_\_属性,指向Object.prototype。

如果我们想要访问一个实例属性，**首先会在该实例中搜索该属性。如果没有找到该属性，则会继续搜索实例的原型**。比如我们要搜索instance实例的`sayName()`方法，搜索过程如下：

---
1. 搜索instance实例属性
2. 搜索SubType.prototype
3. 搜索SuperType.prototype
---

只要是原型链中出现的原型对象，都可以说是该原型链所派生的实例的原型。我们可以通过[instanceof操作符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof)或者[Object.prototype.isPrototypeof()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/isPrototypeOf)方法来检测原型和实例的关系，代码如下：

{% codeblock lang:javascript %}
instance instanceof Object;       // true
instance instanceof SuperType;    // true
instance instanceof SubType;      // true

Object.prototype.isPrototypeOf(instance);       // true
SuperType.prototype.isPrototypeOf(instance);    // true
SubType.prototype.isPrototypeOf(instance);      // true
{% endcodeblock %}

### 原型链的缺点

原型链的缺点还是由于原型的共享性所致，超类型的实例同时也是子类型的原型，而原型中的所有属性和方法为所有实例共享。所以，如果超类型的实例（即子类型的原型)中含有引用类型值的属性，那么将会出现问题，比如一个实例的属性将会影响到另一个实例。所以，这种继承方式很少单独使用。

## 借用构造函数

借用构造函数的基本思想很简单，即在子类型构造函数的内部调用超类型的构造函数。因此需要使用[Function.prototype.call()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)方法或者[Function.prototype.apply()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)方法。

{% codeblock lang:javascript %}
      function SuperType () {
        this.colors = ['red', 'green', 'blue'];
      }

      function SubType () {
        // 继承SuperType
        SuperType.call(this);
      }

      var instance1 = new SubType();
      var instance2 = new SubType();

      instance1.colors.push('black');

      console.log(instance1.colors);    // ["red", "green", "blue", "black"]
      console.log(instance2.colors);    // ["red", "green", "blue"]
{% endcodeblock %}

构造函数也有其不足之处：函数复用无从谈起，也无法继承原型中的方法。

## 组合继承（常用）

组合继承将**原型链**和**借用构造函数**的技术组合到了一起。

它的基本思想是，**使用原型链实现对原型属性和方法的继承，通过借用构造函数来实现对实例属性的继承**。这样，它既通过在原型上定义方法实现了函数复用，又能够保证每个实例都有自己的属性。

{% codeblock lang:javascript %}
function SuperType (name) {
    this.name = name;
    this.colors = ['red', 'green', 'blue'];
}

SuperType.prototype.sayName = function () {
    console.log(this.name);
}

function SubType (name, age) {
    // 继承实例属性——每个实例独占一份
    SuperType.call(this);                        // 第二次调用SuperType()
    this.age = age;
}

// 继承原型方法——所有实例共享
SubType.prototype = new SuperType();             // 第一次调用 SuperType()

SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function () {
  console.log(this.age);
}

var instance1 = new SubType('xwj', 21);
var instance2 = new SubType('hyy', 20);

instance1.colors.push('black');

console.log(instance1.colors);           // ["red", "green", "blue", "black"]
console.log(instance2.colors);           // ["red", "green", "blue"]
{% endcodeblock %}

以上代码图解如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E7%BB%84%E5%90%88%E7%BB%A7%E6%89%BF.png)

组合模式常用，但是它也有一点缺陷：无论什么情况下，它都会调用两次构造函数（代码中已经标明）。况且从上图可以看出，子类型的实例和子类型的原型对象中，包含了同样的属性。

## 原型式继承

基本思想：借助原型可以基于已有的对象创建新对象，同时还不必因此创建自定义类型。原型式继承基于以下函数实现：

{% codeblock lang:javascript %}
// 创建一个临时构造函数，并且将其原型指向传入的对象
// 返回临时构造函数的一个实例
function object (o) {
    function F () {}
    F.prototype = o;
    return new F();
}
{% endcodeblock %}

此函数相当于[Object.create()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)方法只有一个参数的时候。本质上讲，此函数对传入其中的对象进行了一次浅复制，**返回实例对象中的\_\_proto\_\_属性保存了一个指针，指向传入函数的对象**。

下面是原型式继承的实现：

{% codeblock lang:javascript %}
var person = {
    name: 'xwj',
    friends: ['A', 'B', 'C']
};

// 本质：创建了person对象的两个副本
var anoterhPerson1 = object(person);
var anoterhPerson2 = object(person);

anoterhPerson1.friends.push('D');

// 包含引用类型值的属性始终会共享相应的值
console.log(anoterhPerson2.friends);  // ["A", "B", "C", "D"]
{% endcodeblock %}

这种继承方式的缺点也是显而易见，即**所有实例都会共享包含引用类型值的属性**，比如colors。

## 寄生组合式继承

前面也已经提到了**组合继承**的缺点，寄生组合式继承解决了这些问题。

它的思路是：**不必为了指定子类型的原型而实例化超类型的构造函数，我们所需要的无非是超类型原型的一个副本而已**。本质上，就是借用构造函数来继承属性，通过原型链的混成形式来继承方法。

核心代码如下：

{% codeblock lang:javascript %}
// 复制超类型的原型，并将之赋给子类型的原型
function inheritPrototype (subType, superType) {
    var prototype = object (superType.prototype); 
    prototype.constructor = subType;
    subType.prototype = prototype;
}

function SuperType (name) {
    this.name = name;
    this.colors = ['red', 'green', 'blue'];
}

SuperType.prototype.sayName = function () {
    console.log(this.name);
}

function SubType (name, age) {
     // 继承父类的实例属性，每个实例独占一份
     SuperType.call (this, name);

     this.age = age;
}

// 只继承父类的原型中的方法（共享）
// 没有实例化父类对象，所以并没有继承父类实例属性
inheritPrototype(SubType, SuperType);

SubType.prototype.sayAge = function () {
  console.log(this.age);
}

var instance = new SubType('xwj', 22);

instance.sayName();           // 'xwj'
{% endcodeblock %}

这样只调用了一次SuperType构造函数，并且因此避免了在SubType.prototype上面创建不必要的、多余的属性。这是非常理想的继承范式。

## 最后

以上就是对几种继承方式的总结，如果有不正确的地方，欢迎在评论区提出～












