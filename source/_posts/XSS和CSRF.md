title: 理解XSS和CSRF原理
tags: [XSS, CSRF]
categories: 前端
toc: true
date: 2017-10-26 10:08:03
---

XSS和CSRF与网络安全息息相关，这篇文章就来浅析一下这两种攻击方式的原理，以及如何防范这两种攻击。

## XSS

XSS，跨站脚本攻击，即`Cross Site Scripting`，为了和CSS不重名，所以取名为XSS。

### 原理

XSS攻击者通过畸形的输入，将恶意的js代码插入到了页面中。当其他用户浏览该网页时，恶意的js代码会被执行，从而达到攻击的目的。

XSS攻击发生的根本原因是对用户输入的信任。

### 案例

XSS类型分为3种，反射型XSS，存储型XSS，DOM-XSS。

#### 反射型XSS

反射型XSS，即将用户的输入直接传到后台（不保存)，后台并未经过任何过滤，直接将输入数据返回给前台。

比如一个页面，其内容就是根据url中查询参数param的值来生成的。那么我们可以请求如下的url：

{% codeblock lang:javascript %}
http://example.com/scene/?param=<script>alert('这是一个XSS攻击')</script>
{% endcodeblock %}

这样当我们将这个url放出去供其他用户访问时，就会出现alert。因为此时页面内容如下：

{% codeblock lang:html %}
<div><script>alert('这是一个XSS攻击')</script></div>
{% endcodeblock %}

#### 存储型XSS

存储型XSS的攻击过程大致如下：

M用户的畸形输入 -> 后台服务器未经任何处理就存储 -> 其他用户T请求数据 -> M用户的畸形输入返回给T -> 用户T打开页面，XSS代码运行

比如在某个评论区提交了XSS代码，并且这些评论都被存储到服务器中。那么所有打开评论列表页面的用户都会收到XSS代码的攻击。

#### DOM-XSS

DOM-XSS即纯粹发生在客户端的XSS攻击。

比如某个网页中有个输入框，在输入完成之后，会创建一个a标签，并且将输入值作为a标签的href属性。代码大致如下：

html:
{% codeblock lang:html %}
<div class="link"></div>
<input type="text" class="user-input">
<Button class="submit">提交</Button>
{% endcodeblock %}

js:
{% codeblock lang:javascript %}
var $ = (selector) => document.querySelector(selector);
$('.submit').addEventListener('click', function () {
    $('.link').innerHTML = "<a href='" + $('.user-input').value + "'>点我呀</a>"
});
{% endcodeblock %}

这样一来，当我输入`' onclick=alert('哈哈哈') '`，其实就完成了一次简单的XSS攻击（点击a标签就是执行js）。

审查生成的a标签，内容如下：

{% codeblock lang:html %}
<a href="" onclick="alert(123)" ''="">点我呀</a>
{% endcodeblock %}

### 如何防范XSS?

XSS攻击的原因，就在于没有对用户的输入进行检查、过滤，转义等。因此我们要对用户的输入（url参数/post数据等）当做不可信数据来处理。

这里推荐一个[第三方的工具](http://jsxss.com/zh/index.html)来防范XSS，它具有以下特性：

- 白名单控制允许的HTML标签以及各标签的属性
- 通过自定义处理函数，可以对任意标签及其属性进行处理。

## CSRF

CSRF，跨站请求伪造，即`Cross Site Request Forgery`。

### 原理

CSRF可以在受害者毫不知情的情况下以受害者名义发送请求。其原理是**借助受害者的cookie来骗取服务器的信任**。

比如，受害者T登录了某个银行网站，sessionId等信息就会保存在浏览器中。这时候，用户T又打开了CSRF攻击网站，这个网站通过表单自动提交来向银行网站发起一次转账请求，这样，受害者T的银行账户的钱可能就被转走了。

这个过程成功的关键点在于：**用户登录了银行网站，并且session还未过期的时候，打开了CSRF攻击网站**。

### 案例

这个案例构造了一个CSRF攻击网页，当某个登录了`http://www.example.com`的用户点开这个网页时，就会自动在`www.example.com`中发表一个评论，而用户对此毫不知情。

html:
{% codeblock lang:html %}
<!--注意：ajax受浏览器同源策略的影响，但是表单提交是不受这个限制的。-->
<form id="attackForm" action="http://www.example.com/article/59a12398904c4928cc46603c/comment/new" method="post">
    <input type="text" name="content" value="vim是世界上最好的编辑器"/>
</form>
{% endcodeblock %}

js:
{% codeblock lang:javascript %}
// 自动提交表单，这时候，浏览器会带着受害用户在example.com下的cookie去发请求，这样就通过了服务器的认证
window.onload = function () {
   (function () {
       document.querySelector('#attackForm').submit();
   })();
};
{% endcodeblock %}

### 如何防范CSRF?

下面列举了两种防御CSRF攻击的措施：

### 检查HTTP Referer字段

在HTTP的请求头中，有一个Referer字段，用于表示请求来源于哪个地址。上面提到的银行网站，可以在请求到来时，检查Referer字段，如果请求来源地址不在白名单内，就不予回应。

这种方法简单易行，但是也有一定的局限性，比如完全依赖于浏览器发送正确的Referer字段，况且Referer也是很可能被攻击者篡改的。

### 添加校验TOKEN

其原理是服务器在响应请求时，生成一个csrf-token传递到前台，当前台发起请求时，需要带着这个csrf-token以便于服务器端进行校验。这样一来，由于csrf攻击网站无法获取到这个token，所以也无法通过服务器端的校验。

当前台发起请求时，csrf-token可以通过url参数，或者post的请求体来携带，但是最安全的方式，还是`将csrf-token添加到自定义HTTP请求头中（比如X-CSRF-TOKEN）`。

### CSURF

express团队为我们提供了[CSURF](https://github.com/expressjs/csurf)这个中间件，可以帮助我们来轻松实现CSRF的防御。
