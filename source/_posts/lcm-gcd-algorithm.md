---
title: 求最小公倍数和最大公约数
date: 2016/07/08
categories:
- 算法
tags:
- 算法
- Java
---

求最小公倍数和最大公约数
===================

这个是算法中的基础题目了，在此记录一下。


-------
## 公共函数
求两数最小：
```Java
public static int min(int i, int j) {
	return i > j ? j : i;
}
```
求两数最大：
```Java
public static int max(int i, int j) {
	return i > j ? i : j;
}
```
## 最大公约数
### 暴力求解法
暴力法求解：
从两个数中的最小的一个开始，逐次递减，每次都进行判断是不是这两个数的公约数。若是，则直接返回，此时返回的即为最大公约数。
补充：自然数1是任意两个正整数的约数。
```Java
public static int zdgys1(int i, int j) {
	for (int p = min(i, j); p > -1; p--)
		if (i % p == 0 && j % p == 0)
			return p;
	return 1;
}
```
### 辗转相除法
1、辗转相除法(欧几里得算法) - 1
辗转相除法又称为欧几里得算法，这里是用循环的做法。
```Java
public static int zdgys2(int i, int j) {
	int m = min(i, j);
	int n = max(i, j);
	while (true) {
		if (m == 0)
			return n;
		int t = m;
		m = n % m;
		n = t;
	}
}
```
2、辗转相除法(欧几里得算法) - 2
这里是用递归的做法。
```Java
public static int zdgys3(int i, int j) {
	int m = min(i, j);
	int n = max(i, j);
	return gcd(m, n);
}
	
private static int gcd(int m, int n) {
	if (m == 0)
		return n;
	return gcd(n % m, m);
}
```
## 最小公倍数
### 暴力求解法
做法和求最大公约数类似，通过循环加判断。
```Java
public static int zxgbs2(int i, int j) {
	int m = max(i, j);
	for (; m < i * j; m++) {
		if (m % i == 0 && m % j == 0)
			return m;
	}
	return m;
}
```
### 使用最大公约数
使用公式：lcm = ( i * j / gcd( i , j ) )
```Java
public static long zxgbs2(int i, int j) {
	return i * j / zdgys2(i, j);
}
```
## 方法测试

```Java

public static void start() {
	cut = System.currentTimeMillis();
}

public static void end() {
	System.out.println("spend: " + (System.currentTimeMillis() - cut));
}

private static long cut = 0l;

// 最大公约数、最小公倍数
public static void main(String[] args) {
	start();
	System.out.println(zdgys1(1123123124, 21123324));
	end();

	start();
	System.out.println(zdgys1(7, 4));
	end();

	start();
	System.out.println(zdgys2(1123123124, 21123324));
	end();

	start();
	System.out.println(zdgys2(7, 4));
	end();

	start();
	System.out.println(zdgys3(15, 40));
	end();

	start();
	System.out.println(zxgbs1(5, 8));
	end();
	
	start();
	System.out.println(zxgbs2(5, 8));
	end();
	
	start();
	System.out.println(qmqm(5, 8, 17));
	end();
}
```
> 测试结果
4
spend: 80
1
spend: 0
4
spend: 0
1
spend: 1
5
spend: 1
40
spend: 1
40
spend: 1


> 完


