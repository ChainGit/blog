---
title: MVC设计模式和三层结构联系
date: 2017/07/10
categories:
- 学习
tags:
- 学习
- 软件工程
- 转载
- 知识点
---

MVC设计模式和三层结构的联系
==============
这个是最近在简书上看到的，转载了原作者[文章](http://www.jianshu.com/p/71ae09665214?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)，并且自己也做了补充。

-----------------

## 简述

在软件开发中，MVC与三层架构这两个专业词汇经常耳闻，同时总有很多人将它们混为一谈，认为三层架构就是指MVC，给它画上等号，但实际上，这是错误的认知，并不是说它们没有任何关系，而是MVC与三层架构不是简单的相等。
下面将拿java web开发中的MVC（SSM框架）与三层架构进行比较，让大家理清两者之间的关系。

## 概念

### 系统架构

所谓系统架构是指整个应用系统程序大的结构，常见的系统架构有三层架构与MVC。
前面已经说了，三层架构与MVC不是简单的相等，它们存在差别，但又联系。
现在可以肯定的是，这两种系统架构的出现，都是为了降低系统模块间的___耦合度___。

### 三层架构

> 三层架构(3-tier architecture)
通常意义上的三层架构就是将整个业务应用划分为:表现层(Presentation layer)、业务逻辑层(Business Logic Layer)、数据访问层(Data access layer)。
区分层次的目的即为了___"高内聚低耦合"___的思想。
在软件体系架构设计中，分层式结构是最常见，也是最重要的一种结构。
微软推荐的分层式结构一般分为三层，从下至上分别为: ___数据访问层、业务逻辑层(又或称为领域层)、表示层___。

三层架构是指：视图层View、服务层Service、持久层Dao，分别完成不同的功能。

> View层：用于接收用户提交请求的代码在这里编写。

> Service层：系统的业务逻辑主要在这里编写。

> Dao层：直接操作数据库的代码在这里编写。

为了更好的降低各层间的耦合度，在三层架构程序设计中，采用面向抽象编程。即上层对下层的调用，是通过接口实现的。而下层对上层的真正服务提供者，是下层接口的实现类。服务标准（接口）是相同的，服务提供者（实现类）可以更换。这就实现了层间的耦合。

![image](http://upload-images.jianshu.io/upload_images/4050443-8fe4ba1f7fb2d855.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### MVC

MVC是指：Model模型、View视图、Controller控件器。

> View：视图，为用户提供使用界面，与用户直接进行交互。

> Model：模型，承载数据，并对用户提交请求进行计算的模块。其分为两类，一类称为数据承载Bean，一类称为业务处理Bean。所谓数据承载Bean是指实体类，专门承载业务数据的，如Student、User等。而业务处理Bean则是指Service或Dao对象，专门用于处理用户提交请求的。

> Controller：控制器，用于将用户请求转发给相应的Model进行处理，并处理Model的计算结果向用户提供相应响应。

MVC架构程序的工作流程是这样的：

> （1）用户通过View页面向服务端提出请求，可以是表单请求、超链接请求、AJAX请求等。

> （2）服务端Controller控制器接收到请求后对请求进行解析，找到相应 的Model对用户请求进行处理。

> （3）Model处理后，将处理结果再交给Controller。

> （4）Controller在接到处理结果后，根据处理结果找到要作为向客户端发回的响应View页面。页面经渲染（数据填充）后，再发送给客户端。

![image](http://upload-images.jianshu.io/upload_images/4050443-a336ba9cd0097eb3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 关系

### MVC与三层架构的关系

MVC与三层架构很相似，但它们并不一样。如果以三层架构为背景，那么MVC的三个部分分别对应的是什么？

三层架构中的View层简单的说就是跟用户发生直接关系的层，MVC中的V和C就是这样的存在，所以MVC中的V和C均属于三层架构的View层。同时，我们知道MVC中的M（Model）包括了数据承载Bean和业务处理Bean，其中业务处理Bean分为Service或Dao对象，分别对应业务逻辑处理和数据库操作，相应的，它们对应的是三层架构中的Service层和Dao层。故，它们的关系如下图所示：

![image](http://upload-images.jianshu.io/upload_images/4050443-735215803ebede91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### SSM与三层架构的关系

SSM即SpringMVC、Spring、Mybatis三个框架。它们在三层架构中所处的位置是不同的，即它们在三层架构中的功能各不相同，各司其职。

> SpringMVC：作为View层的实现者，完成用户的请求接收功能。SpringMVC的Controller作为整个应用的控制器，完成用户请求的转发及对用户的响应。

> MyBatis：作为 Dao层的实现者，完成对数据库的增、删、改、查功能。

> Spring：以整个应用大管家的身份出现。整个应用中所有的Bean的生命周期行为，均由Spring来管理。即整个应用中所有对象的创建、初始化、销毁，及对象间关联关系的维护，均由Spring进行管理。

![image](http://upload-images.jianshu.io/upload_images/4050443-dd2be8e425ac67c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 补充

### Model和Entity

整理自[知乎](https://www.zhihu.com/question/25256772)

相同：两者都是都是一个类，里面有些属性字段。

区别：Model是一个模型，是一个集合，里面装了各种数据，属于MVC设计模式中的M，将一个Model传递给View，在视图页面就可以使用Model里的数据呈现到页面上。
而Entity是实体，就是和数据表一一对应的，一个实体一张表。

然而，官方的demo没有entity这个说法，他只有model。model既是表，也是传递给view的，换句话说，就是官方的demo内的“model”即是model又是entity。
而我们在实际运用中，需要将Model分成ViewModel和Entity，甚至还要加入DTO。ViewModel是视图所需要的model，说白了就是视图上显示数据的。而entity只是数据表字段。dto是数据传输对象，业务需要哪些数据，就放哪些数据。

比如数据库中用户User表有很多字段，但是注册的时候视图页面只需要账号、密码、手机、注册时间这几个字段。那就可以建立一个RegisterDto，存放部分数据字段，提交时将这个dto的数据传递给model后再写入数据到数据库，至于传递数据可以手动复制给entity也可以使用automapper来自动映射。

再举一个具体的例子：

比如一个新闻页面需要的有：这篇新闻的各种字段(Title，CreateTime，Content等)、最新文章10条、最热文章10条。

那这个视图就需要这样一个ViewModel:

```java
public class NewsView{
    public News news{get;set;} //这篇新闻的全部数据
    public List<News> lastNews{get;set;} //最新10篇新闻
    public List<News> hotNews{get;set;} //最热10篇新闻
}
```

而新闻实体Entity（对应数据库中新闻表news）：

```java
public class News{
    public int id{get;set;}
    public string title{get;set;}
    public string content{get;set;}
    public int clicks{get;set;}
    public Timestamp createTime{get;set;}
}
```

这样在控制器Controller的action中返回的Model就是NewsView，使得页面能够使用ViewModel中的数据显示这篇新闻的内容，还有最新最热的10篇文章列表。

但是最新和最热的文章，一般只需要一个包含标题和日期，没有必要知道所有的内容。
而在上文的ViewModel中最新最热的文章也是直接使用了News，会包含全部的内容，所以在这种情况下需要使用DTO。

```java
public class NewsDto{
    public int id{get;set;}
    public string title{get;set;}
    public Timestamp createTime{get;set;}
}
```

将上面的ViewModel改成：

```java
public class NewsView{
    public News news{get;set;} //这篇新闻的全部数据
    public List<NewsDto> lastNews{get;set;} //最新10篇新闻的简要信息
    public List<NewsDto> hotNews{get;set;} //最热10篇新闻的简要信息
}
```

这样就能实现业务的灵活性。

> 完