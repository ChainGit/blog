---
title: Java中的进制转换和ScaleConvertUtils
date: 2016/12/28
categories:
- 学习
tags:
- 学习
- Java
- 知识点
---

Java中的进制转换和ScaleConvertUtils
==================================
复习Java基础时，学到了进制转换部分，就想着自己搞了一个进制转换的工具类，当然工具类还有不完善的地方。

## 工具简介
进制转换在JavaAPI中已经封装，但是自己实现还是有很多要学习的地方，涉及到无符号数的表示、负数的转换、浮点数在计算机中的表示、位操作等。
本工具也支持多进制之间的转换，进制与字符串之间的互转。

下面贴出简单测试代码：
```java
@Test
public void test() throws Exception {
    int a = -10;
    String s1 = ScaleConvertUtils.toBinaryString(a);
    System.out.println(s1);
    int a1 = ScaleConvertUtils.parseBinaryString(s1);
    System.out.println(a1);

    float b = -12.34f;
    String s2 = ScaleConvertUtils.toBinaryString(b);
    System.out.println(s2);
    float b1 = ScaleConvertUtils.parseBinaryStringToFloat(s2);
    System.out.println(b1);
}
```

输出：
> 11111111111111111111111111110110
-10
11000001010001010111000010100100
-12.34

下载地址：点击[这里下载jar包](/uploads/number-scale-convert-utils/ChainUtils.jar)，源码在[这里](https://github.com/ChainGit/some-tests/blob/master/test04/ScaleConvertUtils.java)。

> 完