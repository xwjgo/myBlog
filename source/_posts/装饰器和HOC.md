title: 装饰器和HOC
tags: [es6, react]
categories: react
toc: true
date: 2017-11-09 10:08:03
---

之所以把js的装饰器和react中的HOC放到一起来写，是因为这两个东西的原理是很相近的。他们都实现了在不修改函数（或react组件）本身代码的情况下，给该函数（或组件）添加一些额外行为的功能。

## js中的装饰器

js中的Decorator语法属于ES7（ES2016）的新特性，这里可能需要了解一下ECMA国际，即`ECMA International`这个企业会员制的国际标准化组织是如何制定新的ECMA规范的。

在ECMA国际，每个标准都会由一个TC（Technical Committees）负责，而ECMAScript的规范，就是由我们经常看到的TC39来负责标准化的。一般每个新的js特性，从开始到完成会经历如下几个阶段：

- stage-0，稻草人（Strawman）阶段，只是一个想法，任何人都可以提PR到[这个仓库](https://github.com/tc39/ecma262)。
- stage-1，提案（proposal）阶段，TC39审阅通过，已经值得好好研究了
- stage-2，草案（draft）阶段，有了最初的规范，并且有两个及以上的实现，比如babel
- stage-3，候选（cadinate）阶段，有了完整的规范和初步的浏览器实现
- stage-4，完成（finish）阶段，会加入到下一年的js版本中

本文要学习的Decorator语法目前正处于`stage-2`的阶段，因此我们需要`babel`的帮助。

### babel配置

首先安装所需的npm包：

{% codeblock lang:json %}
"dependencies": {
    "babel-cli": "^6.26.0",
    "babel-plugin-transform-decorators-legacy": "^1.3.4",
    "babel-preset-env": "^1.6.1"
}
{% endcodeblock %}

新建`.babelrc`文件，配置如下：

{% codeblock lang:json %}
{
    "plugins": ["transform-decorators-legacy"],
    "presets": ["env"]
}
{% endcodeblock %}

最后在package.json中添加如下脚本，表示将src目录下的js编译到lib目录下，并且开启监听：

{% codeblock lang:javascript %}
"scripts": {
    "build": "./node_modules/.bin/babel src --out-dir lib --watch"
}
{% endcodeblock %}

这样我们只要`npm run build`，就可以开心地使用Decorator的语法了。

关于babel，这里需要了解下其中`presets`、`pulugins`以及`stage-X`的概念。

- preset，插件的合集，就类似与一些套装，包含了制定的插件。比如`babel-preset-env`这个preset就包含了es2015/es2016/es2017/latest这4个插件的特性。
- plugin，单个插件，比如使用装饰器语法，就需要安装`babel-plugin-transform-decorators-legacy`这个插件。
- stage-x，相当于实验性质的preset，官方建议使用这些stage-X的时候要格外小心，尤其是stage-3之前的。
- 关于转码执行顺序，plugins会在presets之前执行，pulgins中各个插件是从前到后执行，而presets中各个套件的执行顺序为从后到前。

### 修饰类

js中的装饰器可以用来修饰类和修饰类的方法，先看修饰类的情况：

{% codeblock lang:javascript %}
@testable
@returnNewClass
class Person {
    constructor () {
        this.name = 'xwj';
    }
    sayHello () {
        console.log(`hello, my name is ${this.name}, my age is ${this.age}`);
    }
}

function testable (target) {
    console.log('testable...');
    target.testable = true;
}

function returnNewClass (target) {
    console.log('returnNewClass...');
    return class extends target {   // 这里返回的类，将作为下一个装饰器的输入
        constructor () {
            super();
            console.log('constructor...');    // 先执行装饰器，再执行这里的构造函数
            this.age = 23;
        }
    };
}

const xwj = new Person();
xwj.sayHello();
console.log(Person.testable);

/**
 * 输入结果：
 * returnNewClass...
 * testable...
 * constructor...
 * hello, my name is xwj, my age is 23
 * true
 */
{% endcodeblock %}

从上面代码中我们可以看出：

1. 类修装饰器接受的参数，即`target`为类本身。
2. 如果有多个装饰器同时修饰一个类，这些装饰器的执行顺序为`由近及远`。
3. 某个装饰器返回一个新的类，如果后面还有装饰器，那么新类将作为后续装饰器target传入。
4. 某个装饰器返回一个新的类，如果后面没有装饰器，那么原本的类名（Person）将指向新类。

### 修饰类的方法

{% codeblock lang:javascript %}
class Person {
    constructor () {
        this.name = 'xwj';
    }
    @readOnly
    sayHello () {
        console.log(`hello, my name is ${this.name}`);
    }
}

function readOnly (target, name, descriptor) {
    /**
     * descriptor默认值：
     * {
     *     value: f sayHello(),
     *     enumerable: false,
     *     configurable: true,
     *     writable: true
     * }
     */
   descriptor.writable = false;
   return descriptor;
}

const xwj = new Person();
xwj.sayHello = function () {   // 这里试图修改sayHello方法，会报错
    console.log('你好');
};
{% endcodeblock %}

从以上代码我们可以了解到：

1. 类的方法的装饰器接受3个参数，分别为`类的原型`，被修饰的方法名，该方法的descriptor。
2. 类的方法的装饰器，其实质就是修改并返回一个新的descriptor对象。
3. 新返回的descriptor对象，最终会通过`Object.defineProperty(Person.prototype, 'sayHello', newDesciptor)`来改变该属性的配置。

### 装饰器可以实现哪些功能？

- mixin

mixin其实就是将一个对象的属性或方法混入到另外一个对象中，其实现可以使用了`Object.assign()`。

{% codeblock lang:javascript %}
// 工厂函数的方式来定义装饰器
function mixin (...list) {
    return function (target) {
        console.log(target, list);
        Object.assign(target.prototype, ...list);
    }
}

const obj1 = {
    func1 () {
        console.log('func1...');
    },
    func2 () {
        console.log('func2...');
    }
};
const obj2 = {
    func3 () {
        console.log('func3...');
    },
    func4 () {
        console.log('func4...');
    }
};

@mixin(obj1, obj2)
class Person {
    constructor () {
        this.name = 'xwj';
    }
    sayHello () {
        console.log(`hello, my name is ${this.name}`);
    }
}

const xwj = new Person();
xwj.func1();
xwj.func3();
{% endcodeblock %}

- AOP

AOP，即`Aspect Oriented Programming`，面向切面编程。其核心理念为把一些与核心业务无关的逻辑抽离出来，然后在必要的时候，通过“动态织入”的方式掺入到应用模块中。AOP适用的场景有日志记录，埋点收集，权限控制，错误处理等。

如果用es5的方式，我们可能通过扩展Function的原型来实现AOP：

{% codeblock lang:javascript %}
Function.prototype.after = function (fn) {
    var self = this;
    return function () {
        var ret = self.apply(this, arguments);
        fn.apply(this, arguments);
        return ret;
    }
}
{% endcodeblock %}

现在假如需要在用户点击完登录按钮之后，输出一行日志。最直接的方式可能是修改onLoginIn函数，但是这样可能导致其他的一些问题。使用了AOP之后，我们则不需要修改onLoginIn函数，只需要执行如下的代码：

{% codeblock lang:javascript %}
this.onLogin = this.onLogin.after(function () {
    console.log('用户触发了登录！');
});
{% endcodeblock %}

用ES7的Decorator可以实现与AOP相同的效果：

{% codeblock lang:javascript %}
function doSomethingAfterThis (fn) {
    return function (target, name, descriptor) {
        const cacheValue = descriptor.value;
        descriptor.value = function (...args) {
            let ret = cacheValue.apply(this, ...args);
            fn.apply(this, ...args);
            return ret;
        };
    };
}

function logEvent () {
    console.log('sayHello finish!');
}

class Person {
    constructor () {
        this.name = 'xwj';
    }

    @doSomethingAfterThis(logEvent)
    sayHello () {
        console.log(`hello, my name is ${this.name}`);
    }
}

const xwj = new Person();
xwj.sayHello();
{% endcodeblock %}

以上需要注意的是：装饰器对目标行为的改变，是在编译时发生的，而不是运行时。所以，如果我们把logEvent写到class中，然后像这这样使用`@doSomethingAfterThis(this.logEvent)`是行不通的。

## HOC

HOC，即高阶组件(Higher-Order Component)，整体来说，一个高阶组件就是一个函数，它接收一个react组件，然后返回一个新的react组件。

{% codeblock lang:javascript %}
const EnhancedComponent = higherOrderComponent(WrappedComponent);
{% endcodeblock %}

HOC的主要作用为：**复用组件逻辑**。

下面是一个使用HOC来实现一个简单的目录组件：

{% codeblock lang:javascript %}
// HOC
function makeToggleable (WrappedComp) {
    return class extends React.Component {
        constructor () {
            super();
            this.state = {
                show: false
            }
        }

        toggle () {
            this.setState((prevState) => ({
                show: !prevState.show
            }));
        }

        render () {
            return (
                <div className="toggle-comp">
                    <WrappedComp {...this.props} onClick={this.toggle.bind(this)}/>
                    {this.state.show && this.props.children}
                </div>
            );
        }
    }
}

@makeToggleable
class Menu extends React.Component {
    render () {
        return (
            <div className="menu-comp" onClick={this.props.onClick}>
                <h1>{this.props.title}</h1>
            </div>
        );
    }
}
{% endcodeblock %}

然后我们就可以这样使用组件了：

{% codeblock lang:html %}
<div className="app">
    <Menu title="水果">
        <p>香蕉</p>
        <p>橘子</p>
        <p>榴莲</p>
    </Menu>
    <Menu title="衣服">
        <p>T恤</p>
        <p>裤子</p>
        <p>衬衫</p>
    </Menu>
</div
{% endcodeblock %}

在以上案例中，我们可以学习到：

1. HOC的范式最好为`(Component) => Component`，即传入HOC的参数除了Component外不要有其他，这样一来可以方便地用于ES7的Decorator中，二来也可以方便地进行compose。
2. 如果我们需要更多的参数来定制HOC，那么我们可以使用工厂函数来创建HOC，类似react-redux中的connect函数一样。
3. HOC和WrappedComponent之间的联系纯粹是基于props的。

另外，关于compose: comopse(f, g, h)相当于(...args) => f(g(h(...args)))，所以如果我们所有的HOC都是`(Component) => Component`的范式，就可以方便地组合很多HOC。如下：

{% codeblock lang:javascript %}
// 不实用compose的情况,可读性较差
const EnhancedComponent = hoc1(hoc2(hoc3(WrappedComponent)));

// 使用compose的情况
const enhance = compose(hoc1, hoc2, hoc3);
const EnhancedcComponent = enhance(WrappedComponent);
{% endcodeblock %}

最后，关于HOC我们可能还有如下几点需要注意：

1. 如果向EnhancedComponent传入ref回调，ref将不再指向WrappedComponent。折衷的方案为通过其他prop传递一个普通回调函数到WrappedComponent中。
2. WrappedComponent的所有静态方法将丢失，所以必须手动copy，
3. 不要在render函数中使用HOC，因为HOC每次返回一个全新的组件，它会完全代替旧的组件，并且导致整个组件的state和children都丢失。
4. 我们期望EnhancedComponent和WrappedComponent拥有完全相同的接口。

以上就是我对于Decorator和HOC的总结，还有待在实践中不断加深理解。

