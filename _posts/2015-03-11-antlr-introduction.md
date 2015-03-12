---
layout: post
title: Antlr介绍 - 一个好用的语言分析工具
date: 2015-03-11 17:44:10
category: "antlr"
---

Antlr（ANother Tool for Language Recognition）是一个强大的语言识别工具，可以用来分析文法、提取结构化数据、创建领域语言等。Antrl使用java开发，当前版本只能在java程序中使用，未来会支持c++、c#等语言。


Antlr当前的版本为4.5， [主页](http://www.antlr.org/){:target="_blank"}

### 为什么使用Antlr？
我的开发工作主要与搜索引擎相关，调试搜索结果时，经常有各种各样的参数，例如，一个简单的搜索请求的json格式为：

{% highlight javascript %}
{
    "query": {
        "type": "query_string",
        "content": "title:'hello world'"
    },
    "from": 0,
    "size": 10
}
{% endhighlight %}

如果使用Antlr开发了类SQL的搜索领域语言后，api是这样的：

{% highlight sql %}
select * from index where query is "title:'hello world'"
{% endhighlight %}
通过开发领域语言，使得api简化，方便了调试， 提供工作效率的同时，也降低了别人的学习成本。

### Antlr提供方便界面
Antlr为eclipse, Intellij, NetBeans等提供plugin，提供了包括语法高亮，错误检查，语法树展示等功能。


笔者使用IntelliJ IDEA，Antlr提供了相应插件 -- Antlr plugin for IntelliJ，[下载地址](https://plugins.jetbrains.com/plugin/7358?pr=){:target="_blank"}


### 如何使用Antlr？
本文将使用Antlr中的calculator的例子，在Antlr原始的calculator例子里，是使用visitor模式实现的。这里，我将使用listener模式实现。


首先，在pom中引入Antlr的maven plugin。
{% highlight xml %}
    <plugin>
        <groupId>org.antlr</groupId>
        <artifactId>antlr4-maven-plugin</artifactId>
        <version>4.2</version>
        <configuration>
            <listener>true</listener>
            <visitor>true</visitor>
        </configuration>
        <executions>
            <execution>
                <goals>
                    <goal>antlr4</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
{% endhighlight %}
需要注意的是，在configuration有listener和visitor两个配置，这是antlr使用的两种模式，后面会介绍。


然后，定义calculator的grammer。


Calculator.g4:
{% highlight java %}
grammar Calculator;
 
prog: stat+ ;
 
stat: expr # printExpr
    | ID EQL expr # assign
;
 
expr: expr op=(MUL|DIV) expr # MulDiv
    | expr op=(ADD|SUB) expr # AddSub
    | INT # int
    | ID # id
    | '(' expr ')' # parens
;
 
MUL : '*' ;             // assigns token name to '*' used above in grammar
DIV : '/' ;
ADD : '+' ;
SUB : '-' ;
EQL : '=' ;
ID : [a-zA-Z]+ ;        // match identifiers
INT : [0-9]+ ;          // match integers
NEWLINE:'\r'? '\n' ;    // return newlines to parser (is end-statement signal)
WS : [ \t]+ -> skip ;   // toss out whitespace
{% endhighlight %}
运行antlr plugin，plugin会根据Calculator.g4生成词法、语法分析文件，CalculatorLexer.java，CalculatorParser.java等。

#### Lexers和parsers是什么？
Antlr识别分析一个文法，需要两个步骤，首先通过lexer按照grammer中定义的规则分词，然后使用parser生成语法树。例如,
{% highlight java %}
    1 + 2 * 3 - 4 / 5  -> lexer -> 1|+|2|*|3|-|4|/|5 -> parser -> 语法树
{% endhighlight %}

#### Antlr如何遍历语法树？

在Antlr中，每个树节点表示grammer中定义的语法，这个节点记录了关于语法的所有信息，在Antlr中把这个树节点称之为context节点。


Antlr支持了两种模式，listener和visitor。


#### Lisener模式
Antlr编译grammer文件时会生成相应的context lisener接口，context lisener提供了enter和exit两个回调函数。在遍历语法树时，当访问context节点时，首先会调用enter方法，然后继续访问这个context节点下的子节点，当所有子节点访问完毕后，会触发context节点的exit方法。


下图大概演示了节点的访问路径,


![Antlr parse tree]({{ site.url }}/images/assets/antlr_parser_tree.png)


Lisenter模式的优点是，我们不需要自己去实现遍历方法，context节点的enter和exit方法也不需要访问子节点。

Calculator生成的parser回调函数调用顺序，
{% highlight java %}
enterStat(StatContext)
enterExpr(ExprContext)
enterAssign(AssignContext)
visitTerminal(TerminalNode)
visitTerminal(TerminalNode)
visitTerminal(TerminalNode)
visitTerminal(TerminalNode)
exitAssign(AssignContext)
exitExpr(ExprContext)
exitStat(StatContext)
{% endhighlight %}


#### Visitor模式
当我们想要自定义遍历方法，并访问相应子节点时，我们需要使用Antlr的visitor模式。


在pom.xml中，设置antlr plugin
{% highlight xml %}
    <configuration>
        <visitor>true</visitor>
    </configuration>
{% endhighlight %}
Antlr plugin会生成相应的vistor接口，用户可以自定义遍历方法。


我使用listener模式实现了一个calculator的例子。

Calculator example: [Antlr examples]{:target="_blank"}

[Antlr examples]: https://github.com/philolee/antlr-examples
