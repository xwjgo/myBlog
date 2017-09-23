title: immutable.js在react中的实践
tags: [immutable.js, react]
categories: react
toc: true
date: 2017-09-23 10:08:03
---

react是一个UI=f(prop|state)的框架，通过virtual dom的diff来决定对真实dom的修改，因为操作普通的js对象，比直接操作dom更加的高效。

当一个组件的prop或者state发生变化时，这个组件将进入更新阶段，如果shouldComponentUpdate()返回了true，那么将会比较前后两次render函数执行返回的`react element`是否相等，如果不相等，react将更新真实DOM。这个过程也就是所谓的`reconciliation`。

有时候为了提高组件性能，我们需要避免不必要的reconciliation，其关键就在于`shouldComponentUpdate()`这个方法。默认该方法总是返回true，即只要有prop或者state的变化，那么reconciliation过程一定会执行，这时候是否更新真实dom就取决于前后两次render返回的react element是否完全相等。

如果一些情况下，我们确定这个组件不需要reconciliation，那么我们就可以在shouldComponentUpdate()这个方法中进行前后state和prop的比较，适当地返回一些false。这样就可以避免reconciliation，render函数也就不需要执行，后续的diff当然也就不需要了。比如以下：

{% codeblock lang:javascript %}
shouldComponentUpdate (nextProps, nextState) {
    // 只有在props中的visible或者state中的inputValue发生变化时，才进行reconciliation
    return this.props.visible !== nextProps.visible || this.state.inputValue !== nextState.inputValue;
}
{% endcodeblock %}

当然，为了达到类似的效果，我们可以让组件继承`React.PureComponent`，它和`React.Component`类似，不同之处是它默认在shouldComponentUpdate()中对前后的prop和state进行了浅比较（即引用类型值比较引用），所以使用它的时候应该格外小心。比如下面这个子组件，如果父组件中执行`friends.push('xwj')`，虽然新的props.friends已经发生了变化，但是改子组件并不会更新，因为friends的引用并没有变。如下：

{% codeblock lang:javascript %}
class FriendsList extend React.PureComponent {
    render () {
        const {friends} = this.props;
        return (
            <p>{friends.join(',')}</p>
        );
    }
}
{% endcodeblock %}

## state必须是immuatble的吗？

不一定，我们如果直接修改state，貌似也可以达到我们想要的效果，比如在一个时钟程序中，可以这样更新时间：

{% codeblock lang:javascript %}
tick () {
    this.state.date.second++;
    this.setState({});
}
{% endcodeblock %}

但是这种做法是不被推荐的，原因如下：

1. state的更新是异步的，react可能会将多次setState合并为一次状态修改，以提高性能。上面的修改显然是同步的方式。
2. 如果date被当作props传给了一个子组件，并且该子组件继承了PureComponent，那么即使修改了date中的second，子组件也不会更新。
3. react内部的浅比较或者merge可能会被扰乱，因为现在其实只有一个state，这可能会扰乱react的生命周期函数。
4. 如果想实现Undo/Redo这种逻辑，会很麻烦。

总之，大多数情况下，我们直接修改state并不会影响app的运行，这是因为我们的应用还很小，逻辑也很简单，一旦app变得复杂且庞大，我们的app可能因此表现不正常甚至奔溃。同时，我们的代码也将变得不可维护，跨组件的state也会失去控制。

因此，react官方也强烈建议state为immutable的，这样才算是最佳实践吧。

## 如何让state变得immutable？

通常为了避免state的修改，我们会使用浅复制(shallow copy)或者深复制(deep copy)的方式来得到新的state。比如有下面一个state对象：

{% codeblock lang:javascript %}
const state = {
    name: 'myBookStore',
    owner: 'xwj',
    books: [
        {name: 'book1', price: 100},
        {name: 'book2', price: 200},
    ]
};
{% endcodeblock %}

### 深复制

假设我们想更新一下书店的owner属性，如果采用深复制的方式，那么整个对象都会被复制一份，如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/deepCopy.png)

上图中，圆形代表引用类型，矩形代表基本数据类型。在深复制结束之后，我们就可以通过`newState.owner = 'newOwnerName'`来更新state了。

这样的缺点在于：1. 深复制耗费性能，尤其在数据层级较深的时候。2. 没有为渲染环节提供高渲染效率的铺垫。

### 浅复制

这里的浅复制并非只复制state的引用，而是更新变化的节点及其父节点。同样是更新书店的owner，浅复制模式如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/shallowCopy.png)

因为state内部数据发生变化，所以创建新的state引用。而books中的数据没有发生变化，所以直接复制该引用就可以了。

这种实现immutable的方法不仅性能较好，并且提高了组件渲染的运行效率。比如books数组很大，并且这个books是传递给子组件渲染一个书籍列表，那么深复制的话，所有的book都是全新的引用，所以会重新reconciliation，但是浅复制的方式, 由于books引用没变，就不需要reconciliation，这样性能显然提升很多。当然子组件需要加上如下代码：

{% codeblock lang:javascript %}
shouldComponentUpdate (prevProps) {
    return prevProps.books !== this.props.books;
}
{% endcodeblock %}

当然，这样的更新效果，可以使用es6提供的[Object.assign()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)方法以及[扩展操作符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator)来实现, 这两种方式的本质都是浅复制。下面是简单的例子：

{% codeblock lang:javascript %}
// Object.assign()
const newState = Object.assign({}, state, {'owner': 'newOwnerName'});

// 扩展运算符
const newState = {...state, owner: 'newOwnerName'};
{% endcodeblock %}

上面只是简单地修改了owner，如果想要用浅复制的方式来修改第一个book对象的price为500。我们应该怎么处理呢？最终的新state应该如下图所示：

![](http://7xvlvo.com1.z0.glb.clouddn.com/shaollwCopy2.png)

对应到更新代码，如下：

{% codeblock lang:javascript %}
const newState = Object.assign({}, state, {
    books: [Object.assgin({}, state.books[0], {price: 500}), ...state.books.slice(1)]
});
{% endcodeblock %}

我们可以看到，这种浅复制操作的实现过程是相当繁琐的，如果层级再深一点，代码可读性将大大降低。于是就有了immutable.js这个专门处理immutable data的库，它可以**使用类似赋值的方式来生成浅复制的不变性数据**。

## immutable.js在react中的使用

immutable.js提供了List、Map、Set等不可变数据类型，同时提供了大量与原生Object或者List类似的api。

我们可以使用`fromJS()`来将state.books变为不可变数据类型，使用`toJS()`来将state.books变为普通的js数组。

下面immutable.js在react中的一些使用场景：

{% codeblock lang:javascript %}
// 场景1，增加一本书
this.setState((prevState) => ({
    books: prevState.books.push(fromJS({name: 'book3', price: 300}));
}));

// 场景2，删除第二本书
this.setState((prevState) => ({
    books: prevState.books.delete(2);
}));

// 场景3, 修改第二本书的价格为500
this.setState((prevState) => ({
    books: prevState.books.setIn(['1', 'price'], 500);
}));

// 场景4，如果层级很深，可以使用deleteIn、updateIn、mergeDeepIn等
// 比如删除第3个模块下的第1篇文章的第2个章节
this.setState((prevState) => ({
    modules: prevState.modules.deleteIn(['2', 'articles', '0', 'sections', '1']);
}));
{% endcodeblock %}

其他的api就不一一举例了，我们可以在[immutable.js官网](https://facebook.github.io/immutable-js/docs/#/)的控制台中方便地测试这些api。

关于immutable.js在react中的使用，就先写到这里吧，后面还需要更多的实践来深入学习它。






