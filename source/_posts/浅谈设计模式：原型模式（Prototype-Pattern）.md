---
title: 浅谈设计模式：原型模式（Prototype Pattern）
date: 2017-03-08 21:34:16
tags:
  - 设计
  - 技术
  - 设计模式
---

### 什么是原型模式？

- 官方解释： cloning of an existing object instead of creating new one and can also be customized as per the requirement.(克隆一个现有对象来代替新建一个对象，并且可以按定制要求克隆。)

- 通俗解释：通过新建一个原型对象（该对象实现一个具有克隆接口的抽象类、接口）指明要创建的类型，调用对象自身的克隆接口创建出更多的同类型对象。

<!-- more -->

### 为什么使用原型模式？
1. 简化对象创建的过程，利用现有对象直接调用克隆方法实现对象的创建。
2. 降低子类的需求。
3. 客户获取对象时不需要知道哪一个类型的对象会呗创建。
4. 可在运行时创建、销毁对象。

如何使用原型模式？
原型模式UML图如下：

![](/img/prototype-pattern.png)

- 调用方：在已有一个原型对象的前提下，调用原型对象的克隆方法来获取（大量）同类型对象。
- 原型接口：定义一个clone接口的抽象类（接口）。
- 具体原型类：每一个原型类都要实现（继承）该接口，并实现接口的clone（克隆）方法。这里的clone必须是深克隆的，即在一些具有引用型对象（java的引用、c/c++的指针），必须将引用型对象所指的成员全部复制一遍，而不是仅仅将引用的地址复制一遍。

这里讲到了深复制，为了让不懂这个概念的同学了解一下，那么就来科普一下深复制和浅复制的概念。

- 浅复制：克隆类对象的时候，在类具有引用型成员对象时，仅仅将对象类中的引用性对象的地址复制一份到新的对象去，所以此时被克隆出来新的对象的应用型成员对象指向的还是和原来的对象一样。

- 深复制：克隆类对象的时候，将类内部的引用型成员对象指向的成员也挨个复制一边赋值到新建的类对象中，并且此过程可嵌套，因为引用型成员对象指向的成员也可能具有引用型成员对象，因此在实现克隆的时候，需要考虑克隆的对象是否会出现这个嵌套的过程，再三注意，若嵌套过深造成克隆的性能过差，可考虑放弃使用原型模式（这种情况一般很少出现）。

原型模式使用场景：

- 当类需要在运行时进行实例化。
- 当创建一个对象过于复杂和昂贵。
- 当你想保持应用中类数量最小。
- 当客户端（调用方）不想知道类的创建和表示。


#### 应用实例：

听说过纳米机器人吗？现在有一种这样的纳米机器人，他们只要有足够的能量就能无限自我复制，在许多领域都有强大的作用，那么我们现在来抽象定义一个纳米机器人的原型。

1. 首先肯定是：Prototype抽象类（接口）、该接口具有clone方法。在java中，Object对象具有clone接口，这个接口是声明为protected的，java本身所有类Object的子类，但是需要实现Cloneable接口后才能调用clone方法。因此，在java中Prototype这个接口可直接省略，直接用具体原型对象类实现Cloneable接口即可。题外话，这里的clone方法还是声明为native的，java中该关键字的意思是该声明的方法的实现使用非java代码实现的，详细细节可自行搜索哈。

2. 接着定义具体的原型对象类（naroRobot）并实现原型接口（此处是Cloneable），重写clone方法，克隆的实现必须是深复制的，这里不纠结与深浅复制的实现，之后会有相关详细的介绍。

```java
public class NaroRobot implements Cloneable {
    private String id;
    private String name;
    //……各种属性，为引用型成员时需要进行深复制
    public NaroRobot(String id, String name) {
        this.id = id;
        this.name = name;
    }
    @Override
    protected Object clone() throws CloneNotSupportedException {
        //此处假设该原型类没有引用型对象。
        return super.clone();
    }
}
```

最后是在现有一个naroRobot的情况下，调用对象的clone方法实现大规模纳米机器人的创建
```java
public class Client {
    public static void main(String[] args) {
        NaroRobot naroRobot = new NaroRobot("007", "robot");
        List<NaroRobot> naroRobotList = new ArrayList<>();
        for (int i = 0; i < 100; ++i) {
            try {
                naroRobotList.add(naroRobot.clone());
            } catch (CloneNotSupportedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
 PS：此处的原型接口也可不省略，在有多个同类的原型对象类时，可增加一个抽象类实现Cloneable接口，然后具体原型类再继承该类即可。
