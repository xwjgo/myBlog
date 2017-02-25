title: react基础
tags: react
categories: 前端
toc: true
date: 2017-2-23 12:12:12
---

最近项目中在用到react，这里就整理一下react相关的基础知识。

## 安装

如果使用npm的话，需要安装两个基础的npm包，`react`和`react-dom`。

## 第一个组件

{% codeblock lang:javascript %}
class ShoppingList extends React.Component {
    render () {
        return (
            <div className="shopping-list">
                <h1>{this.porps.name}</h1>
                <ul>
                    <li>显示器</li>
                    <li>主机</li>
                    <li>机械键盘</li>
                </ul>
            </div>
       )
    }
}
{% endcodeblock %}

上面创建了一个最基本的react组件，其中render函数中用到了`JSX`的语法。

JSX仅仅是js的语法糖，其最终还是要被转换成js的，可以使用`webpack`+`babel-loader`。

一个react组件接收一个参数`props`，然后通过render函数返回一个`reactNode`对象，注意是一个，而不能是多个。

接下来就可以这样使用ShoppingList组件：

{% codeblock lang:javascript %}
ReactDom.render(
    <ShoppingList shoppingList='购物清单'/>,
    document.querySelect('#app')
);
{% endcodeblock %}

可以发现`props`在调用组件的时候的传入，就好比我们调用函数时传入参数一样。

## props

上面的组件是一个无状态（stateless)组件，只能接收props，返回reactNode。

所以，它可以简写为如下形式：

{% codeblock lang:javascript %}
const ShoppingList = (props) => (
    <div className="shopping-list">
        <h1>{this.porps.shoppingList}</h1>
        <ul>
            <li>显示器</li>
            <li>主机</li>
            <li>机械键盘</li>
        </ul>
    </div>
);
{% endcodeblock %}

这种react组件被称为`functional component`。

另外还有以下几点需要注意：

1. props是只读的
2. 所有的组件必须表现地像`pure function`，不可以修改props。

## state

当然，既然有stateless组件，肯定也有stateful组件。

stateful的组件就不能定义为`functional component`，而必须是`class component`。后面将要介绍的包含生命周期函数（lifecycle hooks）的组件也只能定义为`class component`。

看这个时钟的案例：

{% codeblock lang:javascript %}
class Clock extend React.Component {
    constructor (props){
        super(props);    // 这个一定要记得调用，且在第一行调用
        this.state = {
            date: new Date()
        }
    }
    componentDidMount () {    // lifecycle hook
        this.timerId = serInterval(
            () => this.tick(),
            1000
        );
    }
    componentWillUnmount () {    // lifecycle hook
        clearInterval(this.timerId);
    }
    tick () {
        this.setState({
            date: new Date()
        });
    }
    render () {
        return (
            <div>
                <h1>当前时间为：</h1>
                <p>{this.state.date}</p>
            </div>
       );
    }
}

ReactDom.render(
    <Clock/>,
    document.querySelector('#app')
);
{% endcodeblock %}

这个组件的运行过程如下：

1. 在通过ReactDom.render来渲染Clock组件到DOM时，调用`constructor()`。
2. 接着调用`render()`并更新DOM。
3. 将Clock挂载到DOM后，调用`componentDidMount()`，它开启了一个定时器，让浏览器每隔1s调用tick()。
4. 1s后，执行`tick()`，并且在tick中执行`setState()`。
5. state的改变会触发`render()`函数的再次调用（注意这时constructor，componentDidMount不会再调用）。
6. 如果Clock组件被从DOM中移除，那么会调用`componentWillUnmount()`，停止定时器。

另外使用state时，还有以下几点需要注意：

---

- 不要直接修改state对象，而要通过`this.setState()`。

{% codeblock lang:javascript %}
// 错误，但是在constructor函数中可以这样写
this.state.commment = 'hello';

// 正确
this.setState({
    comment: 'hello'
});
{% endcodeblock %}

- state和props更新可能异步的，因为react出于性能考虑，可能将多个setState()调用打包到一起来执行。所以我们不能依赖与它当前的值来计算下一个state。

{% codeblock lang:javascript %}
// wrong
this.setState({
    counter: this.state.counter + this.props.increment
});
// right，我们可以传入一个函数来当作setState参数
this.setState((prevState, props) => ({
    counter: prevState.counter + props.increment
}));
{% endcodeblock %}

- state对象的更新是通过merge来实现的。所以，你调用setState的时候，只需要改变你关心的state分片就可以。

{% codeblock lang:javascript %}
constructor (props) {
    super(props);
    this.state = {
        posts: [],
        comments: []
    }
}
componentDidMount () {
    fetchPost().then(response => {
        this.setState({
            posts: response.posts
        });
    });
    fetchComment().then(response => {
        this.setState({
            comments: response.comments
        });
    });
}
{% endcodeblock %}

- state的初始化，即可以在class component中的constructor()中使用`this.state`来赋值，也可以在使用`React.createClass()`中提供`gitInitialState()`方法来初始化。

- 在应用中，所有数据的变化都源于一个`source for truth`，通常就是state。如果多个子组件都需要这份数据，可以将state放在离这些子组件最近的公共祖先组件中，在祖先组件中，通过props将state（分片）分发给各个子组件，这也是react中数据流自上而下流动的体现。

## 组件生命周期

一个React组件的声明周期大致分为三个阶段：

- 挂载阶段（Mounting），发生在组件被挂载到真实DOM时
- 更新阶段（Updating），组件重新渲染成选你DOM并决定真实DOM是否需要更新时
- 卸载阶段（Unmounting），组件从DOM中卸载时

下图列出了在这三个阶段对应的方法以及执行顺序：

![](http://7xvlvo.com1.z0.glb.clouddn.com/react%E7%BB%84%E4%BB%B6%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

尤其注意各个lifecycle hooks在什么阶段会执行，比如constructor()只会在挂载阶段执行，更新阶段不执行。

## 列表项的key

我们常常有这样的需求，根据一个数组，渲染出一个列表。

这时，我们必须为每个列表项提供一个独一无二的属性`key`，React可以用这个key来鉴别出哪个列表项被增加、修改或者删除了。

{% codeblock lang:javascript %}
render () {
    const numbers = [1,2,3,4,5];
    const items = numbers.map((num) => (
        <li key={num.toString()}>num</li>
    ));
    return (
        <ul>
            {items}
        </ul>
    );
}
{% endcodeblock %}

注意：**在哪里生成reactNode数组，就在哪里使用key**。

## Forms

在原生的HTML中，`<input>`,`<textarea>`,`select`这些表单元素维持着自己的状态`value`，并且根据用户的输入来自动更新value。

而在react中，所有组件的状态应该在构造函数中声明，并且只能够通过`setValue`来更新状态。

所以，在react中的表单元素会是下面这个样子。

{% codeblock lang:javascript %}
class InfoForm extends React.Component {
    constructor (props) {
        super(props);
        this.state = {
            title: 'react',
            type: 'frontend',
            content: ''
        }
    }
    handleChange (event) {
        const target = event.target;
        const value = target.value;
        const name = target.name;    // 给每个表单组件加了name属性之后，可通过event.target.name来访问该属性
        this.setState({
            [name]: value
        });
    }
    handleSubmit (event) {
        event.preventDefault();
        console.log(this.state);
    }
    render () {
        return (
            <form onSubmit={this.handleSubmit.bind(this)}>
                <label>
                    标题：
                    <input name='title' type='text' value={this.state.title} onChange={this.handleChange.bind(this)}/>
                </label>
                <br/>
                <label>
                    类型：
                    <select name='type' value={this.state.type} onChange={this.handleChange.bind(this)}>
                        <option value="frontend">前端</option>
                        <option value="backend">后台</option>
                        <option value="other">其他类型</option>
                    </select>
                </label>
                <br/>
                <label>
                    内容：
                    <textarea name='content' value={this.state.content} onChange={this.handleChange.bind(this)}/>
                </label>
                <br/>
                <input type='submit' value='提交'/>
            </form>
        )
    }
}
{% endcodeblock %}

## 合成还是继承？

React给出的答案是合成（composition），而非继承（inheritance)。在React中，可以通过props提供了很强大的合成组件的功能。

### props.children

我们常常有这样的需求，一个组件在定义的时候并不知道具体的内容是什么，比如`SideBar`或者`Dialog`。

这时候，我们使用特殊的`props.children`来定义该组件：

{% codeblock lang:javascript %}
// 定义组件
let SideBar = props => (
    <div className="sidebar">
        {props.children}
    </div>
)

// 调用组件
ReactDom.render(
    <SideBar>
        <ul>
            <li>item1</li>
            <li>item2</li>
            <li>item3</li>
        </ul>
    </SideBar>
);
{% endcodeblock %}

### 多个slot

如果我们想定义一个包含多个插槽（slot）的组件，那么我们可以通过props来实现。

因为props可以向组件传递任何类型的数据，包括对象、函数、ReactNode。

{% codeblock lang:javascript %}
// 定义组件
let SplitPane = props => (
    <div className="split-pane">
        <div className="split-pane-left">
            {this.props.left}
        </div>
        <div className="split-pane-right">
            {this.props.right}
        </div>
    </div>
);

// 调用组件
ReactDom.render(
    <SplitPane
        left={
            <Contracts/>
        }
        right={
            <Chat/>
        }
    >
);
{% endcodeblock %}

## thinking in react

- 组件架构方式有两种：从整体到局部和从局部到整体。在简单的应用中，推荐从整体到局部，在复杂的应用中，推荐从局部到整体。

- 让你的app只保存最小量的state数据。比如在todoList中，state中保存了todos的一个数组，就不需要保存todos数组的长度。因为这可以根据todos数组计算出来。

- 把app所需的所有数据罗列出来，并根据以下3条原则来判定哪些数据要当作state：
1. 如果该数据可以通过props传递给其他组件，那么可能不是state。
2. 如果该数据一直保持不变，可能不是state。
3. 如果该数据可以根据其他state或者props计算得到，可能不是state。

## 最后

以上就是react的基础部分，后面再整理react的进阶知识以及redux，react-redux的实际应用。

