title: redux和react-redux基本应用
tags: [react, redux, react-redux]
categories: 前端
toc: true
date: 2017-2-25 12:12:12
---

redux是一个状态管理工具，它可以不限于react的很多框架结合使用。本文就着重整理出readux的基本理念，以及react-redux在react应用中简单使用。

## redux使用场景

如果我们的应用交互很简单，那我们可能并不需要redux，如果非要牛刀小用，可能为带来更多的复杂性。

redux的使用场景为**多交互、多数据源**。

从组件角度看，下面几种场景可以考虑使用redux：

1. 某个组件的状态，需要共享。
2. 某个状态需要在任何地方都可以拿到。
3. 一个组件需要改变全局状态。
4. 一个组件需要改变另一个组件的状态。

redux的基本教程可以访问其[官方文档](http://redux.js.org/)。

这里再重复一下其中强调的三条原则：

1. 整个应用的唯一一颗状态树state存储在`store`中。
2. 改变state对象的唯一方式是dispatch一个`action`。
3. `reducers`必须是pure function。

## redux工作流

如果想在一个应用中使用redux，那么下面是大致的工作流程：

### 设计状态树

本文就按照官方文档的todoApp为实例，整个应用的状态树如下：

{% codeblock lang:javascript %}
{
    filter: 'SHOW_ALL'|'SHOW_COMPLETED'|'SHOW_ACTIVE',
    todos: [
        {
            id: '1',
            text: 'eat, and sleep',
            completed: false
        },{
            id: '2',
            text: 'compute programming',
            completed: true
        }
    ]
}
{% endcodeblock %}

### 定义actions

一个actions就是一个普通的js对象，必须含有type属性，其他数据自定。
能够返回actions的函数，就是actionCreators。

{% codeblock lang:javascript %}
// 一个普通的action
{
    type: 'ADD_TODO',
    id: 0,
    text: 'eat'
}
// 一个actionCreator
let nextTodoId = 0;
const addTodo = text => ({
    type: 'ADD_TODO',
    id: nextTodoId ++,
    text
});
{% endcodeblock %}

当然，如果actions比较多，我们可以将它单独写到一个js文件中。

### 定义reducers

一个reducer就是一个纯函数，它接收两个参数：state(分片)和action。

reducers的职责就是根据当前state以及action来返回下一个state。

但是我们在定义reducer的时候，有以下几点需要注意：

1. 不要更改传入reducer的state。
2. 对于不识别的action，通过default默认返回state。
3. 每个reducer可以只管理state的一个分支，最后再合并成一个根reducer。
4. 使用redux的`combineReducers()`方法即可合并多个reducers。

{% codeblock lang:javascript %}
// 一个reducer
function visibilityFilter (state = 'SHOW_ALL', action) {
    switch(action.type) {
        case 'SET_VISIBILITY_FILTER':
            return action.filter;
        defalut:
            return state;
    }
}
// 另一个reducer
function todos (state = [], action) {
    switch (action.type) {
        case 'ADD_TODO':
            return [
                ...state,
                {
                    id: action.id,
                    type: 'ADD_TODO',
                    text: action.text,
                    completed: false
                }
            ];
        case 'TOGGLE_TODO':
            return state.map(function (todo) {
                if (action.id === todo.id) {
                    return Object.assign({}, todo, {
                        completed: !todo.completed
                    });
                }
                return todo;
            });
        default:
            return state;
    }
}
// 合并reducer
const todoApp = combineReducers({
    visibileTodoFilter,
    todos
});
{% endcodeblock %}

### 定义全局唯一的store

store的作用就是维护app的唯一一颗状态树，即state对象。

它的生成方式如下：

{% codeblock lang:javascript %}
let store = createStore(reducer, [initialState]); // createStore方法由redux提供
{% endcodeblock %}

store提供了几个api，供react组件使用：

1. getState():获取state对象
2. dispatch(action):分发action来更新state对象（并不修改旧的state对象）
3. subscribe(listener):新state产生时触发listener，并返回解注册的句柄

{% codeblock lang:javascript %}
// 监听新state的产生，并打印新state对象
let unsubscribe = store.subscribe(() => {
    console.log(store.getState());
});
// dispatch一些actions
store.dispatch(addTodo('sleep'));
store.dispatch(addTodo('play basketball'));
store.dispatch(addTodo('learning react'));
store.dispatch(toggeleTodo(1));
store.dispatch(setVisibilityFilter('SHOW_COMPLETED'));
// 停止监听新state的产生
unsubscribe();
{% endcodeblock %}

至此，redux的基本工作流就总结完了。

## redux数据流

下面我们就上面的案例来分析一下redux的数据流向

### 你通过store分发一个action

action只是一个普通的js对象，不过可能是通过actionCreator来动态生成的action。

你可以在app的任何的地方dispatch action。

### store调用你传递的reducer

store将当前state对象和接收到的action作为参数，传递给reducer。

当然reducer很可能是一个复合的reducer（即通过combineReducers生成的），比如todoApp就是`todos`和`visibleTodoFilter`这两个reducer复合而成。

此时对根reducer的调用会转化为对两个子reducer的分别调用，如下：

{% codeblock lang:javascript %}
let nextTodos = todos(state.todos, action);
let nextVisibleTodoFilter = visibleTodoFilter(state.visibleTodoFilter, action);
{% endcodeblock %}

最后，返回两个子reducer的返回结果，构成新的state：

{% codeblock lang:javascript %}
return {
    todos: nextTodos,
    visibleTodoFilter: nextVisibleTodoFilter
}
{% endcodeblock %}

### 新state生成，触发监听器

触发由`subscribe(listenser)`所指定的监听器。

如果你使用了`react-redux`，此时你就可以`setState(nextState)`。

## Presentational组件和Container组件

接下来应该将rudux应用于我们的react应用，也就是介绍`react-redux`的时候了。

但是，在此之前，我们需要先理解react中关于组件的一种分类，即`Presentational`组件和`Container`组件。

下面为翻译整理的内容，原文请看[这里](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.6hyf565p5)。

### Presentational组件

Presentational组件是一种与redux无关，只会从props获取数据，通过props传递的回调函数来修改数据的一种组件。

它具有以下特点：

1. 只关心看起来像什么样子，通常包含DOM节点和Styles。
2. 可能同时包含Presentational组件和Container组件。
3. 是stateless的，不需要和redux的store有直接的联系，一般被当作功能组件。
4. 需要程序员手写实现，比如Page，SideBar，UserInfo等组件。

{% codeblock lang:javascript %}
const Link = ({active, children, onClick}) => {
    if (active) {
        return <span>{children}</span>
    }
    return (
        <a href="#" onClick={e => {
            e.preventDefault();
            onClick();
        }}>
            {children}
        </a>
   );
};
{% endcodeblock %}

### Container组件

Container组件与redux息息相关，并且通过subscribe redux的store来获取数据，通过dispatch redux的action来更新数据。

它具有以下特点：

1. 关心数据是如何运转的，比如data fetching，state update，但是不包含DOM和styles。
2. 可能同时包含Presentational组件和Container组件。
3. 是stateful的，通常作为数据源，通过subscribe来获取store的state更新，并将更新通过props应用于其他组件。
4. 组件由`react-redux`来实现，比如UserListContainer等。

## 使用react-redux来构建app

通过了解`Presentational`组件和`Container`组件的基本概念之后，我们就可以继续写我们的react app了。

现在再重新捋一遍我们的流程：

1. 设计状态树
2. 定义actions(actionCreators)
3. 定义reducers
4. 设计Presentational组件（比如上一小节的Link组件）
5. 生成Contianer组件
6. 生成store对象，并传递到所有组件

本小节主要介绍第5步和第6步。

### 使用connect()生成Container组件

`connect()`是`react-redux`提供的api，它充当react组件和redux store之间的桥梁。顾名思义，也就是将react组件连接到redux的store上。

关于connect()详细的api请移步[这里](https://github.com/reactjs/react-redux/blob/master/docs/api.md)。

下面是关于connect()方法的介绍以及实例。

{% codeblock lang:javascript %}
// api: connect
connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])
{% endcodeblock %}

#### mapStateToProps(state, [ownProps]):stateProps {Function}

1. 如果该函数指定了，那么生成的Container组件将订阅redux store的更新，每当store更新，mapStateToProps就会执行。
2. 如果[ownProps]被指定，那么新生成的Container组件接收到新的props时，也会调用mapStateToProps。
3. 第一个参数state为store刚更新的state，[ownProps]为新生成的Container组件的props对象，可选。
4. 该函数返回一个普通js对象stateProps，该对象将被merge到新生成Container组件的props对象中。
5. 如果不想让该组件subscribe redux的store更新，那么可以将该参数置为`null`或者`undefined`。

#### matchDispatchToProps(dispatch, [ownProps]):dispatchProps {Function|Object}

先说该参数被指定为一个对象的情况：

1. 如果该参数被指定为一个对象，那么这个对象中的每个方法都被当作redux actionCreator。
2. 最终返回一个js对象，其中每个函数都与actionCreator同名，但是函数体中都是`dispatch(actionCreator)`语句。
3. 返回的对象会merge到新生成的Container组件中的props对象中。

返回的js对象可能是这个样子的：

{% codeblock lang:javascript %}
{
    addTodo: function (text) {
        dispatch(addTodo(text));
    },
    toggleTodo: function (id) {
        dispatch(toggleTodo(id));
    }
}
{% endcodeblock %}

所以我们在新生成的Container组件中，可以这样调用：`props.addTodo('write blog')`。

再说该参数被指定为函数的情况（参考该小节标题）：

1. 如果该参数被指定为函数，那么它将接收两个参数`dispatch`和`[ownProps]`。
2. [ownProps]和mapStateToProps中的是一样的，都是新生成Container组件的props对象。
3. 返回一个对象，该对象可能包含多个function，并且可以在function中dispatch actions。
4. 返回的对象会被merge到新生成Container组件的props中。
5. 如果该参数（指[mapDispatchToProps]）被省略，那么只会注入`dispatch`方法到新生成的组件中。

#### mergeProps(stateProps, dispatchProps, ownProps): props {Function}

1. 如果该参数被指定了，参数就是mapStateToProps()、mapDispatchToProps()的执行结果以及ownProps对象。
2. 该函数返回一个对象，将被merge到新生成Container组件的props中。
3. 如果不指定，默认执行`Obejct.assign({}, ownProps, stateProps, dispatchProps)`。

#### options {Object}

如果指定该参数，可以对该`connector`的行为进行定制。

1. pure {Boolean} 默认为true，表示connect()将使用自己维护的相等性检验函数，来对新的state/props进行检验，如果检验通过，将不会重复调用mapStateToProps、mapDispatchToProps、mergeProps等函数，也不会重复渲染组件。为了更好的性能，一般都置为ture。
2. areStateEqual {Function} 在pure为true时，用来检验新旧state是否相等，默认strictEqual(===)。
3. areOwnPropsEqual {Function} 在pure为true时，比较新旧props是否相等，默认shallowEqual。
4. areStatePropsEqual {Function}  在pure为true时，计算`mapStateToProps()`的结果和之前是否一样，默认shallowEqual。
5. areMergedPropsEqual {Function} 在pure为true时，计算`mergePros()`的结果和上一次是否一样，默认shallowEqual。

至于具体什么时候该覆写这些方法，去看官方文档吧！

#### 总结

mapStateToProps为组件提供了监听store中state的能力。

mapDispatchToProps为组件提供了dispatch的能力，可以改变state中的数据。

#### 使用connect()的案例

{% codeblock lang:javascript %}
// 只注入dispatch到AddTodo组件的props，不监听store中state的变化
export default connect()(AddTodo);

// 注入所有actionCreators到AddTodo组件的props，不监听store中state的变化
// 在AddTodo组件中可以这样调用，props.addTodo('listen music')或者props.toggleTodo('watch moive')
import * as actionCreators from './actionCreators';
export default connect(null, actionCreators)(AddTodo);

// 注入dispatch和一个state分片todos，监听store中state变化
function mapStateToProps (state) {
    return {
        todos: state.todos
    }
}
export default connect(mapStateToProps)(TodoApp);

// 使用redux的bindActionCreators(actionCreators, dispatch)方法来打包注入actionCreators
// 同时监听state的变化
// 可以在AddTodo组件中这样调用：props.actions.addTodo('sleep')或者props.actions.toggleTodo('eat')
import * as actionCreators from './actionCreators';
import {bindActionCreators} from 'redux';
function mapStateToProps(state) {
    return {
        todos: state.todos
    }
}
function mapDispatchToProps (dispatch) {
    return {
        actions: bindActionCreators(actionsCreators, dispatch)
    }
}
export default connect(mapStateToProps, mapDispatchToProps)(AddTodo);
{% endcodeblock %}

总之这一块的用法还是非常灵活的，请看官方文档提供的[案例](https://github.com/reactjs/react-redux/blob/master/docs/api.md#examples)。

### 传递store到所有对象

所有的Container组件都需要store的访问权限，因为他们需要订阅它，并且还要dispatch来更新store中的state。

所以，为了更方便的访问store，`react-redux`为我们提供了一个组件`<Provider>`，可以神奇地将store传递到所有组件，这使得我们不比显式得通过porps来传递store。

我们只需要在渲染根组件的时候来使用`<Provider>`组件：

{% codeblock lang:javascript %}
import {Provider} from 'react-redux';
RectDom.render(
    <Provider store={store}>
        <App/>
    </Provider>,
    document.querySelector('#app')
);
{% endcodeblock %}

## 最后

以上就是`redux`和`react-redux`为我们在构建应用时提供的一些思路和方法，后面还需要在实践中进一步体验和学习。
