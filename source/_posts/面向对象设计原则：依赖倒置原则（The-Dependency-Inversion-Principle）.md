---
title: 面向对象设计原则：依赖倒置原则（The Dependency Inversion Principle）
date: 2017-03-07 23:05:35
tags:
 - 技术  
 - 设计原则
---

### 什么是依赖倒置原则？

- 官方解释：高层模块不应该依赖低层模块，二者都应该依赖其抽象；抽象不应该依赖细节；细节应该依赖抽象。
- 通俗解释：高层模块指的是抽象层次较高的抽象类、接口，底层模块则一般是指实现高层模块接口的类，抽象是不能够实例化的类、接口，细节是指继承抽象类、实现接口的可实例化类。明白这个后那么官方解释的依赖倒置原则就很容易理解了。
<!-- more -->

### 为什么遵循依赖倒置原则？

1. 相对于细节（可实例化类）的多变性，抽象（接口、抽象类）则要稳定的多，同常定义后除了拓展，在修改某个接口的功能的时候，基本是以修改细节或者拓展一个新的细节来实现的，因此，以抽象搭建起来的系统架构会比以细节为基础搭建的系统架构要稳定得多。
2. 依赖倒置原则的本质上就是面向接口编程（契约式编程），通过抽象（定义好接口、抽象方法）使各个细节（实现模块）相互独立，各自不影响，实现上述的一种松耦合的稳定性系统架构。
3. 抽象想要达到的效果是，通过指定好规范和契约，不去涉及具体操作细节，把具体的操作细节交给实现类（细节）去完成，同样也遵循着开闭原则。

### 如何遵循依赖倒置原则？
#### 在实际编程中，我们一般需要做到如下3点：

- 低层模块尽量都要有抽象类或接口，或者两者都有。
- 变量的声明类型尽量是抽象类或接口。
- 使用继承时遵循里氏替换原则。

#### 依赖实现的方式大致分为三种：
- 通过构造函数传递依赖的对象。
- 通过setter方法传递依赖的对象。
- 通过接口前置条件（输入参数）传递依赖的对象。


简单举例：

```java
public class Bun {
     public void cook(){
        System. out .println("包子烹饪中……" );
    }
}

public class Ming{
     public void cook(Bun bun){
        System. out .println("小明着手烹饪" );
        bun.cook();
    }
}

public class ClientTest{
     public static void main(String[] args){
        Ming xiaoming = new Ming();
        xiaoming.cook( new Bun());
    }
}
```

输出结果：
>小明着手烹饪

>包子烹饪中……

　　那么现在新的需求来了，小明已经学会了烹饪饺子，那么基于上述代码，很明显由于Ming依赖与Bun的具体实现，而不是CookingMartialFood（烹饪食物）的具体抽象，因此需要大肆改造代码才能拓展功能。如下：

```java
interface CookingMaterialFood{
     public void cook();
}

public class Bun implements CookingMaterialFood{
     public void cook(){
        System.out.println( "包子烹饪中……" );
    }
}

public class Dumpling implements CookingMaterialFood{
     public void cook(){
        System. out .println("饺子烹饪中……" );
    }
}

public class Ming{
     public void cook(CookingMaterialFood food){
        System.out.println( "小明着手烹饪" );
        food.cook();
    }
}

public class ClientTest{
     public static void main(String[] args){
        Ming xiaoming = new Ming();
        xiaoming.cook( new Bun());
        xiaoming.cook( new Dumpling());
    }
}
```

　　那么此时的设计很显然是遵循依赖倒置原则的，细节（Ming）依赖与抽象（CookingMaterialFood）实现了小明和食材的松耦合，对于Ming来说，并不关心具体的食材，当新增烹饪的食材时，仅需实现CookingMaterialFood来拓展烹饪类，而无需修改Ming类。
