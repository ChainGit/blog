---
title: 常见的数组排序算法
date: 2016/03/28
categories: 
- 算法
tags: 
- 算法
- Java
---

常见的数组排序算法(Java版)
===================
在一些项目中经常会使用排序算法，Java的API中也有各种已经提供好的排序算法，比如TreeSet的sort方法。在会使用这些即有算法的同时，自己也加以学习整理出常见的排序算法（均为升序排序）。
> 常见的排序算法：
插入排序，选择排序，冒泡排序，希尔排序，快速排序，交换元素等

推荐一个常见算法的动态演示网站：https://visualgo.net/

文中插图来自网络。

----------

## 常见的排序算法
### 交换元素
在写排序算法之前，先说一下交换元素的方法。交换数组中两个元素主要有三种做法：中间变量法，异或法，加减法。其中，异或法和加减法更适用于整型（包括长整型），中间变量法则是万能交换元素的方法。
```Java
/**
	 * 异或方式交换元素，没有中间变量，不会增加数值长度
	 * 
	 * @param n
	 *            要交换元素的数组
	 * @param i
	 *            第一个交换元素的数组下标
	 * @param j
	 *            第二个交换元素的数组小标
	 */
	public static void swap(int[] n, int i, int j) {
		checkSwapArgs(n, i, j);
		n[i] ^= n[j];
		n[j] ^= n[i];
		n[i] ^= n[j];
	}

	/**
	 * 
	 * 加减方式交换元素，注意可能带来的数据溢出和符号位问题
	 * 
	 * @param n
	 *            要交换元素的数组
	 * @param i
	 *            第一个交换元素的数组下标
	 * @param j
	 *            第二个交换元素的数组小标
	 */
	private static void swap2(int[] n, int i, int j) {
		checkSwapArgs(n, i, j);
		n[i] = n[i] + n[j];
		n[j] = n[i] - n[j];
		n[i] = n[i] - n[j];
	}

	/**
	 * 
	 * 中间变量法，方法简单，带来额外的内存开销，不过开销很小
	 * 
	 * @param n
	 *            要交换元素的数组
	 * @param i
	 *            第一个交换元素的数组下标
	 * @param j
	 *            第二个交换元素的数组小标
	 */
	private static void swap3(int[] n, int i, int j) {
		checkSwapArgs(n, i, j);
		int t = n[i];
		n[i] = n[j];
		n[j] = t;
	}
```

### 冒泡排序
冒泡排序就像水中冒泡一样，每次遍历中都将本次遍历中最大的元素排在最后（上）面。这里的冒泡算法做了一些小优化，如果在排序中，已经出现了数组排序好的情况则停止排序。在学习排序算法的时候也找到一写算法的动图，这里粘贴一下。

```Java
/**
	 * 冒泡排序：相邻比较，每次将尚未排序的序列中最大的元素移动到末尾
	 * 
	 * @param a
	 *            要排序的数组
	 */
	public static void bubbleSort(int[] a) {
		// TODO Auto-generated method stub
		checkArray(a);
		int n = a.length;
		boolean flag = false;// false代表没有排好序
		for (int i = 0; i < n - 1 && !flag; i++) {
			flag = true;
			for (int j = 0; j < n - 1 - i; j++) {
				if (a[j] > a[j + 1]) {
					flag = false;
					swap(a, j, j + 1);
				}
			}
		}
	}
```

![image](http://images2015.cnblogs.com/blog/739525/201603/739525-20160329100443676-1647340243.gif)

![image](http://images2015.cnblogs.com/blog/739525/201603/739525-20160329100034660-1420925220.gif)

这个算法的名字由来是因为越小(或越大)的元素会经由交换慢慢“浮”到数列的顶端。

### 选择排序
选择排序，是指在每次排序中，在尚未排序的序列中选择一个最小的然后接到已经排序好的算法后面。

```Java
/**
	 * 选择排序：每次从剩余尚未排序的序列中选择一个最小的接在已排序的部分后面
	 * 
	 * @param a
	 *            要排序的数组
	 */
	public static void selectSort(int[] a) {
		// TODO Auto-generated method stub
		checkArray(a);
		int n = a.length;
		for (int i = 0; i < n - 1; i++) {
			int min = i;
			for (int j = i + 1; j < n; j++)
				if (a[j] < a[min])
					min = j;
			if (min != i)
				swap(a, i, min);
		}
	}
```
![image](http://images2015.cnblogs.com/blog/739525/201603/739525-20160329102006082-273282321.gif)

![image](http://upload.wikimedia.org/wikipedia/commons/b/b0/Selection_sort_animation.gif)

![image](http://ww2.sinaimg.cn/large/6941baebjw1elxsnb210hg20k3068agd.gif)

### 插入排序
插入排序和选择排序有类似的地方，插入排序是每次从尚未排序的剩余序列中取第一个，然后按照其他如冒泡排序等方法插入到前面已经排序好的序列中。

注意是从已经排序好的序列最后一个开始，倒过来找到合适的位置后插入。

```Java
/**
	 * 插入排序：每次取尚未排序的序列的第一个元素插入到前面序列的合适位置
	 * 
	 * @param a
	 *            要排序的数组
	 */
	public static void insertSort(int[] a) {
		// TODO Auto-generated method stub
		checkArray(a);
		int n = a.length;
		for (int i = 1; i < n; i++) {
			// 方法一：类似降序的冒泡排序法
			// method3a(a, i);

			// 方法二：中间变量
			// method3b(a, i);

			// 方法三：中间变量2
			method3c(a, i);

		}
	}

	private static void method3c(int[] a, int i) {
		int t = a[i];
		int j = i - 1;
		for (; j > -1 && a[j] > t; j--)
			a[j + 1] = a[j];
		a[j + 1] = t;
	}

	@SuppressWarnings("unused")
	private static void method3b(int[] a, int i) {
		int t = a[i];
		int j = i;
		for (; j > 0 && a[j] > t; j--)
			a[j] = a[j - 1];
		a[j] = t;
	}

	@SuppressWarnings("unused")
	private static void method3a(int[] a, int i) {
		for (int j = i - 1; j > -1; j--)
			if (a[i] < a[j])
				swap(a, i, j);
	}
```

![image](http://images2015.cnblogs.com/blog/739525/201603/739525-20160329095145504-1018443290.gif)

![image](http://images2015.cnblogs.com/blog/739525/201603/739525-20160328201132394-577931661.gif)

![image](http://ww2.sinaimg.cn/large/6941baebjw1elxsn8bjp0g20k3068gsv.gif)

插入排序不会动右边的元素，选择排序不会动左边的元素；由于插入排序涉及到的未触及的元素要比插入的元素要少，涉及到的比较操作平均要比选择排序少一半。

### 希尔排序
希尔排序开始时设定一个gap（初始时设置为数组的长度），然后像跳格子一样，等间隔（gap）的取出序列中的数，在对取出的数进行冒泡选择插入排序。每次循环gap减半，逐步细化间隔。

希尔排序，也叫__递减增量排序__，是插入排序的一种更高效的改进版本。希尔排序是不稳定的排序算法。

```Java
/**
	 * 希尔排序：设定一个gap，从序列开头开始，每隔一个gap取一个数，然后对取出的数排序(按常规排序法)。每次 循环后gap减小一半，最终细化。
	 * 
	 * @param a
	 *            要排序的数组
	 */
	public static void shellSort(int[] a) {
		// TODO Auto-generated method stub
		checkArray(a);
		int n = a.length;
		for (int gap = n; gap > 0; gap >>= 1) {
			// 冒泡排序法
			// method4a(n, a, gap);

			// 选择排序法
			// method4b(n, a, gap);

			// 插入排序（希尔排序为插入排序类）
			method4c(n, a, gap);

		}
	}

	private static void method4c(int n, int[] a, int gap) {
		for (int i = gap; i < n; i += gap) {
			int t = a[i];
			int j = i - gap;
			for (; j > -gap && a[j] > t; j -= gap)
				a[j + gap] = a[j];
			a[j + gap] = t;
		}
	}

	@SuppressWarnings("unused")
	private static void method4b(int n, int[] a, int gap) {
		for (int i = gap; i < n - gap; i += gap) {
			int min = i;
			for (int j = i + gap; j < n; j += gap)
				if (a[j] < a[min])
					min = j;
			if (min != i)
				swap(a, i, min);
		}
	}

	@SuppressWarnings("unused")
	private static void method4a(int n, int[] a, int gap) {
		boolean flag = false;
		for (int i = 0; i < n - gap && !flag; i += gap) {
			flag = true;
			for (int j = 0; j < n - gap - i; j += gap) {
				if (a[j] > a[j + gap])
					swap(a, j, j + gap);
				flag = false;
			}
		}
	}
```
![image](http://upload.wikimedia.org/wikipedia/commons/d/d8/Sorting_shellsort_anim.gif)

![image](http://ww4.sinaimg.cn/large/6941baebjw1elxsn4usdbg20k3068gor.gif)


希尔排序是基于插入排序的以下两点性质而提出改进方法的：
1) 插入排序在对几乎已经排好序的数据操作时，效率高，即可以达到线性排序的效率；
2) 但插入排序一般来说是低效的，因为插入排序每次只能将数据移动一位。

希尔排序通过将比较的全部元素分为几个区域来提升插入排序的性能。这样可以让一个元素可以一次性地朝最终位置前进一大步。然后算法再取越来越小的步长进行排序，算法的最后一步就是普通的插入排序，但是到了这步，需排序的数据几乎是已排好的了（此时插入排序较快）。

假设有一个很小的数据在一个已按升序排好序的数组的末端。如果用复杂度为O(n^2)的排序（冒泡排序或直接插入排序），可能会进行n次的比较和交换才能将该数据移至正确位置。而希尔排序会用较大的步长移动数据，所以小数据只需进行少数比较和交换即可到正确位置。

### 快速排序
快速排序是一个不稳定算法。原理形象的说，就是每次递归时，在该序列段内选择一个数（一般为第一个数）做为基准，然后将这个序列中比这个数大的数放在右边，比它小的放在左边。

Java的API中可以使用深度优化的Arrays.sort()方法。对于基础类型，底层使用快速排序。对于非基础类型，底层使用归并排序。

```Java
/**
	 * 快速排序：选择一个数作为基准(一般是第一个数),然后将之后的序列中比这个基准数小的数放在左边，比它大的数放在右边。接着将基准数放在“中间”，通过递归分别再处理左边和右边。
	 * 
	 * @param a
	 *            要排序的数组
	 */
	public static void quickSort(int[] a) {
		// TODO Auto-generated method stub
		checkArray(a);
		int n = a.length;
		int le = 0;
		int ri = n - 1;
		quickFun(a, le, ri);
	}

	private static void quickFun(int[] a, int le, int ri) {

		if (le < 0 || ri > a.length - 1 || le >= ri)
			return;

		int i = le;
		int j = ri;
		int base = a[le];
		while (i < j) {
			while (a[j] > base && i < j)
				j--;
			if (i < j) {
				a[i] = a[j];
				i++;
			}
			while (a[i] < base && i < j)
				i++;
			if (i < j) {
				a[j] = a[i];
				j--;
			}
		}
		a[i] = base;

		quickFun(a, le, i - 1);
		quickFun(a, i + 1, ri);

	}
```
![image](http://upload.wikimedia.org/wikipedia/commons/6/6a/Sorting_quicksort_anim.gif)

![image](http://ww3.sinaimg.cn/large/6941baebjw1elxulu5vbzg20k3068tcj.gif)

对一般快速排序进行一些改进可以提高其效率。
1. 当划分到较小的子序列时，通常可以使用插入排序替代快速排序。
2. 三平均分区法(Median of three partitioning)。

由于快速排序在排序算法中具有排序速度快，而且是就地排序等优点，使得在许多编程语言的内部元素非稳定的排序实现中采用的就是快速排序。

可以参考这篇[博客](http://blog.jobbole.com/79298/)

### 其他排序
其他的排序方法还有：

#### 归并排序
采用“分治”思想，简单将就是分块执行，每部分排序好后，再逐层合并结果。
![image](http://upload.wikimedia.org/wikipedia/commons/c/c5/Merge_sort_animation2.gif)

#### 堆排序
堆排序的存储元素的数据结构为树，并且满足堆的性质（完全二叉树）。
![image](http://upload.wikimedia.org/wikipedia/commons/1/1b/Sorting_heapsort_anim.gif)

## 排序算法的性能
常用的排序算法的时间复杂度和空间复杂度

| 排序法     | 最差时间分析   | 平均时间复杂度 | 稳定度 | 空间复杂度        |
| ---------- | -------------- | -------------- | ------ | ----------------- |
| 冒泡排序   | O($n^{2}$)     | O($n^{2}$)     | 稳定   | O(1)              |
| 快速排序   | O($n^{2}$)     | O($n*\log_2n$) | 不稳定 | O($\log_2n$)~O(n) |
| 选择排序   | O($n^{2}$)     | O($n^{2}$)     | 稳定   | O(1)              |
| 二叉树排序 | O($n^{2}$)     | O($n*\log_2n$) | 不稳定 | O(n)              |
| 插入排序   | O($n^{2}$)     | O($n^{2}$)     | 稳定   | O(1)              |
| 堆排序     | O($n*\log_2n$) | O($n*\log_2n$) | 不稳定 | O(1)              |
| 希尔排序   | O              | O              | 不稳定 | O(1)              |


## 结果测试
只是验证了排序算法的正确性，没有对性能等进行测试。
```Java
public void test() {
		int a[] = RandomIntegerArrayMethod.randomSelectArray(0, 9, 10);
		TestMethod.print(a);

		int b[] = a.clone();
		swap(b, 3, 6);
		TestMethod.print(b);
		swap2(b, 1, 4);
		TestMethod.print(b);
		swap3(b, 5, 2);
		TestMethod.print(b);

		int c1[] = a.clone();
		insertSort(c1);
		TestMethod.print(c1);

		int c2[] = a.clone();
		bubbleSort(c2);
		TestMethod.print(c2);

		int c3[] = a.clone();
		selectSort(c3);
		TestMethod.print(c3);

		int c4[] = a.clone();
		shellSort(c4);
		TestMethod.print(c4);

		int c5[] = a.clone();
		quickSort(c5);
		TestMethod.print(c5);

		// other test
		int d[] = { 2, 4, 5, 6, 2, 0, 2 };
		int d1[] = d.clone();
		insertSort(d1);
		TestMethod.print(d1);

		int d2[] = d.clone();
		bubbleSort(d2);
		TestMethod.print(d2);

		int d3[] = d.clone();
		selectSort(d3);
		TestMethod.print(d3);

		int d4[] = d.clone();
		shellSort(d4);
		TestMethod.print(d4);

		int d5[] = d.clone();
		quickSort(d5);
		TestMethod.print(d5);

		// insertSort(null);
	}
```
>测试结果
6 9 8 7 2 4 0 3 1 5 
6 9 8 0 2 4 7 3 1 5 
6 2 8 0 9 4 7 3 1 5 
6 2 4 0 9 8 7 3 1 5 
0 1 2 3 4 5 6 7 8 9 
0 1 2 3 4 5 6 7 8 9 
0 1 2 3 4 5 6 7 8 9 
0 1 2 3 4 5 6 7 8 9 
0 1 2 3 4 5 6 7 8 9 
0 2 2 2 4 5 6 
0 2 2 2 4 5 6 
0 2 2 2 4 5 6 
0 2 2 2 4 5 6 
0 2 2 2 4 5 6 


