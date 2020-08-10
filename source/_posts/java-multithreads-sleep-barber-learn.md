---
title: Java多线程之熟睡的理发师问题
date: 2017/12/10
categories: 学习
tags:
- 学习
- Java
- 多线程
---

Java多线程之熟睡的理发师问题
============================
理发师问题也是经典问题之一，最经典的三个线程同步互斥问题是：生产消费者问题、哲学家就餐问题、读者写者问题。这三个问题的解决方案有很多，不同的方法也有不同的含义和原理，需要好好理解。各个方法中，信号量是通用的方法，理发师问题也可以用信号量来解决。

[操作系统原理之进程和线程管理](https://www.leechain.top/blog/2017/08/25-operation-system-multithreads.html)
[Java的多线程和并发知识纲要](https://www.leechain.top/blog/2017/08/21-java-concurrency-brief.html)
[Java多线程之生产者和消费者模式](https://www.leechain.top/blog/2017/08/27-java-multithreads-producer-and-consumer-learn.html)
[Java多线程之哲学家就餐问题](https://www.leechain.top/blog/2017/11/20-java-multithreads-philosopher-dinner-learn.html)
[Java多线程之读者写者问题](https://www.leechain.top/blog/2017/11/25-java-multithreads-read-write-learn.html)
[Java多线程之吸烟者问题](https://www.leechain.top/blog/2017/12/08-java-multithreads-smokers-learn.html)

## 问题描述

某理发店里只有一名理发师，一张理发椅，若干张顾客等待椅。理发师每天除了理发就是睡觉。理发店的生意很好，需要理发的人很多。
如果一名需要理发的顾客，进店发现等待椅坐满了，他就走了，决定稍后再来。
如果发现等待椅子上还有空位，他就走进店里坐下来等待，不过只会等待一会（每位顾客能等待的时间不一样），如果超过时间就放弃等待，离店并稍后再来。
坐在等待椅上的顾客会不断注意理发师的状态，如果他不在理发而在睡觉中（哪怕刚睡着），就会尝试去叫醒理发师。
叫醒理发师的顾客可以理发，但是需要抢占理发椅，只有抢占到理发椅的顾客才能离开等待椅，并坐到理发椅上由理发师理发。
理发师理完发后，顾客起身离开理发椅，并离开理发店，对理发师的手艺很满意决定不久后再来。
理发师一理完发就会睡觉，由于理发师只能由等待的顾客叫醒，所以如果理发店如果没有顾客的话理发师将一直睡着。
理发师想睡觉只能睡在理发椅上，所以只有当理发师醒了，顾客才能去抢坐理发椅。

![理发师问题](/uploads/multithreads-basic-knowledge/37.jpg)

## 问题分析

理发师问题看上去比较复杂，不过仔细分析，化繁就简还是能解决的。

假设顾客一直想理发，哪怕是刚刚理完发离店后不久还会再来。

顾客的状态：__门外状态、等待状态、理发状态__。
理发师的状态：__睡觉状态、理发状态__。
椅子（等待椅和理发椅）的状态：__有人、没人__。

状态变化分析：

1、在等待状态的顾客会一直尝试唤醒处在睡觉状态的理发师。
2、在门外状态的顾客会每隔一段时间尝试进店，成为等待状态。
3、在等待状态的顾客可能会等待一段时间后离开，变成门外状态。

具体做法分析：
1、顾客尝试离开或坐入等待椅：
等待椅的数量是一个临界资源chair，只能同时由一名顾客去操作。

2、顾客进门尝试坐在等待椅上：
需要一个进入店门的信号量door，尝试进门的顾客先去door排队（P），获得door的顾客才能进店。
进入店后查看等待椅的状态，如果椅子满了，就转身离开，并归还door信号量（V）。
如果椅子没坐满，那么将等待临界资源chair，修改chair-1。修改好chair后才会归还door信号量（V）。
这样的操作确保进店（拿到door）的顾客一次只有一个，且要么能坐到等待椅上，要么不能。

3、等待中的顾客唤醒理发师：
坐在等待椅上的顾客只做一件事，如果理发师在睡觉就唤醒它。

4、顾客抢占理发椅：
成功叫醒理发师后，所有的等待中的顾客都会尝试去占有理发椅，但是只有一个顾客能成功占有理发椅。其余顾客将继续保持等待状态。
理发椅是临界资源，在理发师清醒状态下，只能由一名顾客占有。
占有理发椅的顾客将进入理发状态，理发完成之后顾客离开理发椅。

## 我的解决

各个对象的状态其实是为了方便理解，在实际代码中并不会使用到。

理发师：

```java
package com.chain.test.day07;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 理发师
 * 
 * @author chain
 *
 */
public class Barber {

	// 理发师的姓名
	private String name = "barber";

	// 理发椅
	private WorkChair workChair;

	private ReentrantLock lock;
	private Condition cond;

	// 理发师理过的顾客数量
	private long times;

	public long getTimes() {
		return times;
	}

	public Condition getCondition() {
		return cond;
	}

	public WorkChair getWorkChair() {
		return workChair;
	}

	public ReentrantLock getLock() {
		return lock;
	}

	public Barber(WorkChair workChair) {
		this.workChair = workChair;
		this.lock = new ReentrantLock();
		this.cond = this.lock.newCondition();
	}

	public String getName() {
		return name;
	}

	/**
	 * 理发师睡觉
	 * 
	 * @throws InterruptedException
	 */
	public void sleep() throws InterruptedException {
		// 如果理发椅上没有顾客就睡觉
		if (workChair.get() == null) {
			lock.lock();
			try {
				if (workChair.get() == null) {
					// 在这里理发师不会睡在理发椅上
					System.out.println(name + " is sleeping");
					cond.await();
					System.out.println(name + " is awake");
				}
			} finally {
				if (lock.isLocked())
					lock.unlock();
			}
		}
	}

	/**
	 * 给顾客理发
	 * 
	 * @throws InterruptedException
	 */
	public void work() throws InterruptedException {
		Customer customer = workChair.get();
		if (customer == null) {
			System.out.println("no customer in chair, the barber is confused");
			return;
		}
		System.out.println(name + " is working");
		customer.enjoy();
		customer.bye(this);
		++times;
		System.out.println(name + " is completed, times " + times);
	}

}
```

顾客：

```java
package com.chain.test.day07;

import java.util.concurrent.locks.ReentrantLock;

/**
 * 顾客
 * 
 * @author chain
 *
 */
public class Customer {

	// 顾客序号
	private int id;

	// 顾客姓名
	private String name;

	// 顾客是否进入店里
	private volatile boolean enter;

	// 顾客理过的次数
	private long times;

	public Customer(int id) {
		this.id = id;
		this.name = "customer-" + id;
	}

	public int getId() {
		return id;
	}

	public String getName() {
		return name;
	}

	/**
	 * 顾客理发中
	 * 
	 * @throws InterruptedException
	 */
	public void enjoy() throws InterruptedException {
		System.out.println(name + " is enjoying");
		process(100);
		times++;
		System.out.println(name + " is finished, times " + times);
	}

	public long getTimes() {
		return times;
	}

	/**
	 * 顾客尝试叫醒理发师，成功叫醒理发师的顾客能得到理发
	 * 
	 * @param waitChairs
	 * @param barber
	 * @return
	 * @throws InterruptedException
	 */
	public boolean awake(Chair waitChairs, Barber barber) throws InterruptedException {
		ReentrantLock lock = barber.getLock();
		WorkChair workChair = barber.getWorkChair();
		// 在这里如果理发椅有人，则理发师一定醒着并在理发中，则该顾客放弃叫醒理发师，继续保持等待
		if (!workChair.isFull()) {
			if (lock.tryLock()) {
				try {
					if (!workChair.isFull()) {
						// 顾客先尝试坐在理发椅上
						workChair.sit(this);
						// 然后再离开等待椅
						waitChairs.leave(this);
						System.out.println(name + " has left wait-chair, and sit on the work-chair");
						// 之后通知理发师
						barber.getCondition().signal();
						return true;
					}
				} finally {
					if (lock.isLocked())
						lock.unlock();
				}
			}

		}
		return false;
	}

	public void outdoor() throws InterruptedException {
		process(50);
	}

	public void dosth() throws InterruptedException {
		process(30);
	}

	// cost越小，越接近高并发
	private void process(int cost) throws InterruptedException {
		Thread.sleep(cost * ((int) (Math.random() * 9) + 1));
	}

	/**
	 * 顾客离开理发椅，并走出理发店
	 * 
	 * @param barber
	 * @throws InterruptedException
	 */
	public void bye(Barber barber) throws InterruptedException {
		barber.getWorkChair().leave();
		leave();
	}

	public void leave() {
		this.enter = false;
	}

	public void enter() {
		this.enter = true;
	}

	public boolean isEntered() {
		return enter;
	}

}
```

椅子（等待椅）：

```java
package com.chain.test.day07;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

/**
 * 椅子
 * 
 * 不同类型的椅子看成一个整体
 * 
 * 类似一个阻塞队列
 * 
 * @author chain
 *
 */
public class Chair {

	// 已经坐在椅子上的顾客
	private Map<Integer, Customer> customers;

	// 已经坐在椅子上的顾客的数量
	protected int n;

	// 该种类椅子总共的数量
	protected volatile int limit;

	// 椅子类型
	private String name;

	public Chair(String name, int limit) {
		this.name = name;
		this.limit = limit;
		this.customers = new ConcurrentHashMap<>(limit << 1);
	}

	public String getName() {
		return name;
	}

	/**
	 * 顾客尝试坐下
	 * 
	 * @throws InterruptedException
	 */
	public synchronized void sit(Customer customer) throws InterruptedException {
		// 避免直接使用ConcurrentHashMap.size()
		while (n >= limit)
			wait();
		customers.put(customer.getId(), customer);
		++n;
		notify();
	}

	/**
	 * 顾客尝试离开
	 * 
	 * @throws InterruptedException
	 * 
	 */
	public synchronized void leave(Customer customer) throws InterruptedException {
		while (n <= 0)
			wait();
		customers.remove(customer.getId());
		--n;
		notify();
	}

	/**
	 * 获得椅子上的顾客
	 * 
	 * @return
	 */
	public Customer get(Customer customer) {
		return customers.get(customer.getId());
	}

	/**
	 * 椅子是否坐满了
	 * 
	 * @return
	 */
	public boolean isFull() {
		return n >= limit;
	}

	/**
	 * 已经坐在椅子上的顾客
	 * 
	 * @return
	 */
	public int current() {
		return n;
	}

}
```

理发椅：

```java
package com.chain.test.day07;

/**
 * 理发椅
 * 
 * @author chain
 *
 */
public class WorkChair extends Chair {

	private Customer customer;

	public WorkChair(String name, int limit) {
		super(name, limit);
	}

	@Override
	public synchronized void sit(Customer customer) throws InterruptedException {
		// 避免直接使用ConcurrentHashMap.size()
		while (n >= limit)
			wait();
		this.customer = customer;
		++n;
		notify();
	}

	@Override
	public synchronized void leave(Customer customer) throws InterruptedException {
		while (n <= 0)
			wait();
		this.customer = null;
		--n;
		notify();
	}

	/**
	 * 获得椅子上的顾客
	 * 
	 * @return
	 */
	public Customer get() {
		return customer;
	}

	/**
	 * 顾客离开椅子
	 * 
	 * @throws InterruptedException
	 */
	public synchronized void leave() throws InterruptedException {
		leave(null);
	}

}
```

主测试方法：

```java
package com.chain.test.day07;

import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.Semaphore;

import org.junit.Test;

/**
 * 主测试方法
 * 
 * @author chain
 *
 */
public class Main {

	private BarberThread barberThread;
	private CustomerThread[] customerThreads;

	private Semaphore door;

	private Chair waitChairs;
	private WorkChair workChair;

	private Barber barber;
	private Customer[] customers;

	private int customersNum;

	private CyclicBarrier barrier;

	private MonitorThread monitorThread;

	@Test
	public void test() throws Exception {
		customersNum = (int) (Math.random() * 8) + 2;

		System.out.println("there is " + customersNum + " customers");

		barrier = new CyclicBarrier(customersNum + 1);

		door = new Semaphore(1);

		workChair = new WorkChair("work-chair", 1);

		// 等待椅的数量小于顾客的数量
		waitChairs = new Chair("wait-chair", customersNum / 2);

		System.out.println("there is " + customersNum / 2 + " wait-chairs");

		barber = new Barber(workChair);
		customers = new Customer[customersNum];
		for (int i = 0; i < customersNum; i++)
			customers[i] = new Customer(i);

		barberThread = new BarberThread();
		customerThreads = new CustomerThread[customersNum];
		for (int i = 0; i < customersNum; i++)
			customerThreads[i] = new CustomerThread(customers[i]);

		monitorThread = new MonitorThread();

		barberThread.start();
		for (int i = 0; i < customersNum; i++)
			customerThreads[i].start();

		monitorThread.start();

		barberThread.join();
		for (int i = 0; i < customersNum; i++)
			customerThreads[i].join();

		calcResult();
	}

	private void calcResult() {
		System.out.println();
		System.out.println("============= calc result =============");
		long barberTimes = barber.getTimes();
		long customersTimes = 0;
		for (int i = 0; i < customersNum; i++) {
			long ct = customers[i].getTimes();
			customersTimes += ct;
			System.out.println(customers[i].getName() + " times：" + ct);
		}
		System.out.println("barber times：" + barberTimes);
		System.out.println("customers total times：" + customersTimes);
		System.out.println("correct：" + (barberTimes == customersTimes));
	}

	private class MonitorThread extends Thread {

		@SuppressWarnings("deprecation")
		@Override
		public void run() {
			try {
				// 持续运行10分钟
				Thread.sleep(10 * 60 * 1000);
			} catch (InterruptedException e) {
				throw new RuntimeException(e);
			}

			barberThread.close();
			for (int i = 0; i < customersNum; i++)
				customerThreads[i].close();

			barberThread.interrupt();
			for (int i = 0; i < customersNum; i++)
				customerThreads[i].interrupt();

			barberThread.stop();
			for (int i = 0; i < customersNum; i++)
				customerThreads[i].stop();
		}

	}

	private class CustomerThread extends Thread {

		private volatile boolean status;

		private Customer customer;

		public CustomerThread(Customer customer) {
			this.customer = customer;
		}

		@Override
		public void run() {
			try {
				status = true;
				barrier.await();
				String name = customer.getName();
				System.out.println(name + " is ready");
				while (status) {
					System.out.println(name + " try to enter this barbershop");
					try {
						if (!door.tryAcquire())
							continue;
						customer.enter();
						System.out.println(name + " has entered this barbershop");
						try {
							if (waitChairs.isFull()) {
								System.out.println(name + " failed to sit for full");
								customer.leave();
								continue;
							}
							waitChairs.sit(customer);
							System.out.println(name + " has sit on one wait-chair, current " + waitChairs.current());
						} finally {
							door.release();
						}

						int max = (int) (Math.random() * 5) + 5;
						int wait = 0;
						while (wait++ < max && !customer.awake(waitChairs, barber))
							customer.dosth();

						// 此时等的不耐烦的顾客可能正要走时却成功叫醒了理发师
						if (wait >= max && waitChairs.get(customer) != null) {
							waitChairs.leave(customer);
							customer.leave();
							System.out.println(name + " couldn't wait any more");
						}
					} finally {
						while (customer.isEntered())
							customer.dosth();
						System.out.println(name + " leave this barbershop");
						customer.outdoor();
					}
				}
			} catch (Exception e) {
				throw new RuntimeException(e);
			}
		}

		public void close() {
			this.status = false;
		}

	}

	private class BarberThread extends Thread {

		private volatile boolean status;

		@Override
		public void run() {
			try {
				status = true;
				barrier.await();
				System.out.println(barber.getName() + " is ready");
				while (status) {
					barber.sleep();
					barber.work();
				}
			} catch (Exception e) {
				throw new RuntimeException(e);
			}
		}

		public void close() {
			this.status = false;
		}
	}

}
```

程序持续运行较长时间后的某一个片段截图（忙碌的理发师，只能小憩一会）：

![运行片段](/uploads/multithreads-basic-knowledge/38.png)

程序正常退出时的运行结果：

![运行结果](/uploads/multithreads-basic-knowledge/39.png)

每个顾客理发的次数也是近似相等的。

原先的代码会导致运行一段时间后出现一个顾客都无法得到理发师理发机会的情况，排查了好久才发现原来的代码会出现一个极端情况，即顾客坐上理发椅后刚换醒理发师，理发师接着自己又睡过去了。从打印输出的结果也可以看出，在某一个时刻，顾客刚刚sit到work-chair，而后紧接着理发师sleep了，再之后就出现所有的顾客无法理发的情况。现在已经解决了这个问题，并重新更换了截图和相关代码。

## 总结思考

将复杂问题分解，分解的过程中自己拿起笔，用文字和图片将抽象的问题形象化，具体化，清晰化。

多线程问题的锻炼也能提高自己在编码时思维的严谨，简单的步骤也需要打磨很久。

在多线程和并发下，各种意想不到的情况都会发生，需要平时的经验积累和深入思考。

[源码](https://github.com/ChainGit/some-tests/tree/master/test09)