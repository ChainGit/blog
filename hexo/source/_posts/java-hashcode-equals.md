---
title: Java中hashCode、equals、toString方法
date: 2017/09/25
categories:
- 学习
tags:
- 学习
- Java
- 知识点
---

Java中hashCode、equals、toString方法
=====================================
这三个方法是Object类中的方法，换句话说是所有的类都会具有的方法。但是这三个方法对初学者来说一直有迷惑性，面试中也常被问到。现在整理一下，也写一写自己的理解。

## Object中的这三个方法

在Jdk中，所有的类的共同根父类Object.class中，这三个方法是这样定义的：

```java
public class Object{

    public native int hashCode();

    public boolean equals(Object obj) {
        return (this == obj);
    }

    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }

}
```

下面一一说明这三个方法：

__1）toString__：

toString()在Object中的默认实现其实是打印出这个类的名称和这个对象实例的hash值。
比如下面的结果：
> java.lang.Object@15db9742

在一些其他的类中，往往会重写这个方法，比如ArrayList的toString就是打印内部存储的数组。

重写toString的方法往往是为了便于调试，即在debug模式下能够很方便的看到一个对象的内部属性情况，当然也可以用System.out.println()打印输出。

ArrayList的toString()方法由父类AbstractList的父类AbstractCollection实现。

AbstractCollection.toString()：

```java
public String toString() {
    Iterator<E> it = iterator();
    if (! it.hasNext())
        return "[]";

    StringBuilder sb = new StringBuilder();
    sb.append('[');
    for (;;) {
        E e = it.next();
        sb.append(e == this ? "(this Collection)" : e);
        if (! it.hasNext())
            return sb.append(']').toString();
        sb.append(',').append(' ');
    }
}
```

__2）equals__：

Object类中的默认equals()方法是对象之间使用“==”比较。默认的“==”是比较对象的__内存引用__。

所以默认的equals方法比较的是两个对象的内存引用是否相等。

但是，在开发或其他的类库中，比较对象的内存引用其实没有什么太大的意义，所以有很多类又会重写equals方法。

__3）hashCode__：

Object类中的hashCode的实现是由JVM实现的，这是一个native方法。

查阅相关资料和结合API文档可以得知：

hashCode方法可以这样理解：

> 首先JVM将对象存放的位置分为几块，或者叫几个“桶”，然后每一个新new出来的对象会先计算其在JVM内存中的“位置”，然后再根据这个“位置”放在合适的“桶”中。

为什么JVM要这样做呢？

如果形象一点理解的话，可以有这样的情景：

假如要查找一个学校的某一个学生，怎样查找更快呢？是拿着点名簿从上往下一个一个找，还是先获得该学生的学号（hash值），再根据学号判断是哪个班级，接着再根据其他信息能直接定位到具体的这个学生，不用再遍历学生的信息表。

> 比如我的学校的某个学生的学号为：208140123，可以判断他所在的年级是14届，班级是01班，班内序号是23，这样能很快的定位到学生。

像HashSet、HashMap等都是利用hash的典型类库。

## equals详解

再来看看equals的方法：

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

其用法是obj1.equals(obj2)，顾名思义是比较两个对象是否相等。

那么两个对象相等具体是什么意思，换句话说什么是相等的对象？

API中有这样一段话（已翻译）：

```java
A 对称性（symmetric）：如果x.equals(y)返回是“true”，那么y.equals(x)也应该返回是“true”。

B 反射性（reflexive）：x.equals(x)必须返回是“true”。

C 类推性（transitive）：如果x.equals(y)返回是“true”，而且y.equals(z)返回是“true”，那么z.equals(x)也应该返回是“true”。

D 一致性（consistent）：如果x.equals(y)返回是“true”，只要x和y内容一直不变，不管你重复x.equals(y)多少次，返回都是“true”。

E 任何情况下，x.equals(null)，永远返回是“false”；x.equals(和x不同类型的对象)永远返回是“false”。
```

也就是说除非特殊需求，判断两个对象是否相等需要遵循以上的规则。

Object中的equals只有“==”，但是却完全符合上面的那个规则，是不是感觉很神奇。

下面写一个Person类，来方便理解。绝大数代码可以直接由Eclipse或者Idea生成，而且是正确的。

Person.java：

```java
package com.chain.blog.test.day07;

public class Person {

	private String name;
	private int age;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

	@Override
	public String toString() {
		StringBuilder builder = new StringBuilder();
		builder.append("Person [name=").append(name).append(", age=").append(age).append("]");
		return builder.toString();
	}

	public Person(String name, int age) {
		super();
		this.name = name;
		this.age = age;
	}

	public Person() {
		super();
	}

	@Override
	public int hashCode() {
		// 初始化一个素数
		final int prime = 31;
		// 初始化结果
		int result = 1;
		result = prime * result + age;
		result = prime * result + ((name == null) ? 0 : name.hashCode());
		return result;
	}

	// 需要满足对称性、反射性、类推性、一致性、和null总为false
	@Override
	public boolean equals(Object obj) {
		// 反射性
		if (this == obj)
			return true;
		// 和null进行比较
		if (obj == null)
			return false;
		// 实例的类不相同的肯定不相等
		// 如果有父类的话，需要使用父类的equals方法进行比较
		if (getClass() != obj.getClass())
			return false;
		Person other = (Person) obj;
		// 对象成员的具体比较，可以根据业务自行定制
		if (age != other.age)
			return false;
		// 和null的比较
		if (name == null) {
			if (other.name != null)
				return false;
		}
		// 调用成员对象的equals方法进行比较
		else if (!name.equals(other.name))
			return false;
		return true;
	}

}
```

接下来再测试一下：

```java
private static void test3() {
    Person peter = new Person("peter", 22);
    Person jack = new Person("jack", 21);
    Person peteralso = new Person("peter", 22);
    Person peteralso2 = new Person("peter", 22);

    // 对象通过“==”比较
    System.out.println(peter == peter);
    System.out.println(peter == jack);
    System.out.println(peter == peteralso);
    System.out.println(peteralso == peter);
    System.out.println(peteralso == peteralso2);

    System.out.println();

    // 反射性
    System.out.println(peter.equals(peter));
    // 对称性
    System.out.println(peter.equals(jack));
    System.out.println(jack.equals(peter));
    // 对称性
    System.out.println(peteralso.equals(peter));
    // 类推性
    System.out.println(peter.equals(peteralso));
    System.out.println(peteralso.equals(peteralso2));
    System.out.println(peter.equals(peteralso2));
    // 和null总为false
    System.out.println(peter.equals(null));
}
```

测试结果：

```java
true
false
false
false
false

true
false
false
true
true
true
true
false
```

由上可以基本掌握equals的用法。

## hashCode的详解

前面讲到hash的基本作用是为了便于快速查找。

那么hash还有其他作用吗？

Java中有一个很重要的集合类：HashSet。

Set具有确定性、互异性、无序性这三大特性。

HashSet的底层是由HashMap来实现。

先来看看HashSet.put()方法：

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

API文档中描述的是如果添加的元素重复（已存在）那么将这个待添加的元素被丢弃并返回false。

那么如何判断要添加的元素是否重复（也可以理解为已存在）呢？

接着看看HashMap的put方法：

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

有源码可以看出，一个新增add元素的大致过程：
1）先计算对象的hash值
2）判断hash表中这个对象的hash值对应的位置是否由元素，
    a) 如果没有元素，则添加上去，并返回；
    b) 如果有元素，则接着判断equals方法，
        i) 如果equals方法不想等，那么添加这个元素到这个hash所在位置链接的红黑树中；
        ii) 如果equals方法相等，那么可以判断元素重复添加，不执行添加操作且返回已存在的那个值。

由此也可以看出hashCode和equals方法的大致关系，后面再讲。

那么hash在查找get元素上的作用，删除remove元素上的作用呢？

可以阅读源码发现和add的作用是类似的。

## hashCode和equals

hashCode和equals可谓是难兄难弟，需要“捆绑”在一起使用和修改。

前面已经知道，像HashSet之类在添加和删除元素时都需要先判断hashCode，然后再判断equals。

先使用hash一是为了快速找到元素，防止遍历查找，二是为了能大致先预判判断元素是否相等。

如果两个对象连hashCode都不相等，那么肯定不是相等的对象，也就不用再进一步判断equals了。

换句话说，如果两个对象不想等，那么hashCode可能相等也可能不想等。

所以如果对象需要修改equals方法的话，也要修改hashCode方法，否则将起不到预期的作用。

比如Person中，如果不重写hashCode方法，只是重写了equals方法，那么如果将Person添加到HashSet集合中时将会在先调用hashCode时返回不相等。
这样就会出现peter和peteralso，乃至peteralso2都添加进了Set中。因为这三者都是不同的对象实例，所以hashCode都不一样的可能性很高。

所以应该谨慎对待hashCode和equals方法。

## 补充例子

下面再来一个例子，更好的说明hashCode和equals方法：

```java
private static void test4() {
    HashSet<Person> list = new HashSet<>();

    Person p1 = new Person("p1", 22);
    Person p2 = new Person("p2", 23);
    Person p3 = new Person("p1", 22);
    Person p4 = new Person("p4", 25);
    Person p5 = new Person("p5", 26);

    list.add(p1);
    list.add(p2);
    list.add(p3);
    list.add(p4);
    list.add(p5);

    System.out.println(list.size());
    System.out.println(list);

    list.remove(new Person("p5", 26));

    System.out.println(list.size());
    System.out.println(list);

    p2.setName("p2_x");

    list.remove(new Person("p2", 23));

    System.out.println(list.size());
    System.out.println(list);

    list.remove(p2);

    System.out.println(list.size());
    System.out.println(list);
}
```

猜一猜结果是什么？

是不是这个：

```java
4
[Person [name=p1, age=22], Person [name=p2, age=23], Person [name=p4, age=25], Person [name=p5, age=26]]
3
[Person [name=p1, age=22], Person [name=p2, age=23], Person [name=p4, age=25]]
2
[Person [name=p1, age=22], Person [name=p4, age=25]]
2
[Person [name=p1, age=22], Person [name=p4, age=25]]
```

其实正确的是这个：

```java
4
[Person [name=p1, age=22], Person [name=p2, age=23], Person [name=p4, age=25], Person [name=p5, age=26]]
3
[Person [name=p1, age=22], Person [name=p2, age=23], Person [name=p4, age=25]]
3
[Person [name=p1, age=22], Person [name=p2_x, age=23], Person [name=p4, age=25]]
3
[Person [name=p1, age=22], Person [name=p2_x, age=23], Person [name=p4, age=25]]
```

这个是为什么呢？

因为修改了p2的name属性后，这个对象的hash和equals都已经发生了改变。

首先hash已经改变，因为hash计算中name参与其中。

而HashSet在数据添加时，对象的位置已经根据对象自己的hash值添加到合适的位置。

而调用了remove方法时，remove方法会先计算要删除元素o的hash值，然后再根据这个hash值去已经成型的hash表中查找。而元素已经改变，因而很有可能是找不到这个元素的，就算找到了也有可能会出现误删。因此要么删除失败，要么就是误删除。

这是一个大坑，所以当使用HashSet之类时，要注意添加进Set的元素的hashCode值不能再改变了。同理equals的结果也一样，不然会违反一致性的要求。

当然hashCode和equals还有其他注意事项，以后继续学习再补充吧。

越是简单的东西有时却越是复杂。