---
title: js中精度问题以及解决方案
tags: [javascript]
categories: javascript
toc: true
date: 2018-03-17 10:08:03
---

js中的数字按照IEEE 754的标准，使用64位双精度浮点型来表示。其中符号位S，指数位E，尾数位M分别占了1，11，52位，并且在[ES5规范](https://es5.github.io/#x8.5)中指出了指数位E的取值范围是`[-1074, 971]`。

## 精度问题汇总

想用有限的位来表示无穷的数字，显然是不可能的，因此会出现一些列精度问题：

- 浮点数精度问题，比如`0.1 + 0.2 !== 0.3`
- 大数精度问题，比如`9999 9999 9999 9999 == 1000 0000 0000 0000 1`
- toFixed四舍五入结果不准确，比如`1.335.toFixed(2) == 1.33`

浮点数精度和toFixed其实属于同一类问题，都是由于浮点数无法精确表示引起的，如下：

{% codeblock lang:javascript %}
(1.335).toPrecision(20);    // "1.3349999999999999645"
{% endcodeblock %}

而关于大数精度问题，我们可以先看下面这个代码片段：

{% codeblock lang:javascript %}
// 能精确表示的整数范围上限,S为1个0，E为11个0，S为53个1
Math.pow(2, 53) - 1 === Number.MAX_SAFE_INTEGER    // true

// 能精确表示的整数范围下限,S为1个1，E为11个0，S为53个1
-(Math.pow(2, 53) - 1) === Number.MIN_SAFE_INTEGER    // true

// 能表示的最大数字，S为1个0，E为971，S为53个1
(Math.pow(2, 53) - 1) * Math.pow(2, 971) === Number.MAX_VALUE    // true

// 能表示的最接近于0的正数，S为1个0，E为-1074，S为0
Math.pow(2, -1074) === Number.MIN_VALUE // true
{% endcodeblock %}

通过以上可以明白，[MIN_SAFE_INTEGER, MAX_SAFE_INTEGER]的整数都可以精确表示，但是超出这个范围的整数就不一定能精确表示。这样就会产生所谓的大数精度丢失问题。

## 解决思路

首先考虑的是如何解决浮点数运算的精度问题，有3种思路：

1. 考虑到每次浮点数运算的偏差非常小(其实不然)，可以对结果进行指定精度的四舍五入，比如可以`parseFloat(result.toFixed(12))`;
2. 将浮点数转为整数运算，再对结果做除法。比如0.1 + 0.2，可以转化为`(1*2)/3`。
3. 把浮点数转化为字符串，模拟实际运算的过程。

先来看第一种方案，在大多数情况下，它可以得到正确结果，但是对一些极端情况，toFixed到12是不够的，比如：

{% codeblock lang:javascript %}
210000 * 10000  * 1000 * 8.2    // 17219999999999.998
parseFloat(17219999999999.998.toFixed(12));    // 17219999999999.998，而正确结果为17220000000000
{% endcodeblock %}

上面的情况，如果想让结果正确，需要`toFixed(2)`，这显然是不可接受的。

再看第二种方案，比如[number-precision](https://github.com/nefe/number-precision)这个库就是使用的这种方案，但是这也是有问题的，比如：

{% codeblock lang:javascript %}
// 这两个浮点数，转化为整数之后，相乘的结果已经超过了MAX_SAFE_INTEGER
123456.789 * 123456.789     // 转化为(123456789 * 123456789)/1000000，结果是15241578750.19052
{% endcodeblock %}

所以，最终考虑使用第三种方案，目前已经有了很多较为成熟的库，比如[bignumber.js](https://github.com/MikeMcl/bignumber.js)，[decimal.js](https://github.com/MikeMcl/decimal.js)，以及[big.js](https://github.com/MikeMcl/big.js)等。我们可以根据自己的需求来选择对应的工具。并且，这些库不仅解决了浮点数的运算精度问题，还支持了大数运算，并且修复了原生toFixed结果不准确的问题。

## 题外话

还有另外一个与js计算相关的问题，即Math.round(x)，它虽然不会产生精度问题，但是它有一点小陷阱容易忽略。下面是它的舍入的策略：

- 如果小数部分大于0.5，则舍入到下一个绝对值更大的整数。
- 如果小数部分小于0.5，则舍入到笑一个绝对值更小的整数。
- 如果小数部分等于0.5，则**舍入到下一个正无穷方向上的整数**。

所以，对Math.round(-1.5)，其结果为-1，这可能不是我们想要的结果。

当然，上面提到的big.js等库，都提供了自己的round函数，并且可以指定舍入规则，以避免这个问题。

