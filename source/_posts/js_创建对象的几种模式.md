title: js创建对象的几种模式
tags: javascript
categories: 前端
toc: true
date: 2016-8-17 12:12:12
---

虽然Object构造函数和对象字面量都可以用来创建单个对象，但是这些方式有一个明显的缺点：即使用同一个接口创建很多对象，会产生大量的重复代码。

这篇文章总结了**工厂模式**、**构造函数模式**、**原型模式**、**组合使用构造函数和原型模式**来创建对象的方法，一定程度上能够解决上面的问题。

重点放在原型模式，以增进对原型的理解。

## 工厂模式

工厂模式将创建对象的细节封装在函数中。

{% codeblock lang:javascript %}
function createPerson (name, age) {
   var o = new Object();
   o.name = name;
   o.age = age;
   o.sayName = function () {
     console.log(this.name);
   };
   return o;
}

var person1 = createPerson('xwj', 21);
{% endcodeblock %}

工厂模式虽然解决了创建多个相似对象的问题，但是却没有解决对象识别问题，都是Object类（怎样知道一个对象的类型）。

## 构造函数模式

{% codeblock lang:javascript %}
function Person (name, age) {
        this.name = name;
        this.age = age;
        this.sayName = function () {
          console.log(this.name);
        }
      }

var person1 = new Person('xwj', 21);
var person2 = new Person('hyy', 20);
{% endcodeblock %}

与工厂模式相比，构造函数模式具有以下特点：

---

1. 没有显式地创建对象
2. 直接将属性和方法赋给了this对象
3. 没有return语句

---

而在创建Person类型的实例时，使用了**new**关键字。

new关键字会使以下几个步骤发生：

---

1. 创建一个新对象
2. 将构造函数的作用域赋给新对象（即this指向了这个新对象）
3. 执行构造函数中的代码
4. 返回新对象

---

这时候我们就可以检测person1的类型了：

{% codeblock lang:javascript %}
person1.constructor === Person;     // true
person2 instanceof Person;          // true
{% endcodeblock %}

构造函数模式的也存在一些问题：即每个方法都会在每个实例中重新创建一遍，比如**sayName()**方法。

{% codeblock lang:html %}
person1.sayName === Person2.sayName;        // fasle
{% endcodeblock %}

其实这两个实例完全可以共享一个sayName()方法。

## 原型模式

我们创建的每一个函数都有一个prototype属性，这个属性是一个指针，它指向一个对象，而这个对象的用途是包含所有实例都能共享的属性和方法。

使用原型对象的好处是可以让所有对象实例共享它所包含的属性和方法。

换句话说，不必在构造函数中定义对象实例的信息，而是在构造函数的原型对象中添加这些信息。

{% codeblock lang:javascript %}
// 创建构造函数
function Dog () {}

// 向原型对象中添加属性，这些属性将被所有实例共享
Dog.prototype.name = 'dog';
Dog.prototype.age = 2;
Dog.prototype.sayName = function () {
  console.log(this.name);
};

var dog1 = new Dog();
var dog2 = new Dog();
{% endcodeblock %}

以上代码，创建了构造函数Dog()，并且在其原型对象中增加了name、age两个属性，以及一个方法sayName()，最后实例化了两个实例对象dog1和dog2。

可以将以上代码中各个对象的关系展示如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/js%E5%8E%9F%E5%9E%8B%E5%9B%BE.png)

### 理解原型对象

我们每创建一个新函数，就会同时创建他们的**prototype**对象，这个对象也会自动获得**constructor**属性。

constructor属性是一个指针，它指向**prototype属性所在函数**。

因为所有实例都能共享构造函数的prototype对象，所以所有实例也会获得constructor属性。

另外，实例内部还包含一个指针，它指向构造函数的原型对象。这个指针没有标准的方式可以访问，但在一些浏览器中可以使用**\_\_proto\_\_**来访问。要始终铭记的一点是：这个连接存在于**实例对象**和**构造函数的原型对象**之间，而不是存在于实例与构造函数之间。

{% codeblock lang:javascript %}
dog1.__proto__ === Dog.prototype;      // true
{% endcodeblock %}

还有另外两种方法**isPrototypeOf()**和**Object.getPrototypeOf()**，可以用来确定实例对象和构造函数原型对象之间是否存在联系：

{% codeblock lang:javascript %}
Dog.prototype.isPrototypeOf(dog1);      // true
Object.getPrototypeOf(dog1);            // true
{% endcodeblock %}

### 原型模式的缺点

原型模式最大的问题，是由其共享的本质所导致的。

对于那些包含基本数据值的属性，可以通过在实例上添加一个同名属性，就可以覆盖。但是对于哪些引用类型值来说，问题就比较突出。以下代码就是很好的说明：

{% codeblock lang:javascript %}
function Dog () {};

// 重写原型对象
Dog.prototype = {
  constructor: Dog,
  name: 'dog',
  age: 20,
  friends: ['dogA', 'dogB']
}

var dog1 = new Dog();
var dog2 = new Dog();

dog1.friends.push('dogC');

// 只想改变dog1的friends，却将dog1和dog2的friends都改变了
console.log(dog1.friends);      // ["dogA", "dogB", "dogC"]
console.log(dog2.friends);      // ["dogA", "dogB", "dogC"]
{% endcodeblock %}

### 重写原型对象

以上代码，在重写原型对象时，增加了`construtor: Dog`，为什么要加这行代码呢？解释如下：

**我们每创建一个函数，就会同时创建它的prototype对象，这个对象也会同时获得constructor属性，指向我们创建的函数。**

我们重写 Dog.prototype = {}，其本质是完全覆盖了原来的prototype对象，也就是说Dog的prototype属性指向了一个新的对象，而这个新的对象虽然也叫prototype，但是它的constructor属性却不指向Dog，而是指向Object，所以需要我们显式地声明。

如果我们在已经创建实例的情况下重写原型，并且没有显式声明新原型对象的constrctor属性，那么就会切断所有现有实例于新原型之间的联系。我们看以下代码：

{% codeblock lang:javascript %}
function Dog () {}

// 先实例化对象
var dog1 = new Dog();

// 然后再重写原型对象
Dog.prototype = {
  name: 'dog',
  age: 20,
};

console.log(dog1.name);                          // undefined
console.log(dog1.__proto__ === Dog.prototype);       // fasle
console.log(dog1.__proto__.constructor === Dog);     // true
{% endcodeblock %}

图解如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E9%87%8D%E5%86%99%E5%8E%9F%E5%9E%8B.png)

## 组合使用构造函数模式和原型模式

创建自定义类型最常见的方式，就是组合使用构造函数模式和原型模式。

构造函数模式用于定义**实例属性**，而原型模式用于定义**方法**和**共享的属性**。

这样，每个实例都会有一份自己的实例属性的副本，但同时又共享这对方法的引用，最大限度地节省了内存。

{% codeblock lang:javascript %}
// 定义构造函数
// 构造函数中所有属性，每个实例都会有自己的副本
function Dog (name, age) {
        this.name = name;
        this.age = age;
        this.friends = ['dogA', 'dogB'];
}

// 原型模式，重写原型
// 原型对象中的方法和引用类型值的属性为所有实例共享
Dog.prototype = {
  constructor: Dog,
  sayName: function () {
    console.log(this.name);
  }
}

var dog1 = new Dog('dog1', 2);
var dog2 = new Dog('dog2', 3);

dog1.friends.push('dogC');

// dog1和dog2的friends都是实例属性，互不影响
console.log(dog1.friends);    // ["dogA", "dogB", "dogC"]
console.log(dog2.friends);    // ["dogA", "dogB"]

// dog1和dog2都可以同时使用sayName()方法
dog1.sayName();         // dog1
dog2.sayName();         // dog2
{% endcodeblock %}

以上就是对创建对象几种模式的简单总结，当然还有其他一些模式，后期逐步完善。



