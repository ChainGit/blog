---
title: Spring的事务提交和回滚
date: 2017/09/26
categories:
- 学习
tags:
- 转载
- 学习
- Java
- Spring
- 知识点
---

Spring的事务提交和回滚
======================
Spring是很多框架的大管家，凭借精巧的结构而整合了很多其他框架。常见的比如SSH、SSM等都用Spring作为“管家”。

AOP和IOC是Spring的灵魂所在，这里主要讨论的是Spring的事务，事务的提交和回滚也是面试常问的问题。

这两天整理了一下一些不错的博客，了解一下Spring的事务提交和回滚的基本知识和使用。

## 什么是事务

先讲一个日常生活中最常做的事（尽管现在移动支付很火热）：__取钱__。

比如去ATM机取1000块钱，大体有两个步骤：
1） 首先输入密码，然后输入取款金额1000，接着银行数据库里你的银行卡扣掉1000元；
2） 最后ATM吐出1000元纸币。

__这两个步骤必须是要么都执行要么都不执行__。

如果银行卡扣除了1000块但是ATM出钱失败的话，你将会损失1000元；
如果银行卡扣钱失败但是ATM却出了1000块，那么银行将损失1000元。

所以，如果一个步骤成功另一个步骤失败对双方都不是好事，如果不管哪一个步骤失败了以后，整个取钱过程都能__回滚__，也就是完全取消所有操作的话，这对双方都是没有问题的。

所以，就诞生了事务。事务（Transaction）就是用来解决类似问题的。

> 事务是一系列的动作，它们综合在一起才是一个完整的工作单元，这些动作必须全部完成，如果有一个失败的话，那么事务就会回滚到最开始的状态，仿佛什么都没发生过一样。

在企业级应用程序开发中，事务管理必不可少的技术，用来确保数据的完整性和一致性。

## 核心接口

Spring事务管理的实现有许多细节，如果对整个接口框架有个大体了解会非常有利于我们理解事务，下面通过讲解Spring的事务接口来了解Spring实现事务的具体策略。

Spring事务管理涉及的接口的联系如下：

![image](http://img.blog.csdn.net/20160324011156424)

具体的实现关系如下：

![image](/uploads/java-spring-transaction-and-rollback/5.jpg)

Spring所有的事务管理策略类都继承自org.springframework.transaction.PlatformTransactionManager接口。

PlatformTransactionManager：

```java
public interface PlatformTransactionManager {

	TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

	void commit(TransactionStatus status) throws TransactionException;

	void rollback(TransactionStatus status) throws TransactionException;
}
```

Spring提供了这个接口，交由具体的数据源如HibernateTransactionManager实现具体事务的提交(commit)和回滚(rollback)操作。

具体可以参看上面的实现关系图。

## 实现方式

Spring支持__编程式事务__管理和__声明式事务__管理两种方式。

也有人认为有五种方式，其实是具体实现的方式的不同，大致可以分为以上两种。

五种方式分别为：
第一种方式：每个Bean都有一个代理；
第二种方式：所有Bean共享一个代理基类；
第三种方式：使用拦截器；
第四种方式：使用tx标签配置的拦截器；
第五种方式：使用注解。

具体配置的方法[参考](Spring事务配置的五种方式)。

编程式事务管理使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。
对于编程式事务管理，Spring推荐使用TransactionTemplate。

声明式事务管理建立在AOP之上的，其本质是对方法前后进行拦截（也可以理解为代理）。
在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明(或通过基于@Transactional注解的方式)，便可以将事务规则应用到业务逻辑中。

显然声明式事务管理要优于编程式事务管理，这也正是Spring倡导的__非侵入式__的开发方式。

声明式事务管理使业务代码不受污染，一个普通的POJO对象，只要加上注解就可以获得完全的事务支持。
和编程式事务相比，声明式事务唯一不足地方是，后者的最细__粒度__只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。
但是即便有这样的需求，也存在很多变通的方法，比如，可以将需要进行事务管理的代码块独立为方法等等。

声明式事务管理的两种常用的__声明方式__具体指的是：
一种是基于tx和aop名字空间的xml配置文件；
另一种就是基于@Transactional注解。

显然基于注解的方式更简单易用，更清爽。

## 事务特性

在PlatfromTransactionManager中，又使用了TransactionDefinition接口和TransactionStatus接口中定义的属性。

这两个接口又包含了事务属性如__隔离规则、回滚规则，传播行为，事务超时与只读__。

![image](http://img.blog.csdn.net/20160325003448793)

接下来详细的看一下接口内容具体表示的意义：

### TransactionDefinition接口

![image](/uploads/java-spring-transaction-and-rollback/1.png)

#### 事务隔离规则

隔离级别是指若干个__并发的事务__之间的隔离程度。

TransactionDefinition 接口中定义了五个表示隔离级别的常量：

```java
TransactionDefinition.ISOLATION_DEFAULT：这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是TransactionDefinition.ISOLATION_READ_COMMITTED。
TransactionDefinition.ISOLATION_READ_UNCOMMITTED：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读，不可重复读和幻读，因此很少使用该隔离级别。比如PostgreSQL实际上并没有此级别。
TransactionDefinition.ISOLATION_READ_COMMITTED：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。
TransactionDefinition.ISOLATION_REPEATABLE_READ：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。该级别可以防止脏读和不可重复读。
TransactionDefinition.ISOLATION_SERIALIZABLE：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。
```

接着还有一个表格能归纳一下上面的关系。

![image](/uploads/java-spring-transaction-and-rollback/2.png)

这里再补充一些概念：

__ACID：__

ACID，指数据库事务正确执行的四个基本要素的缩写。包含：

> 原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）。

一个支持事务（Transaction）的数据库，必需要具有这四种特性，否则在事务过程（Transaction processing）当中无法保证数据的正确性，交易过程极可能达不到交易方的要求。

1）原子性
整个事务中的所有操作，要么全部完成，要么全部不完成，不可能停滞在中间某个环节。
事务在执行过程中发生错误，会被回滚（rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。

2）一致性
一个事务可以封装状态改变（除非它是一个只读的）。事务必须始终保持系统处于一致的状态，不管在任何给定的时间并发事务有多少。
也就是说：如果事务是并发多个，系统也必须如同串行事务一样操作。其主要特征是保护性和不变性(Preserving an Invariant)，以转账案例为例，假设有五个账户，每个账户余额是100元，那么五个账户总额是500元，如果在这个5个账户之间同时发生多个转账，无论并发多少个，比如在A与B账户之间转账5元，在C与D账户之间转账10元，在B与E之间转账15元，五个账户总额也应该还是500元，这就是保护性和不变性。

3）隔离性
隔离状态执行事务，使它们好像是系统在给定时间内执行的唯一操作。
如果有两个事务，运行在相同的时间内，执行相同的功能，事务的隔离性将确保每一事务在系统中认为只有该事务在使用系统。
这种属性有时称为串行化，为了防止事务操作间的混淆，必须串行化或序列化请求，使得在同一时间仅有一个请求用于同一数据。

4）持久性
在事务完成以后，该事务对数据库所作的更改便持久的保存在数据库之中，并不会被回滚。
由于一项操作通常会包含许多子操作，而这些子操作可能会因为硬件的损坏或其他因素产生问题，要正确实现ACID并不容易。
ACID建议数据库将所有需要更新以及修改的资料一次操作完毕，但实际上并不可行。

目前主要有两种方式实现ACID：第一种是Write ahead logging，也就是日志式的方式(现代数据库均基于这种方式)。第二种是Shadow paging。

相对于WAL（Write ahead logging）技术，Shadow paging技术实现起来比较简单，消除了写日志记录的开销恢复的速度也快(不需要redo和undo)。Shadow paging的缺点就是事务提交时要输出多个块，这使得提交的开销很大，而且以块为单位，很难应用到允许多个事务并发执行的情况——这是它致命的缺点。

WAL的中心思想是对数据文件的修改（它们是表和索引的载体）必须是只能发生在这些修改已经记录了日志之后。也就是说，在日志记录冲刷到永久存储器之后。如果我们遵循这个过程，那么我们就不需要在每次事务提交的时候都把数据页冲刷到磁盘，因为我们知道在出现崩溃的情况下，我们可以用日志来恢复数据库。任何尚未附加到数据页的记录都将先从日志记录中重做（这叫向前滚动恢复，也叫做REDO），然后那些未提交的事务做的修改将被从数据页中删除 （这叫向后滚动恢复-UNDO）。

__脏读、幻读、不可重复读：__

1）脏读：
脏读又称无效数据读出。一个事务读取另外一个事务还没有提交的数据叫脏读。
例如：事务T1修改了一行数据，但是还没有提交，这时候事务T2读取了被事务T1修改后的数据，之后事务T1因为某种原因Rollback了，那么事务T2读取的数据就是脏的。

解决办法：把数据库的事务隔离级别调整到READ_COMMITTED

2）不可重复读：
不可重复读是指在同一个事务内，两个相同的查询返回了不同的结果。
例如：事务T1读取某一数据，事务T2读取并修改了该数据，T1为了对读取值进行检验而再次读取该数据，便得到了不同的结果。

解决办法：把数据库的事务隔离级别调整到REPEATABLE_READ

3）幻读：
例如：系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级，但是系统管理员B就在这个时候插入了一条具体分数的记录，当系统管理员A改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样。这就叫幻读。

解决办法：把数据库的事务隔离级别调整到SERIALIZABLE_READ

> 脏读、不可重复读、幻读的级别高低是：脏读 < 不可重复读 < 幻读。

所以，设置了最高级别的SERIALIZABLE\_READ就不用在设置REPEATABLE\_READ和READ\_COMMITTED了

#### 事务传播行为

所谓事务的传播行为（propagation behavior）是指，如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。

在TransactionDefinition定义中包括了如下几个表示传播行为的常量：

```java
TransactionDefinition.PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。这是默认值。
TransactionDefinition.PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
TransactionDefinition.PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
TransactionDefinition.PROPAGATION_NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。
```

PROPAGATION\_NESTED 与PROPAGATION\_REQUIRES\_NEW的区别:

1）两者比较类似,都是嵌套事务，且如果不存在一个活动的事务，都会开启一个新的事务。
2）使用PROPAGATION\_REQUIRES\_NEW时，内层事务与外层事务就像两个__独立__的事务一样，一旦内层事务进行了提交后，外层事务不能对其进行回滚；换言之，两个事务之间互不影响。__两个事务不是一个真正的嵌套事务__。同时它需要JTA事务管理器的支持。
类似的解释：PROPAGATION\_REQUIRES\_NEW 启动一个新的, 不依赖于环境的 “内部” 事务。这个事务可以独立的 commit 或 rollback 而不需要依赖于外部事务。 它拥有自己的隔离范围, 自己的锁, 等等。当内部事务开始执行时, 外部事务将被挂起；内务事务结束时, 外部事务将继续执行。
3）使用PROPAGATION\_NESTED时，外层事务的回滚可以引起内层事务的回滚。而内层事务的异常并不会导致外层事务的回滚，可能某一步出错但不影响整个事务的推进。即可以跳过这个内层事务继续推进。__它是一个真正的嵌套事务__。
类似的解释：PROPAGATION\_NESTED 开始一个 “嵌套的” 事务，它是已经存在事务的一个真正的子事务。嵌套事务开始执行时，它将取得一个savepoint。如果这个嵌套事务失败, 我们将回滚到此 savepoint，而不会继续推进事务，可以选择重试或者再手动放弃整个事务。嵌套事务是外部事务的一部分，只有外部事务结束后它才会被提交。
DataSourceTransactionManager使用savepoint支持PROPAGATION\_NESTED时，需要JDBC 3.0以上驱动及1.4以上的JDK版本支持。其它的JTA TrasactionManager实现可能有不同的支持方式。

由此可见, PROPAGATION\_REQUIRES\_NEW 和 PROPAGATION\_NESTED 的最大区别在于：

> PROPAGATION\_REQUIRES\_NEW 完全是一个新的事务, 而 PROPAGATION\_NESTED 则是外部事务的子事务。

PROPAGATION\_REQUIRED是常用的事务传播行为，它能够满足我们大多数的事务需求。

#### 事务的超时

所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。

这样做的主要目的在于防止事务占用数据库时间太长。

在 TransactionDefinition 中以 int 的值来表示超时时间，其单位是秒。

默认设置为底层事务系统的超时值，如果底层数据库事务系统没有设置超时值，那么就是none，没有超时限制。

#### 事务只读属性

只读事务用于客户代码只读但不修改数据的情形，只读事务用于特定情景下的优化，比如使用Hibernate的时候。

默认为读写事务。

注意：
“只读事务”并不是一个强制选项，它只是一个“暗示”，提示数据库驱动程序和数据库系统，这个事务并不包含更改数据的操作，那么JDBC驱动程序和数据库就有可能根据这种情况对该事务进行一些特定的优化，比方说不安排相应的数据库锁，以减轻事务对数据库的压力，毕竟事务也是要消耗数据库的资源的。

但是你非要在“只读事务”里面修改数据，也并非不可以，只不过对于数据一致性的保护不像“读写事务”那样保险而已。

因此，“只读事务”仅仅是一个性能优化的推荐配置而已，并非强制你要这样做不可。

#### 事务回滚规则

要使得Spring事务管理器能够回滚一个事务的推荐方法是：

> 在当前事务的上下文内抛出异常。

Spring事务管理器会捕捉任何未处理(unchecked)的异常(如RuntimeException)，然后依据事先配置好的规则决定是否回滚这个抛出异常的事务。

默认配置下，Spring只有在抛出的异常为运行时unchecked异常时才回滚该事务，也就是抛出的异常为RuntimeException的子类(Errors也会导致事务回滚)，而抛出checked异常则不会导致事务回滚。

但是可以明确的配置在抛出那些异常时回滚事务，这样就包括checked异常。
同理，也可以明确定义那些异常抛出时不回滚事务。

还可以编程性的通过静态方法setRollbackOnly()方法来指示一个事务必须回滚，在调用完setRollbackOnly()后你所能执行的唯一操作就是回滚。

### TransactionStatus接口

上面讲到的调用PlatformTransactionManager接口的getTransaction()的方法得到的是TransactionStatus接口的一个实现。

这个接口的内容如下：

![image](/uploads/java-spring-transaction-and-rollback/3.png)

接口描述的是一些处理事务的过程中，提供简单的控制事务执行和查询事务状态的方法，在回滚或提交的时候需要应用对应的事务状态。

## Transactional注解

Transactional注解是声明式配置Spring的事务管理的一种方式，也是比较灵活的一种配置方式。

该注解的属性如下：

![image](/uploads/java-spring-transaction-and-rollback/4.png)

注解用法与注意事项:

@Transactional 可以作用于接口、接口方法、类以及类方法上。当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。

虽然 @Transactional 注解可以作用于接口、接口方法、类以及类方法上，但是 Spring 建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。另外， @Transactional 注解__应该只被应用到 public 方法__上，这是由 Spring AOP 的本质决定的。如果你在 protected、private 或者默认可见性的方法上使用 @Transactional 注解，这将被忽略，也不会抛出任何异常。

默认情况下，只有来自外部的方法调用才会被AOP代理捕获。也就是，类内部方法调用本类内部的其他方法并不会引起事务行为，即使被调用方法使用@Transactional注解进行修饰。

## 参考博客

[1、Spring事务管理（详解+实例）](http://www.mamicode.com/info-detail-1248286.html)
[2、Spring事务机制详解](http://www.cnblogs.com/csniper/p/5536633.html)
[3、spring事务配置](http://blog.csdn.net/bao19901210/article/details/41724355)
