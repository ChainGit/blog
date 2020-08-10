---
title: 利用Hash进行大文本文件的行搜索
date: 2017/12/14
categories: 学习
tags:
- Java
- 学习
---

利用Hash进行大文本文件的行搜索
============================
思考一个问题，如何在大文件中搜索一个指定的数据？这也是一道经常被问的面试题，不过搜索了一下，网上一般只提供了思路但是并没有什么实际的代码和案例。现在想着自己能够实现一下，并写下了这篇博客，水平有限，如有不正确的地方欢迎指正。

## 问题描述

现在有两个按行存储的文本文件a.txt和b.txt，已知a.txt内有url共50亿条，b.txt内有url共500条。请问，b中的url在a中是否出现，如果出现，依次输出该url在b中的行号。

## 问题分析

如果采用顺序搜索，对于小数据是可以的，但是对于大文本文件，那个需要的时间可想而知。虽然将数据读入内存后，会加快搜索的速度，但是计算机的内存容量一般远小于磁盘容量，如果将一个2G大小的文件读入内存，那个代价也是承受不起的。其他常见的搜索方法，如二分搜索法，二分搜索法的前提是数据是有序的，而对一个大文本文件排序也是不现实的，所以在这里也不适用。

因此必须采用其他的方法来搜索数据。我这里采用Hash查找做了一个自己觉得还可以的实现。

## Hash查找

常见的Hash算法有很多，这里我采用的是JavaAPI中String.hashCode()方法和取模Hash法结合使用。

### 取模Hash法

任意一个整数a，如果对它进行取模运算，比如a % N，那么取模的结果必然在 0 ~ N-1 之间。
这样对于一个大的随机整数集Q，如果把它们依次进行取模运算，那么这个整数集Q就可以近似被分成 N 组。（如果随机整数集Q足够随机，每个组的元素数量也是近似相等的）
这样的话如果要查找某个数b，那么先将它进行取模运算 b % N，得到的余数p就自然可以被映射到那个已经分好N组的某个组G中，换句话说，b如果在Q中存在，那么必然也只能是在G中。此时再利用线性搜索对这个分组G进行查找的话，由于此时只有Q/N个元素，速度自然会快了不少。

由上可见，N的取值也是比较关键的。

### String.hashCode()

String类重写了hashCode()方法：

```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        hash = h = isLatin1() ? StringLatin1.hashCode(value)
                                : StringUTF16.hashCode(value);
    }
    return h;
}
```

JavaAPI文档的描述是这样的：

```
Returns a hash code for this string. The hash code for a String object is computed as 

s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
 
using int arithmetic, where s[i] is the ith character of the string, n is the length of the string, and ^ indicates exponentiation. (The hash value of the empty string is zero.)
```

这是Java中默认的计算String的hash值的方法，将字符串转为一个整数。

结合这两个计算Hash值的方法，就可以解决上面的问题。

## 我的解决

目标文件均为__按行存储的文本文件__。

第一次采用的尝试实现的方法是：
1、遍历文件a，对每一行数据求hashCode，然后再对hashCode求__N=100__的模，根据求模的结果写入到对应的小文件中，比如h34.txt，并将这一行所在的行号写入到另一个小文件中，比如n34.txt。
2、这样一次遍历后，原来超大的文本文件a可以被分割成100个小文件，原来文本文件中某数据的所在行也可以对应映射记录在100个小文件中。比如原来a中的第1234行的数据，求模的值是78，那么这行数据被添加在h78.txt的末尾，对应的行号也添加在n78.txt末尾。
3、因为只有一个线程在操作，所以不存在多线程安全的问题。数据的映射关系保持正确。
4、输入要查找的字符串数据s，先对s进行取__N=100__模运算，比如得到结果78，然后再读取h78.txt文件，按顺序读取h78.txt文件，并同时记录读取h78.txt的行数L，若在h78.txt找到和s一致的数据时，此时再去n78.txt读取到L行，读出n78.txt的L行数据就是s在原来文件a中的行号。循环下去，直到读完h78.txt为止，此时也相应的得到s在文件a中出现的次数，以及对应的所有行号。

具体实现可以选择InputStream、Reader，对应的实现类BufferInputStream，FileInputStream、BufferedReader，FileReader，LineNumberReader。

第一种方法很好理解，也容易实现。不过需要在磁盘中多出很多额外的小文件，占有更多的磁盘空间。另外频繁的读取磁盘，不仅速度慢，而且也影响了磁盘的寿命。

对应第一种方法中的不足加以改进，第二次采用的尝试实现的方法是：
1、在内存中建立一个Map，Map的键是Integer类型，Map的值是一个List。具体如下：
```java
// Map<Hash值，数据所在行的信息>
private Map<Integer, List<LineInfo>> store;
```
其中LineInfo存储了数据的所在行号、该行的开头在文件中的位置、该行数据的完整hash值。
```java
// 数据所在行的信息
private class LineInfo {
    // 数据所在行号
    private int lineNumber;
    // 该行的开头在文件中的位置
    private long position;
    // 该行数据的完整hash值
    private int hashValue;
}
```
2、遍历文件a，对每一行数据求hashCode，然后再对hashCode求__N=100__的模，得到求模的结果M，将M加入到Map中key=M的List中。
3、查找数据s时和第一种方法类似，不同的是操作的是Map，获得Map中模存储的List，接着再遍历List即可。
4、遍历List时，先判断s的hashCode和List中存储的LineInfo.hashValue是否一致，如果不一致则数据内容肯定不一致，能减少读取磁盘的次数，加快执行速度。然后再根据LineInfo.position定位到数据在文件a中的位置，读取出具体的数据，再使用equals方法进一步判断内容是否一致；由LineInfo.lineNumber可以得到数据在文件a的行号。这样重复操作，就可以解决上述问题。

具体的实现使用RandomAccessFile即可。

不过第二种方法在测试过程中，建立hash表的时间太长，同等文件大小下比第一种方法慢的多。既然都是文件读取操作，时间差别应该不是很大，所以还是需要改进的。

考虑到RandomAccessFile并没有BufferedInputStream和BufferedReader那样的缓存区，而是几乎每次都需要读取磁盘，才读若干行就需要去操作磁盘一次（从任务管理器也可以看的出来），虽然操作系统具有一部分缓存区，但是如果能一次读出一块放在内存中再进行计算，速度岂不是能大大提高。

为此在第二种方法的基础上，又加以改进，得到了第三种方法，即使用RandomAccessFile+byte[]的方式。

速度较慢的构建hash表的方式：
```java
// 慢速
private void build0() {
    store = new HashMap<>();
    try {
        file = new RandomAccessFile(path, "r");
        // 行号从1开始
        int lineNumber = 1;
        long cursor = 0;
        String lineStr = null;
        List<LineInfo> lineInfos = null;
        while ((lineStr = file.readLine()) != null) {
            // 方便find查找
            int realHashValue = lineStr.hashCode();
            int hashCode = Math.abs(realHashValue) & BASE;
            lineInfos = store.get(hashCode);
            if (lineInfos == null) {
                lineInfos = new ArrayList<>();
                store.put(hashCode, lineInfos);
            }
            // 顺序添加，更进一步的意思是按照出现的顺序添加，能简化查找操作
            lineInfos.add(new LineInfo(lineNumber++, cursor, realHashValue));
            cursor = file.getFilePointer();
        }
        System.out.println("build slow-mode has read lines " + (lineNumber - 1));
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```

速度较快的构建hash表的方式：
```java
// 快速
private void build1() {
    store = new HashMap<>();
    try {
        file = new RandomAccessFile(path, "r");
        // 一次读取64M
        int bufLen = 1024 * 1024 * 64;
        byte[] buf = new byte[bufLen];
        // 注意使用long
        long times = 0;
        int readLen = -1;
        // 行号从1开始
        int lineNumber = 1;
        // 回退的位数
        long back = 0;
        while ((readLen = file.read(buf)) != -1) {
            times++;
            int cursor = 0;
            while (cursor < readLen) {
                int start = cursor;
                int offset = 0;
                int left = readLen - start;
                while (offset < left) {
                    // 数据文件中的每一行数据的末尾必须要有'\n'
                    if (buf[start + offset] == '\n')
                        break;
                    offset++;
                    cursor++;
                }
                if (offset == left) {
                    back += left;
                    break;
                } else {
                    if (offset > 0 && buf[start + offset - 1] == '\n')
                        offset--;
                    if (offset > 0 && buf[start + offset - 1] == '\r')
                        offset--;
                    String lineStr = new String(buf, start, offset);
                    // 方便find查找
                    int realHashValue = lineStr.hashCode();
                    // 与方式取模，速度更快，前提是BASE必须是2^n-1
                    int hashCode = Math.abs(realHashValue) & BASE;
                    List<LineInfo> lineInfos = store.get(hashCode);
                    if (lineInfos == null) {
                        lineInfos = new ArrayList<>();
                        store.put(hashCode, lineInfos);
                    }
                    // 顺序添加，更进一步的意思是按照出现的顺序添加，能简化查找操作
                    lineInfos.add(new LineInfo(lineNumber++, (times - 1) * bufLen - back + start, realHashValue));
                }
                cursor++;
            }
            file.seek(times * bufLen - back);
        }
        System.out.println("build fast-mode has read lines " + (lineNumber - 1));
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```

查找数据的方法：
```java
/**
    * 查找
    * 
    * @param search
    * @return
    */
public List<Integer> find(String search) {
    if (search == null)
        throw new RuntimeException("string is null");
    if (store == null)
        throw new RuntimeException("has not built yet");
    int searchHashValue = search.hashCode();
    int hashCode = Math.abs(searchHashValue) & BASE;
    List<LineInfo> lineInfos = store.get(hashCode);
    if (lineInfos == null)
        return null;
    List<Integer> lines = new ArrayList<>();
    try {
        String lineStr = null;
        for (LineInfo line : lineInfos) {
            // hash不一样，内容肯定不一样，能加快判断的速度
            if (searchHashValue != line.hashValue)
                continue;
            file.seek(line.position);
            lineStr = file.readLine();
            // 进一步判断内容
            if (search.equals(lineStr))
                // 行号从1开始
                lines.add(line.lineNumber);
        }
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
    return lines.size() > 0 ? lines : null;
}
```

## 测试结果

如果数据文件中的行数为__10万__，需要查找的数据为__10__条时，执行的结果如下：

```java
====== HASH SEARCH ======
START MAKING
MAKING COST 139

FAST MODE
START BUILDING
BUILDING COST 177

START FINDING
1 http://url.cn/
FIND: 6330
EXAMPLE: 1 5 10 20 22 
FINDING STEP COST 201
49 http://url.cn/Ilq
FIND: 1
EXAMPLE: 49 
FINDING STEP COST 1
19 http://url.cn/r0pX8W4I6
FIND: 1
EXAMPLE: 19 
FINDING STEP COST 0
72 http://url.cn/1mhXGMclLYQk
FIND: 1
EXAMPLE: 72 
FINDING STEP COST 0
120 http://url.cn/J
FIND: 101
EXAMPLE: 120 1778 1798 2787 3335 
FINDING STEP COST 4
58 http://url.cn/ThZfMpoMVhLw
FIND: 1
EXAMPLE: 58 
FINDING STEP COST 0
11 http://url.cn/C0o3lk4FyAOewkP
FIND: 1
EXAMPLE: 11 
FINDING STEP COST 0
91 http://url.cn/NstBXfBXEEJD3
FIND: 1
EXAMPLE: 91 
FINDING STEP COST 0
45 http://url.cn/yx
FIND: 2
EXAMPLE: 45 94836 
FINDING STEP COST 1
93 http://url.cn/A9mo
FIND: 1
EXAMPLE: 93 
FINDING STEP COST 0
FINDING TOTAL COST 207

SLOW MODE
START BUILDING
BUILDING COST 4329

START FINDING
1 http://url.cn/
FIND: 6330
EXAMPLE: 1 5 10 20 22 
FINDING STEP COST 191
49 http://url.cn/Ilq
FIND: 1
EXAMPLE: 49 
FINDING STEP COST 0
19 http://url.cn/r0pX8W4I6
FIND: 1
EXAMPLE: 19 
FINDING STEP COST 0
72 http://url.cn/1mhXGMclLYQk
FIND: 1
EXAMPLE: 72 
FINDING STEP COST 0
120 http://url.cn/J
FIND: 101
EXAMPLE: 120 1778 1798 2787 3335 
FINDING STEP COST 5
58 http://url.cn/ThZfMpoMVhLw
FIND: 1
EXAMPLE: 58 
FINDING STEP COST 0
11 http://url.cn/C0o3lk4FyAOewkP
FIND: 1
EXAMPLE: 11 
FINDING STEP COST 0
91 http://url.cn/NstBXfBXEEJD3
FIND: 1
EXAMPLE: 91 
FINDING STEP COST 0
45 http://url.cn/yx
FIND: 2
EXAMPLE: 45 94836 
FINDING STEP COST 0
93 http://url.cn/A9mo
FIND: 1
EXAMPLE: 93 
FINDING STEP COST 0
FINDING TOTAL COST 198

true
```

由测试结果可知：
1、慢速和快速构建hash表的结果一致(true)。
2、慢表的构建表的速度为4329毫秒，快表的构建表速度是177毫秒。
3、使用Hash表进行文件的查找速度非常快，不用测试都能看出比顺序查找快很多。当数据重复行较多时，操作的主要耗时在读取行并判断equals上，这里还是可以再改进的。

接下来结合上面的内容再测试几组更大的数据规模，并使用Notepad++软件搜索计数功能手动验证部分结果，得到下表：

![image](/uploads/0-common-images/0.jpg)

结果显示使用快速构建Hash表查找的方式的效果还是很不错的，实验的目的基本达成。

__2017年12月22日补充：__

如果将store改成双层hash，如下：
```java
// Map<Hash值，<某行数据的hash值,数据所在行的信息>>
private Map<Integer, Map<Integer, List<LineInfo>>> store;
```
虽然理论上查找速度会略微有提升，但是因为第二层Map有很多长度只有1的List，因此内存占用增大，CPU占用率也会大幅增加，查找速度反而又会下降。所以一层hash的效果更好一些。
不过也应该有其他更好的优化方式，等遇到再说吧。

![image](/uploads/0-common-images/16.jpg)

## 总结思考

在解决这个问题的时候，也简单搜索了一下hash的具体应用，发现hash的用处广范而强大，比如分布式集群控制系统使用hash来实现，真的是简单的东西深挖下去不简单。

多动手，多思考。

[源码](https://github.com/ChainGit/some-tests/tree/master/test11)