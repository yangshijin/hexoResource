---
title: 浅谈设计模式：责任链模式（Chain Of Responsibility）
date: 2017-03-08 22:29:45
tags:
  - 设计
  - 技术
  - 设计模式
---

### 什么是责任链模式？

- 官方解释：
> avoid coupling the sender of a request to its receiver by giving multiple objects a chance to handle the request。
>
(使多个对象都有机会处理请求，从而避免请求的发送者和接受者之间的耦合关系。)

- 通俗解释：一个执行命令可能被多个对象处理，为避免发送命令者和接收（处理）命令者之间具有比较复杂的关联，将这多个处理的对象当作一条责任链中的一个个结点，并使请求命令经过该责任链，这些结点都有机会处理请求。

<!-- more -->
### 为什么使用责任链模式？

- 发送命令者与接受命令者解耦，若不加以包装，可能一种命令要对应一种处理命令的对象，这样一发送者与接收命令者之间关联变得复杂，形成了强耦合关系。这对与系统的拓展并没有好处。将命令处理对象拼接成一条处理命令的责任链，使得发送者与接受者形成弱关联，降低耦合性，易于拓展。

- 命令处理对象之间的链接通过继承者（successor）来实现，使得节点的顺序可灵活变化，并不需要固定一个顺序。

- 当在对象中分派职责时，职责链给你更多的灵活性。你可以通过在运行时刻对该链进行动态的增加或修改来增加或改变处理一个请求的那些职责。你可以将这种机制与静态的特例化处理对象的继承机制结合起来使用。


### 如何使用责任链模式？
责任链模式的UML图如下：

![](/img/chain-of-responsibility.png)

##### 各个组件解释：

- Client：命令的发送者。

- Handler：抽象的命令处理器，定义了处理命令的抽象接口，内聚了一个Handler作为继承者（Successor）。

- ConcreteHandler：具体的命令处理器，实现了处理命令的方法，并确定了当某个命令并不需要自己处理时，下一个处理该命令的继承者是谁。

##### 使用范围：

1. 多个对象可以处理一个请求
2. 命令发送者与接受者之间的关系不想明确指定时，即命令对象希望被发送到一组命令处理器中而并不显示指定其接收器
3. 处理某个指令的对象必须动态指定

##### 纯与不纯责任链：

- 有一种说法，纯的责任链的命令处理节点只需要做到：***处理该命令并返回、不处理该命令并传递。*** 即命令可能传递到应该处理的那个节点后中断传递。

- 不纯的责任链的命令处理节点则是这样：***处理该命令并传递、不处理该命令并传递。*** 即命令一定会传递完整条责任链，其中的所有节点都有可能对该命令进行响应并处理。可能一个节点处理，也可能多个节点处理。Servlet的Filter便是采用了这种设计结构。

- 个人认为，并没有所谓纯与不纯之说，主要看你需要哪一种实现方案，说到底责任链仅仅是提供一个命令处理的包装，使得代码更加整洁，系统更加容易拓展。

##### 应用实例：
现在我们有这么一个需求，我想实现一个文本解析的功能，文件File充当命令，不同后缀格式的文件需要用不同的解析器（接受者）进行解析，那么我们可以这样：

1、定义一个抽象的解析器类，实现了公有的方法，以及声明了解析文件的抽象方法。

```java
public abstract class Parser {
    private Parser successor;
    protected String fomat;
    public void transfer(File file) {
        if (getSuccessor() != null) {
            getSuccessor().parse(file);
        } else {
            System.out.println("不能找到解析该文本的相关处理器");
        }
    }
    public abstract void parse(File file);
    protected boolean canParseFile(File file) {
        return (file != null) && (file.getName().endsWith(this.fomat));
    }
    public void setFomat(String fomat) {
        this.fomat = fomat;
    }
    public Parser getSuccessor() {
        return successor;
    }
    public void setSuccessor(Parser successor) {
        this.successor = successor;
    }
}
```

2、定义具体的解析器， 当文件属于自身解析的范畴时，进行解析，否则传递给继承者。

```java
public class TextParser extends Parser {
    public TextParser(Parser successor) {
        setFomat("txt");
        setSuccessor(successor);
    }
    @Override
    public void parse(File file) {
        if (canParseFile(file)) {
            System.out.println(file.getName() + "文本文件解析成功");
        } else {
            super.transfer(file);
        }
    }
}
```

```java
public class XmlParser extends Parser {
    public XmlParser() {
        setFomat("xml");
    }
    @Override
    public void parse(File file) {
        if (canParseFile(file)) {
            System.out.println(file.getName() + "XML文件解析成功");
        } else {
            super.transfer(file);
        }
    }
}
```

3、在客户端中定义责任链中的解析器的前后顺序。此处简明起见为：Text->Xml。
```java
public class client {
    public static void main(String[] args) {
        XmlParser xmlParser = new XmlParser();
        TextParser textParser = new TextParser(xmlParser);
        textParser.parse(new File("test.xml"));
    }
}
```

***此处可拓展地方为：***
1. 可灵活增加解析器
2. 继承者的初始化可不在构造方法中而使用setter方法
3. 根据需求可将本结构改为一个文件解析多重文本格式的功能，即如上所说的“不纯”的责任链，具体细节各位可以自己研究一下。
