---
title: 源码分析StringBuffer和StringBuilder
date: 2016/09/07
categories: 学习
tags:
- Java
- 学习
- 知识点
---

源码分析StringBuffer和StringBuilder
=====================
一直都说StringBuffer是线程安全的，而StringBuilder是线程不安全的。那么到底StringBuilder不安全在哪，StringBuffer又为什么是安全的呢？

## 背景知识
线程安全：
线程安全简单讲就是多线程访问同一段代码或者同一个变量时，可能会出现不可预料的结果。

## 测试代码
编写了如下测试代码：
MyThread是对StringBuilder的操作线程，对传入的StringBuilder实例进行append操作，循环执行100次。
```Java
package com.chain.test.day02;

public class MyThread implements Runnable {

	private StringBuilder sb;
	private char c;

	public MyThread(StringBuilder sb, char c) {
		super();
		this.sb = sb;
		this.c = c;
	}

	@Override
	public void run() {
		for (int i = 0; i < 100; i++) {
			sb.append(c);
		}
		System.out.println(Thread.currentThread().getName() + ": sb.length: " + sb.length());
	}
}
```

MyThread2是对StringBuffer的操作线程，对传入的StringBuffer实例进行append操作，循环执行100次。
```Java
package com.chain.test.day02;

public class MyThread2 implements Runnable {

	private StringBuffer sb;
	private char c;

	public MyThread2(StringBuffer sb, char c) {
		super();
		this.sb = sb;
		this.c = c;
	}

	@Override
	public void run() {
		for (int i = 0; i < 100; i++) {
			sb.append(c);
		}
		System.out.println(Thread.currentThread().getName() + ": sb.length: " + sb.length());
	}
}
```

最后是测试的方法，每次循环都实例化一个StringBuilder或者StringBuffer，并做为公共的操作对象传入MyThead和MyThread2中。
```Java
package com.chain.test.day02;

public class StringBufferBuilderTest {

	public static void main(String[] args) {
		// new StringBufferBuilderTest().testA();
		new StringBufferBuilderTest().testB();
	}

	public void testA() {
		for (int i = 0; i < 10; i++) {
			StringBuilder sb1 = new StringBuilder();
			test1(sb1);
			try {
				Thread.sleep(5000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println();
			System.out.println("Test1-" + i + ": " + sb1.length());
			System.out.println();
		}
	}

	public void testB() {
		for (int i = 0; i < 10; i++) {
			StringBuffer sb2 = new StringBuffer();
			test2(sb2);
			try {
				Thread.sleep(5000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println();
			System.out.println("Test1-" + i + ": " + sb2.length());
			System.out.println();
		}
	}

	public void test2(StringBuffer sb2) {
		for (int i = 0; i < 10; i++)
			new Thread(new MyThread2(sb2, (char) (i + 65))).start();
	}

	public void test1(StringBuilder sb1) {
		for (int i = 0; i < 10; i++)
			new Thread(new MyThread(sb1, (char) (i + 65))).start();
	}

}
```

## 测试结果
分别执行testA和TestB的结果如下：
执行testA：
```Java
Thread-2: sb.length: 263
Thread-1: sb.length: 297
Thread-3: sb.length: 397
Thread-0: sb.length: 100
Thread-4: sb.length: 497
Thread-7: sb.length: 597
Thread-9: sb.length: 720
Thread-5: sb.length: 818
Thread-6: sb.length: 897
Thread-8: sb.length: 997

Test1-0: 997

Thread-11: sb.length: 100
Thread-13: sb.length: 300
Thread-12: sb.length: 259
Thread-14: sb.length: 992
Thread-16: sb.length: 892
Thread-15: sb.length: 868
Thread-17: sb.length: 692
Thread-18: sb.length: 692
Thread-19: sb.length: 574
Thread-10: sb.length: 400

Test1-1: 992

Thread-20: sb.length: 100
Thread-21: sb.length: 300
Thread-24: sb.length: 500
Thread-23: sb.length: 400
Thread-22: sb.length: 300
Thread-29: sb.length: 1000
Thread-27: sb.length: 900
Thread-26: sb.length: 800
Thread-28: sb.length: 700
Thread-25: sb.length: 600

Test1-2: 1000

Thread-30: sb.length: 100
Thread-32: sb.length: 200
Thread-31: sb.length: 300
Thread-35: sb.length: 600
Thread-36: sb.length: 800
Thread-33: sb.length: 500
Thread-34: sb.length: 400
Thread-39: sb.length: 900
Thread-38: sb.length: 1000
Thread-37: sb.length: 700

Test1-3: 1000

Thread-40: sb.length: 100
Thread-42: sb.length: 200
Thread-41: sb.length: 300
Thread-45: sb.length: 500
Thread-43: sb.length: 700
Thread-44: sb.length: 600
Thread-46: sb.length: 500
Thread-47: sb.length: 1000
Thread-48: sb.length: 1000
Thread-49: sb.length: 1000

Test1-4: 1000

Thread-50: sb.length: 100
Thread-51: sb.length: 200
Thread-52: sb.length: 300
Thread-54: sb.length: 400
Thread-55: sb.length: 578
Thread-53: sb.length: 578
Thread-56: sb.length: 678
Thread-58: sb.length: 778
Thread-59: sb.length: 878
Thread-57: sb.length: 978

Test1-5: 978

Thread-60: sb.length: 100
Thread-61: sb.length: 200
Thread-65: sb.length: 300
Thread-63: sb.length: 400
Thread-62: sb.length: 500
Thread-64: sb.length: 600
Thread-66: sb.length: 700
Thread-68: sb.length: 800
Thread-69: sb.length: 900
Thread-67: sb.length: 1000

Test1-6: 1000

Thread-71: sb.length: 200
Thread-70: sb.length: 200
Thread-74: sb.length: 300
Thread-72: sb.length: 400
Thread-77: sb.length: 500
Thread-73: sb.length: 600
Thread-78: sb.length: 700
Thread-79: sb.length: 800
Thread-75: sb.length: 900
Thread-76: sb.length: 1000

Test1-7: 1000

Thread-80: sb.length: 100
Thread-81: sb.length: 200
Thread-82: sb.length: 300
Thread-83: sb.length: 400
Thread-84: sb.length: 500
Thread-86: sb.length: 600
Thread-85: sb.length: 700
Thread-87: sb.length: 800
Thread-88: sb.length: 900
Thread-89: sb.length: 1000

Test1-8: 1000

Thread-90: sb.length: 142
Thread-91: sb.length: 200
Thread-92: sb.length: 300
Thread-93: sb.length: 400
Thread-94: sb.length: 500
Thread-95: sb.length: 600
Thread-98: sb.length: 899
Thread-97: sb.length: 899
Thread-99: sb.length: 700
Thread-96: sb.length: 999

Test1-9: 999

```

执行testB：
```Java
Thread-1: sb.length: 200
Thread-3: sb.length: 400
Thread-2: sb.length: 300
Thread-0: sb.length: 200
Thread-5: sb.length: 500
Thread-6: sb.length: 658
Thread-7: sb.length: 878
Thread-9: sb.length: 864
Thread-8: sb.length: 1000
Thread-4: sb.length: 900

Test1-0: 1000

Thread-10: sb.length: 100
Thread-11: sb.length: 200
Thread-12: sb.length: 400
Thread-13: sb.length: 300
Thread-15: sb.length: 600
Thread-14: sb.length: 500
Thread-17: sb.length: 800
Thread-16: sb.length: 700
Thread-19: sb.length: 1000
Thread-18: sb.length: 900

Test1-1: 1000

Thread-20: sb.length: 100
Thread-21: sb.length: 279
Thread-23: sb.length: 449
Thread-22: sb.length: 300
Thread-25: sb.length: 600
Thread-24: sb.length: 507
Thread-29: sb.length: 985
Thread-27: sb.length: 1000
Thread-26: sb.length: 845
Thread-28: sb.length: 732

Test1-2: 1000

Thread-30: sb.length: 100
Thread-34: sb.length: 200
Thread-33: sb.length: 400
Thread-32: sb.length: 400
Thread-31: sb.length: 500
Thread-35: sb.length: 600
Thread-36: sb.length: 700
Thread-37: sb.length: 824
Thread-39: sb.length: 1000
Thread-38: sb.length: 1000

Test1-3: 1000

Thread-40: sb.length: 100
Thread-41: sb.length: 200
Thread-43: sb.length: 400
Thread-44: sb.length: 400
Thread-45: sb.length: 600
Thread-42: sb.length: 545
Thread-48: sb.length: 900
Thread-46: sb.length: 700
Thread-49: sb.length: 1000
Thread-47: sb.length: 900

Test1-4: 1000

Thread-50: sb.length: 100
Thread-53: sb.length: 300
Thread-55: sb.length: 400
Thread-54: sb.length: 500
Thread-51: sb.length: 300
Thread-52: sb.length: 700
Thread-56: sb.length: 600
Thread-59: sb.length: 883
Thread-58: sb.length: 1000
Thread-57: sb.length: 1000

Test1-5: 1000

Thread-60: sb.length: 100
Thread-61: sb.length: 200
Thread-64: sb.length: 360
Thread-63: sb.length: 541
Thread-65: sb.length: 400
Thread-62: sb.length: 600
Thread-69: sb.length: 700
Thread-66: sb.length: 913
Thread-67: sb.length: 1000
Thread-68: sb.length: 913

Test1-6: 1000

Thread-70: sb.length: 100
Thread-71: sb.length: 200
Thread-73: sb.length: 400
Thread-72: sb.length: 300
Thread-74: sb.length: 500
Thread-75: sb.length: 600
Thread-77: sb.length: 700
Thread-76: sb.length: 800
Thread-78: sb.length: 1000
Thread-79: sb.length: 1000

Test1-7: 1000

Thread-80: sb.length: 100
Thread-81: sb.length: 200
Thread-82: sb.length: 400
Thread-84: sb.length: 300
Thread-85: sb.length: 500
Thread-83: sb.length: 600
Thread-88: sb.length: 800
Thread-86: sb.length: 700
Thread-87: sb.length: 900
Thread-89: sb.length: 1000

Test1-8: 1000

Thread-90: sb.length: 100
Thread-91: sb.length: 200
Thread-96: sb.length: 421
Thread-95: sb.length: 644
Thread-98: sb.length: 421
Thread-94: sb.length: 1000
Thread-97: sb.length: 521
Thread-93: sb.length: 1000
Thread-92: sb.length: 800
Thread-99: sb.length: 700

Test1-9: 1000

```

## 结果分析
由测试结果可以看到，StringBuilder确实是线程非安全的，而StringBuffer是线程安全的。

代码并不能保证添加字符的前后顺序，但是可以判断最终添加的个数（长度）。

> 注：线程在执行sb.length()时，其他线程仍然在对sb进行append操作，所以打印的并非是100的倍数。但是最后一个线程退出时所打印的值是sb.length()的准确值。

使用反编译工具或者加入java的源码包，然后查看StringBuilder和StringBuffer的源码。

顺便也提一下String的存储方式，String是采用final型的char数组，所以对String的修改操作均返回一个新的String对象，而StringBuilder和StringBuffer使用链式编程风格。

> private final char value[];

StringBuilder为jdk5.0时诞生，而StringBuffer则是在jdk1.0时就存在了。两者均是继承实现了AbstractionStringBuilder。

StringBuilder.append(char)，length()，toString()：
```Java
/**
    * A cache of the last value returned by toString. Cleared
    * whenever the StringBuffer is modified.
    */
private transient char[] toStringCache;

@Override
public synchronized StringBuffer append(char c) {
    toStringCache = null;
    super.append(c);
    return this;
}

@Override
public synchronized int length() {
    return count;
}

@Override
public synchronized String toString() {
    if (toStringCache == null) {
        toStringCache = Arrays.copyOfRange(value, 0, count);
    }
    return new String(toStringCache, true);
}
```

StringBuffer.append(char)，length()，toString()：
```Java
@Override
public StringBuilder append(char c) {
    super.append(c);
    return this;
}

@Override
public String toString() {
    // Create a copy, don't share the array
    return new String(value, 0, count);
}

length()方法为父类AbstractStringBuilder.length()。
```

AbstractStringBuilder.append(char)，length()，toString()：
```Java
/**
* The value is used for character storage.
*/
char[] value;

/**
* The count is the number of characters used.
*/
int count;

@Override
public AbstractStringBuilder append(char c) {
    ensureCapacityInternal(count + 1);
    value[count++] = c;
    return this;
}

private void ensureCapacityInternal(int minimumCapacity) {
    // overflow-conscious code
    if (minimumCapacity - value.length > 0) {
        value = Arrays.copyOf(value,
                newCapacity(minimumCapacity));
    }
}

@Override
public int length() {
    return count;
}

@Override
public abstract String toString();
```

由源码可以看到：
1、StringBuffer采用了一个toStringCache，在调用toString()方法时将已有数据复制到toStringCache，而后再使用toStringCache创建一个String对象，而StringBuilder则是直接使用了new String(value, 0, count)。如果StringBuilder被多个线程修改时，那么value和count是不确定的，所以创建String对象时会存在问题。而StringBuffer则有效避免了这个问题。
2、StringBuffer中大部分方法都使用了synchronized关键字，来确保同一时刻只有一个进程执行代码块，确保了线程安全，但同时也降低了运行效率。
3、问题主要出在append方法，会出现value[count++] = c的覆盖问题。

### 补充说明
可以对执行StringBuilder的部分代码进行局部加锁，可以兼顾线程安全和执行性能。
比如：
```Java
synchronized(lock){
    sb.append(c);
}
```

