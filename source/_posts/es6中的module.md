title: es6中的module
tags: es6
categories: 前端
toc: true
date: 2017-2-26 12:12:12
---

在es6出现之前，已经出现了为js制定的模块加载方案，最主要的有`commonJS`和`AMD`两种。commonJS用于服务器（比如node），AMD用于浏览器。

es6在语言标准层面上，又增加了一种模块语法，可能成为浏览器和服务器端通用的模块解决方案。

## es6模块设计思想

es6中模块的设计思想不同于commonJS和AMD。

### 编译时加载

es6模块加载尽量地保持静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量，这种方式可以只加载通过export导出的指定方法，称为`编译时加载`。

es6模块输出的是值的引用。

### 运行时加载

而commonJS模块依赖关系只能在运行时确定，先加载整个模块（对象），然后再取出某些方法来使用，这种方式称为`运行时加载`。

conmmonJS输出的是值的拷贝。

### 严格模式

es6模块默认采用严格模式。所以，必须遵守严格模式的一些规则，比如：

1. 不能使用arguments.callee
2. 禁止this指向全局作用域（顶层的this指向undefined）
3. 引入块级作用域，函数声明必须在顶层
4. arguments不再追踪参数变化（只指向原本传入的参数）

严格模式是由es5制定的，关于严格模式更多的限制，参考[这里](http://www.ruanyifeng.com/blog/2013/01/javascript_strict_mode.html)。

## 基本语法

模块功能主要有两个基本的命令：`export`和`import`。export指定模块对外提供的接口，import命令引入其他模块提供的功能。

### export

使用export可以导出一个所有**最外层声明的函数、类以及var，let，const声明的变量**。只需要在声明前面加上export关键字，或者使用大括号来来指明想要导出的api名称。

{% codeblock lang:javascript %}
// profile.js
// 方式1：在声明前加export关键字
export let count = 0;
export const xwj = {
    name: 'xuwnejiang',
    age: 21,
    school: 'ouc'
}
export function sayHello () {
    console.log('hello');
};
export class Person (name, age) {
    constructor () {
        this.name = name;
        this.age = age;
    }
};

// 方式二：使用大括号来指明导出列表
let count = 0;
const xwj = {
     name: 'xuwenjiang',
     age: 21,
     school: 'ouc'
};
function sayHello () {
    console.log('hello');
}
class Person (name, age) {
    constructor () {
        this.name = name;
        this.age = age;
    }
}
export {count, xwj, syaHello, Person};
{% endcodeblock %}

### import

我们可以在另外一个模块（js文件）中，导入我们需要的东西。和export一样，import也可以通过大括号来指定一个导入列表。

{% codeblock lang:javascript %}
import {count, xwj, sayHello, Person} from './profile.js';
syaHello();
{% endcodeblock %}

### 重命名

无论是export还是import，我们都可以给接口重命名。这样可以防止一些命名冲突，比如你要import某个接口，但是接口名称与当前模块中的某个变量名称发生了冲突，这时候，重命名的功能就派上用场了。

{% codeblock lang:javascript %}
// export重命名
let xwj = {};
function sayHello () {};
export {
    xwj as person,
    sayHello as action
};

// import重命名，比如从两个不同模块导入相同名称的api
import {sayHello as action1} from './one.js';
import {syaHello as action2} from './two.js';
{% endcodeblock %}

### import整个模块对象

我们可以通过`import *`的方式来一次性导入某个模块的所有api。

{% codeblock lang:javascript %}
import * as person from './profile.js';
person.syaHello();
console.log(person.count);
{% endcodeblock %}

### export default

es6模块是可以和commonJS以及AMD模块实现交互的，比如在一个nodejs的项目中，我们通过npm安装了lodash，那么我们可能会这样导入模块:

{% codeblock lang:javascript %}
// 为什么我们导入的时候既没有加大括号，也没有impot *
import _ from 'lodash';

// 这是因为在commondJS或者AMD模块中都有一个默认导出，上面代码等同与
import {default as _} from 'lodash';
{% endcodeblock %}

所以，如果我们想让模块有一个默认导出，方法很简单，**它与其他的导出唯一的不同之处在于导出api的名称被定为`default`**。

{% codeblock lang:javascript %}
// 方式1：在声明之前加 export default
// 相当于把该对象的值赋值给defalut并导出
export default {
    name: 'xuwenjiang',
    age: 21,
    school: 'ouc'
};

// 方式2：使用大括号导出
const xwj = {
    name: 'xuwenjiang',
    age: 21,
    school: 'ouc'
};
export {xwj as default};

// 注意
// export default后面可以跟声明，但是不可以跟表达式
export default const xwj = {};             // 赋值表达式，错误
export default let sayHello = () => {};     // 函数表达式，错误
export default function () {};  // 函数声明，正确
export default class {};        // 类声明，正确
{% endcodeblock %}

现在我们就可以想导入lodash时候一样导入xwj对象了，不需要使用大括号也不用`import *`。

{% codeblock lang:javascript %}
// 因为默认导出就只有一个，所以可以这样导入，且可以给默认api命名
import person from './profile.js';
console.log(person); // xwj对象
{% endcodeblock %}

### 聚合模块

有时因为模块较多，我们可能将很多模块的api聚合到一个模块中，再导出。那么， 这个模块，就成为聚合模块。

{% codeblock lang:javascript %}
// 方式1，不常用
import {apple} from './apple.js';
import {huawei} from './huawei.js';
import {hammer} from './smartisan.js';

export {apple, hawei, hammer};
// 方式二
export {apple} from './apple.js';
export {huawei} from './huawei.js';
export {hammer} from './smartisan.js';
// 还可以顺便重命名api
export {defalut as mi} from './mi.js';
export {vivo as vivoPlus} from './vivo.js';
{% endcodeblock %}

## import实际做了什么？

1. 语法解析：阅读模块的源代码，检查是否有语法错误。
2. 加载：递归地加载所有引入地模块。这部分还没有完全被标准化。
3. 连接：当遇到一个新加载的模块，为其创建作用域并将模块内所有声明填充到该作用域中，其中包括由其他模块导入的内容。
4. 运行：最终，在每一个新加载的模块中运行所有语句。此时，导入的过程就结束了。

## 参考

[es6-in-depth-modules](https://hacks.mozilla.org/2015/08/es6-in-depth-modules/)

