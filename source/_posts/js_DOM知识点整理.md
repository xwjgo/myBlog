title: DOM知识点整理
tags: javascript
categories: 前端
toc: true
date: 2016-8-11 12:12:12
---
这里总结了常用的DOM属性和方法。

## 节点类型

常用的节点类型有3种：**元素节点，属性节点和文本节点**。

---
| 节点类型 | nodeName | nodeType | nodeValue            |
| ---      | ---      | ---      |
| **元素**     | 元素名称 | 1        | null                 |
| **属性**     | 属性名称 | 2        | 属性值               |
| **文本**     | #text    | 3        | 文本内容（不含html） |
---

## 属性

### 节点信息

{% codeblock lang:javascript %}
nodeType             // 节点类型，为数字
nodeName             // 取决有节点类型，若节点为元素，则为标签名
nodeValue            // 取决于节点类型，若节点为文本，则为文本内容
{% endcodeblock %}

### 节点关系

{% codeblock lang:javascript %}
childNodes           // 返回一个nodeList对象，具有item(pos)方法和length属性，也可用中括号
parentNode           // 对父节点的引用

nextSibling          // 下一个同胞节点
previousSibling      // 上一个同胞节点

firstChild           // 第一个子节点
lastChild            // 最后一个子节点

// 以下专门针对元素节点

children                   // 所有子元素节点
childElementCount          // 所有子元素节点的数量

firstElementChild          // 第一个子元素节点
lastElementChild           // 最后一个子元素节点

nextElementSibling         // 下一个同胞元素节点
previousElementSibling     // 前一个同胞元素节点
{% endcodeblock %}

### 节点内容

{% codeblock lang:javascript %}
innerHTML                 // 返回所有子节点的HTML，不包含调用节点
outerHTML                 // 返回调用节点以及子节点的HTML

innerText                 // 将文档树中所有文本拼接起来并返回，不包含调用节点的文本内容
outerText                 // 范围扩大到包含调用它的节点，其他与innerText一致
{% endcodeblock %}

### 节点属性

{% codeblock lang:javascript %}
attributes     // 返回一个NamdNodeMap，与Nodelist相似，有item(pos)等，不常用，但可以遍历元素特性
classList      // 操作节点的class属性值，返回一个DOMTokenList对象，它具有以下方法

ul.classList.remove('one');       // 移除class属性值'one'
ul.classList.add('four');         // 增加class属性值'four'
ul.classList.contains('four');    // true
ul.classList.toggle('one');       // 若果class属性值包括'one'，则删；反之，则加
{% endcodeblock %}

### 元素大小

用一张图来说明：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E5%85%83%E7%B4%A0%E5%A4%A7%E5%B0%8F.png)

## 方法

### 增加节点

{% codeblock lang:javascript %}
document.createElement(tagName)                  // 创建新元素节点
document.createTextNode(text)                    // 创建新文本节点

parentNode.appendChild(newNode)                  // 在childNodes列表结尾添加新节点
parentNode.insertBefore(newNode, referNode)      // 在参考节点前插入新节点

node.cloneNode(Boolean)             // 克隆节点，如果参数为true，则执行深复制，即克隆整个子节点树

insertAdjacentHTML()                // 用法如下，注意第一个参数必须全部小写

// 作为前一个同辈元素插入
element.insertAdjacentHTML('beforebegin', '<p> hello xwj... </p>')
// 作为第一个子元素插入
element.insertAdjacentHTML('afterbegin', '<p>hello xwj... </p>'
// 作为最后一个子元素插入
element.insertAdjacentHTML('beforeend', '<p>hello xwj... </p>'
// 作为后一个同辈元素插入
element.insertAdjacentHTML('afterend', '<p>hello xwj... </p>'
{% endcodeblock %}

### 删除节点

{% codeblock lang:javascript %}
parentNode.removeChild(deleteNode)     // 移除目标节点
{% endcodeblock %}

### 替换节点

{% codeblock lang:javascript %}
parentNode.replaceChild(newNode, deleteNode)    // 使用新节点替换旧节点
{% endcodeblock %}

### 查获节点

{% codeblock lang:javascript %}
document.getElementById(id)
document.getElementsByTagName(tagName)       // 根据标签名来获取节点
document.getElementsByName(name)             // 根据节点的name属性来获取节点
document.getElementsByClassName(className)   // 根据calss属性值来获取节点

document.querySelector()            // 传入CSS选择符，返回第一个与该选择符匹配的节点
document.querySelectorAll()         // 传入CSS选择符，返回所有匹配元素
{% endcodeblock %}

### 操作特性

{% codeblock lang:javascript %}
element.getAttribute('id')              // 获取特性
element.id                              // 获取特性，一般采用这种方式，但是自定义属性不行

element.setAtrribute('class', 'one')    // 设置特性
element.className = 'color'             // 设置特性,不可以设置自定义属性

element.removeAttribute('class')               // 移除特性
{% endcodeblock %}



## 其他

{% codeblock lang:javascript %}
element.hasChildNodes()                // 若该节点有一个或者多个子节点，返回true

document.hasFocus()                    // 节点可以调用focus()来获取焦点，这个方法可以检测文档是否获取焦点

document.activeElement                 // 始终指向DOM中获得焦点的元素
{% endcodeblock %}
















