---
title: rand5实现rand7
date: 2017/10/17
categories: 学习
tags:
- 学习
- Java
- 求职
---

利用rand5来实现rand7（Java）
============================
这是一道很有趣的笔试题，考察随机和概率。这里整理一下。

## 考试题目

已知rand5能等概率产生1, 2, 3, 4, 5， 现要用rand5来实现rand7。（rand7的意思是要等概率产生1, 2, 3, 4, 5, 6, 7）。

## 我的解决

这个涉及概率和随机，应当用数学的方式去解决，而不是凑结果。

Java的API库实现的Math.random是等概率的，可以用它来实现rand5()。

rand5()产生从1-5的等概随机数，然后可以用rand5来实现rand7。

rand5()可以实现1-5，那么rand5()-1可以实现等概产生0-4。

如果将上者的结果再线性扩大5被，那么就可以等概产生0，5，10，15，20。

这样再将原来的rand5插入其中，就可以等概的产生0到24。

方程可以概括为：

```java
(rand5() - 1) * 5 + (rand5() - 1)
```

以上的过程其实就是将一个随机数经过加上常数和扩大的过程，整个过程是线性的。

接着有了0-24后，需要得到0-6，则需要先裁剪0-24，使之满足7的倍数，即裁剪为0-20这21个数。

最后就可以根据和7取模得到等概的0-6。进而可以得到1-7。

```java
package com.chain.javase.test.day11;

import org.junit.Test;

/**
 * 由rand5()计算出rand7()
 * 
 * @author Chain
 *
 */
public class Test11 {

	private static final long M = 10_0000_0000L;

	// rand5()测试
	@Test
	public void test1() {
		int[] t = new int[5];
		for (long i = 0; i < M; i++) {
			t[rand5() - 1]++;
		}
		print(t);
	}

	// rand7()测试
	@Test
	public void test2() {
		int[] t = new int[7];
		for (long i = 0; i < M; i++) {
			t[rand7() - 1]++;
		}
		print(t);
	}

	// 使用rand5()产生rand7()，等概产生1-7这7个数
	private int rand7() {
		while (true) {
			// 产生的数字从0到24，共25个数字，这25个数字的产生是等可能的
			// 但是25不是7的倍数，最近的是21
			int r = (rand5() - 1) * 5 + (rand5() - 1);
			// 剔除21, 22, 23, 24
			if (r > 20)
				// 虽然每次计算的时间不一样，需要有若干次循环，但是仍然是等概率的
				continue;
			else
				// 映射到1-7
				return r % 7 + 1;
		}
	}

	// 等概产生1-5这5个数
	private int rand5() {
		return (int) (Math.random() * 5 + 1);
	}

	private void print(int[] a) {
		for (int i = 0; i < a.length; i++)
			System.out.println(a[i]);
		System.out.println();
	}

}
```

思考一下，如果是已知rand7，要求实现rand5呢？

道理其实是一样的。

```java
7 * (rand7() - 1) + (rand7() - 1)
```

可以等概生成0-48，共49个数。

45是5的倍数，所以循环中剔除46,47,48,49。

然后使用

```java
t % 5 + 1
```

可以得到结果。

## 测试结果

先测试rand5()：

```java
200010915
200002853
199990426
199994398
200001408
```

再测试rand7()：

```java
142861771
142856121
142860575
142858382
142858689
142843816
142860646
```

可以看到rand5是等概的结果，rand7也是等概的结果。

## 参考博客

[csdn](http://blog.csdn.net/stpeace/article/details/46672035)