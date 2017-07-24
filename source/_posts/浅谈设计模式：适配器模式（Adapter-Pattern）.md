---
title: 浅谈设计模式：适配器模式（Adapter Pattern）
date: 2017-03-09 21:56:10
tags:
  - 设计
  - 技术
  - 设计模式
---
### 什么是适配器模式？

- 官方解释：
> Convert the interface of a class into another interface clients expect。
>
将一个类的接口转换成另外一个客户端想要的接口。

- 通俗解释：客户端期望能有某一个接口，但是当前仅有一个功能相似但又不全部相同的接口，需要一个适配器将这个借口稍加处理包装（继承、组合）成客户端所期望的接口。

<!-- more -->

### 为什么使用适配器模式？

- -原来由于接口不兼容而不能在一起工作的那些类可以一起工作。

- 希望复用一些现存的类，但是接口又与复用环境要求不一致。

- 被适配的接口对于客户端来说是透明的，简单直接。

- 将目标类与被适配者解耦，使得被适配者无需直接在目标类上进行修改处理。

- 一个对象适配器可以把多个不同的适配者类适配到同一个目标

PS：适配器在最初设计的时候就不应出现，适配器只适合在你前期设计考虑不周，后续新的业务需求进来后进行功能上的增加、代码上的重构的时候使用。

### 如何使用适配器模式？

适配器的UML图如下：

![](/img/adapter-pattern.png)

- Client：需要调用到最终适配好的接口的客户端类。

- Target：所期待的接口形式。

- Adapter：适配器，继承了Target。类适配器继承了Adaptee，对象适配器聚合了Adaptee。

- Adaptee：被适配的类。

##### 应用场景：
-  系统需要使用现有的类，而这些类的接口不符合系统的接口规范。

-  想要建立一个可重用类，与一些彼此之间没有关联的类，包括一些可能在将来引进的类一起工作。（很少因为这种情况使用）

-  系统前期开发已经实现了一些功能，新的功能需求只能以另外接口的形式访问，不希望手动更改原有类的时候。（大部分情况）

-  使用第三方组件，组件接口定义和自己定义的不同，不希望修改自己的接口，但是要使用第三方组件接口的功能。（大部分情况）

###### 简单实现（对象适配器）：

```java
public interface Target {
    public void expectedOperation();
}

public class Adaptee {
    public void exitedOperation(){
        System.out.println("this function is old function to do something!");
    }
}

public class Adapter implements Target{

    private Adaptee adaptee = new Adaptee();

    @Override
    public void expectedOperation() {
        adaptee.exitedOperation();
        System.out.println("this function is client expected interface，it was adapted the Adaptee");
    }
}

public class Client {
    private Target target = new Adapter();
    public void bizOperation(){
        target.expectedOperation();
    }

    public static void main(String[] args){
        Client client = new Client();
        client.bizOperation();
    }
}```
