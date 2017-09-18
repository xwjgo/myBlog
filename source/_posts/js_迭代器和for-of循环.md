title: 迭代器和for-of循环
tags: es6
categories: 前端
toc: true
date: 2016-7-14 12:12:12
---

## 从数组遍历谈起

最传统的数组遍历方法应该是以下这样：

{% codeblock lang:javascript %}
for (var i = 0, len = myArr.length; i < len; i ++) {
  console.log(myArr[i]);
}
{% endcodeblock %}

这种方式看起来比较麻烦。

### forEach()

后来ECMAScript5发布之后，我们可以使用[forEach()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)方法来遍历数组：

{% codeblock lang:javascript %}
myArr.forEach((value, key, array) => {
  console.log(key + '-->' + value);
});
{% endcodeblock %}

它可以**读取数组的键，也可以读取数组的值**。

但是，它**不可以在遍历过程中使用break、return来跳出循环，也不能使用return返回数据**。

### for-in

我们也曾使用[for-in](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...in)来遍历数组：

{% codeblock lang:javascript %}
for (var key in myArr) {
  console.log(typeof key);  // string
}
{% endcodeblock %}

for-in循环用于**遍历键（用字符串表示），而不是遍历值**。

它有以下3个明显的缺点：

---
1. 数组的键（索引）为数组，而for-in循环遍历的到的是**字符串'0','1','2'...**
2. for-in除了遍历数组键名外，还会**遍历数组的自定义键名**，比如myArr.name，则name会被遍历出来
3. 一些情况下，for-in会以**任意顺序遍历**键名。
---

### for-of

ES6为我们提供了一种新的循环语法：[for-of](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...of)。

它的基本用法如下：

{% codeblock lang:javascript %}
for (var value of myArr) {
  console.log(value);
}
{% endcodeblock %}

for-of避开了for-in循环的所有缺陷，它**遍历的是数组的值**。

并且，与forEach()不同，它可以**正常响应break、continue和return语句**。

{% codeblock lang:javascript %}
// 只会输出数组中小于3的值
for (var value of myArr) {
  if (value < 3) {
    console.log(value);
  } else {
    break;
  }
}
{% endcodeblock %}

### 对比

---
1. for-in**遍历键（字符串表示）**，适合用来**遍历对象属性**。
2. for-of**遍历值**，并且可以**遍历更多的数据类型**（下面会讲到），并且可以**配合break、continue、return等使用**。
3. forEach()可以**遍历键和值**，但是**不能使用break、continue等退出循环，也不能使用return返回数据**。
---

## 更强大的for-of

for-of循环其实主要为遍历**可迭代对象**（具有**迭代器方法**的一切对象）而生。

ES6规定，迭代器方法为**`[Symbol.iterator]()`**方法。

所以，只要某个对象实现了**`[Symbol.iterator]()`**的属性，那么这个对象就是可迭代对象，它就可以用for-of遍历。

在ES6中，有3类数据结构原生具有迭代器方法: **数组、某些类似数组的对象（NodeList、arguments、字符串）、Set和Map结构**。

### for-of遍历原理

---
1. for-of循环遍历某种数据结构时，会首先调用数据结构的访问器方法，即**`[Symbol.iterator]()`**方法，这个方法**返回一个新的迭代器对象**。迭代器对象是**任何具有`next()`方法的对象**，它不同于可迭代对象。

2. for-of循环将**重复调用迭代器对象中的`next()`方法**，每次循环调用一次。
3. 每次调用`next()`方法，都会**返回一个包含`value`和`done`两个属性的对象**，其中`value`属性值为此次迭代的输出结果，而`done`属性值是一个布尔值，表示遍历是否结束。
---

以下代码定义了一个很简单的可迭代对象:

{% codeblock lang:javascript %}
// 一个简单的可迭代对象
var easyIterator = {
  [Symbol.iterator] () {
    return this;
  },
  next () {
    return {
      value: ++ this.num,
      done: false
    };
  },
  num: 0
};

// 使用for-of循环来遍历
for (let num of easyIterator) {
  if (num < 10) {
    console.log(num);
  } else {
    break;
  }
}

// 输出1-9
{% endcodeblock %}

### 遍历更多数据类型

除了遍历数组外，for-of还可以遍历更多的数据类型，包括**Set和Map结构、类数据对象（比如arguments、DOM NodeList对象、字符串）、Generator对象**。

我们来看以下几个场景：

{% codeblock lang:javascript %}
/*
 * 场景1： 遍历Set和Map结构，其原生具有迭代器方法
 * 遍历的顺序是按照各个成员被添加进数据结构的顺序
 * Set结构遍历时，返回的是一个值；但是Map结构则返回一个数组，包含key和value
 */
 
var colors = new Set(['red', 'red', 'green', 'blue']);
for (let color of colors) {
  console.log(color);
}
// red
// green
// blue

var xwj = new Map();
xwj.set('name', 'xuwenjiang');
xwj.set('age', 20);
xwj.set('school', 'OUC');
for (let [key, value] of xwj) {
  console.log(key + '-->' + value);
}
// name-->xuwenjiang
// age-->20
// school-->OUC

/*
 * 场景2： 遍历计算生成的数据结构
 * ES6的数组、Set、Map都部署了以下三个方法，调用后都返回迭代器对象
 * entries()：返回一个迭代器对象，用来遍历[key, value]
 * keys(): 返回一个迭代器对象，用来遍历所有键名
 * values(): 返回一个迭代器对象，用来遍历所有键值
 */

var arr = ['a', 'b', 'c'];
for (let item of arr.entries()) {
  console.log(item);
}
// [0,'a']
// [1,'b']
// [2,'c']

/*
 * 场景3： 遍历类似数组的对象，原生具有迭代器方法
 * 类似数组的对象有：字符串、DOM NodeList、arguments
 */

// 1. 字符串
var myStr = 'xwj';
for (let c of myStr) {
  console.log(c);       // x   w   j
}

// 2. DOM NodeList对象
var balls = document.querySelectorAll('.ball');
for (let ball of balls) {
  ball.classList.add('big');
}

//3. arguments对象
function sum () {
  let sum = 0;
  for (let x of arguments) {
    sum += x;
  }
  return sum;
}
console.log(sum(1,2,3,4));    // 10

/*
 * 场景4：当然，并非所有的类数组对象都具有迭代器方法
 * 但是我们可以使用Array.from()方法将其转换为数组
 * 类数组对象：拥有一个length属性和若干索引属性的任意对象
 */

var car = {
  length: 2,
  0: 'redColor',
  1: 'bigSize'
};
console.log(Array.from(car));    // ["redColor", "bigSize"]
for(let x of Array.from(car)) {
  console.log(x);
}
{% endcodeblock %}

### 遍历普通对象

for-of**不能直接遍历普通对象**，只有实现了迭代器方法`[Symbol.iterator]()`才能使用。

这种情况下，我们可以**使用for-in来遍历键名**。

或者采用[Object.keys()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)来**将对象的键名生成一个数组，然后遍历这个数组**，代码如下：

{% codeblock lang:javascript %}
var xwj = {
  name: 'xuwenjiang',
  age: 20,
  school: 'OUC'

};
for (let key of Object.keys(xwj)) {
  console.log(key + '-->' + xwj[key]);

}
{% endcodeblock %}

## 迭代器对象的return()和throw()

迭代器对象，除了具有`next()`方法，还有`return()`和`throw()`方法。

其中**next()方法是必须的，而return()和throw()方法时可选的**。

return()方法的使用场合是：如果for-of循环因为**出错**，或者因为**break、continue语句而提前退出**，就会调用return()方法。

throw()方法主要是配合Generator函数使用，一般的迭代器对象用不到这个方法。

## 默认调用迭代器方法的场合

有一些场合会默认调用可迭代对象的迭代器方法，即`[Symbol.iterator()]`方法，除了for-of循环，还有以下几个场合。

### [解构赋值](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

对数组和Set解构赋值时，会默认调用迭代器方法。以下代码证明了这一点：

{% codeblock lang:javascript %}
var mySet = new Set().add('a').add('b').add('c');
let [x, y, z] = mySet;
console.log(x + '-->' + y + '-->' + z);    // a-->b-->c

// 假如我们加上以下代码
// 则输出结果为 'next-->next-->next'
mySet[Symbol.iterator] = () => {
  return {
    next () {
      return {
        value: 'next',
        done: false
      };
    }
  };
};
{% endcodeblock %}

也就是说，我们可以**将任何具有迭代器方法的数据结构，进行解构赋值**。

### [扩展运算符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Spread_operator)

扩展运算符也会默认调用迭代器方法：

{% codeblock lang:javascript %}
// 例一
var str = 'xwj';
console.log(...str);        // ['x','w','j']

// 例二
var arr = ['b', 'c']
console.log(['a', ...arr, 'd']);        // ['a','b','c','d']
{% endcodeblock %}

也就是说，我们可以**将任何具有迭代器方法的数据结构，通过扩展运算符，将它转化为数组**。

### 其他场合

由于数组的遍历会调用迭代器方法，所以任何可以接收数组作为参数的场合，其实都调用了迭代器方法。下面是一些例子：

---
1. [Array.from()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/from)
2. Map(),Set()
3. Promise.all()
4. Promise.race()
---

## 参考资料

- [ES6 In Depth: Iterator and the for-of loop](https://hacks.mozilla.org/2015/04/es6-in-depth-iterators-and-the-for-of-loop/)
- [阮大大的ES6教程](http://es6.ruanyifeng.com/#docs/iterator#字符串的Iterator接口)
- [可迭代对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/iterable)
- [for...of](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...of)



