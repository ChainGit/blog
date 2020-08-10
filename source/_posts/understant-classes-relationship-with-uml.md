---
title: 结合UML图理解类之间的关系(Java)
date: 2016/11/18
categories: 学习
tags:
- 学习
- Java
- 知识点
---

结合UML图理解类之间的关系(Java)
=========================
Java是一门面对对象的语言，用来抽象的描述不同对象之间的关系。对象之间的关系主要有__泛化、实现、依赖、关联、聚合、组合__。刚刚学习Java时，觉得很陌生，现在查阅一些资料后，整理出这篇博客。

## 基本概念

__泛化(generalization)__
概念：指的是一个类（称为子类、子接口）__继承__另外的一个类（称为父类、父接口）的功能，并可以增加它自己的新功能的能力。继承是类与类或者接口与接口之间最常见的关系。
体现：在Java代码中此类关系通过关键字__extends__明确标识，且在设计时一般没有什么二义性。
代码：
```java
public class Student extends Person {}
```
别名：is-a
图示：
![image](/uploads/understant-classes-relationship-with-uml/Generalization.png)

__实现(Realization)__
概念：指的是一个class类__实现interface接口__（可以是多个）的功能；实现是类与接口之间最常见的关系。子类比父类拓展了功能。
体现：在Java代码中此类关系通过关键字__implements__明确标识，且在设计时一般没有什么二义性。
别名：(is-)like-a
代码：
```java
public class BookDaoImpl implements BookDao {}

public class BookDaoImpl extends BaseDaoImpl implements BookDao {}
```
图示：
![image](/uploads/understant-classes-relationship-with-uml/Realization.png)

__依赖(Dependency)__
概念：可以简单的理解，就是一个类A使用到了另一个类B，而这种使用关系是具有偶然性的、临时性的、非常弱的，但是B类的变化会影响到A。
体现：在Java代码中，类B作为参数被类A在某个method方法中使用，也就是__局部变量__。
代码：
```java
public void drink(Water water) {}
```
别名：use-a
图示：
![image](/uploads/understant-classes-relationship-with-uml/Dependency.png)

__关联(Association)__
概念：体现的是两个类、或者类与接口之间语义级别的一种强依赖关系。这种关系比“依赖”更强、不存在“依赖”关系的偶然性，也不是临时性的，一般是长期性的，而且双方的关系一般是平等的、关联可以是单向、双向的、自身关联。
体现：在Java代码中，被关联类B以类属性的形式出现在关联类A中，也可能是关联类A引用了一个类型为被关联类B的__成员变量__。
代码：
```java
public class Employee {
    Computer computer;
}
```
别名：(has-a)
图示：
![image](/uploads/understant-classes-relationship-with-uml/Association.png)

__聚合(Aggregation)___
概念：聚合是关联关系的一种__特例__，他体现的是整体与部分的关系，拥有的关系，可以是多和一的关系，此时整体与部分之间是__可分离__的，他们可以具有各自的生命周期，部分可以属于多个整体对象，也可以为多个整体对象共享；
体现：表现在代码层面，其实和关联关系是一致的，没有什么太大的区别，只能从语义级别来区分。
代码：
```java
public class Car {
    Set<Wheel> wheel;
    Set<Door> doors;
}
```
别名：has-a
图示：
![image](/uploads/understant-classes-relationship-with-uml/Aggregation.png)


__组合(合成)(Composition)__
概念：组合也是关联关系的一种__特例__，这种关系比聚合更强，也称为__强聚合__；他同样体现整体与部分间的关系，但此时整体与部分是__不可分__的，整体的生命周期结束也就意味着部分的生命周期结束，两者生命周期一致。
体现：表现在代码层面，和关联关系是一致的，只能从语义级别来区分。
代码：
```java
public class Person {
    Head head = new Head();
}

public class Company {
    Set<Department> departments = new HashSet<>();
}
```
别名：contains-a
图示：
![image](/uploads/understant-classes-relationship-with-uml/Composition.png)

__补充说明：__
对于__继承、实现__这两种关系没多少疑问，他们体现的是一种类与类、或者类与接口间的__纵向关系__；
而其他的四类关系则体现的是类与类、或者类与接口间的引用，是一种__横向关系__，也是比较难区分的。
有很多事物间的关系要想准确定性是很难的，前文也提到这四种关系都是语义级别的，所以从代码层面并不能完全清楚的区分。
但总的来说，后几种关系所表现的__强弱程度__依次为：____组合 > 聚合 > 关联 > 依赖____；

## UML示例
这个是我根据前文基本概念画的一张经典UML类关系图，包含泛化，实现，依赖，关联，聚合，组合。
图片是SVG格式，如果浏览器不支持，可以[查看JPEG图片文件](/uploads/understant-classes-relationship-with-uml/Main.jpg)。
![image](/uploads/understant-classes-relationship-with-uml/Main.svg)

图中的接口实现的箭头不是标准的。
正确的关系表示箭头如下：
![image](/uploads/understant-classes-relationship-with-uml/Classes.jpg)

## UML代码
根据这张UML图，可以生成对应的Java代码。这里我使用Idea根据UML图写了一个测试案例，可以点击[这里下载](/uploads/understant-classes-relationship-with-uml/Test.rar)。

这里只贴出Test部分的代码：
```java
package com.chain.test.day01;

import org.junit.Test;

public class UMLTest {

    // 被 依赖 的对象需要提前创建好。
    private static Air air = new Air();
    private static Water water = new Water();

    @Test
    public void test() {
        Company company = new Company("Oracle");

        Department department1 = new Department("Database");
        Department department2 = new Department("Router");

        // 公司和部门是 组合 关系，公司不能没有部门。
        company.addDepartment(department1);
        company.addDepartment(department2);

        Computer computer1 = new Computer();
        Computer computer2 = new Computer();
        Computer computer3 = new Computer();

        // 电脑和员工是 关联 关系，员工可以没有电脑，只不过没有电脑的员工不能办公(work)。
        Employee employee1 = new Employee("Jack", computer1);
        Employee employee2 = new Employee("Mark");
        Employee employee3 = new Employee("Peter");
        Employee employee4 = new Employee("James");
        Employee employee5 = new Employee("May");

        employee3.setComputer(computer2);
        employee4.setComputer(computer3);

        // 部门和员工是 聚合 关系，部门是整体，员工是部门，员工可以不隶属于任何部门内。
        // 比如employee2就不属于任何部门
        department1.addEmployee(employee1);
        department1.addEmployee(employee3);

        department2.addEmployee(employee4);
        department2.addEmployee(employee5);

        // 输出公司内的部门，以及各部门内的员工信息
        System.out.println(company);

        // 调用员工的work方法，没有电脑的员工不能办公(work)。
        employee1.work();
        employee2.work();

        employee3.think();

        employee4.breath(air);
        employee5.drink(water);
    }

}
```

输出：
> Company{name='Oracle', departments=[Department{name='Database', employees=[Employee{name=Jack}, Employee{name=Peter}]}, Department{name='Router', employees=[Employee{name=May}, Employee{name=James}]}]}
> Employee Jack work with the computer.
> Employee Mark doesn't have a computer.
> Head doSomething
> Air doSomething
> Water doSomething

## 参考资料
[参考一](http://blog.csdn.net/kevin_darkelf/article/details/11371353)
[参考二](http://www.it165.net/pro/html/201604/65753.html)
[参考三](http://www.cnblogs.com/SceneryHao/p/5355915.html)
