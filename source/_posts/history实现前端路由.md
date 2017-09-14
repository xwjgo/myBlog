title: history实现一个前端路由
tags: [history]
categories: javascript
toc: true
date: 2017-08-29 10:08:03
---

history也是HTML5新增的一个api，这里简单总结一下它的基本用法，最后探究一下使用history来实现前端路由的整体思路。

## 在历史记录之间穿梭

history提供了几个非常方便的方法，帮助我们在浏览器的历史记录之间方便地进行切换，这相当于我们手动点击浏览器工具栏中的前进或者后退按钮。

{% codeblock lang:javascript %}
window.history.back();       // 后退一页
window.history.forward();    // 前进一页
window.history.go(-2);       // 后退两页，当前页索引为0
{% endcodeblock %}

## pushState()与replaceState()

如果我们再浏览器控制台中直接输入history，就可以打印出history对象，可以看到它的length属性和state属性。在它的prototype中除了上面提到了go()、back()、forward()方法以外，它还有两外两个常用的方法：`pushState()`和`replaceState()`。这两个方法分别用来创建新的history实体（可以理解为新的history状态）和改变当前history实体。

pushState()接收3个参数，如下：

1. stateObj，一个普通的js对象，可以包含任何数据，它包含在新的hsitory实体的state属性中。当用户导航到该状态时，一个`popstate`事件将会被触发，并且在该事件的`e.state`中会包含这个stateObj。
2. title，目前没什么用，我们可以传入一个空字符串，或者一个表示新history状态的字符串。
3. URL，可选参数，这将会改变浏览器当前的url，但是新的url并不会立刻加载。当时用户此时跳转到另外一个页面，然后再点击后退，该url会加载，并且触发popstate事件。

看个案例，比如我们现在进入了`www.example.com`：

{% codeblock lang:javascript %}
// 浏览器地址会变为http://www.example.com/profile，但并不会加载。同时history的length会加1，state会变为传入的stateObj
history.pushState({name: 'xwj', age: 22}, '', '/profile');
// 此时如果从当前页跳转到www.baidu.com，然后点击返回，那么http://www.example.com/profile会被加载。
// 同时触发popstate事件，刚才的{name: 'xwj', age: 22}会存在与e.state中。
{% endcodeblock %}

replaceState()和pushState()用法基本一样，不过它只是更改当前history实体，并不会创建新的history实体，通常用来改变当前history的state和url，而不刷新页面。

此外，pushState一定程度上类似于`window.location='#foo'`，因为他们都创建并激活了一个history实体，但是pushState()的优点更多一点，如下：

1. 新的url可以是任何url，而后者只能是当前document。
2. 可以在新的history实体通过state属性挂载任何对象，而hash的方式则不可以。
3. 可以单纯地添加一个history实体而不改变url（将url属性置空），而hash必定会造成url的变化。

## popstate事件

只有触发一个浏览器行为，比如前进或者后退，popstate事件才会被触发。这里需要注意，当我们调用pushState()或者replaceState()时并不会触发popstate事件。下面是一个简单的案例：

{% codeblock lang:javascript %}
window.onpopstate = function (e) {
  console.log(e.state);  // 这里的state即我们在调用pushState()或者replaceState()时传入的state
};

history.pushState({page: 1}, '', '?page=1');
history.pushState({page: 2}, '', '?page=2');
history.replaceState({page: 3}, '', '?page=3');
history.back();  // state:{page: 1}
history.back(); // state:null
history.go(2); // state:{page: 3}
{% endcodeblock %}

## 实现一个简单的路由系统

有了以上基础，我们就可以思考如何使用hitory来构建一套简单的前端路由系统了。当然，要想实现类似`http://www.example.com/profile`这样好看的前端路由，我们还是需要后台来配合的，否则当用户在浏览器直接访问这个url时，就会返回404。所以，后台要做的处理就是**当匹配不到对应的资源时，就返回index.html，而这个页面就是我们app依赖的界面**，类似下面这样：

{% codeblock lang:javascript %}
app.get('/', (req, res) => {
    res.sendFile(path.join(__dirname, 'index.html'));
});
// 如果匹配不到任何界面，则返回index.html
app.use((req, res) => {
    res.sendFile(path.join(__dirname, 'index.html'));
});
{% endcodeblock %}

后台启动成功之后，我们再来实现前端路由。前端路由的本质，就是url到DOM结构的映射，即路由到某个url时，执行对应的方法来改变DOM。

下面是一个简单的实现思路：

1. 用户提供一个routeMap，是url到改变DOM方法的映射。
2. 初始化页面，使用replaceState来改变浏览器显示的url（无刷新），并存储当前path到state，同时调用`routeMap[location.pathname]()`来改变DOM。
3. 用户进入新的页面，使用pushState来改变浏览器URL，存储当前path到state，同时调用对应的改变DOM的方法。
4. 通过监听window的`popstate`事件来响应浏览器的回退和前进，在事件处理方法中，通过`e.state.path`来得到当前path，并执行`routeMap[path]()`来改变DOM。

实现和核心代码如下：

{% codeblock lang:javascript %}
const $ = (selector) => document.querySelector(selector);
class Route {
    constructor (routeMap) {
        this.routeMap = routeMap;
        this._bindPopState();
    }

    init (path) {
        path = Route.correctPath(path);
        history.replaceState({path: path}, '', path);
        this.routeMap[path] && this.routeMap[path]();
    }

    go (path) {
        path = Route.correctPath(path);
        history.pushState({path: path}, '', path);
        this.routeMap[path] && this.routeMap[path]();
    }

    _bindPopState () {
        window.addEventListener('popstate', (e) => {
           const path = e.state && e.state.path;
           this.routeMap[path] && this.routeMap[path]();
        });
    }

    static correctPath (path) {
        if (path !== '/' && path.slice(-1) === '/') {
            path = path.match(/(.+)\/$/)[1];
        }
        return path;
    }
}

const routeMap = {
    '/': () => {
        const content = $('.content');
        content.innerHTML = '<div>welcome to Home Page</div>';
    },
    '/profile': () => {
        const content = $('.content');
        content.innerHTML = '<div>welcome to Profile Page</div>';
    },
    '/articles': () => {
        const content = $('.content');
        content.innerHTML =
            '<div>' +
            '<p>welcome to Article Page</p>' +
            '<ul>' +
            '<li>文章1</li>' +
            '<li>文章2</li>' +
            '<li>文章3</li>' +
            '</ul>' +
            '</div>';
    }
};
const router = new Route(routeMap);
router.init(location.pathname);
$('.menu').addEventListener('click', (e) => {
    if (e.target.tagName === 'A') {
        e.preventDefault();
        router.go(e.target.getAttribute('href'))
    }
});
{% endcodeblock %}

完整的代码请访问[这里](https://github.com/xwjgo/history-learning)。

## 参考

- [https://developer.mozilla.org/en-US/docs/Web/API/History_API](https://developer.mozilla.org/en-US/docs/Web/API/History_API)



