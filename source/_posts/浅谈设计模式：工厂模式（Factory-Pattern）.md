---
title: 浅谈设计模式：工厂模式（Factory Pattern）
date: 2017-03-08 21:01:33
tags:
  - 设计
  - 技术
  - 设计模式
---

### 什么是工厂模式？

- 官方解释：
> define an interface or abstract class for creating an object but let the subclasses decide which class to instantiate
>
(定义一个创建对象的接口或者抽象类，但让子类去决定哪一个类去实例化)

- 通俗解释：当存在大量的公共的创建对象接口时，将其抽象定义一个接口或者出抽象类来创建对象，并由其子类来决定创建的是哪一个对象。
<!-- more -->

### 为什么使用工厂模式？

- 不对客户端暴露创建对象的实例化逻辑。在***实际的项目开发中，类的实例化有时候会是很复杂、很多样的，工厂模式将类的实例化逻辑封装起来，使其对于客户端（调用者）是透明的，只需要依赖相关的工厂类来创建需要的类即可。***


- 通过一个公共的接口去创建新的对象，并且这个创建的对象是延迟到子类去决定的。


- 由于不必在代码中关联应用特定的类，促进了松耦合。同样的，***特定的类的实例化逻辑往往也是很复杂的，可能需要想实例化相关联的类带进客户端中，使得客户端变得不纯粹，产生变更时造成的影响面更大，而使用工厂模式，即使是你想要创建的类的具体实现（或实例化逻辑）发生变化时，对于客户端来说也是透明的，使得系统更加稳定。***


### 如何使用工厂模式？
工厂类可以说是实际开发中使用得最多的模式，那么什么时候该使用咧？

* When a class doesn't know what sub-classes will be required to create.（当一个类不知道将要创建哪一个子类时。）


* When a class wants that its sub-classes specify the objects to be created.（当一个类希望其子类指定要创建的对象时）


* When the parent classes choose the creation of objects to its sub-classes.（当父类选择创建子类的对象时）


通俗点就是：

* 当一种产品具有多个产品簇时
* 当产品类的实例化逻辑不希望被客户端依赖时
* 当一个系统的多个类可以被抽象为一个共同的特性并能被抽象为一个创建对象的接口或抽象类时。


　　说到底，就是当你希望你的系统希望具有松耦合、少依赖的特性，并且当前的设计环境符合工厂的设计模型时，均可使用抽象工厂。但是，这里需要注意，当一个类的实例化逻辑非常简单，对于客户端依赖微乎其微以至于变更不产生相互影响或影响甚少时，可直接使用new进行实例化。

#### 具体应用实例：

假如我现在有一个产品A，产品A的实例化需要有两个前置条件（S、Y）,于是呼你可能这样做：

```java
 public class ProductA {
     public ProductA(CoditionS s, CoditionY y){
         //initialize
        System. out .println("成功生产产品A" );
    }
}
public class CoditionS {
}
public class CoditionY {
}
public class TestFactory {
     public static void main(String[] args){
         //这里是客户端，即调用方
        CoditionS s = new ConditionS();
        CoditionY y = new ConditionY();
        ProductA  a = new ProductA(s, y);
    }
}
```
- 这里很明显就是客户端已经将产品A所依赖的ConditionS、ConditionY给直接依赖进来了，事实上这两个条件和客户端并没有半毛钱关系，只是与产品A有关系而已，因此这样做的话，严重违反迪米特法则（Law of Rule）即只与直接朋友联系，不要和“陌生人”联系,而且你想想看，如果产品需要的条件不只两个呢，如果产品依赖的条件其中有一两个Condition实例化的时候可能又会依赖到其他的类或变量呢，这是一个恶性循环的依赖过程，这对于一个系统应用来说是一个噩梦，也是一个程序员不应该犯的错误。


#### 工厂模式来实现：

```java
public class Product {
     //产品抽象出来的公共属性
}
public class ProductA extends Product {
     private ConditionS s;
     private ConditionY y;
     public ProductA(CoditionS s, CoditionY y){
         //initialize
         this .s = s;
         this .y = y;
        System.out.println( "成功生产产品A" );
    }   
}
public class ProductAFactory {
     public Product createProductA(){
         return new ProductA( new CondistionS(), new ConditionY());
    }
}
public class TestFactory {
     public static void main(String[] args){
         //这里是客户端，即调用方
        ProductAFactory productAFactory = new ProductAFactory();
        Product productA = productAFactory.createProductA();
    }
}
```

- 这个时候，客户端的耦合度明显减少，仅依赖与工厂，对于工厂如何生产产品A的具体逻辑并不关心，并且这个时候整个框架看起来很容易扩展，假设客户端还需要一个属于这个产品族的产品B，这个时候产品B的实例化逻辑再复杂，对于客户端来说也是毫无关系的，它仅仅需要使用工厂来调研那个创建产品B的方法就行，至于具体的创建逻辑，则交给工厂去解决。
