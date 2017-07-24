---
title: 浅谈设计模式：状态模式（State Pattern）
date: 2017-03-09 21:46:55
tags:
  - 设计
  - 技术
  - 设计模式
---
### 什么是状态模式？
- 官方解释：
> State pattern is one of the behavioral design pattern. State design pattern is used when an Object change it’s behavior based on it’s internal state.
>
状态模式允许一个对象在其内部状态改变的时候改变其行为。这个对象看上去就像是改变了它的类一样。

<!-- more -->
### 为什么使用状态模式？
- 如果一个类的某个方法在不同的状态下可能展现出不同的行为，那么为了减少if语句，可以采用状态模式。

- 将不同的行为与相关的类解耦，使得某种状态下的状态只与相关的状态有关。

- 封装了转换过程，对于外部来说，暴露的一个具有行为变换的接口。

##### 缺点：当状态过多的时候，程序会出现过多的类。结构会变得比较分散，阅读的时候比较不方便。

### 如何使用状态模式？
状态模式的UML图如下：

![](/img/state-pattern-01.png)

- Context：客户类，聚合了一个State接口对象，该对象随着类的某些操作会动态改变其具体的引用。

- State：状态接口，定义了某些接口，以封装与Context的本状态相对应的行为。

- ConcreteState（AState、BState等）：具体状态，每一个类实现了该状态下与Context的某些行为相对应的具体行为。

##### 简单实现：
如上的UML图所示，该类的AB行为具有ABC三种状态，状态之间的转换由状态自己决定。假设上述的状态的状态机如下：

![](/img/state-pattern-02.png)

即：
- A状态：执行操作A后状态转换成B，执行操作B后状态转换成C。

- B状态：执行操作A后状态转换成C，执行操作B后状态转换成A。

- C状态：执行操作A后状态转换成A，执行操作B后状态转换成B。

##### 具体代码实现：

```java
public interface State {
     State operationA();
     State operationB();
}

public class AState implements State {
    @Override
    public State operationA() {
        System.out.println("A state's operationA");
        return new BState();
    }

    @Override
    public State operationB() {
        System.out.println("A state's operationB");
        return new CState();
    }
}

public class BState implements State {
    @Override
    public State operationA() {
        System.out.println("B state's operationA");
        return new CState();
    }

    @Override
    public State operationB() {
        System.out.println("B state's operationB");
        return new AState();
    }
}

public class CState implements State{
    @Override
    public State operationA() {
        System.out.println("C state's operationA");
        return new AState();
    }

    @Override
    public State operationB() {
        System.out.println("C state's operationB");
        return new BState();
    }
}

public class Context {
    private State state = new AState();

    public void operationA(){
        state = state.operationA();
    }

    public void operationB(){
        state = state.operationB();
    }


    public static void main(String [] args){
        Context context = new Context();
        for(int i = 0; i < 12; ++i){
            context.operationA();
        }
        for(int i = 0; i < 12; ++i){
            context.operationB();
        }
    }
}```

- 状态模式的实现方式有很多种，你可以使用纯接口方式（即上述方式），也可以使用枚举方式（结合java强大的枚举功能很容易实现），但千变万化不离其终，即状态模式本身想表达的允许一个对象在其内部状态改变的时候改变其行为。
- 所以说不要纠结于何种实现方式比较好，需要考虑的因素很多，具体的业务场景，不同的语言特性等等，可以这么说，模式是死的，设计是活的（可变的），业务场景是活的（可变的），实现代码是活的（灵活可变的），实现思想的不变的。
