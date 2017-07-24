---
title: 浅谈设计模式：解释器模式（Interpreter Pattern）
date: 2017-03-08 22:51:56
tags:
  - 设计
  - 技术
  - 设计模式
---

### 什么是解释器模式？

- 官方解释：
>to define a representation of grammar of a given language, along with an interpreter that uses this representation to interpret sentences in the language。
>
定义一个给定语言的语法表达式，并用该表达式作为一个解释器来解释语言中的句子。

- 通俗解释：给定一种语言及相关语法，根据这些语法定义一个语法表达式的解释器，客户端可以使用这个解释器来解释这个语言中句子。

<!-- more -->
### 为什么使用解释器模式？

- 语法表达式进行抽象封装，易于修改及拓展，当这个语言新增了某种特性，可以通过继承抽象表达式类来实现新的语言特性。

- 每一条语法都可以表示为一个表达式类，实现起来比较容易。

PS：该模式由于其结构特性，对于复杂语法很难维护，执行效率比较低，因此实际开发中几乎不适用这个模式，但是其本身的结构以及思想还是可以学习借鉴一下的。

### 如何使用解释器模式？
UML图如下：

![](/img/interpreter-pattern.png)

##### 各个组件解释：

- AbstractExpression（抽象表达式）：声明一个抽象的解释操作interpreter，这个接口为所有具体表达式角色（抽象语法树中的节点）都要实现的。

- TerminalExpression（终结表达式）：实现了抽象表达式角色所要求的接口，主要是一个interpret()方法；文法中的每一个终结符都有一个具体终结表达式与之相对应。比如有一个简单的公式R=R1+R2，在里面R1和R2就是终结符，对应的解析R1和R2的解释器就是终结符表达式。

- NonTerminalExpression（非终结表达式）：文法中的每一条规则都需要一个具体的非终结符表达式，非终结符表达式一般是文法中的运算符或者其他关键字，比如公式R=R1+R2中，“+”就是非终结符，解析“+”的解释器就是一个非终结符表达式。

- Client（客户端）：使用解释器的角色。

- Context（上下文）：这个角色的任务一般是用来存放文法中各个终结符所对应的具体值，比如R=R1+R2，我们给R1赋值100，给R2赋值200。这些信息需要存放到环境角色中，很多情况下我们使用一个映射（Map）来充当环境角色就足够了。

##### 应用实例：

有这么一个简单的需求，给予一个字符，让你判断是否是数字字符（‘0’-‘9’），可以这么实现：

1、定义一个抽象表达式

```java
interface AbstractExpression {
    boolean interpret(Character character);
}```

2、定义终结表达式，即直接判断字符是否是数字字符

```java
public class TerminalExpression implements AbstractExpression {
    @Override
    public boolean interpret(Character character) {
        //是否是数字字符
        return character.isDigit(character);
    }
}```

3、定义简单的非终结表达式，and 、not 、or

```java
public class AndExpression implements AbstractExpression {
    private AbstractExpression leftExpression;
    private AbstractExpression rightExpression;

    public AndExpression(AbstractExpression leftExpression, AbstractExpression rightExpression) {
        this.leftExpression = leftExpression;
        this.rightExpression = rightExpression;
    }

    @Override
    public boolean interpret(Character character) {
        return leftExpression.interpret(character) && rightExpression.interpret(character);
    }
}

public class NotExpression implements AbstractExpression {
    private AbstractExpression expression;

    public NotExpression(AbstractExpression expression) {
        this.expression = expression;
    }

    @Override
    public boolean interpret(Character character) {
        return !expression.interpret(character);
    }
}

public class OrExpression implements AbstractExpression {
    private AbstractExpression leftExpression;
    private AbstractExpression rightExpression;

    public OrExpression(AbstractExpression leftExpression, AbstractExpression rightExpression) {
        this.leftExpression = leftExpression;
        this.rightExpression = rightExpression;
    }

    @Override
    public boolean interpret(Character character) {
        return leftExpression.interpret(character) || rightExpression.interpret(character);
    }
}```

4、客户端使用，这里由于相对简单，不需要使用context组件：

```java
public class Client {
    public static void main(String[] args) {
        Character digitCharacter = new Character('1');
        Character notdigitCharacter = new Character('l');
        AbstractExpression terminalExpression = new TerminalExpression();
        AbstractExpression notExpression = new NotExpression(terminalExpression);
        AbstractExpression andExpression = new AndExpression(terminalExpression, notExpression);
        AbstractExpression orExpression = new OrExpression(terminalExpression, notExpression);

        System.out.println(andExpression.interpret(digitCharacter));
        System.out.println(andExpression.interpret(notdigitCharacter));
        System.out.println(orExpression.interpret(digitCharacter));
        System.out.println(orExpression.interpret(notdigitCharacter));
    }
}```
