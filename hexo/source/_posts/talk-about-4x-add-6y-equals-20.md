---
title: 简单的题目，编程的思想
date: 2017/10/10
categories:
- 学习
tags:
- 学习
- Java
---

简单的题目，编程的思想
=======================
实验室需要带萌新，简单帮学弟入门编程，自己也是在大学的最后一年了。

出了一道简单的编程题：

> 求满足4x+6y=20的所有非负整数解。

自己当时也没想太多，纯粹是让练练手，不过大家提交的答案却能投射出编程的一些思想。

## 做法一

循环执行次数：21*21=441。

这个也是最直观的做法。

```java
private static void test1() {
    for (int i = 0; i <= 20; i++) {
        for (int j = 0; j <= 20; j++) {
            if (4 * i + 6 * j == 20)
                System.out.println(i + " " + j);
        }
    }
}
```

## 做法二

利用2层while循环，在执行过程判断x,y的和与20的关系。

循环执行次数：5+4+4+3+2+2=20。

```java
private static void test2() {
    int x = 0;
    int y = 0;
    int z = 20;
    while (true) {
        y = 0;
        while (true) {
            int sum = 4 * x + 6 * y;
            if (sum == z) {
                System.out.println(x + " " + y);
            } else if (sum > z) {
                break;
            }
            y++;
        }
        x++;
        if (4 * x > z)
            break;
    }
}
```

## 做法三

较做法1缩小了范围。

化简4x+6y=20为2x+3y=10。

循环执行次数：4*6=24。

```java
private static void test3() {
    for (int y = 0; y <= 3; y++) {
        for (int x = 0; x <= 5; x++) {
            int sum = 4 * x + 6 * y;
            if (sum == 20)
                System.out.println(x + " " + y);
        }
    }
}
```

## 做法四

是做法三的改进，一个简单的数学不等式，将一个未知数用另一个未知数表示。

循环执行次数：14。

```java
private static void test4() {
    for (int y = 0; y <= 3; y++) {
        for (int x = 0; x <= (10 - 3 * y) / 2; x++) {
            int sum = 4 * x + 6 * y;
            if (sum == 20)
                System.out.println(x + " " + y);
        }
    }
}
```

## 做法五

利用做法四的数学关系，进一步优化。

循环执行次数：4。

```java
private static void test5() {
    for (int y = 0; y <= 3; y++) {
        int t = 10 - 3 * y;
        int x = t / 2;
        int n = t % 2;
        if (n == 0 && x >= 0 && x <= 5)
            System.out.println(x + " " + y);
    }
}
```

## 总结

简单的题目，详细探究还是能体现编程的一些思想的。

接下来，在一起讨论下，总结出了以下的通用做法，还是很不错的哈。

求解二元一次方程的通用做法：

```java
private static void test6(int x, int y, int z) {
    // 先进行优化，化简
    {
        int t = gcd(x, y);
        t = gcd(t, z);
        if (t != 1) {
            x = x / t;
            y = y / t;
            z = z / t;
        }
    }

    final int m = max(x, y);
    final int n = min(x, y);

    final int p = z / m;
    final int q = z / n;
    for (int i = 0; i <= p; i++) {
        int e = z - m * i;
        int u = e / n;
        int r = e % n;
        if (r == 0 && u >= 0 && u <= q) {
            if (m == x) {
                System.out.println("x=" + i + ",y=" + u);
            } else {
                System.out.println("x=" + u + ",y=" + i);
            }
        }
    }
}

// 循环法求最大公约数
private static int gcd(final int i, final int j) {
    int m = min(i, j);
    int n = max(i, j);
    int t = 0;
    while (true) {
        if (m == 0)
            return n;
        t = m;
        m = n % m;
        n = t;
    }
}

private static int min(final int m, final int n) {
    return m > n ? n : m;
}

private static int max(final int m, final int n) {
    return m > n ? m : n;
}
```