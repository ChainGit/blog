---
title: 常见的查找算法
date: 2016/03/24
categories: 
	- 算法
tags: 
	- 算法
	- Java
---

常见的数组查找算法(Java版)
===================

在一些算法题中，或者是做项目中，常常需要查找某个数据，为此将一些常见的查找算法加以整理。还有很多高级的查找算法，这里只是一些基础的查找算法，数组均为整型类型。
> 常见的查找算法：
1. 找出最值
2. 线性查找
3. 折半查找
4. 哈希查找

2017年12月13日，加入哈希查找。

----------
## 常见的查找算法
### 找出最值
顾名思义就是找出数组中的最大值和最小值，基础做法是循环遍历即可。
```Java
/**
	 * 找出整型数组中的<i>最小值</i>
	 * 
	 * @param a
	 *            要查找的数组
	 * @return 返回数组中最小元素的值
	 */
	public static int getMin(int[] a) {
		checkArray(a);
		int p = 0;
		for (int i = 0; i < a.length; i++)
			if (a[p] > a[i])
				p = i;
		return a[p];
	}

	/**
	 * 找出整型数组中的<i>最大值</i>
	 * 
	 * @param a
	 *            要查找的数组
	 * @return 返回数组中最大元素的值
	 */
	public static int getMax(int[] a) {
		checkArray(a);
		int p = 0;
		for (int i = 0; i < a.length; i++)
			if (a[p] < a[i])
				p = i;
		return a[p];
	}

	private static void checkArray(int[] a) {
		if (a == null || a.length < 1)
			throw new RuntimeException("array is null or empty");
	}
```

### 线性查找
所谓线性查找，简单讲是将数组遍历一遍。
```Java
/**
	 * 线性查找
	 * 
	 * @param a
	 *            待查找数组
	 * @param k
	 *            查找的数据
	 * @return 若找到则返回元素所在数组的<i>下标</i>，若没有找到则返回-1
	 */
	public static int orderSearch(int[] a, int k) {
		checkArray(a);
		int index = -1;
		for (index = 0; index < a.length; index++) {
			if (a[index] == k)
				return index;
		}
		return -1;
	}

```

### 折半查找
折半查找又叫二分查找。需要查找的数组需要是已经排序好的。这里参考别人博客里的一张图，演示二分查找的基本过程。
![image](http://img.blog.csdn.net/20130708154419484?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbHVvd2VpZnU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
```Java
/**
	 * 折半查找(二分查找)，<b>必须有序数组或者已知固定数组</b>
	 * 
	 * @param a
	 *            待查找数组
	 * @param k
	 *            查找的数据
	 * @return 若找到则返回元素所在数组的<i>下标</i>，若没有找到则返回查找的元素在序列<i>可插入的位置的负数</i>
	 */
	public static int halfSearch(int[] a, int k) {
		checkArray(a);
		int min, max, mid;
		min = 0;
		max = a.length - 1;

		do {
			mid = (max + min) >>> 1;

			if (k < a[mid])
				max = mid - 1;
			else if (k > a[mid])
				min = mid + 1;
			else if (k == a[mid])
				return mid;

		} while (min <= max);
		return -1 - min;
	}
```

### 哈希查找
哈希查找是利用hash值建表进行查找。hash主要耗时是在第一次建表的过程，只要数据没有发生改变，之后的查找速度会快的多。
```java
/**
	 * 利用hash查找无序表中的某一个元素（只是演示算法，并不是最优的）
	 * 
	 * @param a
	 *            待查找数组
	 * @param k
	 *            查找的数据
	 * @return 若找到则返回元素所在数组的<i>下标</i>，若没有找到返回-1
	 */
	public static int hashSearch(int[] a, int k) {
		int base = 10;
		int len = a.length;
		if (len < base)
			return orderSearch(a, k);
		int[] f = new int[base];
		int[][] h = new int[base][len];
		int[][] s = new int[base][len];
		for (int i = 0; i < len; i++) {
			int v = Math.abs(a[i]) % base;
			h[v][f[v]] = a[i];
			s[v][f[v]++] = i;
		}
		int e = Math.abs(k) % base;
		int p = orderSearch(h[e], k);
		return p == -1 ? -1 : s[e][p];
	}
```

## 测试结果

测试代码：

```Java
package com.chain.test;

import org.junit.Test;

import com.chain.utils.IntegerArraySearchUtils;
import com.chain.utils.IntegerArraySortUtils;
import com.chain.utils.PrintUtils;
import com.chain.utils.RandomIntegerArrayUtils;

public class IntegerArraySearchUtilsTest {

	@Test
	public void test() {
		int[] a = RandomIntegerArrayUtils.randomSelectArray(0, 9, 10);
		PrintUtils.show(a);

		System.out.println(IntegerArraySearchUtils.getMax(a));
		System.out.println(IntegerArraySearchUtils.getMin(a));

		System.out.println(IntegerArraySearchUtils.orderSearch(a, 4));
		System.out.println(IntegerArraySearchUtils.orderSearch(a, 12));

		IntegerArraySortUtils.quickSort(a);
		PrintUtils.show(a);
		System.out.println(IntegerArraySearchUtils.halfSearch(a, 8));
		System.out.println(IntegerArraySearchUtils.halfSearch(a, 10));
		System.out.println(IntegerArraySearchUtils.halfSearch(a, 20));

		a = new int[] { 2, 4, 7, 9, 11, 15, 23, 27, 29, 31, 35, 40, 43, 50, 65 };
		System.out.println(IntegerArraySearchUtils.halfSearch(a, 43));

		a = RandomIntegerArrayUtils.randomSelectArray(0, 1000, 500);
		System.out.println(IntegerArraySearchUtils.orderSearch(a, 100));
		System.out.println(IntegerArraySearchUtils.hashSearch(a, 100));

		// other test
		// System.out.println(IntegerArraySearchUtils.halfSearch(null, 8));
	}

}
```

测试结果：

```java
7 2 5 9 8 1 4 6 0 3 
9
0
6
-1
0 1 2 3 4 5 6 7 8 9 
8
-11
-11
12
245
245
```

