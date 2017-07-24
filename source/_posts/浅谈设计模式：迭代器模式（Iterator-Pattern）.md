---
title: 浅谈设计模式：迭代器模式（Iterator Pattern）
date: 2017-03-09 21:22:52
tags:
  - 设计
  - 技术
  - 设计模式
---
### 什么是迭代器模式？

- 官方解释：
> to access the elements of an aggregate object sequentially without exposing its underlying implementation
>
顺序地访问集合对象的元素并且不暴露它的内部实现

- 通俗解释：假设给定一个集合对象，定义一个与之相关联的Iterator（迭代器），该迭代器能够访问集合对象的内部元素，通过迭代的方法能够按照顺序依次访问集合对象的每一个元素。

<!-- more -->

### 为什么使用迭代器模式？

- 它能让集合内部元素结构对于客户端透明的情况下遍历该集合。保证了封装性，透明性。
- 提供了一个统一的方法去访问你的数据对象（集合），并且你不需要知道数据对象的类型。
- 实用性很强，得到了广泛的应用，并且拓展性很强，容易自己进行拓展。

### 如何使用迭代器模式？

UML图如下：

![](/img/iterator-pattern.png)

各个组件解释：

- Iterator：抽象迭代器，定义了迭代器最基本的接口。如：hasNext()表示是否该迭代是否还有下一个元素，next则是返回迭代的下一个元素。

- Aggregate：抽象集合，定义了集合的基本操作接口。如：add()向该集合增加一个原色，remove()则表示删除某个元素。

- ComcreteAggregate：具体数据集合类，定义了集合元素的结构，操作细节。同时在类内部定义了一个具体迭代器并提供一个能够返回迭代器对象的方法。

- ConcreteIterator：具体迭代器，通常定义在聚合类的内部，以此来达到访问聚合类内部数据结构的目的，同时实现了迭代访问聚合类元素的方法。

使用范围：

- 当你想要在不暴露其内部表示的情况下访问一个对象集合

- 当你想要有多种遍历方式来遍历一个对象集合，如正序、倒序、跳表访问

##### 应用举例：
假设现在有一个学生对象集合类，客户端并不想关心其内部细节，只需要能遍历学生对象（或者学生对象的某些属性）即可。使用迭代器模式来实现这个需求。

1、定义迭代器以及集合的抽象接口。
```java
interface Iterator<T> {
    T next();
    boolean hasNext();
}
interface Collection<T> {
    boolean add(T element);
    boolean remove(T element);
}```

2、定义学生对象类，简明起见，不定义太多复杂属性

```java
public class Student {
    private String id;
    private String name;
    public Student(String id, String name) {
        this.id = id;
        this.name = name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getId() {
        return id;
    }
    public void setId(String id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
}```

3、实现具体学生对象聚合类，并在内部依赖定义具体迭代器。

```java
public class StudentList implements Collection<Student> {
    List<Student> elementList = new ArrayList<>();
    @Override
    public void add(Student element) {
        elementList.add(element);
    }
    @Override
    public boolean remove(Student element) {
        return elementList.remove(element);
    }
    public Iterator<Student> createIterator(){
        return new StudentIterator(elementList);
    }
    //具体迭代器类
    private class StudentIterator implements Iterator<Student> {
        private List<Student> students;
        private int position;
        public StudentIterator(List<Student> students) {
            this.students = students;
            this.position = 0;
        }
        @Override
        public Student next() {
            if(hasNext()) {
                return students.get(position++);
            }
            return null;
        }
        @Override
        public boolean hasNext() {
            return position < students.size();
        }
    }
}```

4、使用客户端调用，此处mock一个测试数据，方便调用。

```java
public class Client {
    public static void main(String[] args) {
        StudentList studentList = new StudentList();
        studentList.add(new Student("001", "张三"));
        studentList.add(new Student("002", "李四"));
        studentList.add(new Student("004", "赵五"));
        studentList.add(new Student("004", "钱六"));

        Iterator<Student> iterator = studentList.createIterator();
        while(iterator.hasNext()){
            Student student = iterator.next();
            System.out.println(student.getId() + student.getName());
        }
    }
}```

注：此处的抽象迭代器、抽象集合接口，均只是声明了最简单常用的方法，之际开发中可以根据需求增加各种各样的方法.

*** 　　迭代器则可以根据具体的聚合，选择不同的遍历方式：正序、倒序、跳表等。而遍历的可以不是整个对象，或许你指向遍历对象的某个属性呢，如上例你指向要遍历学生的学号，或者说学号用得最勤快，那么你另外定义一个迭代方法nextId来遍历学生的学号呀。***
