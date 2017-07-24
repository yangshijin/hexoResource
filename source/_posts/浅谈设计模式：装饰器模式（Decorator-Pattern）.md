---
title: 浅谈设计模式：装饰器模式（Decorator Pattern）
date: 2017-03-09 21:51:54
tags:
  - 设计
  - 技术
  - 设计模式
---

### 什么是装饰器模式？

- 官方解释：
>The Decorator Pattern attaches additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.
>
装饰器模式动态地给一个对象添加了一些额外的职责，装饰器提供了一个灵活可变的子类来拓展功能。

- 通俗解释：装饰器模式可以在运行时动态地给一个类对象增加功能。

<!-- more -->
### 为什么使用装饰器模式？

- 使用组合的方式，而不是通过继承对象来拓展一个类的功能。

- 把复杂的功能简单化，分散化，然后根据需要来动态组合。

- 将各个功能点解耦，使得每个功能点能独立存在不依赖其他类。

- 在不影响其他对象的情况下，以动态、透明的方式给对象添加职责。

### 如何使用装饰器模式？

装饰器的UML图如下：

![](/img/decorator-pattern.png)

- Component：功能类接口，定义了类的一些功能接口。

- ConcreteComponent：具体功能类定义，实现了功能类接口的方法。

- Decorator：装饰器基类，实现了功能类接口，并简单定义了装饰方法。

- ConcreteDecorator：具体装饰器类，继承自Decorator，具体定义了要装饰的内容（操作）。

##### 样例：

　　假设我们有这么一个需求，我要渲染一个窗口，这个窗口可以是实体的，也可以是半透明的，除此之外这个窗口还可以带有一些附属控件，比如菜单、子窗口等。现在要求每一个类型的窗口都可以自由搭配不同的附属控件。

##### 实现：
1、 定义一个渲染简单窗口的接口。
```java
public interface Window {
    void drawSimpleWindow();
}```

2、 继承上述接口定义窗口的类型。
```java
public class SolidWindow implements Window {
    @Override
    public void drawSimpleWindow() {
        System.out.println("渲染一个实体不透明窗口！！！");
    }
}

public class TranslucenceWindow implements Window {
    @Override
    public void drawSimpleWindow() {
        System.out.println("渲染一个半透明窗口！！！");
    }
}```

3、 定义一个装饰器基类，聚合了窗口接口，其中声明了渲染操作前后要装饰内容的接口。
```java
public abstract class Decorator implements Window{
    protected Window window;
    public Decorator(Window window){
        this.window = window;
    }
    @Override
    public void drawSimpleWindow() {
        doBeforeDraw();
        window.drawSimpleWindow();
        doAfterDraw();
    }

    protected abstract void doBeforeDraw();
    protected abstract void doAfterDraw();
}```

4、 定义具体装饰器类，并实现要装饰的内容。
```java
public class MenuDecorator extends Decorator {
    public MenuDecorator(Window window) {
        super(window);
    }

    @Override
    protected void doBeforeDraw() {
        System.out.println("渲染一个窗口的菜单！！！");
    }

    @Override
    protected void doAfterDraw() {
        //do nothing
    }
}

public class SubWindowDecorator extends Decorator {
    public SubWindowDecorator(Window window) {
        super(window);
    }

    @Override
    protected void doBeforeDraw() {
        //do nothing
    }

    @Override
    protected void doAfterDraw() {
        System.out.println("渲染一个窗口的子窗口！");
    }
}```

5、测试

```java
public class DecoratorTest {
    public static void main(String[] args){
        //渲染一个带有菜单和子窗口的实体窗口
        Window windowA = new SubWindowDecorator(new MenuDecorator(new SolidWindow()));
        windowA.drawSimpleWindow();
        System.out.println("---------------------------------------");
        //渲染一个带有菜单的半透明窗口
        Window windowB = new MenuDecorator(new TranslucenceWindow());
        windowB.drawSimpleWindow();
    }
}```
