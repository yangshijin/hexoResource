---
title: 浅谈设计模式：单例模式（Singleton）
date: 2017-03-08 21:55:05
tags:
  - 设计
  - 技术
  - 设计模式
---

### 什么是单例模式？

- 一个类有且仅有一个实例，由系统自行实例化并通过一个全局访问点向整个系统提供。

<!-- more -->
### 为什么使用单例模式？

- 节省内存，不需要在每次使用的时候都实例化一个对象出来
- 一个实例全局提供重复利用
- 某些环境下保证类有且只有一个实例非常重要，如：windows下任务管理器。

### 如何使用单例模式？
单例模式的写法千奇百怪，各种各种的写法都有，最终逃不过以下三个要点：

- 有且仅有一个实例
- 无需手动创建，系统自行实例化
- 通过唯一一个全局访问点向整个系统提供使用

此外不同环境下可能还会有不同要求：保证线程安全、保证序列化、反序列化时不产生多个实例、 多个类加载器加载时产生多个实例等等。

##### 第一种实现：

```java
public class Singleton {
    private static Singleton singleton = null;
    private Singleton(){
    }
    public static Singleton getInstance(){
         if(singleton == null){
             singleton = new Singleton();
        }
         return singleton ;
    }
}
```
这一种实现Singleton类被装载了，instance不一定被初始化。因为SingletonHolder类没有被主动使用，只有显示通过调用getInstance方法时，才会显示装载SingletonHolder类，从而实例化instance。

##### 第二种实现：

```java
public class Singleton {
    private static Singleton singleton = new Singleton();
    private Singleton(){
    }
    public static Singleton getInstance(){
         return singleton ;
    }
}
```

这一种单例的实例化基于classloader机制，避免了多线程安全的方式，同时也不需要在实例每一次使用时判断是否已经实例化，属于最常用的方式。类似实现还可以使用静态块来进行单例的实例化，不要累述。

##### 第三种实现：
```java
public class Singleton {
    private static class SingletonHolder {
         private static final Singleton INSTANCE = new Singleton();
    }
    private Singleton (){}
    public static final Singleton getInstance() {
    return SingletonHolder. INSTANCE;
    }
}
```

这一种实现Singleton类被装载了，instance不一定被初始化。因为SingletonHolder类没有被主动使用，只有显示通过调用getInstance方法时，才会显示装载SingletonHolder类，从而实例化instance。

##### 第四种实现：
```java
public enum  Singleton{
    SINGLETON;
     public void doWhatever(){};
}
```

这一种实现使用枚举来实现，能避免多线程同步问题，还能防止反序列化重复创建对象的问题，但由于枚举是在jdk1.5才加入的新特性，因此目前还很少看到有人在实际项目中使用。

##### 第五种实现：
```java
public class Singleton {
    private volatile static Singleton singleton;
    private Singleton (){}
    public static Singleton getSingleton() {
        if (singleton == null) {
            synchronized (Singleton. class) {
              if (singleton == null) {
                  singleton = new Singleton();
              }
            }
        }
        return singleton ;
    }
}
```

已经被摒弃使用的一种写法，具体可戳….
http://blog.csdn.net/kufeiyun/article/details/6166673

此外，在阅读uncle bob 的一篇关于单例和“仅创建一个实例”的文章中，他提供了另一种可行的方式：
```java
public class   Singleton {
     private Singleton(){}
     public static NonSingleton nonSingleton = new NonSingleton( new Singleton());
}
public class   NonSingleton {
     public NonSingleton(Singleton singleton) {
         assert (singleton != null);
    }
     public void doWhatever(){

    }
}
```

这一种实现主要是因为uncle bob 认为单例模式自身违背了SRP（单一职责原则）即一个类只做一件事的原则，而他的这种实现即是将单例实例化、以及单例做的事情分割开来。
