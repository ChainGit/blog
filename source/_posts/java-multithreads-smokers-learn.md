---
title: Java多线程之吸烟者问题
date: 2017/12/08
categories: 学习
tags:
- 学习
- Java
- 多线程
---

Java多线程之吸烟者问题
============================
吸烟者问题有点类似生产者消费者问题，但是生产者的生产的商品是多样的，商品不一样，对应的消费者也不一样。

[操作系统原理之进程和线程管理](https://www.leechain.top/blog/2017/08/25-operation-system-multithreads.html)
[Java的多线程和并发知识纲要](https://www.leechain.top/blog/2017/08/21-java-concurrency-brief.html)
[Java多线程之生产者和消费者模式](https://www.leechain.top/blog/2017/08/27-java-multithreads-producer-and-consumer-learn.html)
[Java多线程之哲学家就餐问题](https://www.leechain.top/blog/2017/11/20-java-multithreads-philosopher-dinner-learn.html)
[Java多线程之读者写者问题](https://www.leechain.top/blog/2017/11/25-java-multithreads-read-write-learn.html)

## 问题描述

假设有三个抽烟者和一个供应者。每个抽烟者不停地卷烟并抽掉它，但是要卷起并抽掉一支烟，抽烟者需要同时具备三种材料：烟草、纸和胶水。三个抽烟者中，第一个拥有无限烟草、第二个拥有无限纸，第三个拥有无限胶水，抽烟者之间互不提供材料，各自缺失的另外两种材料仅由供应者提供。供应者能无限地提供这三种材料，但是供应者每次只会将其中的两种材料放在桌子上，并等待抽烟者的信号。此时拥有缺失的那种材料的抽烟者可以从桌子上拿走材料，卷上一根烟并抽掉它。抽烟者卷好烟后（不需要等到抽完）就会给供应者一个信号，告诉供应者可以了，供应者就会清理并随机再选两种材料放在桌上，这种过程一直重复。

![image](/uploads/multithreads-basic-knowledge/34.jpg)

## 问题分析

供应者和吸烟者是同步关系，当供应者提供好材料时，吸烟者可以进行卷烟操作；吸烟者卷好烟后，供应者就可以重新放置材料。
但是，供应者每次只能满足一个吸烟者，也就是说吸烟者之间的卷烟动作是互斥的。

这里的材料可以用信号量来表示，表示的方法可以有两种。

第一种：烟草、纸、胶水各自为单独信号量，每个吸烟者等待各自需要的两个材料（两个信号量）。
第二种：烟草+纸，纸+胶水，胶水+烟草组合作为信号量，每个吸烟者等待各自需要的那个信号量。

## 我的解决

无论是烟草、纸、胶水的单独信号量，还是烟草+纸，纸+胶水，胶水+烟草的组合信号量，其实本质上都是一样的。

### 公共类和方法

供应者抽象类：

```java
package com.chain.test.day06;

/**
 * 供应者抽象类
 * 
 * @author chain
 *
 */
public abstract class AbstractSupplier {

	/**
	 * 供应者提供材料
	 * 
	 * @throws Exception
	 */
	public abstract void put() throws Exception;
}
```

吸烟者抽象类：

```java
package com.chain.test.day06;

/**
 * 吸烟者抽象类
 * 
 * @author chain
 *
 */
public abstract class AbstractSmoker {

	/**
	 * 吸烟者等待材料
	 * 
	 * @throws Exception
	 */
	public abstract void take() throws Exception;

}
```

材料的抽象类：

```java
package com.chain.test.day06;

import java.util.HashMap;
import java.util.Map;

/**
 * 材料的抽象类
 * 
 * @author chain
 *
 */
public abstract class AbstractMaterials {

	protected Map<Integer, String> list;

	public AbstractMaterials() {
		list = new HashMap<>();
	}

	/**
	 * 获取材料的名称
	 * 
	 * @param id
	 * @return
	 */
	public abstract String get(int id);

}
```

主测试方法、吸烟者、提供者、材料以及信号量的具体实现参看文末的源码链接。

### 具体做法

材料、吸烟者、提供者、信号量需要根据不同的情况一一对应。

#### 烟草、纸、胶水的单独信号量

提供者每次提供三种材料中的两个，比如烟草、纸。吸烟者等待自己需要的两种材料。

test.properties：

```
smoker.class=com.chain.test.day06.SmokerA
supplier.class=com.chain.test.day06.SupplierA
semaphore.class=com.chain.test.day06.SemaphoreA
material.class=com.chain.test.day06.MaterialsA
```

吸烟者：
```java
package com.chain.test.day06;

/**
 * 吸烟者
 * 
 * 烟草、纸、胶水的单独信号量
 * 
 * @author chain
 *
 */
public class SmokerA extends AbstractSmoker {

	private AbstractSemaphore[] takes;

	public SmokerA(int id, AbstractSemaphore[] semaphores, AbstractSemaphore finish, AbstractMaterials materials) {
		super(id, semaphores, finish, materials);
		takes = semaphores;
	}

	@Override
	public void take() throws Exception {
		int mlen = takes.length;
		int notTake = getId();

		StringBuffer sb = new StringBuffer();
		for (int i = 0; i < mlen; i++) {
			if (i == notTake)
				continue;
			sb.append(materials.get(i) + " ");
		}

		String msg = sb.toString();

		// 等待供应者的材料
		System.out.println(getName() + " wait for materials： " + msg);

		for (int i = 0; i < mlen; i++) {
			if (i == notTake)
				continue;
			takes[i].P();
		}

		System.out.println(getName() + " get materials： " + msg);

		// 告知供应者自己的到所需要的材料
		finish.V();

		System.out.println(getName() + " signal supplier");

		process();
	}

	/**
	 * 卷烟和吸烟
	 */
	private void process() {
		try {
			Thread.sleep(100 * ((int) (Math.random() * 9 + 1)));
		} catch (InterruptedException e) {
			throw new RuntimeException(e);
		}
		System.out.println(getName() + " cigarette and smoke");
	}

}
```

提供者：

```java
package com.chain.test.day06;

import java.util.Random;

/**
 * 供应者
 * 
 * 烟草、纸、胶水的单独信号量
 * 
 * @author chain
 *
 */
public class SupplierA extends AbstractSupplier {

	private Random random;

	public SupplierA(int id, AbstractSemaphore[] semaphores, AbstractSemaphore finish, AbstractMaterials materials) {
		super(id, semaphores, finish, materials);
		this.random = new Random();
	}

	@Override
	public void put() throws Exception {
		int mlen = semaphores.length;
		int notMake = Math.abs(random.nextInt()) % mlen;

		System.out.println(getName() + " make materials");

		StringBuffer sb = new StringBuffer();
		sb.append(getName() + " has put materials： ");
		for (int i = 0; i < mlen; i++) {
			if (i == notMake)
				continue;
			sb.append(materials.get(i) + " ");
		}

		String msg = sb.toString();

		process();

		for (int i = 0; i < mlen; i++) {
			if (i == notMake)
				continue;
			semaphores[i].V();
		}

		System.out.println(msg);

		finish.P();
	}

	// 处理其他事情
	private void process() {
		try {
			Thread.sleep(100 * ((int) (Math.random() * 9 + 1)));
		} catch (InterruptedException e) {
			throw new RuntimeException(e);
		}
	}

}
```

材料：

```java
package com.chain.test.day06;

/**
 * 材料
 * 
 * 烟草、纸、胶水的单独信号量
 * 
 * @author chain
 *
 */
public class MaterialsA extends AbstractMaterials {

	public MaterialsA() {
		// 编译器会自动添加
		// super();
		init();
	}

	private void init() {
		// 烟草
		list.put(0, "tobacco");
		// 胶水
		list.put(1, "gluewater");
		// 纸
		list.put(2, "paper");
	}

	@Override
	public String get(int id) {
		return list.get(id);
	}

}
```

![测试结果](/uploads/multithreads-basic-knowledge/35.png)

#### 烟草+纸，纸+胶水，胶水+烟草的组合信号量

提供者每次提供三种材料组合中的任意一个，比如烟草+纸。吸烟者等待自己需要的材料组合。

test.properties：

```java
semaphore.class=com.chain.test.day06.SemaphoreA
smoker.class=com.chain.test.day06.SmokerB
supplier.class=com.chain.test.day06.SupplierB
material.class=com.chain.test.day06.MaterialsB
```

供应者：

```java
package com.chain.test.day06;

import java.util.Random;

/**
 * 供应者
 * 
 * 烟草+纸，纸+胶水，胶水+烟草的组合信号量
 * 
 * @author chain
 *
 */
public class SupplierB extends AbstractSupplier {

	private Random random;

	public SupplierB(int id, AbstractSemaphore[] semaphores, AbstractSemaphore finish, AbstractMaterials materials) {
		super(id, semaphores, finish, materials);
		this.random = new Random();
	}

	@Override
	public void put() throws Exception {
		int toMake = Math.abs(random.nextInt()) % semaphores.length;
		String material = materials.get(toMake);

		System.out.println(getName() + " make materials");

		// 提供者放置材料
		process();

		semaphores[toMake].V();

		System.out.println(getName() + " has put materials： " + material);

		// 等待吸烟者取走材料
		finish.P();
	}

	// 处理其他事情
	private void process() {
		try {
			Thread.sleep(100 * ((int) (Math.random() * 9 + 1)));
		} catch (InterruptedException e) {
			throw new RuntimeException(e);
		}
	}

}
```

吸烟者：

```java
package com.chain.test.day06;

/**
 * 吸烟者
 * 
 * 烟草+纸，纸+胶水，胶水+烟草的组合信号量
 * 
 * @author chain
 *
 */
public class SmokerB extends AbstractSmoker {

	private AbstractSemaphore take;

	public SmokerB(int id, AbstractSemaphore[] semaphores, AbstractSemaphore finish, AbstractMaterials materials) {
		super(id, semaphores, finish, materials);
		take = semaphores[id];
	}

	@Override
	public void take() throws Exception {
		String material = materials.get(getId());

		System.out.println(getName() + " wait for material " + material);

		// 等待供应者的材料
		take.P();

		System.out.println(getName() + " get material " + material);

		// 告知供应者自己的到所需要的材料
		finish.V();

		System.out.println(getName() + " signal supplier");

		process();
	}

	/**
	 * 卷烟和吸烟
	 */
	private void process() {
		try {
			Thread.sleep(100 * ((int) (Math.random() * 9 + 1)));
		} catch (InterruptedException e) {
			throw new RuntimeException(e);
		}
		System.out.println(getName() + " cigarette and smoke");
	}

}
```

材料：

```java
package com.chain.test.day06;

/**
 * 材料
 * 
 * 烟草+纸，纸+胶水，胶水+烟草的组合信号量
 * 
 * @author chain
 *
 */
public class MaterialsB extends AbstractMaterials {

	public MaterialsB() {
		// 编译器会自动添加
		// super();
		init();
	}

	private void init() {
		// 烟草+纸
		list.put(0, "tobacco+paper");
		// 胶水+纸
		list.put(1, "gluewater+paper");
		// 烟草+胶水
		list.put(2, "gluewater+tobacco");
	}

	@Override
	public String get(int id) {
		return list.get(id);
	}

}
```

第二种方式运行结果和第一种效果一致，但是第二种代码更简洁，运行速度也会快一点。

![测试结果](/uploads/multithreads-basic-knowledge/36.png)

## 总结思考

无论是烟草、纸、胶水的单独信号量，还是烟草+纸，纸+胶水，胶水+烟草的组合信号量，其实本质上都是一样的，差别就在于信号量具体的PV操作。
但是使用组合方式，是这个问题比较好的解决方案。

吸烟者问题有点类似生产者消费者问题，但是生产出的商品是多样的，消费者也是多样的。

搞定经典问题，做到举一反三。

[源码](https://github.com/ChainGit/some-tests/tree/master/test08)



