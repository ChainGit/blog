---
title: HashMap的高并发下的死循环问题
date: 2017/08/16
categories:
- 学习
tags:
- 学习
- Java
- 转载
---

HashMap的高并发下的死循环问题
===========================
在公司实习中，师傅陆陆续续的给我介绍了很多知识，有常用的框架使用，有新的技术，也有多线程和高并发的一些知识。多线程和高并发一直是大公司的必问知识点，不过自己的目前的知识水平还未达到那个层次，学习的路永无止境。前天介绍了一个HashMap在高并发状态下存在的问题，回去也学习了一下，找到一篇很不错的[文章](https://coolshell.cn/articles/9606.html)，在此整理一下。

## 死循环复现
下面模拟实现在多线程（多用户）下的HashMap的死循环复现：
```Java
public static void main(String[] args) {
    final Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < 100; i++) {
        Thread t = new Thread() {
            Random rnd = new Random();

            @Override
            public void run() {
                for (int i = 0; i < 100; i++) {
                    map.put(rnd.nextInt(), 1);
                }
            }
        };
        t.start();
    }
}
```
程序运行没多久，可以在任务管理器中发现CPU的占用率达到100%，而且正是我们在跑的这个Java程序。

使用jstack可以定位到那个耗时最长的进程的具体代码，具体操作文章可以[参考](http://www.cnblogs.com/chengJAVA/p/5821218.html)。

定位的结果显示，耗时最多的是HashMap的get(i)方法，为什么get方法会耗时最长呢？

## Hash表数据结构
回顾一下HashMap这个Java中的经典数据结构。

HashMap通常会用一个指针数组（假设为table[]）来做分散所有的key，当一个key被加入时，会通过Hash算法通过key算出这个数组的下标i，然后就把这个&lt;key, value&gt;插到table[i]中，如果有两个不同的key被算在了同一个i，那么就叫__冲突__，又叫碰撞，这样会在table[i]上形成一个链表(在Java8中已经优化为红黑树)。

我们知道，如果table[]的尺寸很小，比如只有2个，如果要放进10个keys的话，那么碰撞非常频繁，于是一个O(1)的查找算法，就变成了链表遍历，性能变成了O(n)，这是Hash表的缺陷（可参看《Hash Collision DoS 问题》）。

所以，Hash表的尺寸和容量非常的重要。一般来说，Hash表这个容器当有数据要插入时，都会检查容量有没有超过设定的thredhold，__如果超过，需要增大Hash表的尺寸，但是这样一来，整个Hash表里的无素都需要被重算一遍__。这叫rehash，这个成本相当的大。

相信大家对这个基础知识已经很熟悉了。

## HashMap的rehash源代码
下面，我们来看一下Java的HashMap的源代码。

Put一个Key,Value对到Hash表中：
```Java
public V put(K key, V value) {
    ......
    //算Hash值
    int hash = hash(key.hashCode());
    int i = indexFor(hash, table.length);
    //如果该key已被插入，则替换掉旧的value （链接操作）
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    //该key不存在，需要增加一个结点
    addEntry(hash, key, value, i);/****/
    return null;
}
```

检查容量是否超标
```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
    //查看当前的size是否超过了我们设定的阈值threshold，如果超过，需要resize
    if (size++ >= threshold)
        resize(2 * table.length);/****/
}
```

新建一个更大尺寸的hash表，然后把数据从老的Hash表中迁移到新的Hash表中。
```Java
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    ......
    //创建一个新的Hash Table
    Entry[] newTable = new Entry[newCapacity];
    //将Old Hash Table上的数据迁移到New Hash Table上
    transfer(newTable);/****/
    table = newTable;
    threshold = (int)(newCapacity * loadFactor);
}
```

迁移的源代码，注意高亮处：
```Java
void transfer(Entry[] newTable) {
    Entry[] src = table;
    int newCapacity = newTable.length;
    //下面这段代码的意思是：
    //  从OldTable里摘一个元素出来，然后放到NewTable中
    for (int j = 0; j < src.length; j++) {
        Entry<K,V> e = src[j];
        if (e != null) {
            src[j] = null;
            do {
                Entry<K,V> next = e.next;
                int i = indexFor(e.hash, newCapacity);
                //注意这一段
                e.next = newTable[i];/****/
                newTable[i] = e;/****/
                e = next;/****/
            } while (e != null);
        }
    }
}
```

好了，上面的代码一般使用的话，没有什么问题。

### 正常ReHash的过程

画了个图做了个演示。

我假设了我们的hash算法就是简单的用key mod 一下表的大小（也就是数组的长度）。
最上面的是old hash 表，其中的Hash表的size=2, 所以key = 3, 7, 5，在mod 2以后都冲突在table[1]这里了。
接下来的三个步骤是Hash表 resize成4，然后所有的&lt;key,value&gt; 重新rehash的过程

![image](https://coolshell.cn/wp-content/uploads/2013/05/HashMap01.jpg)

### 并发下的ReHash

1）假设我们有两个线程。我用红色和浅蓝色标注了一下。

我们再回头看一下我们的 transfer代码中的这个细节：

```Java
do {
    Entry<K,V> next = e.next; // <--假设线程一执行到这里就被调度挂起了
    int i = indexFor(e.hash, newCapacity);
    e.next = newTable[i];
    newTable[i] = e;
    e = next;
} while (e != null);
```

而我们的线程二执行完成了。于是我们有下面的这个样子。

![image](https://coolshell.cn/wp-content/uploads/2013/05/HashMap02.jpg)

注意，因为Thread1的 e 指向了key(3)，而next指向了key(7)，其在线程二rehash后，指向了线程二重组后的链表。我们可以看到链表的顺序被反转后。

2）线程一被调度回来执行。

先是执行 newTalbe[i] = e;
然后是e = next，导致了e指向了key(7)，
而下一次循环的next = e.next导致了next指向了key(3)

![image](https://coolshell.cn/wp-content/uploads/2013/05/HashMap03.jpg)

3）一切安好。

线程一接着工作。把key(7)摘下来，放到newTable[i]的第一个，然后把e和next往下移。

![image](https://coolshell.cn/wp-content/uploads/2013/05/HashMap04.jpg)

4）环形链接出现。

e.next = newTable[i] 导致  key(3).next 指向了 key(7)

注意：此时的key(7).next 已经指向了key(3)， 环形链表就这样出现了。

![image](https://coolshell.cn/wp-content/uploads/2013/05/HashMap05.jpg)

于是，当我们的线程一调用到，HashTable.get(11)时，悲剧就出现了——Infinite Loop。

## 解决

HashMap不支持并发，并发情况下可以使用
1、ConcurrentHashMap，其通过分段锁和其他技术实现了高并发，支持原子条件更新操作，不会抛出ConcurrentModificationException，实现了弱一致性。
2、final Map&lt;Integer, Integer&gt; map = Collections.synchronizedMap(new HashMap&lt;Integer, Integer&gt;());