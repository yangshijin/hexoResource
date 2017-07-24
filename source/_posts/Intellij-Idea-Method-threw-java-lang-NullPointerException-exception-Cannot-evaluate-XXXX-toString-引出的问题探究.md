---
title: >-
  Intellij Idea Method threw java.lang.NullPointerException exception. Cannot
  evaluate XXXX.toString()引出的问题探究
date: 2017-03-13 20:29:48
tags:
  - java
  - 日常问题
---

今天用IntelliJ Idea14在调试一个用例的时候，在某一个步骤，发现了一个Idea自身报错的现象，如图：

![](/img/toString-NPE-01.png)

<!-- more -->

虽然平时正常运行的时候，并不会报这个错误，但好奇驱使下，我决定研究一下，于是我检查了一下报错的类代码（由于出错代码是公司项目代码，我自己重写了一个能触发同样错误的类）：

```java
public class NPETestDO {

  @Getter
  @Setter
  private String testA;

  @Getter
  @Setter
  private String testB;

  @Override
  public int hashCode() {
      return NPETestDO.class.getName()
              .concat(this.testA)
              .concat(this.testB)
              .hashCode();
  }

  @Override
  public boolean equals(Object obj) {

      if(obj == this){
          return true;
      }
      if (!(obj instanceof NPETestDO)) {
          return false;
      }
      NPETestDO other = (NPETestDO) obj;
      return this.testA.equals(other.testA)
              && this.testB.equals(other.testB);
  }

}

```
  乍看之下似乎并没有什么问题，这里只重写了hashCode()和equals()方法，都没有重写toString方法，toString方法怎么会报NPE错误呢，于是我看了一个Object的toString实现，瞬间了然：

```java
public class Object {
  //其他无关代码忽略并没有贴上来

  public String toString() {
      return getClass().getName() + "@" + Integer.toHexString(hashCode());
  }
}

public final class String {
  public String concat(String str) {
    int otherLen = str.length();
    if (otherLen == 0) {
        return this;
    }
    int len = value.length;
    char buf[] = Arrays.copyOf(value, len + otherLen);
    str.getChars(buf, len);
    return new String(buf, true);
}
}
```

　　原来是Object的toString实现中 调用了自身的hashCode()方法,而在这个类重写的的hashCode()方法中，将自身的变量testA和testB作为参数传入到了String.concat()的方法中,当testA和testB其中一个为null的时候，concat的第一行实现代码int ohterLen = str.length()就会触发空指针异常。

　　而Idea在调试模式的时候，调试板上面会默认会调用类对象的toString()方法，然后将返回内容展示在调试窗口中，当这个类的testA或testB还没有被初始化值的时候，就会报上图所展示的那个错误。

然后我就想这个调试会调用类对象的toString()方法功能能不能关闭，找了一圈之后，发现还真的可以关闭，而且还提供指定类列表才调用toString方法的功能。就在：
> settings->Build,Execution,Deployment->Debugger->Data views ->java->Enable 'toString()' Object View 选项。


　　然后我觉得这个异常在运行时也很有可能出现，于是把类的hashCode()和equals()方法改了一下：

```java
public class NPETestDO {

  @Getter
  @Setter
  private String testA;

  @Getter
  @Setter
  private String testB;

  @Override
  public int hashCode() {
      if(this.testA == null || this.testB == null){
        return 0;
      }
      return NPETestDO.class.getName()
              .concat(this.testA)
              .concat(this.testB)
              .hashCode();
  }

  @Override
  public boolean equals(Object obj) {
      if(obj == null){
        return false;
      }
      if(obj == this){
          return true;
      }
      if (!(obj instanceof NPETestDO)) {
          return false;
      }
      if(this.testA == null || this.testB == null){
        return false;
      }
      NPETestDO other = (NPETestDO) obj;
      return this.testA.equals(other.testA)
              && this.testB.equals(other.testB);
  }
}

```

　　这样即使在类对象还没有实例化完全的时候被使用来做比较和hashCode或toString，也不会出现NPE异常了。

　　任何一个可能出现异常或错误的代码都应该被修复，即使这个错误只有千万分之一的几率发生，必须怀着敬畏之心看待每一行代码。
