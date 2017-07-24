---
title: 浅谈设计模式：建造者模式（Builder Pattern）
date: 2017-03-08 21:21:25
tags:
  - 设计
  - 技术
  - 设计模式
---

### 什么是建造者模式?

- 官方解释： 将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。

- 通俗解释：一个产品抽象成的类可能属性会很多并且这些属性很难在一个简单的（构造）方法中全部初始化完毕，或者说全部构建完毕，这个时候使用一个抽象构造器（Builder），将这类产品的构造属性抽象成几组构建方法，不同的建造者（构建器）可以有不同的组织构建实现，最终在一个流程总控的类（Director）中控制流程化调用Builder的构建方法，这样的创建型逻辑便为建造者模式。

<!-- more -->

### 为什么使用建造者模式？

- 造者模式与工厂模式很相似，同样具有很好的封装性，客户端不需要知道产品的内部组成细节。

- Builder（建造者）相对独立，对外部其他的Builder并没有依赖，因此扩展性非常好。

- 对象建造过程中，各个流程责任细分，逐步细化，局部细节的变化对于其他模块不会产生什么影响。

### 如何使用建造者模式？
建造者模式的常用UML图如下：

![](/img/builder-pattern.png)


各个组件的定义如下：

- Product（产品）：一个结构复杂的对象，实例化流程比较复杂，具有较多的代码量。实际项目中，该角色也可以是抽象产品，衍生出各种复杂的子类对象。

- Builder（抽象建造者）：为创建一个Product而抽象出流程各个步骤的抽象接口。主要用此接口来进行扩展出各式各样的产品建造者。

- ConcreteBuilder（具体建造者）：实现Builder的抽象接口，是一个具体的产品建造者，一般会有两个以上的方法，一个或一个以上接口用来实例化产品，一个用来返回实例化后的产品对象。

- Director（导演类）：内聚一个Builder接口对象，产品建造的流程总控，对建造者的各个实例步骤方法进行顺序调用，并且利用多态实现不同复杂对象的实例化。一般不与产品类发生直接依赖的关系，仅依赖于抽象层次下的Builder接口。是与客户端（调用方）直接交互的组件。

***适用范围：***

- 需要生成的产品对象内部具有复杂的结构。

- 需要生成的产品对象内部属性相互依赖，若使用常规工厂方法实例化，可能导致系统具有较高的耦合性，因此可采用建造者模式强制内部属性的生成顺序。

- 一个复杂对象的使用工厂模式的实例化算法较为稳定，而需求的变化，经常需要对该对象的局部细节进行修改，此时建造者模式的流程化构建明显满足需求。

#### 应用例子：
　　就说说英雄联盟这个游戏吧，游戏的每一个召唤师英雄的属性都是极为复杂的，那么在此简化一下，我想要构建每一个英雄的q、w、e、r四个技能，其他暂时忽视，此时在builder可抽象四个构建这四个技能的方法，以及一个获取最终产品的方法，然后我们先实现builder接口定义一下寒冰射手的英雄Builder（果然对寒冰情有独钟）,接着定义一个Director类，内聚一个Builder接口，定义一下construct方法，方法体内依次调用builder的四个技能构建接口，然后返回构建完召唤师英雄类。

```java
/**
  抽象builder，抽象英雄的基本构造
**/
public abstract class Builder {
    public abstract void buildQSkill();
    public abstract void buildWSkill();
    public abstract void buildESkill();
    public abstract void buildRSkill();
    public abstract GameRole getRole();
}
/**
  具体builder，寒冰射手艾希的具体builder
**/
public class HanBingBuilder extends Builder {
    private GameRole hangBingRole;
    public HanBingBuilder() {
        this.hangBingRole = new GameRole();
    }
    @Override
    public void buildQSkill() {
        hangBingRole.setqSkill("冰霜射击");
    }
    @Override
    public void buildWSkill() {
        hangBingRole.setwSkill("万箭齐发");
    }
    @Override
    public void buildESkill() {
        hangBingRole.seteSkill("鹰击长空");
    }
    @Override
    public void buildRSkill() {
        hangBingRole.setrSkill("魔法水晶箭");
    }
    @Override
    public GameRole getRole() {
        return hangBingRole;
    }
}

/**
  产品，即游戏英雄角色
**/
public class GameRole {
    private String qSkill;
    private String wSkill;
    private String eSkill;
    private String rSkill;
    public void setqSkill(String qSkill) {
        this.qSkill = qSkill;
    }
    public void setwSkill(String wSkill) {
        this.wSkill = wSkill;
    }
    public void seteSkill(String eSkill) {
        this.eSkill = eSkill;
    }
    public void setrSkill(String rSkill) {
        this.rSkill = rSkill;
    }
    public String getqSkill() {
        return qSkill;
    }
    public String getwSkill() {
        return wSkill;
    }
    public String geteSkill() {
        return eSkill;
    }
    public String getrSkill() {
        return rSkill;
    }
}

/**
  导演类
**/
public class Director {
    Builder roleBuilder = null;
    public void setRoleBuilder(Builder roleBuilder) {
        this.roleBuilder = roleBuilder;
    }
    public GameRole construct() {
        roleBuilder.buildQSkill();
        roleBuilder.buildWSkill();
        roleBuilder.buildESkill();
        roleBuilder.buildRSkill();
        return roleBuilder.getRole();
    }
}

/**
  客户端测试
**/
public class ClientTest {
    public static void  main(String[] args){
        Builder hanBingBuilder = new HanBingBuilder();
        Director director = new Director();
        director.setRoleBuilder(hanBingBuilder);
        GameRole hanBingRole = director.construct();
        System.out.println("q技能是:" + hanBingRole.getqSkill());
    }
}
```
***此后，要想新增一个新的召唤师英雄，比如光辉女郎，仅需实现Builder接口定义四个技能构建方法的实现即可，对于其他组件弱依赖，变更成本很低。***
