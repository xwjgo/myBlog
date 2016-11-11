title: 正则表达式的基本使用
tags: 正则表达式
categories: 乱七八糟
toc: true
date: 2016-9-19 12:12:12
---

最近状况有些糟糕，健康陪伴了我两年的电脑，硬盘彻底报废，大学里积攒的所有数据全部丢失（怪我没有备份），主板也烧了好几处，导致现在发不出声音。另外，由于为了修电脑，跑动跑西，我竟然错过了携程和滴滴出行的两场校招笔试！唉，祸不单行啊，希望情况能快点好起来，校招季不好熬-\_-

## 什么是正则表达式

开头先来推荐两个正则表达式可视化工具： [REGEXPER](https://regexper.com/)和[RegExr](http://regexr.com/)。

再来推荐一个适合用来查询的文档：[MSDN正则参考手册](https://msdn.microsoft.com/zh-cn/library/ae5bf541(v=vs.100).aspx)。

正则表达式（**Regular Expression**）是我们处理字符串的利器，它主要用来**描述匹配字符串的规则**，主要来解决字符串的**查找**和**替换**两大需求。这里就结合js来总结一下正则表达式的用法。

## 基本使用

正则表达式包括**普通字符**（比如a到z之间的字母）和**元字符**。

而元字符又包含了单字符元字符和多字符元字符。

### 单字符元字符

以下总结了单字符元字符及其行为。

{% codeblock lang:javascript %}
*     // 零次或者多次匹配前面的字符，等效于{0,}
+     // 一次或者多次匹配前面的字符，等效于{1,}
?     // 零次或者一次匹配前面的字符，等效于{0,1}
      // 当?紧随其他限定符（*，+，？，{n,m}）之后时，表示非贪婪匹配
^     // 匹配搜索字符串开始的位置
      // 如果将^用作括号表达式中的第一个字符，则会对字符集求反
$     // 匹配搜索字符串结尾的位置
.     // 匹配除换行符\n之外的任何单个字符。如果要匹配所有字符，请用[\s\S]
[]    // 匹配括号范围内的某个元素
{}    // 量词，表示前一个字符有多少个
()    // 分组
|     // 或，在两个或者多个项(不同于字符)之间选择，比如/(?:love|hate) you/
\     // 将下一字符标记为特殊字符
{% endcodeblock %}

### 多字符元字符

以下列出了多字符元字符以及他们的行为：

{% codeblock lang:javascript %}
\b    // 字符边界
\B    // 非字符边界
\d    // 数字字符，等同于[0-9]
\D    // 非数字字符，等同与[^0-9]
\w    // 等同与[A-Za-z0-9_]
\W    // 等同与[_A-Za-z0-9_]
(模式)    // 与模式匹配，并且保存匹配项（当作分组，可反向引用）
(?:模式)  // 与模式匹配，但不保存匹配项（不当作分组，不可以反向引用）
(?=断言)  // 正向前瞻
(?!断言)  // 负向前瞻，对后面要匹配的内容做与判断
\num      // 此处num是一个正整数，是对以保存的匹配项的引用
          // 比如(\d)\2是对连续两个相同数字的匹配
{% endcodeblock %}

### 非打印字符

此外还有一些非打印字符：

{% codeblock lang:javascript %}
\n        // 换行符
\r        // 回车符
\s        // 任何空白字符，包括空格、制表符
\S        // 任何非空白字符
\t        // tab字符
{% endcodeblock %}

### 优先级

优先级从高到低依次如下：

{% codeblock lang:javascript %}
\                           // 转义符
(), (?:), (?=), (?!), []    // 括号和中括号
*, +, ?, {n,m}              // 限定符
^, $, /                     // 任何元字符
|                           // 或（优先级最低）
{% endcodeblock %}

## 正向前瞻与负向前瞻

前瞻分为正向前瞻和负向前瞻。

正向前瞻：比如`/(?=XXX).*/`，它的作用是对后面要匹配的内容做一个预判断，如果后面的内容是XXX，则此段内容匹配成功，继续向后匹配。

负向前瞻：比如`/(?!xxx).*/`，它的作用是**对后面要匹配的内容做一个预判断，如果后面的内容是XXX，则此段内容匹配失败**，跳过去重新开始匹配。

我们来看一个实例：

{% codeblock lang:javascript %}
// 正向前瞻，语法为 expression(?=assert)
// 放在expression之后，若匹配assert（断言）成功，继续向前匹配expression
// assert匹配字符串并不会出现在结果中

var reg = /\w+(?=ing)/g;
var str = 'cooking eating';

str.match(reg);              // ["cook", "eat"]

// 负向前瞻，语法为 expression(?!assert)
// 放在expression之后，若匹配assert（断言）成功，停止向前匹配expression
// assert匹配字符串并不会出现在结果中

var reg = /\d(?!px)/g;
var myStr = '1pt 2px 3em 4px';

str.match(reg);             // ["1", "3"]
{% endcodeblock %}

## 贪婪与懒惰

正则表达式**默认为贪婪模式**，即尽可能多的匹配。

如果想使用非贪婪模式，只需要**在量词（\*，+，？，{n,m}）之后加上`?`即可**。

{% codeblock lang:javascript %}
// 贪婪模式，默认
var reg = /\d{3,6}/g;
var myStr = '123456789';

str.match(reg);             // ["123456"]

// 非贪婪模式
var reg = /\d{3,6}?/g;
var myStr = '123456789';

str.match(reg);             // ["123", "456", "789"]
{% endcodeblock %}

## 反向引用

反向引用，即**引用捕获组匹配到的引用**，它**既可以在正则表达式内部进行引用，也可以在正则表达式外部进行引用**。

捕获组包括两种：**普通捕获组**和**命名捕获组**。

### 普通捕获组

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E6%99%AE%E9%80%9A%E6%8D%95%E8%8E%B7%E7%BB%84.png)

再看一个案例：

{% codeblock lang:javascript %}
// '2016-09-20'   --->  '2016/09/20'
// 外部引用
var reg = /(\d{4})-(\d{2})-(\d{2})/;
var str = '2016-09-20';
var newStr = str.replace(reg, '$1/$2/$3')

// 匹配连续4个数字
// 内部引用
var reg = /(\d)\d\1\1\1/g;
var str = '12111 234 65666';
str.match(reg);      // ["12111", "65666"]
{% endcodeblock %}

### 命名捕获组

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E5%91%BD%E5%90%8D%E6%8D%95%E8%8E%B7%E7%BB%84.png)

但是，**js目前还不支持命名捕获组**。

## js中使用正则表达式

js中使用[RegExp](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp/exec)和[String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/replace)两个对象的某些方法都可以使用正则表达式。

### RegExp的test()

test方法很简单，看正则表达式与指定字符串是否匹配。返回true或者false。

{% codeblock lang:javascript %}
regexObj.test(str)
{% endcodeblock %}

### RegExp的exec()

{% codeblock lang:javascript %}
// 如果成功匹配，返回一个数组，并且更新正则表达式对象的属性。
// 返回的数组包括匹配的字符串作为第一个元素，紧接着是捕获组中的字符串。
// 如果匹配失败，返回null
regexObj.exec(str)
{% endcodeblock %}

来看一个实例：

{% codeblock lang:javascript %}
var reg = /(\w+)ing\s(\w+)/g;
var str = 'eating apple eating orange';

var ret = reg.exec(str);         // ["eating apple", "eat", "apple"]
var lastIndex = reg.lastIndex;   // 12

var ret2 = reg.exec(str);        // ["eating orange", "eat", "orange"]
var lastIndex2 = reg.lastIndex;  // 26

var ret3 = reg.exec(str);        // null
var lastIndex3 = reg.lastIndex;  // 0

// 输出结果的属性
var index1 = ret.index;      // 0
var index2 = ret2.index;     // 13
var input = ret.input;       // "eating apple eating orange"

// reg的属性值
var ignorCase = reg.ignorCase;     // 是否使用了i
var global = reg.global;           // 是否使用了g
var multiline = reg.multiline;     // 是否使用了m
var source = reg.source;           // "(\w+)ing\s(\w+)"
{% endcodeblock %}

### String的方法

String类型也提供了一些可以使用正则表达式的方法，具体如下：

{% codeblock lang:javascript %}
// var reg = /\d{3}/g;
var str = 'xu0wen123jiang456789';

// String.prototype.search()
// 返回第一个匹配项的索引，找不到返回-1
var index = str.search(reg);      // 6

// String.prototype.match()
// 非全局匹配：返回第一个匹配项和分组匹配项组成的数组
// 全局匹配：返回所有匹配项组成的数组
var rets = str.match(reg);        // ["123", "456", "789"]

// String.prototype.split()
var arrs = str.split(reg);        // ["xu0wen", "jiang", "", ""]

// String.prototype.replace()
// 其第二个参数除了可以是字符串或者正则之外，还可以是一个函数，具体见最后案例
var newStr = str.replace(reg, '0');    // "xu0wen0jiang00"
{% endcodeblock %}

## 一个案例

{% codeblock lang:javascript %}
// 'a1b2c3d4'   --->  'a2b3c4d5'

var reg = /\d/g;
var str = 'a1b2c3d4';

// 对每一个匹配项执行回调，并将返回值作为替换
var newStr = str.replace(reg, (match, index, origin) => {
  return parseInt(match) + 1;
});                                   // "a2b3c4d5"
{% endcodeblock %}




