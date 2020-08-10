---
title: Java多线程之生产者和消费者模式
date: 2017/08/27
categories: 学习
tags:
- 学习
- Java
- 多线程
---

Java多线程之生产者和消费者模式
============================
这几天学习了下JUC的基础知识，并实现了经典的生产者和消费者模型。生产者和消费者广义上可以指数据的产生和数据的使用者，可以用在很多场景中。这里只是做了简单的实现。

[操作系统原理之进程和线程管理](https://www.leechain.top/blog/2017/08/25-operation-system-multithreads.html)
[Java的多线程和并发知识纲要](https://www.leechain.top/blog/2017/08/21-java-concurrency-brief.html)

## 公共类和方法
本文中的生产和消费的模型，主要是指 __生产 => 商店 => 消费__ 这样的模型。

相关文章网上有很多，实现具体细节也不一样。

生产者和消费者的可以使用自定义的粗糙实现，也可以使用JavaAPI提供的JUC相关类。

### 商店
商店的接口（也可以是抽象类）Shop：
```java
package com.chain.blog.test.day02;

/**
 * 商店接口
 * 
 * @author Chain
 *
 */
public interface Shop {

	/**
	 * 最大的商品库存数
	 */
	public static final int MAX = 10;

	/**
	 * 最小的商品库存数
	 */
	public static final int MIN = 0;

	/**
	 * 查找商品的库存
	 * 
	 * @return
	 */
	public int now();

	/**
	 * 商店进货
	 */
	public boolean in();

	/**
	 * 商店卖货
	 */
	public boolean out();
}
```

接下来是生产者和消费者，在这里我使用了CountDownLatch，用于最后的main线程的统计已生产和已消费的数量。

在这里，所有的生产者和消费者均尝试生产和消费20次。

### 常量
存储在Constant接口里：
```java
package com.chain.blog.test.day02;

public interface Constant {

	public static final int TIMES = 20;
}
```

### 生产者
生产者模型Producer：
```java
package com.chain.blog.test.day02;

import java.util.concurrent.CountDownLatch;

/**
 * 生产者
 * 
 * @author Chain
 *
 */
public class Producer implements Runnable {

	private Shop shop;

	private CountDownLatch latch;

	public Producer(Shop shop, CountDownLatch latch) {
		this.shop = shop;
		this.latch = latch;
	}

	@Override
	public void run() {
		try {
			for (int i = 0; i < Constant.TIMES; i++) {
				try {
					// 随机暂停一段时间，模拟不同的情况
					// Thread.sleep((int) (Math.random() * 301 + 100));
					Thread.sleep(100);
				} catch (InterruptedException e) {
				}

				put();
			}
		} finally {
			latch.countDown();
		}
	}

	private void put() {
		boolean res = shop.in();
		if (res) {
			System.out.println("producer " + Thread.currentThread().getName() + " has put one");
		} else {
			System.out.println("producer " + Thread.currentThread().getName() + " put one fail");
		}
	}

}
```

### 消费者
消费者模型Consumer:
```java
package com.chain.blog.test.day02;

import java.util.concurrent.CountDownLatch;

/**
 * 消费者
 * 
 * @author Chain
 *
 */
public class Consumer implements Runnable {

	private Shop shop;

	private CountDownLatch latch;

	public Consumer(Shop shop, CountDownLatch latch) {
		this.shop = shop;
		this.latch = latch;
	}

	@Override
	public void run() {
		try {
			for (int i = 0; i < Constant.TIMES; i++) {
				try {
					// 随机暂停一段时间，模拟不同的情况
					// Thread.sleep((int) (Math.random() * 301 + 100));
					Thread.sleep(300);
				} catch (InterruptedException e) {
				}

				buy();
			}
		} finally {
			latch.countDown();
		}
	}

	/**
	 * 消费者购买商品
	 */
	private void buy() {
		boolean res = shop.out();
		if (res) {
			System.out.println("consumer " + Thread.currentThread().getName() + " has buy one");
		} else {
			System.out.println("consumer " + Thread.currentThread().getName() + " buy one fail");
		}
	}

}
```
## 非线程安全的做法

先来测试一下非线程安全的情况，可能会出现：
1、仓库爆满货仓库为负的情况；
2、生产者在仓库满时仍然尝试生产，以及消费者在仓库空时仍然尝试消费的情况；
3、仓库最后有空余的情况。

### 商店实现
ShopImplA：
```java
package com.chain.blog.test.day02;

/**
 * 不使用任何多线程机制
 * 
 * 存在<br>
 * 1、内存可见性<br>
 * 2、库存可能爆仓货、库存可能为负<br>
 * 3、当库存已满时，生产者继续放货，可能出现商品生产丢失的情况<br>
 * 4、当库存不足时，消费者会继续尝试不断的买货，造成没有意义的购买尝试<br>
 * 的情况<br>
 * 
 * @author Chain
 *
 */
public class ShopImplA implements Shop {

	private int product;
	// private volatile int product;

	public int inTimes;
	public int outTimes;

	@Override
	public boolean in() {
		if (now() >= Shop.MAX) {
			System.out.println("shop is full");
			return false;
		}
		product++;
		inTimes++;
		System.out.println("after in, shop left is " + now());
		return true;
	}

	@Override
	public boolean out() {
		if (now() <= Shop.MIN) {
			System.out.println("shop is empty");
			return false;
		}
		product--;
		outTimes++;
		System.out.println("after out, shop left is " + now());
		return true;
	}

	@Override
	public int now() {
		return product;
	}

}
```

### 测试代码
TestA：
```java
package com.chain.blog.test.day02;

import java.util.concurrent.CountDownLatch;

public class TestA {

	public static void main(String[] args) {
		// test1();
		test2();
	}

	// 多个生产者和消费者，每个生产者和消费者也具有多个线程
	// 会出现爆仓和负仓的问题，且容易出现生产和消费数据不匹配问题
	private static void test2() {
		CountDownLatch latch = new CountDownLatch(8);
		Shop shop = new ShopImplA();
		Consumer consumer1 = new Consumer(shop, latch);
		Producer producer1 = new Producer(shop, latch);
		Consumer consumer2 = new Consumer(shop, latch);
		Producer producer2 = new Producer(shop, latch);
		new Thread(consumer1, "consumerA1").start();
		new Thread(producer1, "producerA1").start();
		new Thread(consumer1, "consumerA2").start();
		new Thread(producer1, "producerA2").start();
		new Thread(consumer2, "consumerB1").start();
		new Thread(producer2, "producerB1").start();
		new Thread(consumer2, "consumerB2").start();
		new Thread(producer2, "producerB2").start();

		while (latch.getCount() != 0) {
			try {
				Thread.sleep(2000);
			} catch (InterruptedException e) {
			}
		}

		System.out.println();
		System.out.println("in times: " + ((ShopImplA) shop).inTimes);
		System.out.println("out times: " + ((ShopImplA) shop).outTimes);
		System.out.println("shop left: " + shop.now());
	}

	// 只有一个消费者和一个生产者，似乎没有出现爆仓和负仓的问题，但是商品有时有剩余，不能刚好买多少卖多少
	private static void test1() {
		CountDownLatch latch = new CountDownLatch(2);
		Shop shop = new ShopImplA();
		Consumer consumer = new Consumer(shop, latch);
		Producer producer = new Producer(shop, latch);
		new Thread(consumer, "consumerA").start();
		new Thread(producer, "producerA").start();

		while (latch.getCount() != 0) {
			try {
				Thread.sleep(2000);
			} catch (InterruptedException e) {
			}
		}

		System.out.println();
		System.out.println("in times: " + ((ShopImplA) shop).inTimes);
		System.out.println("out times: " + ((ShopImplA) shop).outTimes);
		System.out.println("shop left: " + shop.now());
	}

}
```

### 测试结果
调整Consumer和Producer的Thread.sleep中的值，可以模拟
1、生产效率快，消费速度慢
2、生产效率慢，消费速度快
3、生产和消费速度随机波动
的情况。

以下是测试过程中的异常情况：

1、出现爆仓：

![image](/uploads/java-producer-and-consumer-learn/one-overflow.png)

2、出现负仓：

![image](/uploads/java-producer-and-consumer-learn/one-fucang.png)

3、仍有库存：

![image](/uploads/java-producer-and-consumer-learn/one-left.png)

由上可见，这样的方式是不安全的。

## 线程安全的做法

### 信号量与互斥量

semaphore + mutex 也是PV模型

### 监控法

while + wait/notify

### 隐式锁synchronized

使用java语法直接支持的synchronized和Object自有的wait/notify方法。

在这里需要注意虚假唤醒的问题。

#### 商店实现
ShopImplB：
```java
package com.chain.blog.test.day02;

/**
 * 使用synchronized、wait/notify实现线程同步
 * 
 * 注意虚假唤醒的问题
 * 
 * @author Chain
 *
 */
public class ShopImplB implements Shop {

	private volatile int product;

	public int inTimes;
	public int outTimes;

	@Override
	public synchronized int now() {
		return product;
	}

	@Override
	public synchronized boolean in() {
		// 注意这里需要是while，不能是if，因为线程除了会唤醒out操作，也会唤醒in的操作
		while (now() >= Shop.MAX) {
			System.out.println("shop is full");
			try {
				wait();
			} catch (InterruptedException e) {
			}
			// return false;
		}
		product++;
		inTimes++;
		System.out.println("after in, shop left is " + now());
		notifyAll();
		return true;
	}

	@Override
	public synchronized boolean out() {
		while (now() <= Shop.MIN) {
			System.out.println("shop is empty");
			try {
				wait();
			} catch (InterruptedException e) {
			}
			// return false;
		}
		product--;
		outTimes++;
		System.out.println("after out, shop left is " + now());
		notifyAll();
		return true;
	}

}
```

#### 测试代码
和非同步原型类似，只需将ShopImplA替换成ShopImplB即可。

#### 测试结果

结果总是正确的，如果不使用while而使用if会出现和非同步一样的现象，这个是虚假唤醒。

> 比如：生产线程A执行add操作时，再判断完if(new()>=MAX)后就暂停后，线程B执行add操作时也同A一样暂停在判断后，那么在消费线程唤醒所有的线程时，会执行了两个add操作，这样就可能爆仓。

### 显式锁ReentrantLock

显示锁操作比较麻烦，比隐式锁麻烦，不过隐式锁jvm会不断优化，还是建议使用synchronized。

#### 商店实现
ShopImplC：
```java
package com.chain.blog.test.day02;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 使用显式锁和显示条件
 * 
 * @author Chain
 *
 */
public class ShopImplC implements Shop {

	private volatile int product;

	private ReentrantLock lock = new ReentrantLock();
	private Condition conditionIn = lock.newCondition();
	private Condition conditionOut = lock.newCondition();

	public int inTimes;
	public int outTimes;

	@Override
	public int now() {
		lock.lock();
		try {
			return product;
		} finally {
			lock.unlock();
		}
	}

	@Override
	public boolean in() {
		lock.lock();
		try {
			while (now() >= Shop.MAX) {
				System.out.println("shop is full");
				try {
					conditionIn.await();
				} catch (InterruptedException e) {
				}
			}
			product++;
			inTimes++;
			System.out.println("after in, shop left is " + now());
			conditionOut.signalAll();
			return true;
		} finally {
			lock.unlock();
		}
	}

	@Override
	public boolean out() {
		lock.lock();
		try {
			while (now() <= Shop.MIN) {
				System.out.println("shop is empty");
				try {
					conditionOut.await();
				} catch (InterruptedException e) {
				}
			}
			product--;
			outTimes++;
			System.out.println("after out, shop left is " + now());
			conditionIn.signalAll();
			return true;
		} finally {
			lock.unlock();
		}
	}

}
```

#### 测试代码
和非同步原型类似，只需将ShopImplA替换成ShopImplC即可。

#### 测试结果
测试结果和synchronized相比，显式锁比隐式锁感觉上要流畅一些。

### 使用阻塞队列

#### 自定义阻塞队列MyBlockArrayQueue

##### 商店实现
ShopImplD：
```java
package com.chain.blog.test.day02;

import java.util.ArrayDeque;
import java.util.Queue;

/**
 * 使用自定义的阻塞队列，基于synchronized、wait/notify
 * 
 * @author Chain
 *
 */
public class ShopImplD implements Shop {

	private MyBlockArrayQueue product = new MyBlockArrayQueue(Shop.MAX);

	public int inTimes;
	public int outTimes;

	@Override
	public int now() {
		return product.size();
	}

	@Override
	public boolean in() {
		product.add();
		synchronized (this) {
			inTimes++;
			return true;
		}
	}

	@Override
	public boolean out() {
		product.poll();
		synchronized (this) {
			outTimes++;
			return true;
		}
	}

}

/**
 * 自定义的阻塞队列
 * 
 * @author Chain
 *
 */
class MyBlockArrayQueue {

	private int limit;
	private int floor;

	private Queue<Object> queue;

	public MyBlockArrayQueue(int limit) {
		this.limit = limit;
		this.queue = new ArrayDeque<>(limit);
	}

	public synchronized void add() {
		while (size() >= limit) {
			try {
				System.out.println("queue is full");
				wait();
			} catch (InterruptedException e) {
			}
		}
		queue.add(new Object());
		System.out.println("after add, queue left is " + size());
		notifyAll();
	}

	public synchronized void poll() {
		while (size() <= floor) {
			System.out.println("queue is empty");
			try {
				wait();
			} catch (InterruptedException e) {
			}
		}
		queue.poll();
		System.out.println("after poll, queue left is " + size());
		notifyAll();
	}

	public synchronized int size() {
		return queue.size();
	}
}
```

##### 测试代码
和非同步原型类似，只需将ShopImplA替换成ShopImplD即可。

##### 测试结果
测试结果正确。

#### 自定义阻塞队列MyBlockArrayQueue2

##### 商店实现
ShopImplE：
```java
package com.chain.blog.test.day02;

import java.util.ArrayDeque;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 使用自定义的阻塞队列，基于显式锁和显示条件
 * 
 * @author Chain
 *
 */
public class ShopImplE implements Shop {

	private MyBlockArrayQueue2 product = new MyBlockArrayQueue2(MAX);

	public int inTimes;
	public int outTimes;

	@Override
	public int now() {
		return product.size();
	}

	@Override
	public boolean in() {
		product.add();
		synchronized (this) {
			inTimes++;
			return true;
		}
	}

	@Override
	public boolean out() {
		product.poll();
		synchronized (this) {
			outTimes++;
			return true;
		}
	}

}

class MyBlockArrayQueue2 {

	private int limit;
	private int floor;

	private Queue<Object> queue;

	public MyBlockArrayQueue2(int limit) {
		this.limit = limit;
		this.queue = new ArrayDeque<>(limit);
	}

	private ReentrantLock lock = new ReentrantLock();
	private Condition notEmpty = lock.newCondition();
	private Condition notFull = lock.newCondition();

	public void add() {
		lock.lock();
		try {
			while (size() >= limit) {
				System.out.println("queue is full");
				try {
					notFull.await();
				} catch (InterruptedException e) {
				}
			}
			queue.add(new Object());
			System.out.println("after add, queue left is " + size());
			notEmpty.signalAll();
		} finally {
			lock.unlock();
		}
	}

	public void poll() {
		lock.lock();
		try {
			while (size() <= floor) {
				System.out.println("queue is empty");
				try {
					notEmpty.await();
				} catch (InterruptedException e) {
				}
			}
			queue.poll();
			System.out.println("after poll, queue left is " + size());
			notFull.signalAll();
		} finally {
			lock.unlock();
		}
	}

	public int size() {
		lock.lock();
		try {
			return queue.size();
		} finally {
			lock.unlock();
		}
	}

}
```

##### 测试代码
和非同步原型类似，只需将ShopImplA替换成ShopImplE即可。

##### 测试结果
测试结果正确，且感觉上优于隐式锁实现方式。

#### JavaAPI支持的ArrayBlockQueue

官方的API基于显式锁和显式条件实现。

##### 商店实现
ShopImplF：
```java
package com.chain.blog.test.day02;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

/**
 * 使用java的ArrayBlockingQueue，基于显式锁和显示条件
 * 
 * @author Chain
 *
 */
public class ShopImplF implements Shop {

	private BlockingQueue<Object> product = new ArrayBlockingQueue<>(MAX);

	public int inTimes;
	public int outTimes;

	@Override
	public synchronized int now() {
		return product.size();
	}

	@Override
	public synchronized boolean in() {
		while (product.size() >= MAX) {
			System.out.println("shop is full");
			try {
				wait();
			} catch (InterruptedException e) {
			}
		}
		product.add(new Object());
		inTimes++;
		System.out.println("after in, shop left is " + now());
		notifyAll();
		return true;
	}

	@Override
	public synchronized boolean out() {
		while (product.size() <= MIN) {
			System.out.println("shop is empty");
			try {
				wait();
			} catch (InterruptedException e) {
			}
		}
		product.poll();
		outTimes++;
		System.out.println("after out, shop left is " + now());
		notifyAll();
		return true;
	}

}
```

##### 测试代码
和非同步原型类似，只需将ShopImplA替换成ShopImplD即可。

##### 测试结果
测试结果正确。

## 另一个案例
多线程实现交替打印ABC

直接贴上源码：
```java
package com.chain.juc.day03;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 面试题
 * 
 * ABC交替打印
 * 
 * @author Chain
 *
 */
public class Test04ABC {

	public static void main(String[] args) {
		test1();
	}

	private static void test1() {
		MyABC abc = new MyABC();

		new Thread(new Runnable() {

			@Override
			public void run() {
				while (true) {
					abc.printA();
				}
			}

		}).start();

		new Thread(new Runnable() {

			@Override
			public void run() {
				while (true) {
					abc.printB();
				}
			}

		}).start();

		new Thread(new Runnable() {

			@Override
			public void run() {
				while (true) {
					abc.printC();
				}
			}

		}).start();
	}

}

class MyABC {

	private ReentrantLock lock = new ReentrantLock();
	private Condition conditionA = lock.newCondition();
	private Condition conditionB = lock.newCondition();
	private Condition conditionC = lock.newCondition();

	private volatile int current = 1;

	public void printA() {
		lock.lock();
		try {
			while (current != 1) {
				try {
					conditionA.await();
				} catch (InterruptedException e) {
				}
			}

			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
			}

			System.out.println("A");
			current = 2;
			conditionB.signal();
		} finally {
			lock.unlock();
		}
	}

	public void printB() {
		lock.lock();
		try {
			while (current != 2) {
				try {
					conditionB.await();
				} catch (InterruptedException e) {
				}
			}

			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
			}

			System.out.println("B");
			current = 3;
			conditionC.signal();
		} finally {
			lock.unlock();
		}
	}

	public void printC() {
		lock.lock();
		try {
			while (current != 3) {
				try {
					conditionC.await();
				} catch (InterruptedException e) {
				}
			}

			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
			}

			System.out.println("C");
			current = 1;
			conditionA.signal();
		} finally {
			lock.unlock();
		}
	}

}
```

## 总结
生产者消费者模型是多线程中的经典模型了，当然还是有很多东西要学习的。加油!

源码在[这里](https://github.com/ChainGit/some-tests/tree/master/test05)