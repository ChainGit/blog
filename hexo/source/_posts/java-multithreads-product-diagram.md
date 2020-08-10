---
title: Java多线程之工作流网图
date: 2017/12/11
categories: 学习
tags:
- 学习
- Java
- 多线程
- 数据结构
---

Java多线程之工作流网图
============================
工作流网图基于前驱图实现。较之前几个问题，这个问题也是生产中的常见的模型，有人把这个模型称为AOE和AOV。

[操作系统原理之进程和线程管理](https://www.leechain.top/blog/2017/08/25-operation-system-multithreads.html)
[Java的多线程和并发知识纲要](https://www.leechain.top/blog/2017/08/21-java-concurrency-brief.html)
[Java多线程之生产者和消费者模式](https://www.leechain.top/blog/2017/08/27-java-multithreads-producer-and-consumer-learn.html)
[Java多线程之哲学家就餐问题](https://www.leechain.top/blog/2017/11/20-java-multithreads-philosopher-dinner-learn.html)
[Java多线程之读者写者问题](https://www.leechain.top/blog/2017/11/25-java-multithreads-read-write-learn.html)
[Java多线程之吸烟者问题](https://www.leechain.top/blog/2017/12/08-java-multithreads-smokers-learn.html)
[Java多线程之熟睡的理发师问题](https://www.leechain.top/blog/2017/12/10-java-multithreads-sleep-barber-learn.html)

## AOV和AOE

AOV网与工作流网（AOE）在模型结构上是相似的，它们都是基于图的结构，以节点表示活动，有向边表示流程的流向。

AOV网的有向边仅仅只表示活动的前后次序，也可以说是流程中的流程流向。工作流网中的有向边却不仅如此，它可以在每条边上设置不同的条件来决定活动的下一环节是什么，它的出度就不一定是所有有向边了。

因此，AOV网其实是工作流网（AOE）的一种特例，是一种全入全出的有向无环工作流网。

AOE网的定义是在带权有向图中若以顶点表示事件，有向边表示活动，边上的权值表示该活动持续的时间。
从定义上来看，很容易看出两种网的不同，__AOV网的活动以顶点表示，而AOE网的活动以有向边表示__。

纵观这两种网图，其实它们总体网络结构是一样的，仅仅是活动所表示的方式不同，因此从AOV网转换成AOE网应该是可行的。

## 问题描述

在生产中，比如玩具组装车间情景里，有这样的流水线（可能不准确，只是为了说明工作流网）：

![工作流网图](/uploads/multithreads-basic-knowledge/40.png)

某一个节点，比如P3，它需要等待P0和P2的工作完成后才能进行工作。

## 问题分析

AOE是边上有权值，节点值为“0”（只是为了理解，更准确的是“没有”）。
AOV可以转换成AOE，只要将路径上的权值设置为“0”，节点值设置为非“0”。

在这个问题下，工作流图的图存储：
1、如果使用邻接矩阵的话，存储边上的权值比较方便，但是去寻找节点的出度需要遍历操作。
2、如果使用邻接表的话，存储边上的权值比较麻烦，但是去寻找节点的出度比较方便。

进程之间的工作完成的通知可以使用信号量。

## 我的解决

我采用的工作流图的图存储方式是邻接矩阵，矩阵内的元素包含__权值和信号量__，这样既能方便找到某节点的出度和入度到某节点的源节点，也能迅速定位到这条边所使用的信号量。使用邻接表的话则不大容易实现。

将上图的工作流网图加上具体的边上权值，节点值为“0”，如下图：

![工作流网图](/uploads/multithreads-basic-knowledge/41.png)

### 源码实现

具体的详细源码可以参看文末的链接。

```java
private class NodeThread extends Thread {

	private int id;
	private String name;

	// 该节点生产时间
	private int time;

	// 生产的次数
	private int amount;

	private volatile boolean status;

	public NodeThread(int id, int time) {
		this.id = id;
		this.name = "node-" + id;
		this.time = time;
	}

	@Override
	public void run() {
		try {
			status = true;
			barrier.await();
			System.out.println(name + " is ready");
			long begin = System.currentTimeMillis();
			while (status) {
				System.out.println(name + " wait for semaphores");
				long start = System.currentTimeMillis();
				recv();
				System.out.println(name + " is working");
				process();
				long end = System.currentTimeMillis();
				System.out.println(name + " finish work, cost " + (end - start) + ", amount " + amount);
				send();
				System.out.println(name + " has sent semaphores");
			}
			long finish = System.currentTimeMillis();
			System.out.println(name + " IS DONE, TOTAL COST " + (finish - begin) + ", FINAL AMOUNT " + amount);
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
	}

	// 等待所有入该节点的信号量
	public void recv() throws InterruptedException {
		for (int i = 0; i < nums; i++) {
			Edge e = diagram[i][id];
			if (e == null)
				continue;
			e.semaphore.P();
		}
	}

	// 通知所有出该节点的信号量
	public void send() {
		for (int i = 0; i < nums; i++) {
			Edge e = diagram[id][i];
			if (e == null)
				continue;
			executor.schedule(new Msg(e.semaphore), e.value, TimeUnit.SECONDS);
		}
	}

	// 生产该节点负责的部分
	public void process() throws InterruptedException {
		Thread.sleep(time * 1000);
		if (++amount == goods)
			status = false;
	}

}
```

### 结果分析

#### P7和P0不相连，AOE

P7和P0不相连，AOE，运行过程中某一个片段情况如下图：

![P7和P0不相连，AOE，运行过程中某一个片段情况](/uploads/multithreads-basic-knowledge/42.png)

从运行片段情况可以看出，整个网图第一次生产的过程耗时是最长的，花费了14012毫秒，同时也就是关键路径P0->P2->P3->P5->P7所耗费的时间。
由于整个网络的生产是并行进行的，所以之后的生产速度就快了很多，这也是流水线生产模式的优势。

如果生产的产品数量是固定的，比如10件，即每个节点生产10次。
此时观察运行结果，可以看到各个节点结束运行的先后顺序是：P0->P2->P1->P4->P3->P6->P5->P7。这个结束的顺序，对照上面的流图分析也是吻合的。

生产10件产品的总时间是14018毫秒（稍大于14秒）。

#### P7和P0相连，AOE

如果将P7和P0相连，即增加一条图的有向边P7->P0。
也就意味着P0必须等待P7生产完才能生产，那么整个流水线就变成同时只能生产一个完整产品。

假设第一次执行时P0无需等待P7的信号。

此时的运行过程中某一个片段情况如下图：

![P7和P0相连，AOE，运行过程中某一个片段情况](/uploads/multithreads-basic-knowledge/43.png)

第一次循环时各个节点的执行时间与流图吻合，但是之后的循环却显示出每个节点等待的时间等于关键路径所需要的时间，如下图所示：

![P7和P0相连，AOE，等于关键路径所需要的时间](/uploads/multithreads-basic-knowledge/44.png)

P7和P0相连，AOE，生产10件产品的总时间：

![P7和P0相连，AOE，生产10件产品的总时间](/uploads/multithreads-basic-knowledge/45.png)

此时观察运行结果，每次循环的执行顺序是固定的：P0->P2->P1->P4->P3->P6->P5->P7，与流图吻合。

生产10件产品的总时间是140081毫秒（约14*10秒，远大于14秒）。

#### P7和P0不相连，AOV

如果更改每个edge的权值为“0”，节点值为非“0”，那么AOE就转变成AOV，如下图：

![转换成AOV的工作流网图](/uploads/multithreads-basic-knowledge/46.png)

此时的运行过程中某一个片段情况（也是第一次循环时各个节点的执行时间，耗时15秒）如下图：

![P7和P0不相连，转换成AOV，运行过程中某一个片段情况](/uploads/multithreads-basic-knowledge/49.png)

P7和P0不相连，AOV，生产10件产品的总时间：

![P7和P0不相连，转换成AOV，生产10件产品的总时间](/uploads/multithreads-basic-knowledge/47.png)

此时观察运行结果，可以看到各个节点结束运行的先后顺序是：P0->P2->P1->P4->P3->P5->P6->P7，与流图吻合。

生产10件产品的总时间是60023毫秒（约60秒，15+(2+3)*9）。

#### P7和P0相连，AOV

在AOV的情况下，如果P7和P0相连的话，工作情况和AOE下P7和P0相连的情况是类似的。

P7和P0相连，AOV，生产10件产品的总时间：

![P7和P0相连，转换成AOV，生产10件产品的总时间](/uploads/multithreads-basic-knowledge/48.png)

此时观察运行结果，每次循环的执行顺序是固定的：P0->P2->P1->P4->P3->P5->P6->P7，与流图吻合。

生产10件产品的总时间是150044毫秒（约15*10秒）。

## 总结思考

AOE示意图：
![AOE示意图](/uploads/multithreads-basic-knowledge/50.png)

AOV示意图：
![AOV示意图](/uploads/multithreads-basic-knowledge/51.png)

具体AOE和AOV的知识还是等自己深入学习到数据结构--图再说吧，水平还不足请谅解。

多线程实现工作流图，也是线程之间的同步和协作的例子，还是比较有意思的哈。同时自己的数据结构知识还有待加强，学的越多圆圈越大，不知道的也越多。

[源码](https://github.com/ChainGit/some-tests/tree/master/test10)