---
title: 酷狗校招-买橙子
date: 2017/09/27
categories: 
- 学习
tags: 
- 学习
- Java
- 求职
---

酷狗校招-买橙子
=========================
酷狗的校招题目有点怪，和网易的有点像，编程题也少。好像只有一题编程题吧，在此也记录下，没有测试，所以代码可能不是正确的。

## 考试题目

不得不说，QQ的识别文字功能真的很强大。

小明去附近的水果店买橙子,水果商贩只提供整袋购买,有每袋6个和每袋8个的包装(包装不可拆分)。可是小明只想
购买恰好n个橙子，并且尽量少的袋数方便携带。如果不能购买恰好n个橙子,小明将不会购买。

请根据此实现一个程序。

要求：
输入一个整数n,表示小明想购买n(1s n s 100)个苹果
输出一个整数表示最少需要购买的袋数，如果不能买恰好n个橙子则输出- 1

例如：
输入20，输出3。

## 我的解决

直接上代码吧，暴力法。

大袋子由多至少，小袋子由少至多。

```java
package com.chain.blog.test.day06;

import java.util.Scanner;

public class BuyFruitTest {

	public static void main(String[] args) {
		try (Scanner in = new Scanner(System.in)) {
			int min = test(in);
			System.out.println(min);
		}
	}

	private static final int M = 3;
	private static final int N = 4;

	private static int test(Scanner in) {
		int n = in.nextInt();
		if (n < 1 || n > 100 || n % 2 != 0) {
			return -1;
		}

		n >>= 1;
		int min = -1;
		int topN = n / N;

		for (int i = topN; i >= 0; i--) {
			int leftM = n - i * N;
			int topM = leftM / M;
			for (int j = 0; j <= topM; j++) {
				int r = M * j + N * i;
				if (r == n) {
					int t = i + j;
					if (t < min || min == -1)
						min = t;
				}
			}
		}
		return min;
	}

}
```

## 测试结果

> 输入：
20

输出：
3

## 其他答案

可以参考牛客网上的解答。