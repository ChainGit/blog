---
title: 艺龙校招-逃脱神凛幻域
date: 2017/09/26
categories: 
- 学习
tags: 
- 学习
- Java
- 求职
---

艺龙校招-逃脱神凛幻域
=========================
做了艺龙的在线笔试，题目难度不大，两道编程题都AC了，哈哈。这里贴一下最后一道题目。

最后一道题目主要在于理解题意，理解好后就容易多了。

## 考试题目

不得不说，QQ的识别文字功能真的很强大。

__逃脱神凛幻域__

时间限制:C/C++ 语言1000MS ;其他语言3000MS
内存限制:C/C++ 语言65536KB ;其他语言589824KB

题目描述：
受到小人的设计，主人公暮小云落入一个名叫神凛幻域的奇特地方。对于迷失在这里的人而言这个
空间没有绝对的方向，想要脱离这个地方就必须向前走出n步。由于在这个空间内没有方向的概
念，无论何时向任何方向迈出一步都是等效的(哪怕你是原地转圈，只要走出N步即可脱离幻
境)。不过，由于空间壁垒的原因，不同时刻向不同方向走所耗费的体力不同。现在已知不同时刻
向某个方向跨出一步所需要耗费的体力，请你告诉暮小云怎么走最省体力，以及需要耗费的最小体
力。

输入：
有多个输入样例，输入的第一行是样例个数T(1<= T <= 50)。每个样例的第一行是一个整数n(1
< = n < = 100000)。紧接着是四行，依次表示东南西北四个方向的体力耗费情况，每行n个数字，
分别表示第n步向该方向走需要花费的体力值xi(0 <= xi <= 1000000)。某一步的多个方向体力值
均为最小值时按照东南西北的顺序取优先方向。

输出：
第一行输出需要的最小体力值。第二行输出行走方案分别用符号ESWN表示东南西北。

## 我的解决

直接上代码吧。

```java
package com.chain.blog.test.day04;

import java.util.Scanner;

public class MainTest {

	public static void main(String[] args) {
		test1();
	}

	private static void test1() {
		Scanner in = null;
		try {
			in = new Scanner(System.in);
			int n = in.nextInt();
			for (int i = 0; i < n; i++) {
				process(in);
			}
		} finally {
			in.close();
		}

	}

	private static final int E = 0;
	private static final int S = 1;
	private static final int W = 2;
	private static final int N = 3;

	private static void process(Scanner in) {
		int n = in.nextInt();
		int[][] t = new int[n][4];

		// 矩阵的转置
		for (int i = 0; i < 4; i++) {
			for (int j = 0; j < n; j++) {
				t[j][i] = in.nextInt();
			}
		}

		long cost = 0;
		StringBuffer sb = new StringBuffer();
		// 找出每行的最小值的下标的模值
		for (int i = 0; i < n; i++) {
			int min = 0;
			for (int j = 0; j < 4; j++) {
				if (t[i][j] < t[i][min])
					min = j;
			}
			int r = min % 4;
			cost += t[i][min];
			switch (r) {
			case E:
				sb.append("E");
				break;
			case S:
				sb.append("S");
				break;
			case W:
				sb.append("W");
				break;
			case N:
				sb.append("N");
				break;
			}
		}
		System.out.println(cost);
		System.out.println(sb.toString());
	}
}
```

## 测试结果

> 输入：
1
4
1 10 100 1000
100 10 1000 1
10 10 10 10
50 5 15 55

输出：
17
ENWS

## 其他答案

可以参考牛客网上的解答。