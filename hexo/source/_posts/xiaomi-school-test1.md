---
title: 小米校招题-名称转换
date: 2017/09/19
categories:
- 学习
tags:
- 学习
- Java
- 求职
---

小米校招题-名称转换
=======================
昨天尝试做了小米的在线校招题，发现使用的平台是[赛码网](http://www.acmcoder.com/index)，学校一直没有什么宣传，自己也是第一次知道这个网站。网站是很好的平台，很多公司都在上面举行在线笔试，自己也可以用来练习算法和结构，类似leetcode一样。

这次参加的是小米的服务器开发的在线笔试，题目主要是考察算法和数据结构，也是自己的短板啊。跌跌撞撞的写了一道编程题，__名称转换__。字符串操作其实是比较繁琐的，不像一些图论等有基本的章法，字符串题目而言，读懂题意是最关键的。

## 考试题目
题目比较长，自己记得也比较模糊了，写个大概吧。

给定名称，将名称转为C/C++中的宏定义的格式，比如：
```java
> myHeader => _MY_HEADER_
```

类名中包含“.”作为分隔符，比如
```java
> my.ABCAbc => _MY_ABC_ABC_
```

类名中只会包含大小写字母和数字，且不以数字开头。

转换规则：
1、在开头和结尾都添加下划线
2、将“.”转换为下划线
3、使用下划线将单词分割，具体如下：
  1）第一个大写字母和后面的小写字母组成一个单词，如果不以大写字母开头，则连续的小写字母视为一个单词。
  2）连续的大写字母视为一个单词。如果最后一个大写字母后面跟着小写字母，则最后一个大写字母不包含在内。
  3） 连续的数字视为一个单词。

## 我的解决
我的解决方案比较繁琐，也没有太多考虑性能和简洁性，用C语言直接写过程会更简练。

大致思路就是两个指针，分别指向last，current，使用substring方法一段一段的截取到List中，然后用"_"连接起来。

直接贴上代码：
```java
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

/**
 * 第一题：转为下划线且大写格式
 * 
 * @author Chain
 *
 */
public class Main {

	public static void main(String args[]) {
		test1();
		// test2();
	}

	private static void test2() {
		Scanner in = new Scanner(System.in);
		String str = in.nextLine();
		in.close();
		int len = str.length();
		List<String> list = new ArrayList<>();
		process(list, str, len);
		String con = concat(list);
		List<String> res = new ArrayList<>();
		if (con != null)
			res.add(con);
		printResList(res);
	}

	private static void test1() {
		Scanner in = new Scanner(System.in);
		while (in.hasNext()) {
			String str = in.nextLine();
			int len = str.length();
			if (len < 1)
				continue;
			List<String> list = new ArrayList<>();
			process(list, str, len);
			String con = concat(list);
			System.out.println(con);
		}
		in.close();
	}

	private static void printResList(List<String> res) {
		if (res == null || res.size() < 1)
			return;

		for (String s : res) {
			System.out.println(s);
		}
	}

	private static final String CONCAT = "_";

	private static String concat(List<String> list) {
		StringBuffer sb = new StringBuffer();
		sb.append(CONCAT);

		if (list != null) {
			int size = list.size();
			for (int i = 0; i < size; i++) {
				String str = list.get(i);
				str = str.replaceAll("[^a-zA-Z0-9]", "");
				str = str.toUpperCase();
				if (str.length() > 0) {
					sb.append(str);
					sb.append(CONCAT);
				}
			}
		}

		return sb.toString();
	}

	private static int current = 0;
	private static int last = current;

	private static void process(List<String> list, String str, int len) {
		if (list == null)
			return;

		current = 0;
		last = current;

		while (current < len) {
			char[] chs = str.toCharArray();
			findJumpPoint(chs, len);
			String part = getPart(str);
			list.add(part);
			str = str.substring(current);
			len = str.length();
			current = 0;
			last = current;
			// System.out.println(str);
		}

	}

	private static String getPart(String str) {
		String subStr = str.substring(last, current);
		last = current;
		return subStr;
	}

	private static void findJumpPoint(char[] chs, int len) {
		while ((++current) < len) {
			int before = chs[current - 1];
			int now = chs[current];
			int typeBefore = getType(before);
			int typeNow = getType(now);
			if (typeBefore != typeNow) {
				int res = getCutPoint(chs, typeBefore, typeNow);
				if (res != -1) {
					current = res;
					break;
				}
			}
		}
	}

	private static int getCutPoint(char[] chs, int typeBefore, int typeNow) {
		if (typeNow == LOWER_CASE) {
			if (typeBefore == LOWER_CASE) {

			} else if (typeBefore == UPPER_CASE) {
				if (current - 2 > -1) {
					int beforeBefore = chs[current - 2];
					if (getType(beforeBefore) == UPPER_CASE)
						return current - 1;
				}
				return -1;
			} else if (typeBefore == SPECIAL) {

			}
		} else if (typeNow == UPPER_CASE) {
			if (typeBefore == LOWER_CASE) {

			} else if (typeBefore == UPPER_CASE) {

			} else if (typeBefore == SPECIAL) {

			}
		} else if (typeNow == SPECIAL) {
			if (typeBefore == LOWER_CASE) {

			} else if (typeBefore == UPPER_CASE) {

			} else if (typeBefore == SPECIAL) {

			}
		}
		return current;
	}

	private static final int LOWER_CASE = 0;
	private static final int UPPER_CASE = 1;
	private static final int SPECIAL = 2;

	private static int getType(int p) {
		if (p > 64 && p < 91)
			return UPPER_CASE;
		else if (p > 96 && p < 123)
			return LOWER_CASE;
		return SPECIAL;
	}
}
```

## 测试结果

输入：
```java
> a
my.ABC
my.Abc
My.ABCAbc
MY.ASTParser12
My.ABCAbcABCAbcABC
foo.bar12bar13A
```

输出：
```java
> _A_
_MY_ABC_
_MY_ABC_
_MY_ABC_ABC_
_MY_AST_PARSER_12_
_MY_ABC_ABC_ABC_ABC_ABC_
_FOO_BAR_12_BAR_13_A_
```

## 其他答案

今天又发现赛码网上大牛们的[解答](http://discuss.acmcoder.com/topic/59bfd250b855ca0b3d48b386)，真的是厉害。