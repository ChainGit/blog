---
title: 过桥问题和倒水问题
date: 2016/12/19
categories: 
- 算法
tags: 
- 算法
- Java
- 转载
---

过桥问题和倒水问题
===================
过桥问题和倒水问题都是笔试面试中的热门智力题，不但微软、GOOGLE、百度、腾讯等公司采用，甚至在IQ测试与公务员考试中都能见到。

在此整理下具体的做法。

## 过桥问题

> 在漆黑的夜里，四位旅行者来到了一座狭窄而且没有护栏的桥边。如果不借助手电筒的话，大家是无论如何也不敢过桥去的。不幸的是，四个人一共只带了一只手电筒，而桥窄得只够让__两个人同时通过__。如果各自单独过桥的话，四人所需要的时间分别是1，2，5，8分钟；而如果两人同时过桥，所需要的时间就是走得比较慢的那个人单独行动时所需的时间。问题是，你如何设计一个方案，让用的时间最少。

这个题目被微软、GOOGLE、百度、华硕、建设银行等很多公司用作笔试题或面试题。当然也有用在IQ测试中。

解答其实也容易，关键就是在于__能者多劳——用时短的人必须要多跑几趟以便传递手电筒__。

设这四个人叫做A，B，C，D，他们所需要的时间分别是1，2，5，8分钟。

更加细化的解决方案是：
1、要么是最快者将最慢的送过桥，
2、要么是最快的2个将最慢的2个送过桥。

即将过桥的人按其过桥的时间从小到大排列，设为A，B，…… Y，Z。其中A和B是最快的二个，Y和Z是最慢的二个。

那么就有二种方案：

__方案一 最快者将最慢者送过桥__

第一步：A和Z过桥，花费Z分钟。

第二步：如果桥那边还有人，则A回来，花费A分钟，没有人则不用回来。

这两步后总人数就减小1个，花费时间为A + A + Z分钟。

__方案二 最快的2个将最慢的2个送过桥__

第一步：A和B过桥，花费B分钟。

第二步：A回来，花费A分钟。

第三步：Y和Z过桥，花费Z分钟。

第四步：B回来，花费B分钟。

这四步后总人数同样减小2个，花费时间为A + B + B + Z分钟。

之后比较这两个方法的时间，找到时间消耗最小就行。

接下来就是代码时间：

假设一次过桥的人数最多是2人。

```java
package com.chain.blog.test.day09;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Scanner;

/**
 * 过桥问题：遵循“能者多劳”的准则
 * 
 * @author Chain
 *
 */
public class CrossBridgeTest {

	public static void main(String[] args) {
		test1();
	}

	// 两种方案分别最短耗时
	private static int cost1 = 0;
	private static int cost2 = 0;

	// 两种方案各自的具体实施方案
	private static List<String> list1 = new ArrayList<>();
	private static List<String> list2 = new ArrayList<>();

	private static void test1() {
		try (Scanner in = new Scanner(System.in)) {
			int n = in.nextInt();
			List<Integer> speed1 = new ArrayList<>();
			for (int i = 0; i < n; i++) {
				int s = in.nextInt();
				speed1.add(s);
			}
			Collections.sort(speed1);
			List<Integer> speed2 = new ArrayList<>(speed1);
			process1(speed1, 0);
			process2(speed2, 0);
			print();
		}
	}

	private static void print() {
		System.out.println("方案一耗时: " + cost1);
		System.out.println("方案一具体实施: ");
		for (String s : list1) {
			System.out.println(s);
		}

		System.out.println();

		System.out.println("方案二耗时: " + cost2);
		System.out.println("方案二具体实施: ");
		for (String s : list2) {
			System.out.println(s);
		}
	}

	// 方案二
	private static void process2(List<Integer> t, int times) {
		int size = t.size();

		// 如果人数少于4个人，使用方案一
		if (size < 4) {
			cost2 = process1(t, times, list2, cost2);
			return;
		}

		list2.add("第" + (times + 1) + "次循环：");

		int fast1 = 0;
		int fastSpeed1 = t.get(fast1);
		int fast2 = fast1 + 1;
		int fastSpeed2 = t.get(fast2);
		int slow1 = size - 1;
		int slowSpeed1 = t.get(slow1);
		int slow2 = slow1 - 1;
		int slowSpeed2 = t.get(slow2);

		list2.add("第" + (fast1 + 1) + "个人\t和\t第" + (fast2 + 1) + "个人\t过河");
		cost2 += fastSpeed2;
		list2.add("第" + (fast1 + 1) + "个人\t回头");
		cost2 += fastSpeed1;
		list2.add("第" + (slow1 + 1) + "个人\t和\t第" + (slow2 + 1) + "个人\t过河");
		cost2 += slowSpeed1;
		list2.add("第" + (fast2 + 1) + "个人\t回头");
		cost2 += fastSpeed2;

		t.remove(slow1);
		t.remove(slow2);

		process2(t, times + 1);
	}

	private static void process1(List<Integer> t, int times) {
		cost1 = process1(t, times, list1, cost1);
	}

	// 方案一
	private static int process1(List<Integer> t, int times, List<String> list, int cost) {
		list.add("第" + (times + 1) + "次循环：");

		int size = t.size();
		// 列表为空时直接返回
		if (size < 1) {
			list.add("过河人数为0");
			return cost;
		}

		int fast = 0;
		int fastSpeed = t.get(fast);

		// 如果只有一个人，直接过河就行
		if (size < 2) {
			list.add("第" + (fast + 1) + "个人\t单独\t过河");
			cost += fastSpeed;
			return cost;
		}

		// 最快者将最慢者送过桥
		int slow = size - 1;
		int slowSpeed = t.get(slow);

		list.add("第" + (fast + 1) + "个人\t和\t第" + (slow + 1) + "个人\t过河");
		cost += slowSpeed;
		t.remove(slow);
		size = t.size();
		// 如果桥那边还有人
		if (size > 1) {
			list.add("第" + (fast + 1) + "个人\t回头");
			cost += fastSpeed;
			cost = process1(t, times + 1, list, cost);
		}
		return cost;
	}

}
```

测试结果：

```java
4
1 2 5 10
方案一耗时: 19
方案一具体实施: 
第1次循环：
第1个人	和	第4个人	过河
第1个人	回头
第2次循环：
第1个人	和	第3个人	过河
第1个人	回头
第3次循环：
第1个人	和	第2个人	过河

方案二耗时: 17
方案二具体实施: 
第1次循环：
第1个人	和	第2个人	过河
第1个人	回头
第4个人	和	第3个人	过河
第2个人	回头
第2次循环：
第1个人	和	第2个人	过河
```

## 倒水问题
这个题目的版本非常多，有微软版的，腾讯版的，新浪版的等等。

> 给你一个容量为5升的桶和一个容量为3升的桶，水不限使用，要求精确得到4升水。

解法肯定有很多，可以用__宽度优先搜索（BFS），也可以用穷举法__。

__穷举法__实现比较方便，其基本思想是用：__用小桶容量的倍数对大桶的容量进行取余__。

比如3升的桶和5升的桶得到4升水可以这样做：

3 % 5 = 3

6 % 5 = 1

9 % 5 = 4

成功得到4升水。（PS：上面的过程用如何用文字描述了？）

同样，用7升的桶和11升的桶得到2升水可以这样做：

7 % 11 = 7

14 % 11 = 3

21 % 11 = 10

28 % 11 = 6

35 % 11 = 2

成功得到2升水。

但是也需要注意下的就是可能存在无解的情况。

比如用2升的桶和4升的桶得到3升水。因为2和4的最大公约数是2即GCD(2，4)=2，而2无法整除3。

倒水问题也也容易推广到多个桶的情况，分析方法和上文差不多，这里就不再赘述了。