title: jison学习笔记
tags: [jison]
categories: 乱七八糟
toc: true
date: 2018-03-03 10:08:03
---

最近在工作中遇到一个由js精度引发的bug：比如用户自定义了一个表单提交的校验条件`SUM(a, b) <= 0.3`，此时，当`a=0.1`，`b=0.2`的时候，`SUM(a, b)`的值为0.30000000000000004，这导致校验条件得不到满足，表单也无法提交。

## 原因分析

这个根本原因是js中的数字全部是采用IEEE 754标准的`64双精度浮点数`。改规范定义了浮点数在内存中的表示格式：

- 符号位S：第0位，0表示正数，1表示负数。
- 指数位E：第1到11位，共11位，存储指数部分。
- 尾数位M：第12位到63位，52位，存储小数部分。

举个栗子，比如-8.25这个浮点数转化为二进制为`-1000.01`，然后转化为二进制的科学计数法为`-1.00001*2^3`，总之，任何数字都可以表示为`(-1)^S*1.EEE*2^MMM`的形式。这样，对于`-1.00001*2^3`，我们只需要保存3个部分的值，即符号（正负），指数，尾数（即1.XXX后面的XXX，因为前面的1是固定的），这三部分正好分别存储在IEEE 754标准所定义的符号位S，指数位E，尾数位M中。

由此可见，任何一个数字转化成二进制之后，其有效数字最长为53个二进制位（52位尾数加上前面的1）。这样某个数字的二进制表示如果比53位长，那么多余部分将无法表示。比如0.1和0.2这样的数字：

{% codeblock lang:javascript %}
// 0.1和0.2转化成二进制后，尾数部分是无限循环的，所以超出部分将会截去
(0.1).toString(2);    // "0.0001100110011001100110011001100110011001100110011001101"
(0.2).toString(2);    // "0.001100110011001100110011001100110011001100110011001101"
{% endcodeblock %}

这样0.1+0.2的结果就是`0.0100110011001100110011001100110011001100110011001100`，转化成10进制就是`0.30000000000000004`。

其实不只是加法，四则元算都会有这个问题:

{% codeblock lang:javascript %}
// 加法
0.1 + 0.2 == 0.30000000000000004
0.7 + 0.1 == 0.7999999999999999
// 减法
1.5 - 1.2 == 0.30000000000000004
0.3 - 0.2 == 0.09999999999999998
// 乘法
19.9 * 100 == 1989.9999999999998
39.7 * 100 == 3970.0000000000005
// 除法
0.3 / 0.1 == 2.9999999999999996
0.69 / 10 == 0.06899999999999999
{% endcodeblock %}

综上，我们可以把这种js精度的问题归结为：js采用的IEEE 754标准无法准确表示某些浮点数，类似十进制中我们无法精确表示1/3，二进制中我们也无法精确表示0.1。

## 问题解决

发现了问题的原因，我们就可以对症下药了。其实实际的计算结果与正确的计算结果的差距非常小，所以我们可以通过`toFixed`来根据指定的精度进行四舍五入，并且使用`parseFloat`来抹掉后面多余的0。比如：

{% codeblock lang:javascript %}
parseFloat((0.1 + 0.2).toFixed(10));   // 0.3
parseFloat((19.9 * 100).toFixed(10));  // 1990
{% endcodeblock %}

这样只解决掉了最最基本的情况。但是在实际项目中，用户可以输入类似`IF(SUM(0.1, 0.2) <= 0.3, 19.9 * 100, 0.69/10)`这样稍显复杂的表达式。在这个表达式中，任何一个四则运算都可能导致精度问题。因此，我们解决问题的重心就转移到了**如何劫持（或者是重载）这个表达式中的所有加减乘除运算符，并且让每个运算符都走自己实现的方法**。目前es5中并没有提供重载运算符的工具，所以我们必须自己解析这个表达式。

## 最开始的思路

说到解析，一般有3个步骤：

- 将源码转化成AST(Abstract Syntax Tree)，即抽象语法树。对应工具为[esprima](http://esprima.org/index.html)
- 遍历并更新AST。对应工具[estraverse](https://github.com/estools/estraverse)
- 将AST重新生成源码。对应工具[escodegen](https://github.com/estools/escodegen)

其中esprima提供了一个在线转AST的[工具](http://esprima.org/demo/parse.html)，可以一览AST的面貌。

但是最终我并没采用这一套方法，主要原因是我现在要解析的只是一些函数和基本表达式，并非全部的js语法，所以这套工具略显沉重。另外，有了AST之后，我们还需要手动遍历AST来劫持加减乘除运算符，这样的算法不太容易实现。

还好后来发现了jison这个工具，恰好可以满足我们的需求。

## jison

[jison](https://github.com/zaach/jison)就是[Bison](http://dinosaur.compilertools.net/bison/bison_4.html#SEC7)的js实现。它可以根据我们定义的grammar来生成parser，然后我们可以使用parser来解析表达式。由于jison的文档较少，所以，建议参看Bison的文档。下面是我看Bison文档过程中记录的一些要点：

### 上下文无关文法(Context Free grammar)

Bison如果想要解析一种语言，那么这种语言必须能够被上下文无关文法来描述。这意味着我们需要指定一个或者多个语法分组（syntactic grouping)，并且给出这些语法分组的规则（rules)。比如在c语言中，“一个表达式可能由一个减号和左右两个表达式来组成”，“一个表达式可以是一个整数”，由这两条规则可以看出，规则往往是递归的，并且必须有一条规则可以结束递归。

### 巴克斯范式(Backus Normal Form)

巴克斯范式是最常用的描述这些rules的系统，简称BNF。任何使用BNF来描述的语法，都是上下文无关文法。而BNF可以作为Bison的输入。

### 非终止符（nonterminal symbols) & 终止符(terminal symbols)

在描述rules的语法中，非终止符是那些由更小的单元组成的符号，在Bison中被称为`grouping`，一般用小写字母表示，比如'expression', 'args'等。

终止符指那些不能被降解的符号，在Bison中被称为`token`，一般用大些字母表示，比如'NUMBER', 'FUNCTION'等。但是如果终止符只有一个字符，类似'+', '-'，那么我们就可以用完全一样的'+', '-'来表示。

### 语义值(semantic value)

每个token，都可以有一个对应的语义值。比如input中的一个token，可能对应的token type为NUMBER，对应的语义值为200；而另外一个input token，对应的token type可能也是NUMBER，但是对应的语义值为5000。

grouping也是一样的，比如一个expression可以有对应的语义值为5000。

### 语义动作(semantic action)

在Bison中，每个rule都可以对应一个`action`，这样我们就可以基于input来产生输出。action可以写C语言（在jison中则是js），解析input的过程中，每次匹配到这个rule，就会执行对应的action。大多数情况下，action就是为了计算某个grouping的semantic value，如下：

{% codeblock %}
expr: expr '+' expr {$$ = $1 + $3} ;
{% endcodeblock %}

上面的grammar中，$$代表第一个expr这个grouping的语义值，$1，$3则分别代表第二个和第三个expr的语义值。

action可以省略，Bison会自动添加`$$ = $1`。

如果有多个rules，那么语法将会是这样:

{% codeblock %}
result: component1 {action1}
    | component2 {action2}
    | component3 {action3}
;
{% endcodeblock %}

不仅action可以被省略，其实component也可以被省略，这时代表匹配到**空字符串**，但是注意，此时默认的action将毫无意义，所以需要手动指定。写法通常如下：

{% codeblock lang:javascript %}
result: /* empty */ {action1}
    | component2 {action2}
;
{% endcodeblock %}

### 左递归 OR 右递归

递归是指在component中又使用result grouping，而左递归和右递归可以看下面案例：

{% codeblock %}
// good
// 左递归：第三行的args是作为最左边的component
args: /* empty */ {$$ = [];}
    | e {$$ = $1;}
    | args ',' e {$1.push($3); $$ = $1;}
;
// bad
// 右递归：第三行的args是作为最右边的component
args: /* empty */ {$$ = [];}
    | e {$$ = $1;}
    | e ',' args {$3.push($1); $$ = $3;}
;
{% endcodeblock %}

Bison文档中推荐的方式是永远左递归。

### 结合性 & 优先级

`%left`和`%right`代表改运算符是左结合还是右结合。

行号越靠前，运算符优先级越低。

关于js中的运算符优先级和结合性，可以参考[这里](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)。

## jison的使用案例

了解了Bison中一些基础概念之后，我们可以将它们迁移到jison中来。这个小节来尝试使用jison来完成我们的公式解析。

### 写好grammar file

以下为FormulaCalc.jison的内容：

{% codeblock %}
/* lexical grammar */
%lex
%%

\s+                                                            /* skip whitespace */
[0-9]+(\.[0-9]+)?\b                                            return 'NUMBER'
("==="|"!=="|">="|"<="|"=="|"!="|">"|"<")                      return 'COMPARE_OP'
"*"                                                            return '*'
"/"                                                            return '/'
"-"                                                            return '-'
"+"                                                            return '+'
<<EOF>>                                                        return 'EOF'
.                                                              return 'INVALID'

/lex

/* operator associations and precedence */

%left COMPARE_OP
%left '+' '-'
%left '*' '/'

%start expressions

%% /* language grammar */

expressions
    : e EOF
        { return $1; }
    ;

e
    : e '+' e
        {$$ = FormulaCalcParser.plus($1, $3);}
    | e '-' e
        {$$ = FormulaCalcParser.minus($1, $3);}
    | e '*' e
        {$$ = FormulaCalcParser.multiple($1, $3);}
    | e '/' e
        {$$ = FormulaCalcParser.divide($1, $3);}
    | e COMPARE_OP e
        {$$ = eval($1 + $2 + $3);}
    | NUMBER
        {$$ = Number($1);}
    ;
{% endcodeblock %}

以上有几个需要注意的点：

- 第一部分的lex grammar，左边为正则表达式，右边为对应的token。根据输入来匹配时是从上到下，从左到右的，所以正则的顺序需要注意。
- 最后一部分的language grammar中描述了rules及其对应的action，其中`%prec`表示'-'和'+'的运算符优先级和第二部分定义的UNARY相同。
- 其中FormulaParser是自己实现的一个对象，其中的加减乘除对应的方法，都对运行结果进行了精度处理。

### 生成parser

有了grammar file，jison就可以根据它来生成parser：

{% codeblock %}
jison FormulaCalc.jison
{% endcodeblock %}

这条命令会在当前目录生成`FormulaCalc.js`文件，即我们语言的parser。

### 使用parser

最后将`FormulaCalc.js`打包到项目代码中，然后就可以调用`FormulaCalc.parse(input)`来解析我们的表达式了。效果如下：

{% codeblock %}
FormulaCalc.parse('0.1 + 0.2');   // 0.3
{% endcodeblock %}

## 最后

以上就是我对jison的初步认识，更多的概念，还是去读Bison的文档吧!
