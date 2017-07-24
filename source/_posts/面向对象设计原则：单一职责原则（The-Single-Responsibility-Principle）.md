---
title: 面向对象设计原则：单一职责原则（The Single Responsibility Principle）
date: 2017-03-07 22:56:31
tags:
 - 技术  
 - 设计原则
---

### 什么是单一职责原则？
- 官方解释：一个类应该只有一种改变的原因
- 通俗解释：一个类被修改、拓展的时候，应该只能因为一种职责（功能）的扩展，而不应该有第二种职责导致类的修改，一个也不能有另一种职责存在。

<!-- more -->

### 为什么遵循单一职责原则？

- 降低类的复杂度，一个类负责一种职责，逻辑上也变得直观简单。
- 使代码变得灵活，提高系统的维护性。
- 变更的风险降低，有需求就会有变更，单一职责原则可降低变更的影响面。
- 保持松耦合，提供内聚。

### 如何使用单一职责原则？

> 单一职责原则是设计原则中最简单的，最直观的，但是，往往一些很有经验的开发者在比较负责的项目开发的时候，也经常会犯下错误，一个负责两个或多个职责。因为代码是经常发生变更的，有时候为了快速完成开发，或者代码逻辑并不复杂，改动不大，可能在代码逻辑上不遵循这一原则，或者在方法层次不遵循这一原则，视情况而定，在某些情况下是可行的。但更有甚者，在类层次、包层次不遵循这一原则，导致代码可读性差，影响面增大，脆弱难以维护，这是没有经验的程序员才会干出的事情。

简单举例：现在我在一个开发一个即时聊天工具，现在我想实现发送消息功能，我需要一个SendMessager类。
```java
 public class SendMessager{
      public boolean sendMessage(String content, String sender, String receiver){
           println(sender + " send message to " + receiver);
           return true;
      }
 }
```

这里没什么问题，但是你发现发送内容并没有经过加密，你想要有包含数字的内容被加密后再发送。于是呼
```java
 public class SendMessager{
      public boolean sendMessage(String content, String sender, String receiver){
           /**
                如果包含数字
           */
           if(content contains numeric){
                content = md5(content);
                println(sender + " send message to " + receiver);
                return true;
           }
           println(sender + " send message to " + receiver);
           return true;
      }
 }  
```

 或者完全遵循单一原则：
```java
 public class SendMessager{
      public boolean sendMessage(String content, String sender, String receiver){
           println(sender + " send message to " + receiver);
           return true;
      }
 }

 public class SendMd5Messager{
       /**
            发送经过加密后的消息
       */
      public boolean sendMessage(String content, String sender, String receiver){
                content = md5(content);
                println(sender + " send message to " + receiver);
                return true;
      }
 }    
```
- 在这样简单的逻辑上看，似乎这三种情况都是可行并且也不怎么影响代码的负责度。但是，当职责分化的时候，或者需求增加的时候，三种设计的影响差别就很明显了。

- 比如，现在我想增加一种加密方式，用RSA算法来加密，那么第一种实现，将会影响到代码层次上的变更，第二种实现，会影响到方法层次上的变更，第三种，并没有产生什么影响，仅仅是新增一个SendRSAMessager类而已。

- 因此，我个人认为，***三种设计层次，视职责分化的可能性而定，方法层次上的设计是可以被允许的，最好是用第三种方法，遵循单一职责原则。***
