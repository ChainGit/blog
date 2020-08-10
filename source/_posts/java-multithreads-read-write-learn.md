---
title: Java多线程之读者写者问题
date: 2017/11/25
categories: 学习
tags:
- 学习
- Java
- 多线程
---

Java多线程之读者写者问题
============================
读者写者问题也是经典的多线程问题，在实际生产应用中也是存在必要的意义的。

[操作系统原理之进程和线程管理](https://www.leechain.top/blog/2017/08/25-operation-system-multithreads.html)
[Java的多线程和并发知识纲要](https://www.leechain.top/blog/2017/08/21-java-concurrency-brief.html)
[Java多线程之生产者和消费者模式](https://www.leechain.top/blog/2017/08/27-java-multithreads-producer-and-consumer-learn.html)
[Java多线程之哲学家就餐问题](https://www.leechain.top/blog/2017/11/20-java-multithreads-philosopher-dinner-learn.html)

## 问题描述

有读者和写者两组并发进程，共享一个文件。当两个或以上的读进程同时访问共享数据时不会产生副作用，但若某个写进程和其他进程（读进程或写进程）同时访问共享数据时则可能导致数据不一致的错误。

![读者和写者](/uploads/multithreads-basic-knowledge/22.png)

因此要求：
①允许多个读者可以同时对文件执行读操作；
②只允许一个写者往文件中写信息；
③任一写者在完成写操作之前不允许其他读者或写者工作；
④写者执行写操作前，应让已有的读者和写者全部退出。

## 问题分析

分析问题容易看出，读者和写者是互斥的，写者和写者也是互斥的，而读者和读者之间不存在互斥问题。

当读者和写者都存在时，就会出现优先级问题，到底是读的优先还是写的人优先。

因此，这个问题可以分为__读者优先、弱写者优先（公平竞争）、强写者优先__三种情况。

对每一种情况要具体问题具体分析，找出每种情况下的访问规则。

## 我的解决

不论当前的状态如何，任意写者进程与其他任何进程都互斥，用互斥信号量的PV操作即可。
读者的问题比较复杂，它必须实现与写者互斥的同时还要实现与其他读者的同步。

因此，仅仅简单的一对PV操作是无法解决的。
这里可以使用一个计数器count，用它来判断当前是否有读者读文件。
当有读者的时候写者是无法写文件的，此时读者会一直占用文件；当没有读者的时候写者才可以写文件。
同时不同的读者对该计数器的访问也应该是互斥的。

这里我假设了以下的情景：
写者进程针对share.txt不断进行写操作，读者进程则不断读取share.txt中的内容，并写到另一个文件中。
从而实现文件的实时拷贝，可以用于的场景比如：
写服务器不断的去记录log到同一个文件，然后其他服务器不断的去抓取这个log文件的内容拷贝到自己本地。

### 公共类和方法

读者的抽象类：

```java
package com.chain.test.day05;

import java.io.OutputStream;
import java.io.RandomAccessFile;

/**
 * 读者抽象类
 * 
 * @author chain
 *
 */
public abstract class AbstractReader {

	protected volatile RandomAccessFile file;
	protected OutputStream os;

	private int id;
	private String name;

	public AbstractReader(int id, RandomAccessFile file, OutputStream os) {
		this.id = id;
		this.name = "Reader-" + id;
		this.file = file;
		this.os = os;
	}

	public int getId() {
		return id;
	}

	public String getName() {
		return name;
	}

	/**
	 * 读文件
	 * 
	 * @throws Exception
	 */
	public abstract void read() throws Exception;

	/**
	 * 停止读取
	 */
	public abstract void stop();

	/**
	 * 开始读取
	 */
	public abstract void begin();
    
	/**
	 * 是否已完成工作
	 * 
	 * @return
	 */
	public abstract boolean isFinished();

	/**
	 * 休息一会
	 * 
	 * @throws InterruptedException
	 */
	public void rest() throws InterruptedException {
		System.out.println(getName() + " have a rest");
		Thread.sleep(100 * ((int) (Math.random() * 9) + 0));
	}

}
```

写者的抽象类：

```java
package com.chain.test.day05;

import java.io.RandomAccessFile;
import java.util.Random;

/**
 * 写者抽象类
 * 
 * @author chain
 *
 */
public abstract class AbstractWriter {

	protected volatile RandomAccessFile file;

	protected Random rand;

	private int id;
	private String name;

	public AbstractWriter(int id, RandomAccessFile file) {
		this.id = id;
		this.name = "Writer-" + id;
		this.file = file;
		this.rand = new Random();
	}

	public int getId() {
		return id;
	}

	public String getName() {
		return name;
	}

	/**
	 * 写文件
	 * 
	 * @throws Exception
	 */
	public abstract void write() throws Exception;

	/**
	 * 停止读取
	 */
	public abstract void stop();

	/**
	 * 开始读取
	 */
	public abstract void begin();

	/**
	 * 是否已完成工作
	 * 
	 * @return
	 */
	public abstract boolean isFinished();

	/**
	 * 休息一会
	 * 
	 * @throws InterruptedException
	 */
	public void rest() throws InterruptedException {
		System.out.println(getName() + " have a rest");
		Thread.sleep(100 * ((int) (Math.random() * 9) + 0));
	}
}
```

读者写者规则的抽象类：

```java
package com.chain.test.day05;

/**
 * 读者写者的规则的抽象类
 * 
 * @author chain
 *
 */
public abstract class AbstractReaderWriterMethod {

	/**
	 * 读者读文件
	 * 
	 * @param reader
	 * @throws Exception
	 */
	public abstract void read(AbstractReader reader) throws Exception;

	/**
	 * 写者写文件
	 * 
	 * @param writer
	 * @throws Exception
	 */
	public abstract void write(AbstractWriter writer) throws Exception;

}
```

主测试方法、Reader、Writer、读写规则、自定义信号量等的具体实现可以参看文末的源码链接。

### 读者优先

test.properties：

```java
readers.num=5
writers.num=5
method.class=com.chain.test.day05.ReaderWriterMethod01
file.path=C:\\Temps\\reader_writer
file.name=share.txt
```

```java
package com.chain.test.day05;

/**
 * 读者优先算法
 * 
 * 自定义信号量做法
 * 
 * @author chain
 *
 */
public class ReaderWriterMethod01 extends AbstractReaderWriterMethod {

	// 用于记录当前读者进程在读的数量
	private volatile int count;
	// 保证count的互斥操作
	private AbstractSemaphore mutex;
	// 用于保证读者和写者能互斥的访问文件
	private AbstractSemaphore rw;

	public ReaderWriterMethod01() {
		rw = new Semaphore01(1);
		mutex = new Semaphore01(1);
	}

	@Override
	public void read(AbstractReader reader) throws Exception {
		System.out.println(reader.getName() + " wait for reading");

		// 读者进程互斥访问count
		mutex.P();
		if (count == 0)
			// 阻止写者进程写
			rw.P();
		// 在执行读操作的读者进程+1
		++count;
		mutex.V();

		reader.read();

		mutex.P();
		--count;
		if (count == 0)
			// 允许写者进程写
			rw.V();
		mutex.V();

		System.out.println(reader.getName() + " finish reading");

		reader.rest();
	}

	@Override
	public void write(AbstractWriter writer) throws Exception {
		System.out.println(writer.getName() + " wait for writing");

		// 阻止读者进程读
		rw.P();
		writer.write();
		// 允许读者进程读
		rw.V();

		System.out.println(writer.getName() + " finish writing");

		writer.rest();
	}

}
```

在该算法中，读者进程是优先的，也就是说，当存在读者进程时，写者操作将被推后。而且只要有一个读者进程活跃（在读文件），随后而来的读者进程也是允许读取文件的。这样的方式下，可能会导致写者进程长时间等待，即存在__写者进程“饿死”的情况__。

如果读者和写者的数量都为1时，可以看到两者交替进行，比较融洽：

![读者和写者的数量都为1](/uploads/multithreads-basic-knowledge/26.png)

如果读者数量为1，写者数量为多个（2个），可以看到写者不会同时写，读者和写者也是互斥的，工作也是正常的：

![读者数量为1，写者为多个](/uploads/multithreads-basic-knowledge/27.png)

如果读者数量较少（2个），写者数量适中，可以看到大部分都是读者进程在执行，但是写者进程还是有机会进行写操作的：

![读者优先正常工作](/uploads/multithreads-basic-knowledge/23.png)

但是如果读者数量较多（5个），可以看到写者进程进行写操作的机会很少，基本是读者进程在进行读操作：

![写者饿死的情况](/uploads/multithreads-basic-knowledge/24.png)

程序正常结束运行后检查各个文件的信息，可以看到信息一致。

![文件信息](/uploads/multithreads-basic-knowledge/25.png)

修改方法中的信号量为JavaAPI信号量也是可以的，运行效果一致。

### （强）写者优先

要让写者能优先得到执行的机会，可以参照读者优先中read如何让读者优先的部分，对照设计写者write部分。

具体做法：
为了能做到写者优先，需要对读者部分增加一个队列信号量q，由写者进程控制这个队列信号量q。
每当有一个写进程在执行写操作时，队列就+1；直到队列为0，读进程才能进行读操作。
换句话说，只要有一个写进程得到了队列信号量q，那么之后就会一直占着q不放，直到所有的写进程都完成了写操作后才会释放q，这样的做法就能提高了写进程的优先级。

```java
readers.num=5
writers.num=5
method.class=com.chain.test.day05.ReaderWriterMethod04
file.path=C:\\Temps\\reader_writer
file.name=share.txt
```

```java
package com.chain.test.day05;

/**
 * 写者优先
 * 
 * 自定义信号量做法
 * 
 * @author chain
 *
 */
public class ReaderWriterMethod04 extends AbstractReaderWriterMethod {

	// 用于记录当前读者进程在读的数量
	private volatile int readCount;
	// 保证readCount的互斥操作
	private AbstractSemaphore readMutex;

	// 用于记录当前写者进程在写的数量
	private volatile int writeCount;
	// 保证writeCount的互斥操作
	private AbstractSemaphore writeMutex;

	// 用于保证读者和写者能互斥的访问文件
	private AbstractSemaphore w;

	private AbstractSemaphore q;

	public ReaderWriterMethod04() {
		w = new Semaphore01(1);
		q = new Semaphore01(1);
		writeMutex = new Semaphore01(1);
		readMutex = new Semaphore01(1);
	}

	@Override
	public void read(AbstractReader reader) throws Exception {
		System.out.println(reader.getName() + " wait for reading");

		// 只有当没有写进程在执行时，才能进行读操作
		q.P();

		readMutex.P();
		if (readCount == 0)
			w.P();
		++readCount;
		readMutex.V();

		q.V();

		reader.read();

		readMutex.P();
		--readCount;
		if (readCount == 0)
			w.V();
		readMutex.V();

		System.out.println(reader.getName() + " finish reading");

		reader.rest();
	}

	@Override
	public void write(AbstractWriter writer) throws Exception {
		System.out.println(writer.getName() + " wait for writing");

		writeMutex.P();
		if (writeCount == 0)
			q.P();
		++writeCount;
		writeMutex.V();

		w.P();
		writer.write();
		w.V();

		writeMutex.P();
		--writeCount;
		if (writeCount == 0)
			// 没有要写操作的写者，读者可以读文件
			q.V();
		writeMutex.V();

		System.out.println(writer.getName() + " finish writing");

		writer.rest();
	}

}
```

在该算法中，写者进程是优先的，也就是说，当存在写者进程时，读者操作将被推后。写操作之间通过w信号量保证互斥，读和写进程通过w信号量保持互斥，写进程通过q信号量控制写进程。这样的方式下，可能会导致读者进程长时间等待，即存在__读者进程“饿死”的情况__。

如果读者和写者的数量都为1时，可以看到两者交替进行，比较融洽：

![读者和写者的数量都为1](/uploads/multithreads-basic-knowledge/30.png)

如果读者和写者的数量都为2时，可以看到大部分都是写者进程在执行，但是读者进程还是有机会进行读操作的：

![读者和写者的数量都为2](/uploads/multithreads-basic-knowledge/31.png)

但是如果写者数量较多（3个），可以看到读者进程很少有机会进行读操作，基本是写者进程在进行写操作：

![写者饿死的情况](/uploads/multithreads-basic-knowledge/32.png)

程序正常结束运行后检查各个文件的信息，可以看到信息一致。

由于这是强写者优先，所以都为3的情况下，读进程读完所有的内容时间所需太长（读进程获得执行的机会太少），不过可以修改Reader中一次buffer的大小，加快读取的速度。

![文件信息](/uploads/multithreads-basic-knowledge/33.png)

### 读写公平（弱写者优先）

如果希望读写公平，即在多次操作下，读写操作的执行频率是差不多的。

具体的做法：在读者优先的基础上只增加一个信号量q就可以做到读写公平。

test.properties：

```java
readers.num=10
writers.num=10
method.class=com.chain.test.day05.ReaderWriterMethod05
file.path=C:\\Temps\\reader_writer
file.name=share.txt
```

```java
package com.chain.test.day05;

/**
 * 公平算法，在读者优先的基础上
 * 
 * 使用自定义信号量实现
 * 
 * @author chain
 *
 */
public class ReaderWriterMethod05 extends AbstractReaderWriterMethod {

	// 用于记录当前读者进程在读的数量
	private volatile int count;
	// 保证count的互斥操作
	private AbstractSemaphore mutex;
	// 用于保证读者和写者能互斥的访问文件
	private AbstractSemaphore w;
	// 用于保证只有在写进程不需要写文件时读进程才去读文件
	private AbstractSemaphore q;

	public ReaderWriterMethod05() {
		q = new Semaphore01(1);
		w = new Semaphore01(1);
		mutex = new Semaphore01(1);
	}

	@Override
	public void read(AbstractReader reader) throws Exception {
		System.out.println(reader.getName() + " wait for writer to have a rest");

		// 只有在写进程不需要写文件时读进程才去读文件
		q.P();

		System.out.println(reader.getName() + " wait for reading");

		mutex.P();
		if (count == 0)
			w.P();
		++count;
		mutex.V();

		// 写进程可以尝试去写文件
		// 但是写进程仍然需要去判断rw
		q.V();

		reader.read();

		mutex.P();
		--count;
		if (count == 0)
			w.V();
		mutex.V();

		System.out.println(reader.getName() + " finish reading");

		reader.rest();
	}

	@Override
	public void write(AbstractWriter writer) throws Exception {
		System.out.println(writer.getName() + " wait for reader to have a rest");

		// 写进程在读进程无需读取时进入
		q.P();

		System.out.println(writer.getName() + " wait for writing");

		w.P();

		writer.write();

		// 读进程可以进行读操作
		q.V();

		w.V();

		System.out.println(writer.getName() + " finish writing");

		writer.rest();
	}

}
```

这样修改后，即使有再多的读写进程，两种操作的频率是差不多的：

![公平算法](/uploads/multithreads-basic-knowledge/28.png)

程序正常结束运行后检查各个文件的信息，可以看到信息一致：

![文件信息](/uploads/multithreads-basic-knowledge/29.png)

## 总结思考

为了做到读者之间互斥，可以给读者的读操作增加一个信号量w。
为了做到读者和写者之间的互斥，可以使用上面的信号量w。

为了做到读者优先，可以个读者操作增加一个计数器read\_count，只要有一个读者能得到执行，就会阻止写者进行操作，从而其他的读者也能执行。直到没有读者去进行读操作时，写者才有机会去执行。

为了做到读写公平，可以在读者优先的基础上，给读者增加一个信号量q，这个信号量由写者控制。如果写者想执行操作，可以阻塞q，使得读者不能进行read\_count操作，进而也就不能进行读操作。这种情况也被称为弱写者优先。

为了做到写者优先，可以在读者优先的基础上，给写者也配备一个计数器write\_count，读者增加一个信号量q，这个信号量由写者控制。如果写者想执行操作，和弱写者优先一样，阻塞q即可。但是不同的是，一旦写者获得执行权，写者的计数器write\_count的作用和read\_count一样，会保证其他的写者也能得到执行。直到没有写者去进行写操作时，读者才有机会去执行。这种情况也被称为强写者优先。

[源码](https://github.com/ChainGit/some-tests/tree/master/test07)



