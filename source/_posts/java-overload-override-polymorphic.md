---
title: Java重写、重载、多态和自动装拆箱的简单探究
date: 2018/04/19
categories: 学习
tags:
- 学习
- Java
- 知识点
---

Java重写、重载、多态和自动装拆箱的简单探究
==========================
转眼已经在公司实习了快两个月，这两个月过的也是感觉蛮快的，也学到了不少东西。不过实习到现在，说好的博客和GitHub练习却中止了，也是惭愧。现在正好是项目QA阶段，老大抛给我了一个面试题，他刚用来问别人的，这个问题表面上是涉及到一些Java基础，再深层次就是编译和JVM的知识了。

## 情况一
```java
class Fruit {}

class Apple extends Fruit {}

class Banana extends Fruit {}

class People {
    public void eat(Fruit f) {
        System.out.println("People eat Fruit");
    }

    public void eat(Apple f) {
        System.out.println("People eat Apple");
    }

    public void eat(Banana f) {
        System.out.println("People eat Banana");
    }
}

class Boy extends People {

    public void eat(Fruit f) {
        System.out.println("Boy eats Fruit");
    }

    public void eat(Apple f) {
        System.out.println("Boy eats Apple");
    }

    public void eat(Banana f) {
        System.out.println("Boy eats Banana");
    }
}
```
测试方法：
```java
public void test() {
    Fruit apple = new Apple();
    Fruit banana = new Banana();

    People boy = new Boy();
    boy.eat(apple);
    boy.eat(banana);
}
```
结果是啥？
```java
Boy eats Fruit
Boy eats Fruit
```

神奇？？？

## 情况二

增加一个接口：
```java
interface Eatable {
    void eat(Fruit f);
}
```
增加一个女孩类：
```java
class Girl extends People implements Eatable {

    public void eat(Apple f) {
        System.out.println("Girl eats Apple");
    }

    public void eat(Banana f) {
        System.out.println("Girl eats Banana");
    }

}
```
测试方法：
```java
public void test() {
    Apple apple = new Apple();
    Banana banana = new Banana();

    Eatable eatable = new Girl();
    eatable.eat(apple);
    eatable.eat(banana);
}
```
结果是？
```java
People eats Fruit
People eats Fruit
```

什么？？？

## 情况三

新增一个水果篮子类：
```java
class Basket {

    public void takeBy(People p) {
        System.out.println("takenBy People");
    }

    public void takeBy(Eatable p) {
        System.out.println("takenBy Eatable");
    }
}
```
测试方法：
```java
public void test() {
    Basket basket = new Basket();
    Girl girl = new Girl();

    basket.takenBy(girl);
}
```
结果是？？

IDE报错！__指定不明确__。

真的假的！！！

## 情况四

自动拆箱

```java
class Box {
    public void show(int i){
        System.out.println("show int");
    }

    public void show(Object obj){
        System.out.println("show obj");
    }

    public void show(Number number){
        System.out.println("show number");
    }

    public void show(Comparable comparable){
        System.out.println("show comparable");
    }
}
```
测试代码：
```java
public void test() {
    Box box = new Box;
    Integer integer = Integer.valueOf(100);
    box.show(integer);
}
```

结果是？

依然报错，原因和情况三是一样的。__指定不明确__。

如果是这样的测试代码呢？
```java
public void test() {
    Box box = new Box;
    Number integer = Integer.valueOf(100);
    box.show(integer);
}
```

结果是？
```java
show number
```

哈哈，有点眉头了？

## 情况五

和情况四差不多，只是去掉了一个方法

自动拆箱

```java
class Box {
    public void show(int i){
        System.out.println("show int");
    }

    public void show(Object obj){
        System.out.println("show obj");
    }

    // public void show(Number number){
    //     System.out.println("show number");
    // }

    public void show(Comparable comparable){
        System.out.println("show comparable");
    }
}
```
测试代码：
```java
public void test() {
    Box box = new Box;
    Integer integer = Integer.valueOf(100);
    box.show(integer);
}
```

结果是：
```java
show comparable
```

弄懂情况三就可以理解了。

## 情况六
和情况五差不多，只是再去掉了一个方法

自动拆箱

```java
class Box {
    public void show(int i){
        System.out.println("show int");
    }

    public void show(Object obj){
        System.out.println("show obj");
    }

    // public void show(Number number){
    //     System.out.println("show number");
    // }

    // public void show(Comparable comparable){
    //     System.out.println("show comparable");
    // }
}
```
测试代码：
```java
public void test() {
    Box box = new Box;
    Integer integer = Integer.valueOf(100);
    box.show(integer);
}
```

结果是：
```java
show obj
```

向上寻找？

## 情况七
和情况六差不多，只是再去掉了一个方法

自动拆箱

```java
class Box {
    public void show(int i){
        System.out.println("show int");
    }

    // public void show(Object obj){
    //     System.out.println("show obj");
    // }

    // public void show(Number number){
    //     System.out.println("show number");
    // }

    // public void show(Comparable comparable){
    //     System.out.println("show comparable");
    // }
}
```
测试代码：
```java
public void test() {
    Box box = new Box;
    Integer integer = Integer.valueOf(100);
    box.show(integer);
}
```

结果是：
```java
show int
```

这才是正确的拆箱姿势嘛。

## 情况八

再来看看自动装箱

自动装箱

```java
class Box {
    public void show(int i){
        System.out.println("show int");
    }

    public void show(Object obj){
        System.out.println("show obj");
    }

    public void show(Number number){
        System.out.println("show number");
    }

    public void show(Comparable comparable){
        System.out.println("show comparable");
    }
}
```
测试代码：
```java
public void test() {
    Box box = new Box;
    int i = 100;
    box.show(i);
}
```

结果是，IDE报错了！__指定不明确__

## 情况九

自动装箱

```java
class Box {
    public void show(int i){
        System.out.println("show int");
    }

    public void show(Object obj){
        System.out.println("show obj");
    }

    // public void show(Number number){
    //     System.out.println("show number");
    // }

    public void show(Comparable comparable){
        System.out.println("show comparable");
    }
}
```
测试代码：
```java
public void test() {
    Box box = new Box;
    int i = 100;
    box.show(i);
}
```

结果是：
```java
show comparable
```

嗯？

## 情况十

自动装箱

```java
class Box {
    public void show(int i){
        System.out.println("show int");
    }

    public void show(Object obj){
        System.out.println("show obj");
    }

    // public void show(Number number){
    //     System.out.println("show number");
    // }

    public void show(Comparable comparable){
        System.out.println("show comparable");
    }

    public void show(Integer integer) {
        System.out.println("show integer");
    }
}
```
测试代码：
```java
public void test() {
    Box box = new Box;
    int i = 100;
    box.show(i);
}
```

结果是：
```java
show integer
```

嗯。这才是正确的自动装箱的姿势嘛。

## 总结

做了那么多的实验，和诧异的结果，到底是怎么回事呢？

书中自有黄金屋：《深入理解java虚拟机 第二版》

上面的测试大致面向几个方面：多态、重载、重写，以及自动装拆箱。

个人猜测（__不一定正确__）：

__均建立在对象不为空的情况下，尤其是包装数据类型__。

### 该由谁来执行，执行参数是什么

![image](/uploads/0-common-images/28.png)

执行的方法是在执行者的引用类（2，People）里去找，方法的参数是（1，Fruit）；
如果People里指定方法的参数没有Fruit，就会找Fruit的父类，直到找到为止（Object），如果找不到就会报错（编译器或IDE报错）。
如果找到了就去真正的执行者（3，Boy）里去找刚才找到的符合参数的方法，找到就执行，找不到就按纵向查找，即执行者父类依次查找，直到找到People为止（沿着父类查找的过程中参数类型不变）。
如果找到的方法是抽象或接口方法则报错。

Java是一门静态多分派且动态单分派的语言。
具体我也不懂，可以看书和查询相关博客。

[参考博客](https://blog.csdn.net/sunxianghuang/article/details/52280002)

语言也是一直变化的，没有是绝对不变的。

### 对于既不是基本数据类型，也不是包装数据类型
![image](/uploads/0-common-images/30.png)

结合“该由谁来执行，执行参数是什么”按先横再纵。
比如：
第一次：male.eat(apple) -> male.eat(fruit)
第二次：boy.eat(fruit) -> male.eat(fruit)
...
就这样直到找到匹配的。

### 对于基本数据类型
先去找基本数据类型，没找到再尝试进行类型提升，比如int转成long，继续寻找。
如果没有找到就尝试进行包装，比如int包装成Integer，然后再按“既不是基本数据类型，也不是包装数据类型”处理。
如果还是没找到就报错。

![image](/uploads/0-common-images/29.png)

__包装类型不会进行自动提升__。

### 对于包装数据类型
先按照“既不是基本数据类型，也不是包装数据类型”进行处理，找不到最后再尝试拆箱成基本数据类型(比如Integer拆箱成int)进行查找。
转成基本数据类型后弱火还是没找到就再尝试进行类型提升，比如int转成long，继续寻找。
如果还是没找到就报错。

__目前是没有基本数据类型自动类型下降的__。

## 思考

为什么要纠结这个问题呢？有什么意义？

一个案例：
```java
class BaseService {
    public void process() {
        System.out.println("process old");
    }
}

class NewBaseService extends BaseService {
    public void process() {
        System.out.println("process new");
    }
}

class GoodsService {
    public void service(BaseService baseService) {
        System.out.println("service old process old");
        baseService.process();
    }
}


class NewGoodsService extends GoodsService {
    public void service(NewBaseService baseService) {
        System.out.println("service new process new");
        baseService.process();
    }
}

@Test
public void test3() {
    BaseService baseService = new NewBaseService();
    GoodsService goodsService = new NewGoodsService();
    goodsService.service(baseService);
}
```

结果是：
```java
service old process old
process new
```

但是需要的效果却是：
```java
service new process new
process new
```

怎么办呢？需要对原来的旧类进行改造才行。
改成如下：

```java
class BaseService {
    public void process() {
        System.out.println("process old");
    }
}

class NewBaseService extends BaseService {
    public void process() {
        System.out.println("process new");
    }
}

class GoodsService {
    public void service(BaseService baseService) {
        System.out.println("service old process old");
        baseService.process();
    }

    public void service(NewBaseService baseService) {
        System.out.println("service old process new");
        baseService.process();
    }
}


class NewGoodsService extends GoodsService {
//        public void service(BaseService baseService) {
//            System.out.println("service new process old");
//            baseService.process();
//        }

    public void service(NewBaseService baseService) {
        System.out.println("service new process new");
        baseService.process();
    }
}

@Test
public void test3() {
    NewBaseService baseService = new NewBaseService();
    GoodsService goodsService = new NewGoodsService();
    goodsService.service(baseService);
}
```

对比就知道了原因了。







