---
title: 浅谈设计模式：命令模式（Command Pattern）
date: 2017-03-08 22:38:10
tags:
  - 设计
  - 技术
  - 设计模式
---

### 什么是命令模式？

- 官方解释：
> encapsulate a request under an object as a command and pass it to invoker object. Invoker object looks for the appropriate objectwhich can handle this command and pass the command to the corresponding object and that object executes the command。
>
> （封装一个请求对象作为一个命令并传递给调用器（命令发送者）对象。调用期对象寻找合适的能处理这条命令的对象并传递这条命令到应答（命令接受者）对象中，接着该对象执行这条命令.）

- 通俗解释：将一条命令封装成一个（请求）对象，定义一个命令调用器（发送者），一个命令执行器（接受者），将命令与命令接受者进行关联，并且由命令发送者来承载这条命令（一个命令调用器可承载多条命令），当客户端需要下达某个命令时，通过命令发送者调用相关命令即可。

<!-- more -->
### 为什么使用命令模式？

- 将命令抽象封装起来，使得命令易修改，可拓展，高复用性。拓展时并不需要动旧命令的代码。可讲两个简单的命令组合复用成一个新的命令。

- 将命令与命令具体的操作（接受者进行）分离，使得命令并不需要关心命令具体的操作，仅需确定命令的接受者即可。

- 将命令交由命令发送者（命令调用期）进行相关联，使得客户端并不需要关心命令的任何细节，仅需要使用命令发送者出发某个命令即可。

### 如何使用命令模式？
命令模式的UML图如下：

![](/img/command-pattern.png)

##### 各个组件解释说明：

- Command（抽象命令）：命令的抽象体，定义一个命令执行操作的接口。

- Invoker（命令调用器）：命令的调用器（发送者），内聚了一个或多个命令对象，由客户端调用某个命令。

- ConcreteCommand（具体命令）：具体的某个命令，关联了一个命令接受者，并借此实现了命令执行操作（由关联的命令接受者决定具体操作）的方法。

- Receiver（命令接受者）：定义了一系列命令动作的具体响应操作，与某些命令相关联。

- Client（客户端）：出发命令的客户端，在此之前需要将各个关联关系、内聚关系确定。注：此处也可再进行一次包装，将确定各个组件之间的关系的代码包装成一个第三方类，并由客户端调用。

##### 命令模式使用范围：
- 当一个场景中具有请求–>响应的功能，并且请求种类不少，希望响应功能对客户端透明。
- 当一个场景中，需要根据请求操作参数对象执行某些操作。
- 当你希望一个请求对象的创建和执行不在同一个时间。即命令创建后不用马上使用，在需要的时候使用。
- 当一个场景中你需要支持回滚、恢复、记录等功能时。

##### 应用实例：
现在我们有这么一个场景，还记得之前Builder模式的例子吗，我们构造出寒冰射手这个Role之后，我们要发出命令来让英雄释放技能呀，我们可以这么做：

1、我们先抽象出一个Command接口，定义一个执行命令的接口。接着再抽象出Role接口，定义了QWER四个技能的释放方法接口。并用HanBingRole实现这一接口及技能释放方法。

```java
interface Command {
    public void excute();
}
public interface Role {
    void QSkill();
    void WSkill();
    void ESkill();
    void RSkill();
}
public class HanBingRole implements Role {
    @Override
    public void QSkill() {
        System.out.println("冰霜射击！！！");
    }
    @Override
    public void WSkill() {
        System.out.println("万箭齐发！！！");
    }
    @Override
    public void ESkill() {
        System.out.println("鹰击长空！！！");
    }
    @Override
    public void RSkill() {
        System.out.println("魔法水晶箭");
    }
}```

2、抽象完毕，那么我们需要定义具体的命令类，并与命令接受者（Role）相依赖，让接受者自己来执行命令的具体操作（role.XSkill()）。

```java
public class QSkillCommand implements Command {
    private Role role;
    public QSkillCommand(Role role) {
        this.role = role;
    }
    @Override
    public void excute() {
        role.QSkill();
    }
}
public class WSkillCommand implements Command {
    private Role role;
    public WSkillCommand(Role role) {
        this.role = role;
    }
    @Override
    public void excute() {
        role.WSkill();
    }
}
public class ESkillCommand implements Command {
    private Role role;
    public QSkillCommand(Role role) {
        this.role = role;
    }
    @Override
    public void excute() {
        role.ESkill();
    }
}
public class RSkillCommand implements Command {
    private Role role;
    public QSkillCommand(Role role) {
        this.role = role;
    }
    @Override
    public void excute() {
        role.RSkill();
    }
}```

3、具体命令和接收者都定义好了，接下来我们需要一个命令发送者（Invoker），由于具体命令较多，此处可定义一个技能类型枚举SkillType（QWER），并在Invoker定义一个map，让客户端决定技能类型与具体命令的关联。

```java
enum SkillType {
    QSKILL,WSKILL,ESKILL,RSKILL
}
public class SKillInvoker {
    private Map<SkillType, Command> commandMap;

    SKillInvoker(Map<SkillType, Command> commandMap) {
        this.commandMap = commandMap;
    }
    public void castQSkill(){
        commandMap.get(SkillType.QSKILL).excute();
    }
    public void castWSkill(){
        commandMap.get(SkillType.WSKILL).excute();
    }
    public void castESkill(){
        commandMap.get(SkillType.ESKILL).excute();
    }
    public void castRSkill(){
        commandMap.get(SkillType.RSKILL).excute();
    }
}
public class client {
    public static void main(String[] agrs) {
        HanBingRole hanBingRole = new HanBingRole();
        Map<SkillType, Command> commandMap = new HashMap<>();
        commandMap.put(SkillType.QSKILL, new QSkillCommand(hanBingRole));
        commandMap.put(SkillType.WSKILL, new WSkillCommand(hanBingRole));
        commandMap.put(SkillType.ESKILL, new ESkillCommand(hanBingRole));
        commandMap.put(SkillType.RSKILL, new RSkillCommand(hanBingRole));

        SKillInvoker sKillInvoker = new SKillInvoker(commandMap);

        sKillInvoker.castQSkill();
        sKillInvoker.castWSkill();
        sKillInvoker.castESkill();
        sKillInvoker.castRSkill();
    }
}```

- 此处HanBingRole的其他属性以及实例化方式我为简明起见做了简化，并没有使用Builder模式。
- 实际开发中,HanBingRole可能是用其他的创建型模式进行实例化，创建型模式的具体用法可阅读我的前几篇文章。
- 另外，此处技能类型与命令的映射也是用了最直接的方法，实际开发中，可能使用配置方式或其他易修改可拓展的方式来实现。
