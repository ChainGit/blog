---
title: 一个有趣的面试题：Integer的缓存机制
date: 2017/07/12
categories: 学习
tags:
- Java
- 学习
- 知识点
---

一个有趣的面试题：Integer的缓存机制
==========================
微信上看到了一个有趣的面试题，其实也是关于Integer的底层实现的一些细节。
整理记录一下哈哈。

## 面试题
这是一个简单的Java程序，真的很简单的。
```Java
public class IntegerCacheTest {

	@Test
	public void test() {
		Integer i1 = 100;
		Integer i2 = 100;
		System.out.println(i1 == i2);

		Integer i3 = 1000;
		Integer i4 = 1000;
		System.out.println(i3 == i4);
	}

}
```
那么输出的结果是什么呢？
是下面这个吗？
> true
> true

结果当然不是
是这个：
> true
> false

这是为什么呢？？

## 分析
使用Eclipse的Java Class Decomplier插件反编译上面的class文件。
class文件可以在项目目录里找到，如果是Java Project，就在Bin里，如果是Dynamic Web Project在build里可以找到。
反编译上面的class文件如下：
```Java
import org.junit.Test;

public class IntegerCacheTest {
	@Test
	public void test() {
		Integer i1 = Integer.valueOf(100);
		Integer i2 = Integer.valueOf(100);
		System.out.println(i1 == i2);
		Integer i3 = Integer.valueOf(1000);
		Integer i4 = Integer.valueOf(1000);
		System.out.println(i3 == i4);
	}
}
```
可以看大Java的自动装拆箱机制，使用了Integer.valueOf的方法。

再来看看Integer.valueOf方法：
```Java
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```
结合源码中的java doc文档：
> 
Returns an Integer instance representing the specified int value. If a new Integer instance is not required, this method should generally be used in preference to the constructor Integer(int), as this method is likely to yield significantly better space and time performance by __caching frequently__ requested values. This method will always __cache values in the range -128 to 127__, inclusive, and may cache other values outside of this range.

看到了吗，原来是先在缓存中（-128~127，也是一个short）找找，有就返回，没有就创建一个新的Integer对象。

再来看看那个IntegerCache：
```Java
private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```

仔细阅读源码可以发现，IntegerCache为Integer的静态且不可实例化的内部类。最小的值为-127，最大的值默认为127，也可以根据getSavedProperty("java.lang.Integer.IntegerCache.high")获得自定义的值，但是不能小于127，也不能大于Integer.MAX_VALUE - (-low) -1。

然后创建长度为(high - low) + 1的Integer数组，再由循环依次创建Integer的对象。

这个就是IntegerCache的原理了。

## 结论
__结论：__
面试题中第一个为100，在默认缓存的数组中，直接返回缓存中的Integer对象。而1000不在缓存范围，就每次都新建一个。

而且Integer中大多数为静态方法，final常量，且没有setInteger的方法，Integer内存放的value类型也为final,不能被修改。那么缓存中Integer对象也没有安全问题。
```Java
 private final int value;
```

## 补充
那么下面代码的执行结果呢？
```Java
Integer i5 = new Integer(100);
Integer i6 = new Integer(100);
System.out.println(i5 == i6);

Integer i7 = new Integer("1000");
Integer i8 = new Integer("1000");
System.out.println(i7 == i8);
```

看看new Integer(int value)和new Integer(String value)是怎么实现的吧
```Java
public Integer(int value) {
    this.value = value;
}

public Integer(String s) throws NumberFormatException {
    this.value = parseInt(s, 10);
}
```

输出：
> false 
> false

一个是类似普通的对象创建的过程，另一个就是使用parseInt过程。
这两个过程都是新建对象的方式。

另外再看看equals和hashCode的方法：
```Java
@Override
public int hashCode() {
    return Integer.hashCode(value);
}

public static int hashCode(int value) {
    return value;
}

public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}
```

而 == 比较的是对象的引用，是否引用了同一个对象（其实也是内存地址的比较）；如果是基本数据类型，则是值的比较。

而Object类的equals()方法比较是否是一个对象的方法是内存地址的比较，是可以重写的。

Integer类重写了equals()方法，只要instanceof 是Integer.class，且value相等，就认为相等。

Integer类也重写了hashCode()方法，直接返回vlaue的值。

一般来说，一个类如果涉及到比较，应该重写equals()方法，因为内存地址比较没有意义。

在集合类中还有hashCode的重写。

## 代码
```Java
public class IntegerCacheTest {

	@Test
	public void test() {
		Integer i1 = 100;
		Integer i2 = 100;
		System.out.println(i1 == i2);

		Integer i3 = 1000;
		Integer i4 = 1000;
		System.out.println(i3 == i4);

		Integer i5 = new Integer(100);
		Integer i6 = new Integer(100);
		System.out.println(i5 == i6);

		Integer i7 = new Integer("100");
		Integer i8 = new Integer("100");
		System.out.println(i7 == i8);

		System.out.println(i1.equals(i2));
		System.out.println(i3.equals(i4));
		System.out.println(i5.equals(i6));
		System.out.println(i7.equals(i8));
		
		System.out.println(i1.hashCode());
		System.out.println(i3.hashCode());
		System.out.println(i5.hashCode());
		System.out.println(i7.hashCode());
	}

}
```
输出结果：
> true
false
false
false
true
true
true
true
100
1000
100
100

## 思考
在平时编程中，往往有些整数会常常用到，比如-1，0，1，2等等，也会经常遇到int和Integer之间的转换，Integer是不可变对象，再加上Integer的缓存机制，可以减少对象的建立，加快运行的速度，降低内存的占用。


