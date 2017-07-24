---
title: 浅谈设计模式：模版方法（Template Method）
date: 2017-03-09 21:43:41
tags:
  - 设计
  - 技术
  - 设计模式
---

### 什么是模版方法？
- 官方解释：
> Define the skeleton of an algorithm in an operation, deferring some steps to subclasses。
>
定义一个操作的算法骨架，将某些步骤延迟到子类实现。

- 通俗解释：一次性实现一个算法的不变的部分，并将可变的行为留给子类来实现。

<!-- more -->
### 为什么使用模版方法：

- 模板方法允许子类定义一个算法的某些步骤，而不让它们改变算法的结构。

- 各子类中公共的行为提取出来并集中到一个公共父类中以避免代码重复。提高代码的可用性、复用性，提供了一个很好的代码复用平台。

- 算法骨架不变，而某些步骤可以延迟到子类来实现，不同的子类定义了这些可变操作的不同实现。具有很高的灵活性。

### 如何使用模版方法：

![](/img/template-method.png)

- AbstractTemplateClass:抽象模版方法类，定义了一个算法的骨架，如图所示，主操作方法（operationMain）中一次定义了操作的组合顺序。operationA、B、C方法为抽象方法，没有提供具体实现，延迟到子类实现。

- ComcreteTemplateClass：具体模版方法实现类，继承了抽象模版方法类的主操作方法，实现了算法骨架可变部分的方法，如图中的operationA、B、C。

##### 示例：
　　有这样一个场景，本地一个商品数据库管理者一些商品、商家、店铺的数据，每次从数据库取出来的数据和写入数据的操作是不变的，对数据的操作处理是可变的，可能是对这一块数据的商家部分处理或者商品部分处理。

##### 实现：

1、 定义一个抽象模版方法：AbstractDataProcessor，其中包含了数据操作主方法，方法内调用了读数据、写数据的不可变部分，以及处理数据的可变部分（抽象方法）。
```java
abstract public class AbstractDataProcessor {
    public void dealDataMain(){
        readData();
        dealData();
        writeData();
    }
    protected void readData(){
        System.out.println("从数据库中读取数据");
    }
    protected void writeData(){
        System.out.println("将处理完的数据写入到数据库");
    }
    protected abstract void dealData();
}```

2、 分别实现商品数据处理器、商家数据处理器类。
```java
public class ItemDataProcessor extends AbstractDataProcessor{
    @Override
    protected void dealData() {
        System.out.println("处理商品数据，如改价格、数量等操作！");
    }
}
public class SellerDataProcessor extends AbstractDataProcessor {
    @Override
    protected void dealData() {
        System.out.println("处理商家数据，如改商家信用值、商家关联店铺等！");
    }
}```

3、 编写测试类
```java
public class TemplateMethodTest {
    public static void main(String args[]){
        AbstractDataProcessor itemDataProcessor = new ItemDataProcessor();
        itemDataProcessor.dealDataMain();

        AbstractDataProcessor sellerDataProcessor = new SellerDataProcessor();
        sellerDataProcessor.dealDataMain();
    }
}```
