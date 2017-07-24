---
title: 浅谈设计模式：抽象工厂模式（Abstract Factory Pattern）
date: 2017-03-08 21:07:58
tags:
  - 设计
  - 技术
  - 设计模式
---

### 什么是抽象工厂模式？

- 官方解释：
> define an interface or abstract class for creating families of related (or dependent) objects but without specifying their concrete sub-classes。
>
（定义一个接口或抽象类用于创建一系列相关（或依赖）的对象并且不指定他们具体的子类）

- 通俗解释：普通工厂即具体的工厂生产相对应的一种具体产品，一般情况下具有一个或一组生产产品的方法，具有比较强的单一性，而抽象工厂则是希望一个工厂生产多种不同等级结构的产品，并且具有一个或多个工厂生产同个产品族不同等级结构的产品。
<!-- more -->
例如：
1. 奶茶店生产：小杯奶茶、大杯奶茶（同一个产品族、不同的产品等级结构（小、大））;

2. 咖啡店生产：小杯咖啡、大杯咖啡（同一个咖啡产品族，不同的产品等级结构（小、大）） 两个工厂之间产品不一样，但是等级结构相似或者说相互依赖，此时便适合使用抽象工厂模式。


### 为什么使用抽象工厂模式？

- 当一个工厂等级结构可以创建出分属于不同产品等级结构的一个产品族中的所有对象时，抽象工厂模式比工厂方法模式更为简单、更有效率。

- 工厂模式具有的优点，抽象工厂都具有….

### 如何使用抽象工厂模式？
先说说抽象工厂各个成员的定义：

1. 产品等级结构：如上述，小杯奶茶可抽象为“小杯饮品”这个产品等级结构，同理中杯奶茶则属于“中杯饮品”这个产品等级结构。又或者大众生产的奥迪Q3可抽象为“乘用车”、宝马生产的x1也可抽象为“乘用车”，两者属于同一个产品等级结构。

2. 产品族：指同一个工厂下生产的不同产品等级结构的产品，如奶茶店生产的“小杯饮品”奶茶、“中杯饮品”奶茶。

3. 抽象工厂(AbstractFactory)：最顶层抽象，定义了一个组创建同一个产品族产品的方法。

4. 具体工厂(ConcreteFactory)：具体工厂，实现创建工厂内不同产品等级结构的产品的方法。

5. 抽象产品(AbstractProduct)：定义一个不同产品族但具有相同产品等级结构的产品。

6. 具体产品(ConcreteProduct)：具体产品，实现了各个产品族在该产品等级结构下的产品。

![](/img/abstract-factory.png)

***如上图所述，Aproduct、BProduct抽象了同个产品等级结构的产品，AProductA、AProductB属于同一个工厂生产的产品，即具体工厂ConcreteFactory生产的产品。抽象工厂AbstractFactory定义了一组创建XXProductA、XXProductB产品族的方法，ConcreteFactory、AnotherConcreteFactory则实现各自工厂内该等级结构下的产品的创建方法。***

#### 实例：
```java
/**
抽象工厂，创建饮料工厂
**/
interface DrinkFactory {
    SmallDrink createSmallDrink();
    MiddleDrink createMiddleDrink();
    LargeDrink createLargeDrink();
}
/**
  具体工厂，咖啡饮料工厂
**/
public class CafeFactory implements DrinkFactory {
    @Override
    public SmallDrink createSmallDrink() {
        return new SmallCafeDrink();
    }
    @Override
    public MiddleDrink createMiddleDrink() {
        return new MiddleCafeDrink();
    }
    @Override
    public LargeDrink createLargeDrink() {
        return new LargeCafeDrink();
    }
}
/**
  具体工厂，奶茶饮料工厂
**/
public class TeaFactory implements DrinkFactory {
    @Override
    public SmallDrink createSmallDrink() {
        return new SmallTeaDrink();
    }
    @Override
    public MiddleDrink createMiddleDrink() {
        return new MiddleTeaDrink();
    }
    @Override
    public LargeDrink createLargeDrink() {
        return new LargeTeaDrink();
    }
}

/**
  抽象产品，小杯饮料
**/
public class SmallDrink {
}

/**
  具体产品，小杯咖啡
**/
public class SmallCafeDrink extends SmallDrink {
    public SmallCafeDrink() {
        System.out.println("小杯咖啡");
    }
}

/**
  具体产品。小杯奶茶
**/
public class SmallTeaDrink extends SmallDrink {
    public SmallTeaDrink() {
        System.out.println("小杯奶茶");
    }
}

/**
  抽象产品，中杯饮料
**/
public class MiddleDrink {
}

/**
  具体产品，中杯咖啡
**/
public class MiddleCafeDrink extends MiddleDrink {
    public MiddleCafeDrink() {
        System.out.println("中杯咖啡");
    }
}

/**
  具体产品，中杯奶茶
**/
public class MiddleTeaDrink extends MiddleDrink {
    public MiddleTeaDrink() {
        System.out.println("中杯奶茶");
    }
}

/**
  抽象产品，大杯饮料
**/
public class LargeDrink {
}

/**
  具体产品，大杯咖啡
**/
public class LargeCafeDrink extends LargeDrink {
    public LargeCafeDrink() {
        System.out.println("大杯咖啡");
    }
}

/**
  具体产品，大杯奶茶
**/
public class LargeTeaDrink extends LargeDrink {
    public LargeTeaDrink() {
        System.out.println("大杯奶茶");
    }
}

/**
  测试客户端
**/
public class ClientTest {
    public void static main(String[] args) {
        DrinkFactory cafeFactory = new CafeFactory();
        SmallDrink smallCafe = cafeFactory.createSmallDrink();
        MiddleDrink middleCafe = cafeFactory.createMiddleDrink();
        LargeDrink largeCafe = cafeFactory.createLargeDrink();

        DrinkFactory teaFactory = new TeaFactory();
        SmallDrink smallTea = teaFactory.createSmallDrink();
        MiddleDrink middleTea = teaFactory.createMiddleDrink();
        LargeDrink largeTea = teaFactory.createLargeDrink();
    }
}
```
