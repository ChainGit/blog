---
title: 小米校招题-密码破解
date: 2017/09/19
categories:
- 学习
tags:
- 学习
- Java
- 求职
---

小米校招题-密码破解
=======================
这个是小米的校招题第三题，题目是密码破解。考试时间不足，没做出来，唉。结束后想了一个解决方法，当然也不是最好的解法，不过问题至少解决了。

## 考试题目
大致如此：

已知一个字符串，包含数字。有如下对应规则：
```java
1 -> a
2 -> b
...
26 -> z
```
现在需要求出一个字符串的所有破译的可能结果。

## 我的解决
大致思路其实是递归。

```java
package com.chain.blog.test.day03;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

/**
 * 第三题：密码破译
 * 
 * @author Chain
 *
 */
public class Main {

	public static void main(String[] args) {
		test1();
	}

	private static void test1() {
		Scanner in = null;
		try {
			in = new Scanner(System.in);
			while (in.hasNext()) {
				String str = in.nextLine();
				List<String> list = new ArrayList<>();
				process(list, str);
				printList(list);
			}
		} finally {
			if (in != null)
				in.close();
		}
	}

	private static void process(List<String> list, String str) {
		StringBuffer sb = new StringBuffer();
		bfs(list, str, sb);
	}

	private static void bfs(List<String> list, String str, StringBuffer sb) {
		if (str.isEmpty()) {
			list.add(sb.toString());
			sb.delete(0, sb.length());
			return;
		}

		char[] chs = str.toCharArray();
		int first = chs[0];
		sb.append(intToChar(first));
		String sback = sb.toString();

		if (str.length() > 1) {
			bfs(list, str.substring(1), sb);
		} else {
			bfs(list, EMPTY, sb);
		}

		if (str.length() < 2)
			return;

		sb = new StringBuffer(sback);
		int second = chs[1];
		int len = sb.length();
		sb.deleteCharAt(len - 1);
		char ch = intToChar(first, second);
		if (ch == NO)
			return;
		sb.append(ch);

		if (str.length() > 2) {
			bfs(list, str.substring(2), sb);
		} else {
			bfs(list, EMPTY, sb);
		}

	}

	private static final String EMPTY = "";
	private static final char NO = '#';

	private static char intToChar(int i, int j) {
		i = i - '0';
		j = j - '0';
		int k = i * 10 + j;
		if (k < 1 || k > 26)
			return NO;
		k = k - 1 + 'a';
		return (char) k;
	}

	private static char intToChar(int i) {
		return (char) (i - '0' - 1 + 'a');
	}

	private static void printList(List<String> list) {
		int size = list.size();
		for (int i = 0; i < size; i++) {
			System.out.print(list.get(i));
			if (i < size - 1) {
				System.out.print(" ");
			}
		}
		System.out.println();
	}

}
```

## 测试结果

> 输入：
1
12
123
1234
12345


> 输出：
a
ab l
abc aw lc
abcd awd lcd
abcde awde lcde


## 其他答案
赛码网上大牛们的[解答](http://discuss.acmcoder.com/topic/59bfd250b855ca0b3d48b386)。

