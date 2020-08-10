---
title: Java实现自定义队列（链表和数组两种方式）
date: 2017/09/04
categories:
- 学习
tags:
- 学习
- Java
- 数据结构
---

Java实现自定义队列（链表和数组两种方式）
=======================================
队列有两种实现方式，一种是基于循环数组，一种是基于链表。在这里主要是模拟自定义的队列，实现这两种方式。

集合框架[源码](https://github.com/ChainGit/data-structure-learn/tree/master/Data-Structure-Test)。

## 队列的抽象类
一个队列，无外乎几种常见的方法，比如poll()、remove()、peek()，还有push()、last()操作。如果是双端队列，那么还有对尾元素的删除操作等。

这里实现的是单端队列。

队列抽象类：
AbstractQueue：
```java
package com.chain.algorithm.test.day01;

import java.util.List;

public abstract class AbstractQueue<T> {

	/**
	 * 将新节点加到队列末尾
	 * 
	 * @param value
	 */
	public abstract void push(T value);

	/**
	 * 查看队列最后一个节点的值
	 * 
	 * @return
	 */
	public abstract T last();

	/**
	 * 获得队列的第一个节点的值，并移除第一个节点
	 * 
	 * @return
	 */
	public abstract T pull();

	/**
	 * 查看队列的第一个节点的值
	 * 
	 */
	public abstract T peek();

	/**
	 * 队列是否为空
	 * 
	 * @return
	 */
	public abstract boolean isEmpty();

    /**
	 * 判断数组是否已满
	 * 
	 * @return
	 */
	public boolean isFull();

	/**
	 * 
	 * 清空队列
	 * 
	 */
	public abstract void clear();

	/**
	 * 
	 * 队列的现有节点数
	 * 
	 */
	public abstract int size();

	/**
	 * 
	 * 转换为List
	 * 
	 * @return
	 */
	public abstract List<T> toList();

}
```

## 队列的扩展接口

ArrayQueueInterface：
```java
package com.chain.algorithm.test.day01;

public interface ArrayQueueInterface<T> {

	/**
	 * 扩大数组容量
	 * 
	 * @param old
	 * @return
	 */
	public void extend();

}
```

## 队列节点元素

队列中插入的具体元素类。

QueueNode：
```java
package com.chain.algorithm.test.day01;

/**
 * 队列的节点
 * 
 * @author Chain
 *
 * @param <T>
 */
public class QueueNode<T> {

	private T value;
	private QueueNode<T> nextNode;

	public T getValue() {
		return value;
	}

	public void setValue(T value) {
		this.value = value;
	}

	public QueueNode<T> getNextNode() {
		return nextNode;
	}

	public void setNextNode(QueueNode<T> nextNode) {
		this.nextNode = nextNode;
	}

	public QueueNode(T value) {
		super();
		this.value = value;
	}

}
```

## 循环数组实现

循环数组的实现主要是通过两个指针，一个是head，一个是tail，这两指针分别代表的是队列的头节点和尾节点的__下一个空节点__。所以总有一个位置是被空出来的。

对于队列是否空和满有两种判断方式，一种是通过head、tail和capacity的数学关系判断，一种是通过count来判断。

当然还实现了数组的扩展机制，是两倍扩展的方法。

下面分别介绍使用数学关系和count来判断的不同实现：

使用数学关系：
ArrayQueue：
```java
package com.chain.algorithm.test.day01;

import java.util.ArrayList;
import java.util.List;

/**
 * 使用数组实现循环队列
 * 
 * 判断的时候可以使用数学公式，也可以使用count来判断 ；使用数学公式有一个会浪费，使用count则可以不浪费空间
 * 
 * @author Chain
 *
 */
public class ArrayQueue<T> extends AbstractQueue<T> implements ArrayQueueInterface<T> {

	private Object[] data;

	// 不使用count也是可以的
	private int count;

	// 指向队列的头元素所在的位置的下标
	private int head;
	// 指向队列的尾元素所在的位置的后一个下标
	private int tail;

	private static int capacity = 10;

	public ArrayQueue() {
		super();
		this.data = new Object[capacity];
	}

	@Override
	public void push(T value) {
		if (isFull()) {
			extend();
		}

		data[tail] = value;
		tail = (tail + 1) % capacity;
		count++;
	}

	@Override
	public T last() {
		if (isEmpty())
			return null;

		int last = (tail + capacity - 1) % capacity;
		return (T) data[last];
	}

	@Override
	public T pull() {
		if (isEmpty())
			throw new RuntimeException("queue is empty");

		T temp = (T) data[head];
		data[head] = null;
		head = (head + 1) % capacity;
		count--;
		return temp;
	}

	@Override
	public T peek() {
		if (isEmpty())
			return null;

		T temp = (T) data[head];
		return temp;
	}

	@Override
	public boolean isEmpty() {
		// 如果head和tail重叠，则为空
		if (head == tail)
			return true;
		return false;
	}

	@Override
	public void clear() {
		head = tail = count = 0;
		capacity = 10;
		data = new Object[capacity];
	}

	@Override
	public int size() {
		int size = (capacity - (head - tail)) % capacity;
		return size;
	}

	@Override
	public boolean isFull() {
		// 假设head和tail是不重叠的，即tail和head之间空一格
		if ((tail + 1) % capacity == head)
			return true;
		return false;
	}

	@Override
	public List<T> toList() {
		List<T> lst = new ArrayList<>();
		for (int i = 0; i < data.length; i++) {
			lst.add((T) data[i]);
		}
		return lst;
	}

	@Override
	public synchronized void extend() {
		try {
			// 方法很多，这里使用pull再push的方法
			int newCapacity = capacity << 1;
			Object[] newData = new Object[newCapacity];
			int size = size();
			int index = 0;
			while (size-- > 0) {
				newData[index++] = pull();
			}

			head = 0;
			tail = count = index;
			capacity = newCapacity;
			data = newData;
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

}
```

使用count来判断：
```java
package com.chain.algorithm.test.day01;

import java.util.ArrayList;
import java.util.List;

/**
 * 使用数组实现循环队列
 * 
 * 判断的时候可以使用数学公式，也可以使用count来判断 ；使用数学公式有一个会浪费，使用count则可以不浪费空间
 * 
 * @author Chain
 *
 */
public class ArrayQueue2<T> extends AbstractQueue<T> implements ArrayQueueInterface<T> {

	private Object[] data;

	// 不使用count也是可以的
	private int count;

	// 指向队列的头元素所在的位置的下标
	private int head;
	// 指向队列的尾元素所在的位置的后一个下标
	private int tail;

	private static int capacity = 10;

	public ArrayQueue2() {
		super();
		this.data = new Object[capacity];
	}

	@Override
	public void push(T value) {
		if (isFull()) {
			extend();
		}

		data[tail] = value;
		tail = (tail + 1) % capacity;
		count++;
	}

	@Override
	public T last() {
		if (isEmpty())
			return null;

		int last = (tail + capacity - 1) % capacity;
		return (T) data[last];
	}

	@Override
	public T pull() {
		if (isEmpty())
			throw new RuntimeException("queue is empty");

		T temp = (T) data[head];
		data[head] = null;
		head = (head + 1) % capacity;
		count--;
		return temp;
	}

	@Override
	public T peek() {
		if (isEmpty())
			return null;

		T temp = (T) data[head];
		return temp;
	}

	@Override
	public boolean isEmpty() {
		return size() == 0;
	}

	@Override
	public void clear() {
		head = tail = count = 0;
		capacity = 10;
		data = new Object[capacity];
	}

	@Override
	public int size() {
		return count;
	}

	@Override
	public boolean isFull() {
		return size() == capacity;
	}

	@Override
	public List<T> toList() {
		List<T> lst = new ArrayList<>();
		for (int i = 0; i < data.length; i++) {
			lst.add((T) data[i]);
		}
		return lst;
	}

	@Override
	public synchronized void extend() {
		try {
			// 方法很多，这里使用pull再push的方法
			int newCapacity = capacity << 1;
			Object[] newData = new Object[newCapacity];
			int size = size();
			int index = 0;
			while (size-- > 0) {
				newData[index++] = pull();
			}

			head = 0;
			tail = count = index;
			capacity = newCapacity;
			data = newData;
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

}
```

## 使用链表实现

链表实现就相对数组而言容易多了，不需要数组扩容的方法。

LinkedQueue：
```java
package com.chain.algorithm.test.day01;

import java.util.ArrayList;
import java.util.List;

/**
 * 基于链表的队列
 * 
 * @author Chain
 *
 */
public class LinkedQueue<T> extends AbstractQueue<T> {

	private QueueNode<T> headNode;

	private int count;

	public LinkedQueue() {
		super();
		headNode = new QueueNode<>(null);
	}

	/**
	 * 将新节点加到队列末尾
	 * 
	 * @param value
	 */
	@Override
	public void push(T value) {
		QueueNode<T> node = new QueueNode<>(value);

		QueueNode<T> currentNode = headNode;
		QueueNode<T> lastNode = currentNode;
		while (currentNode != null) {
			lastNode = currentNode;
			currentNode = currentNode.getNextNode();
		}

		lastNode.setNextNode(node);
		count++;
	}

	/**
	 * 查看队列最后一个节点的值
	 * 
	 * @return
	 */
	@Override
	public T last() {
		QueueNode<T> currentNode = headNode;
		QueueNode<T> lastNode = currentNode;
		while (currentNode != null) {
			lastNode = currentNode;
			currentNode = currentNode.getNextNode();
		}

		return lastNode.getValue();
	}

	/**
	 * 获得队列的第一个节点的值，并移除第一个节点
	 * 
	 * @return
	 */
	@Override
	public T pull() {
		if (isEmpty()) {
			throw new RuntimeException("queue is empty");
		}

		QueueNode<T> nextNode = headNode.getNextNode();
		T val = nextNode.getValue();
		QueueNode<T> nextNextNode = nextNode.getNextNode();
		if (nextNextNode == null)
			headNode.setNextNode(null);
		else
			headNode.setNextNode(nextNextNode);

		count--;

		return val;
	}

	/**
	 * 查看队列的第一个节点的值
	 * 
	 */
	@Override
	public T peek() {
		if (isEmpty()) {
			return null;
		}

		QueueNode<T> nextNode = headNode.getNextNode();
		T val = nextNode.getValue();
		return val;
	}

	/**
	 * 队列是否为空
	 * 
	 * @return
	 */
	@Override
	public boolean isEmpty() {
		QueueNode<T> nextNode = headNode.getNextNode();
		return nextNode == null;
		// return count == 0;
	}

	/**
	 * 
	 * 清空队列
	 * 
	 */
	@Override
	public void clear() {
		headNode.setNextNode(null);
		count = 0;
	}

	/**
	 * 队列的现有节点数
	 * 
	 */
	@Override
	public int size() {
		return count;
	}

	/**
	 * 转变为List
	 * 
	 */
	@Override
	public List<T> toList() {
		List<T> lst = new ArrayList<>();
		if (isEmpty())
			return lst;

		QueueNode<T> currrentNode = headNode.getNextNode();
		while (currrentNode != null) {
			lst.add(currrentNode.getValue());
			currrentNode = currrentNode.getNextNode();
		}
		return lst;
	}

}
```

## 测试结果

测试代码就不贴了，源码[传送门](https://github.com/ChainGit/algorithm-learn/tree/master/chain/test01)

