---
title: 浅谈设计模式：策略模式（Strategy Pattern）
date: 2017-03-09 21:37:54
tags:
  - 设计
  - 技术
  - 设计模式
---


### 什么是策略模式？
- 官方解释：
> Definea family of algorithms,encapsulate each one, and make them
interchangeable. Strategy lets the algorithmvary independently from
clients that use it.
>
定义一系列算法并将其封装，使他们能互相替换，该模式能使这些算法独立于使用它的客户端。

<!-- more -->
- 通俗解释：定义一种算法类型并具有多种具体实现，让使用这些算法的客户端不强依赖具体的算法实现而仅仅依赖于算法的类型，使得具体算法独立于客户端而能灵活变化。

### 为什么使用策略模式？
- 客户端不必依赖具体的算法实现，减小两者的耦合性。
- 具体算法实现可以有多种，增强客户使用的灵活性。
- 客户与具体算法互相独立，算法可重用性强。
- 消除了冗余的if –else 或switch-case等代码语句，当一个类的某一个行为具有多种具体方式时，通常将一些互补相关的代码整合到一个方法中，使用if-else或swich-case来进行判断使用，该模式实现客户与算法分离独立，减少了这些冗余的判断。

##### 缺点：
- 客户端虽然与算法相独立，但是必须知道所有的算法实现，并自行决定使用哪一个。
- 增加了客户与算法之间的通信开销，可能某些算法需要的参数不一致而导致某些算法并没有（不需要）用到实际通信传输的参数。
- 模式实现将造成类数量比较多，多少个算法实现变有多少的算法子类。

### 如何使用策略模式？

![](/img/strategy-pattern.png)

- Context：具体使用策略的客户，维护一个对Strategy对象的引用。
- Strategy：抽象的策略接口，Context直接使用该接口来调用具体的策略方法。
- StrategyA：具体策略实现。

##### 使用场景：
- 一个类（或一种类）具有多个相似行为的方法（接口）。

- 某一个算法（功能）有多种不同的变体（实现方式）。

- 用策略模式以避免暴露复杂的、与算法相关的数据结构。

- 一个类的某一个方法集成了多种相似的行为，并使用if-else等判断语句来进行判断使用哪种行为，策略模式可以将相关行为以接口方式移植进去来避免使用if-else等判断语句。

##### 使用样例：
假设现在有一个商品类，包含了商品id、名称、价格等信息，但是通过不同的支付方式价格可能会有变化，如：支付宝支付是9折、微信支付是9.5折，现金支付不打折（假设只支持者三种支付）。

##### 实现：
1、 设计一个计算商品价格的策略接口Calculator：
```java
public interface Calculator {

    public double calculatePrice(double price);
}```

2、 提供三种策略实现 AlipayCalculator、WeixinPayCalculator、CashCalculator：
```java
public class AlipayCalculator implements Calculator {

    @Override
    public double calculatePrice(double price) {
        return price * 0.9;
    }
}

public class WeixinPayCalculator implements Calculator {
    @Override
    public double calculatePrice(double price) {
        return price * 0.95;
    }
}

public class WeixinPayCalculator implements Calculator {
    @Override
    public double calculatePrice(double price) {
        return price * 0.95;
    }
}```

3、 设计商品类：
```java
public class Item {
    private Calculator calculator;
    private double itemPrice;
    private String itemId;
    private String itemName;
    public Item(String itemId, String itemName, double itemPrice){
        this.itemId = itemId;
        this.itemName = itemName;
        this.itemPrice = itemPrice;
    }
    public double getFinalPrice(){
        return calculator.calculatePrice(this.itemPrice);
    }
    public Calculator getCalculator() {
        return calculator;
    }
    public void setCalculator(Calculator calculator) {
        this.calculator = calculator;
    }
}```

4、 测试
```java
public class StrategyTest {
    public static void main(String [] args){
        Item meat = new Item("1001", "猪肉", 100L);
        Calculator alipayCalculator = new AlipayCalculator();
        Calculator weixinPayCalculator = new WeixinPayCalculator();
        Calculator cashCalculator = new CashPayCalculator();

        meat.setCalculator(alipayCalculator);
        System.out.println("支付宝支付价格：" + meat.getFinalPrice());
        meat.setCalculator(weixinPayCalculator);
        System.out.println("微信支付价格：" + meat.getFinalPrice());
        meat.setCalculator(cashCalculator);
        System.out.println("现金支付价格：" + meat.getFinalPrice());
    }
}```

PS：这里仅是提供了一种简单的样例实现，实际中价格是不能用double类型来实现的，一般价格用的是Long型，单位为分，另外商品一般也不会依赖到计算接口，一般是有另外的优惠计算服务类依赖优惠计算器。
