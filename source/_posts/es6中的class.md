title: es6中的class
tags: es6
categories: 前端
toc: true
date: 2017-2-22 12:12:12
---

## 基本语法

es6中引入了class的概念，通过class关键字，可以定义一个类。如下：

{% codeblock lang:javascript %}
class Blog {
    // 构造方法
    constructor (title, author) {
        this.title = title,
        this.author = author
    }
    // 该方法被定义在原型对象上
    publish () {
        console.log(`${this.author}发布了${this.title}!`)
    }
}
{% endcodeblock %}

关于es6的class语法，有下面几点需要知晓：

---
1. ES6的class只是一种语法糖，它的绝大部分功能，es5都可以做到。
2. 定义方法时，不需要加function关键字。
3. 方法之间不需要用逗号分隔，加了会报错。
4. 类中的所有方法，都定义在原型对象上。
5. constructor方法默认返回实例对象，即this
---

## class的继承

类似与传统的面向对象语言，es6的calss之间可以通过`extends`关键字来实现继承。

下面让Dog类继承了Animal类：

{% codeblock lang:javascript %}
class Animal {
    constructor (name, age) {
        this.name = name;
        this.age = age;
    }

    speak () {
        console.log(`I am ${this.name}`);
    }
}

// Dog继承Animal
class Dog extends Animal {
    constructor (name, age, color) {
        super(name, age);    // 调用父类的constructor
        this.color = color;
    }
}

// 实例一个dog
const dog1 = new Dog('xwj', 21, 'gray');

dog1.speak();    // "I am xwj"
{% endcodeblock %}

在实现继承的时候，要注意以下几点：

---
1. 子类没有自己的this对象，而是继承父类的this对象，然后对其加工并返回。
2. 所以子类必须在constructor对象中调用super方法，且必须在构造方法第一行调用。否则得不到this对象，也无法在后面代码中使用this对象。
3. ES5的继承，实际上是创造子类的实例对象this，然后将父类方法添加到this上面。
---

另外，如果没有显示定义constructor方法，这个方法会被默认添加：

{% codeblock lang:javascript %}
constructor (...args) {
    super(...args);
}
{% endcodeblock %}

## super关键字

super在子类中，既可以当作**父类构造方法**来使用，也可以当作**父类原型对象**来调用。

{% codeblock lang:javascript %}
class Super {
    print () {
        console.log('super')
    }
}

class Sub extends Super {
    constructor () {
        super();    // 当作父类的构造方法使用，this指向子类实例
        super.print();  // 当作父类的prototype对象来使用
    }
}
{% endcodeblock %}

## 原生构造函数的继承

js中有一些原生的构造函数，比如Array(), String(), Date()等。

如果我们想继承这些原生的构造函数，在es5中是无法实现的，这是因为es5的继承总是先创建子类实例对象this，然后再用父类的构造函数修饰this对象。

而**原生构造函数的this恰恰无法通过apply或者call来动态绑定，这导致子类实例拿不到原生构造函中的内部属性**。

下面是继承失败的案例：

{% codeblock lang:javascript %}
function MyArray () {
    Array.apply(this, arguments);
}

MyArray.prototype = new Array();

var arr = new MyArray(1,2,3);
arr.pop();    // undefined
{% endcodeblock %}

但是es6允许继承原生的构造函数，因为es6是先创建父类的实例对象this，然后再用子类的构造函数修饰this。

{% codeblock lang:javascript %}
class MyArray extends Array {
    constructor (...args) {
        super(...args);
    }
}
let arr = new MyArray(1,2,3);
arr.pop();    // 3
{% endcodeblock %}

## class的getter和setter

和es5一样，在class内部可以使用`get`和`set`函数来拦截属性的存取行为。

{% codeblock lang:javascript %}
class Person {
    constructor (name, age) {
        this._name = name;
        this._age = age;
    }
    get age () {
        return this._age - 10;
    }
    set age (value) {
        this._age = value;
    }
}

let hyy = new Person ('hyy', 21);
hyy.age = 20;
console.log(hyy.age); // 10
{% endcodeblock %}

## 静态方法

类相当于实例的原型，所有在类中定义的方法，都会被实例继承。

如果在一个方法前，加上`static`关键字，那么这个方法就成为了静态方法，只能通过类名来调用。

{% codeblock lang:javascript %}
class Animal {
    static eat () {
        console.log('Animal like to eat food')
    }
}
Animal.eat();
{% endcodeblock %}

另外，**父类的静态方法，会被子类继承，并且可以从super关键字上来调用**。

## 静态属性

es6规定，class内部只有静态方法，没有静态属性。

如果想定义静态属性，这有下面这种方式可行。

{% codeblock lang:javascript %}
class Animal {
    // 这里只有方法，没有属性
}
Animal.love = 'food';    // 静态属性
{% endcodeblock %}

## 最后

关于es6的class，就先总结到这里。更详细的讲解可以看阮一峰老师的教程。
