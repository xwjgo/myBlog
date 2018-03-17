title: jison VS PEG.js
tags: [jison]
categories: 乱七八糟
toc: true
date: 2018-03-09 10:08:03
---

## 前言

如果我们需要解析一门语言。那么通常我们有3种方式：

- 自己实现一个parser，这种方式代价比较大。
- 使用仅支持解析特定语言的第三方库，但这些库通常仅支持较为流行的语言，可定制性较低。
- 使用一个可以生成parser的工具，这些工具被称为`Parser Generator`。

其中第三种方式最具有灵活性，并且节省空间。因为我们可以自定义grammar来定制语言的规则，并且最后只需要引入生成的parser文件，而不需要引入库本身。

而`jison`和`PEG.js`这两个工具都属于Parse Generator，本文就以解析四则运算为例，来比较一下这两个工具的异同。

## 一些基础概念

### parser

一个parser通常由两部分组成：`lexer`和`parser`。

其中lexer又称为scanner或者tokenizer，主要工作是词法分析，即将input转化为一个个小的token。而parser的主要工作是组合lexer产生的tokens来计算结果。

![](http://7xvlvo.com1.z0.glb.clouddn.com/lexer-parser-center.png)

### grammar

grammar是对某一种语言的正规描述，简单地说，它声明了一系列规则（rules）来定义语言的结构组成。其中每条rule可以引用另外一条rule(甚至是自身)或者某个token，因此规则往往是递归的，但是至少需要有一条规则可以走出递归。比如：

{% codeblock %}
expr: expr '+' expr
    {$$ = $1 + $3;}
    | INTEGER
    {$$ = Number($1);}
;
{% endcodeblock %}

通常不同的语言类型，对应不同的grammar来描述：

- 如果一门语言可以用一系列的正则表达式来描述，且不包含递归，那么称为regular language，其对应Regular grammar。
- 如果grammar中包含递归，比如HTML语言，那么可以称为context free language，其对应Context Free grammar(CFG)。
- 后来又出现了Parsing Expression grammar(PEG)，它和CFG都很强大，但是它能够更加自然地描述变成语言。

CFG和PEG都可以描述我们常见的编程语言，但是PEG不需要进行词法分析，PEG.js就采用了这种grammar，而jison则是采用CFG。

## jison和PEG.js的异同

jison和PEG.js都是针对js的Parser Generaoter。其中jison是js版的Bison，也因此得名。PEG.js则采用PEG来描述语言，并且自称拥有出色的错误报告。

### 相同点

- 工作流一样：编写grammar -> 根据grammar生成parser -> 使用parser.parse(input)来解析并生成结果。
- grammar中基本概念相同：都包含一系列rules，每个rule都包含rule name和匹配规则，以及匹配之后要执行的action。
- 都比较流行，很多项目都依赖于这两个工具。

### 不同点

- grammar写法不同，jison使用CFG，而PEG.js使用PEG。
- bison趋于稳定且更加普及，像CoffeeScript, handlebars.js都在使用。而PEG.js还处于不断更新的阶段，在其官方文档中明确写到“PEG.js目前仍然有很多工作正在进行中，在版本V1.0之前没有兼容性保证”，而当前PEG.js的版本才发布到V0.10.0。
- jison的文档较弱，需要参考Bison的文档，而PEG.js的文档则较为全面。
- PEG.js性能稍优于jison，详情见下一小节。

## 性能对比

这里以js中四则运算为例(支持小数点，正负号，小括号，空格)，使用jison和PEGjs写出的grammar分别如下：

JisonCalc.jison:

{% codeblock %}
/* lexical grammar */
/* 这部分依次从上到下匹配，因此排列顺序很重要 */
%lex
%%

\s+                                             /* skip whitespace */
[0-9]+(\.[0-9]+)?\b                             return 'NUMBER'
"*"                                             return '*'
"/"                                             return '/'
"-"                                             return '-'
"+"                                             return '+'
"("                                             return '('
")"                                             return ')'
<<EOF>>                                         return 'EOF'
.                                               return 'INVALID'

/lex

/* operator associations and precedence */

%left '+' '-'
%left '*' '/'
%right UNARY

%start expressions

%% /* language grammar */

expressions
    : e EOF
        { return $1; }
    ;

e
    : e '+' e
        { $$ = $1 + $3; }
    | e '-' e
        { $$ = $1 - $3; }
    | e '*' e
        { $$ = $1 * $3; }
    | e '/' e
        { $$ = $1 / $3; }
    | '-' e %prec UNARY
        { $$ = -$2; }
    | '+' e %prec UNARY
        { $$ = +$2; }
    | '(' e ')'
        { $$ = $2; }
    | NUMBER
        { $$ = Number($1); }
    ;
{% endcodeblock %}

PegCalc.pegjs:

{% codeblock %}
START
    = _ e:EXPRESSION _ {return e;}

EXPRESSION
    = head:SUB_EXPR tail:(OP_1 SUB_EXPR)* {
        return tail.reduce(function (result, element) {
            if (element[0] === '+') return result + element[1];
            if (element[0] === '-') return result - element[1];
        }, head);
    }

SUB_EXPR
    = head:UNARY tail:(OP_2 FACTOR)* {
        return tail.reduce(function (result, element) {
            if (element[0] === '*') return result * element[1];
            if (element[0] === '/') return result / element[1];
        }, head);
    }

UNARY
    = sign:OP_1? factor:FACTOR _ {
        return sign === '-' ? -factor : factor;
    }

FACTOR
    = _ "(" _ expr:EXPRESSION _ ")" _ {return expr;}
    / NUM

NUM
    = _ head:[0-9]+ tail:('.' [0-9]+)? _ {
        var result = head.join('');
        if (tail && tail[1]) {
            result += '.' + tail[1].join('');
        }
        return parseFloat(result);
    }

OP_1
    = _ op:[\+\-] _ {return op;}

OP_2
    = _ op:[\*\/] _ {return op;}

_ "whitespace"
    = [ \t\n\r]*
{% endcodeblock %}

从上面可以看出，两者在grammar的一些区别：

- 两者grammar的复杂度差不多，在多看几遍文档后应该都可以理解。
- jison中需要分别声明分词语法，运算符的结合性和优先级，规则这3个部分，而PEG.js则将这些都统一到规则中体现。
- jison采用了CFG，主张使用左递归，而PEG.js使用的PEG则需要消除左递归，否则会有错误提示。

写好grammar之后，我们分别使用jison和PEG.js来生成对应的parser：

{% codeblock%}
// jison
npm install -g jison
jison JisonCalc.jison

// PEG.js
npm install -g pegjs
pegjs --format globals --export-var PegCalc PegCalc.pegjs
{% endcodeblock %}

- 这两者在生成parser时都提供了一些配置，比如指定生成parser的文件名，模块规范（Common.js, AMD）等，但是PEG.js支持的配置更多一些，比如指定暴露出的全局变量名，支持缓存解析结果。
- 生成的JisonCalc.js为22K，而PegCalc.js为21K。

在生成了`JisonCalc.js`和`PegCalc.js`这两个parser后，我们可以在项目中直接引入使用并测试，如下：

index.html:

{% codeblock lang:javascript %}
<script type="text/javascript" src="./parser/JisonCalc.js"></script>
<script type="text/javascript" src="./parser/PegCalc.js"></script>
<script type="text/javscript">
var testExpr = "-1.25 + 23.56*10*(10 + 4/(2-1.5)) + 9/3.2*8", // 4262.5
    testTimes = 100000;
// 解析结果验证
JisonCalc.parse(testExpr); // 4262.5
PegCalc.parser(testExpr);  // 4262.5
eval(testExpr);            // 4262.5
// jison测试
console.time();
for(var i = 0; i < testTimes; i ++) {
    JisonCalc.parse(testExpr);
};
console.timeEnd(); // 2456.34814453125ms
// PEG.js测试
console.time();
for(var i = 0; i < testTimes; i ++) {
    PegCalc.parse(testExpr);
};
console.timeEnd(); // 1191.030029296875ms
</script>
{% endcodeblock %}

从上面可以看出，在对同一个公式进行10万次解析的情况下，PEG.js的耗时约等于jison的1/2，在解析速度方面，PEG.js显然更具优势。

## 最后

jison和PEG.js各有自己的优点，在grammar的写作层面，个人感觉jison要更加容易一点。在生成parser时，PEGjs提供了较多的配置项，parser的大小两者相差无几。此外，jison更加稳定且在很多知名项目中使用，而PEG.js在不断更新中，且有更快的解析速度。如果不考虑大批量解析公式的场景，可能jison更适合用在生产环境中。

