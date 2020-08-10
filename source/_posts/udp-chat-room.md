---
title: 基于UDP的可用于公网的多线程设计的多人聊天室
date: 2017/12/05
categories: 案例
tags:
- 案例
- 学习
- Java
- 计算机网络与通信
---

基于UDP的可用于公网的多线程设计的多人聊天室
============================================
一转眼大四上学期也快过去了，最近也在巩固多线程的基础知识。
我的感觉是，多线程和并发编程既需要不断的去接触和思考，不断的去深入挖掘，也需要项目实战的经验积累。
之前在学习Java基础时写过一个简单的单线程局域网的UDP聊天室，现在结合多线程知识，设计一个复杂一点的利用多线程的聊天室。
在设计过程中，也更进一步学习了NAT和路由打洞的一些知识。

## 源码下载

欢迎[下载使用](https://raw.githubusercontent.com/ChainGit/mine-demos/master/demo03/chat-room/chat_room.rar)

使用时注意修改conf下的chatroom.properties，服务器端在命令行模式下运行，客户端可直接打开运行。

源码在[这里](https://github.com/ChainGit/mine-demos/tree/master/demo03)

## 知识难点

__Java编程：__

1、线程的唤醒和等待可以使用三种方式：
1）Object自带的wait/notify
2）JUC的Condition中的await/signal
3）while循环中使用Thread.sleep和if判断

2、多线程中保证并发访问的三种方式：
1）Java的关键字synchronized
2）JUC的ReentrantLock
3）CAS无锁算法

3、多线程并发访问下可使用的集合类：
1）ConcurrentHashMap
2）ConcurrentLinkedQueue
3）LinkedBlockingQueue

4、多线程并发访问下的一个特殊关键字：volatile

5、生产者和消费者的模型

__计算机网络与通信：__

1、心跳机制的实现和意义
2、路由的消息转发和寻路、NAT、打洞
3、心跳机制和路由打洞的联系
4、UDP通信和UDP编程

相关知识点这里就不解释了。

## 架构设计

具体的UML类图和时序图、流程图就不画了，需要费很大功夫。

客户端：
[png](/uploads/udp-chat-room/client.png)

![image](/uploads/udp-chat-room/client.svg)

服务器：
[png](/uploads/udp-chat-room/server.png)

![image](/uploads/udp-chat-room/server.svg)

其中客户端和服务器都还有一个Register，所需的Service等都注册于其中，方便管理和控制。

客户端和服务器之间的通信所有的数据是存储在Msg中。
发送数据时，Msg会转成Json字符串，进而转成byte数据；
接受数据时，会将byte数据转成Json字符串，进而转成Msg对象。
其中Msg存储的data数据可以是各种类型。

Msg中成员变量有：

![image](/uploads/udp-chat-room/msg.png)

Msg.Type即消息类型总共有一下几种：

![image](/uploads/udp-chat-room/type.png)

客户端和服务器的Service的handle方法会根据Msg具体的Type类型进行相应的处理。

## 测试截图

阿里云公网服务器工作截图：

![image](/uploads/udp-chat-room/4.png)

客户端截图（每个客户端都在不同的主机和不同的网路环境下）：

具体的测试环境：移动4G网，电信4G网，校园宽带网

![image](/uploads/udp-chat-room/2.jpg)

![image](/uploads/udp-chat-room/3.jpg)

![image](/uploads/udp-chat-room/1.png)

客户端可以在同一台电脑上运行多个实例，另外即使电脑的网络环境发生变化（比如切换到不同的网络）也能做到不掉线。

这个聊天DEMO没有确保UDP消息一定能做到可靠，没有遗漏，只是为了学习多线程和计算机网络与通信。

## 总结感悟

自己虽然是通信专业，但是在编程过程中才发现自己的理论的学习只是浮于表面，比如路由的转发，只是学到了一些文字上的IP的替换，路由的寻路方法。在这个知识背景下，开始的做法就是，当客户端发送消息给服务器时，服务器收到了消息，然后根据存储的客户端的IP和Port转发给客户端，却发现在跨网段情况下客户端并不能接收到数据包，局域网下是可以的。路由从内而外很容易，但是由外而内却不能直接访问。这就涉及到了打洞知识，而后也就能顺利的解决了问题，实现了公网下的聊天。心跳机制一方面是为了告诉服务器当前客户端是在线的，另一方面也是为了路由的打洞，及时的更新服务器中存储客户端NAT后的不断变化的IP和Port，便于服务器使用这个新的IP和Port和客户端进行通信。

自己在做一些小项目时，其中最大的感受就是（也是一种软件开发的模式），一定要先想好大致的框架和流程，画好草图，想好模块的大致和软件的分层，然后在具体的实施过程对每一个模块进行细致的设计，这样开发出的软件结构优美，秩序井然，且易于修改和增减功能。

当和实验室的伙伴一起测试可以在公网下进行聊天的服务器时，心里的那个激动，爽哈!

