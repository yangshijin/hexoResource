---
title: 浅谈设计模式：观察者模式（Observer Pattern）
date: 2017-03-09 21:28:37
tags:
  - 设计
  - 技术
  - 设计模式
---

### 什么是观察者模式？

- 官方解释：
> Defines a one-to-many dependency between objects so that when one
object changes state, all its dependents are notified and updated
automatically.
>
在对象之间定义一个一对多的依赖关系，当一个对象状态变更时，所有依赖的对象也随着自动更新。

<!-- more -->

- 通俗解释：某些对象（观察者）对某个特定的对象（被观察者）感兴趣，想让这个对象在状态变更的时候能即时得到通知，好做出相对应的变化。因此这个对象便注册了这个观察者对象，增加到自身的观察者依赖列表中。当自身状态变更时将信息通知到这个依赖列表的所有观察者中。

### 为什么使用观察者模式？

- 当抽象个体有两个互相依赖的层面时。封装这些层面在单独的对象内将可允许程序员单独地去变更与重复使用这些对象，而不会产生两者之间交互的问题。

- 当其中一个对象的变更会影响其他对象，却又不知道多少对象必须被同时变更时。

- 当对象应该有能力通知其他对象，又不应该知道其他对象的实做细节时。

### 如何使用命令模式？

![](/img/observer-pattern.png)

- Subject：抽象主题类，内聚了一个抽象Observer列表，定义了增加观察者、删除观察者、通知观察者等基本方法。

- Observer：抽象观察者类，声明了一个更新接口。

- ConcreteSubject：具体主题类，定义了特有的状态信息，以及要通知到的信息。

- ConcreteObserver：具体观察者类，定义了当依赖的主题类信息变更的时的具体操作。

##### 具体用例：

场景：现在有一个互联网报社基站，需要当有新的报刊信息将信息的发送到订阅了的用户邮箱上。一个基站可能包含了不同的分类报刊信息，如JAVA、C++、PHP等，而用户的喜好也各不相同，仅仅选择喜欢的分类来订阅。

实现：

```java
/**
  抽象主题类
**/
public class MessagePublisher {
    private List<Observer> subscriberList = new ArrayList<Observer>();
    public void addSubscriber(Observer observer) {
        subscriberList.add(observer);
    }

    public void remove(Observer observer) {
        subscriberList.remove(subscriberList.indexOf(observer));
    }
    public void notifySubscribers(String state) {
        for(Observer observer : subscriberList){
            observer.update(state);
        }
    }
}

/**
  抽象观察者
**/
interface Observer {
    void update(String state);
}

/**
  具体主题类
**/
public class JavaMessagePublisher extends MessagePublisher {
    private String state;
    public String getState() {
        return state;
    }
    public void setState(String state) {
        this.state = state;
        notifySubscribers(state);
    }
}

/**
  具体主题类
**/
public class PHPMessagePublisher extends MessagePublisher {
    private String state;
    public String getState() {
        return state;
    }
    public void setState(String state) {
        this.state = state;
        notifySubscribers(state);
    }
}

/**
  具体观察者
**/
public class Subscriber implements Observer {
    private String userName;
    private String userMail;
    public Subscriber(String userName, String userMail){
        this.userName = userName;
        this.userMail = userMail;
    }
    @Override
    public void update(String state) {
        System.out.println(userName + "("+ userMail + ") got a new message:" + state);
    }
}

public class Main {

    public static void main(String[] args) {
        JavaMessagePublisher javaMessagePublisher = new JavaMessagePublisher();
        Subscriber userA = new Subscriber("zhangsan", "zhangsan@gmail.com");
        Subscriber userB = new Subscriber("lisi", "lisi@gmail.com");
        javaMessagePublisher.addSubscriber(userA);
        javaMessagePublisher.addSubscriber(userB);

        javaMessagePublisher.setState("jdk 1.8 publish!!!");

        PHPMessagePublisher phpMessagePublisher  = new PHPMessagePublisher();
        Subscriber userC = new Subscriber("zhaowu", "zhaowu@gmail.com");

        phpMessagePublisher.addSubscriber(userA);
        phpMessagePublisher.addSubscriber(userC);

        phpMessagePublisher.setState("php has a new version!!!");

    }
}```
