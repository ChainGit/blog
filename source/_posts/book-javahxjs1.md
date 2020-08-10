---
title: Java核心技术--卷I
date: 2016/06/22
categories:
- 阅读
tags:
- 阅读
---

## Java核心技术--卷I
终于看完这本书了，看的中文版，文字读起来有些别扭，不过结合看过的毕老师的JavaSE视频，也是有所收获。

## 每章小结
### 第一章 Java程度设计概述
```
1.1 Java程序设计平台
1.2 Java"白皮书"的关键术语
简单性，面对对象，网络技能，健壮性，安全性，体系结构中立，可移植性，解释型，高性能，多线程，动态性
1.3 Java Applet 与 Internet
1.4 Java发展简史
1.5 关于Java的常见误解
```

### 第二章 Java程序设计环境
```
2.1 安装Java开发工具箱
2.2 选择开发环境
2.3 使用命令行工具
2.4 使用集成开发环境(IDE)
2.5 运行图形化应用程序
2.6 建立并运行Applet
```

### 第三章 Java的基本程序设计结构
```
3.1 一个简单的Java应用程序
3.2 注释
3.3 数据类型
整形，浮点类型，char类型，boolean类型
3.4 变量
变量的初始化，常量
3.5 运算符
自增运算符与自减运算符，关系运算符与boolean运算符，位运算符，数学函数与常量，数值类型之间的转换，强制类型转换，括号与运算符级别，枚举类型
3.6 字符串
子串，拼接，不可变字符串，检测字符串是否相等，空串与Null串，代码点与代码单元，字符串API，阅读联机API文档，构建字符串
3.7 输入输出
读取输入，格式化输出，文件输入与输出
3.8 控制流程
块作用域，条件语句，循环，确定循环，多重选择:switch语句，中断控制流程语句(continue,break)
3.9 大数值
3.10 数组
foreach 循环，数组初始化以及匿名数组，数组拷贝，命令行参数，数组排序，多维数组，不规则数组
```

### 第四章 对象与类
```
4.1 面对对象程序设计概述(OOP)
类（Class），对象（Object，行为behavior，状态state，标识identity），识别类，类之间的关系(依赖uses-a，聚合has-a，继承is-a)
构造（Construct），实例（Instance），封装（Encapsulation，数据隐藏，黑盒），实例域（instance field），方法（method），状态（state），继承（Inheritance）
Algorithms+Data Structures=Programs
面向过程，算法第一位，数据结构第二位
面向对象，数据第一位，算法第二位
4.2 使用预定义类
对象与对象变量，更改器方法与访问类方法
4.3 用户自定义类
多个源文件的使用，从构造器开始，隐式参数与显式参数，封装的优点(getXXX,setXXX，数据private)，基于类的访问权限，私有方法，final实域类
4.4 静态域与静态方法(static)
静态域，静态常量，静态方法，工厂方法，main方法
4.5 方法参数
Java目前只有值传递，没有C语言内的地址传递。Java引用居多。
4.6 对象构造
重载(重载解析，overloading resolution)，默认域初始化(尽量显式赋初值)，无参数的构造器，显示域初始化，参数化（this），调用另一个构造器，初始化块(initialization block)，对象析构与finalize方法
4.7 包
类的导入(import)，静态导入(静态方法，静态域)，将类放入包中，包的作用域
4.8 类路径
4.9 文档注释（javadoc）
注释的插入，类注释，方法注释，域注释，通用注释，包与概述注释，注释的抽取
4.10 类设计技巧
保障数据私有(private)，显式的数据初始化，类内数据需要认真设计，类间关系想想清楚，类内关联降低，不是所有的类都需要set和get，类的分解恰当，类和方法名要与实际对应。
```

### 第五章 继承
```
5.1 类、超类（superclass,baseclass,parentclass）和子类(subclass,derivedclass,childclass)
继承（Inheritance，复用,拓展,覆盖override）层次（inheritance hierarchy），多态（子类对象赋给超类变量，但不能将超类的引用赋给子类变量），动态绑定，阻止继承：final类和方法，强制类型转换（instanceof检测），抽象类（abstract），受保护访问（private,protected,public,default）
super和this，Java不支持多继承
5.2 Object：所有类的超类
equals方法，相等测试与继承，hashCode方法，toString方法
5.3 泛型数组列表
访问数组列表元素（ArrayList<Object>），类型化与原始数组列表的兼容性，对象包装器与自动装箱(autoboxing)
@SuppressWarnings("unchecked")
5.4 参数数量可变的方法(范例printf)
5.6 枚举类(Enum)
5.7 反射（reflection，强大工具）
Class类，捕获异常(try..catch)，利用反射分析类的能力(Field,Method,Constructor；private,protected,public,default)，在运行时使用反射分析对象，使用反射编写泛型数组代码，调用任意方法（对比接口interface和内部类，不建议使用Method对象，效率相对较低而且不易维护）
5.8 继承设计的技巧
将公共操作和域放在超类；
不要使用受保护的域，确保封装性private；
使用继承实现“is-a”关系；
除非所有继承的方法都有意义，否则不要使用继承；
在覆盖方法时，不要改变预期的行为；
使用多态、接口，而非类型信息；
不要过多的使用反射，反射多用于工具类。
```

### 第六章 接口与内部类
```
6.1 接口（interface）
接口的特性
接口可以白看作没有实例域的抽象类，但有区别。
实现（implement）
如果类遵从某个特定接口，那么就履行这项服务（必须实现接口内的方法）。
任意的x,y，sgn(x.compareTo(y))==-sgn(y.compareTo(x))
接口不是类，不能用new运算符实例化一个接口，即构造接口的对象，但能声明接口的变量。
接口变量必须引用实现了接口的类对象，使用instanceof可以检查对象是否实现了某个特定的接口。
像类可以继承一样，接口也可以被扩展。
接口不能有实例域或静态方法，但是可以包含常量。
接口方法将被标记为public，域标记public static final。
接口与抽象类
Java不支持多继承（mulitiple inheritance，比较复杂而且容易造成效率降低），即只能有一个超类，但是可以实现多个接口。
6.2 对象克隆
拷贝与克隆，浅拷贝与深拷贝。Clonable实现的是深拷贝（约等于完全拷贝了一个独立的新对象）。
6.3 接口与回调（callback）
Java中函数指针的调用类似物--Method对象，但速度反而略慢，且麻烦，而且没有编译时的安全检查。
回调典型如Timer。
6.4 内部类（inner class）
内部类是定义在另一个类中类。
内部类可以访问该类包括私有域的数据。
内部类可以相对同一个包而言可以隐藏起来。
匿名内部类可以定义一个回调函数且不用编写大量的代码。
6.4.1 使用内部类访问对象状态
内部类与接口是Java重要的基本特性。
内部类的对象总有一个隐式引用，指向了创建它的外部类对象，由编译器自动添加。
内部类当且仅当外部类需要时才会有实例域。
常规类具有包可见性或者公共可见性，而内部类可以是私有类。
6.4.2 内部类的特殊语法规则
内部类中，使用外部类的引用正规语法：OuterClass.this
在外围类的作用域之外，可以这样的语法来引用内部类：OuterClass.InnerClass
6.4.3 内部类是否安全，必要，有用
编译器将内部类翻译成$来分割外部类名和内部类名的常规类文件，虚拟机对此一无所知。
内部类不是绝对安全的。
例：static boolean access$0(com.chain.chapter6.TalkingClock);
编译器在外围类添加静态方法，返回作为参数传递给它的对象域。这个可以被调用用作访问类内私有域数据，但难度较高。
6.4.4 局部内部类
局部类对外界的世界完全隐藏起来，当然也不能用public或private。
6.4.5 由外部方法访问final变量
局部类不仅可以访问包含它们的外部类，也可以访问局部变量，但这些局部变量需要被声明为final。原理在于编译器会自动的为局部变量创建副本。
例：final com.chain.chapter6.TalkingClock this$0;
public com.chain.chapter6.TalkingClock$TimePrinter(com.chain.chapter6.TalkingClock);
这是内部类自动生成的构造器，作为副本。
6.4.6 匿名内部类（Annoymous inner class）
例：Person psn1=new Person("Tom");  //这是Person类的一个实例
Person psn2=new Person("Jack"){ ... }; //an object of an inner class extending person
其实匿名内部类与普通内部类可以相互转化，匿名内部类有时可以减少代码的编写，但其实不易维护。
ArrayList<String> friends=new ArrayList<>();
friends.add("Andy");
friends.add("Dan");
invite(friends);
//接下来friends数组列表可以不要了
==>
invite(new ArrayList<String>{{add("Andy");add("Dan");}});
匿名类使用equals时需注意。

new SuperType(construction parameters){
inner class methods and data
}

new InterfaceType(){
methods and data
}

6.4.7 静态内部类
仅需隐藏一个类在另一个类的内部时，并不需要内部类引用外部类对象时，可将内部类声明为static，以便取消编译器自动添加的外部类的引用。static class and method。
6.5 代理（Proxy）
动态加载类，invoke(Object proxy,Method method,Object[] args)
```

### 第七章到第九章 GUI部分
```
省略
```

### 第十章 部署应用程序和applet
```
10.1 JAR文件（Jav归档文件）
将应用程序进行打包，可以包含类文件，图像和声音的资源文件。
JAR文件是压缩的，使用了熟悉的ZIP压缩格式。
使用JAVA自带的jar工具可以创建JAR文件。
清单文件（MANIFEST.MF)
可运行的jar文件
10.2 Java WEb Start
密封，沙箱，签名代码,JNLP API
10.3 applet
HTML标记语属性，object标记，使用参数向applet传递信息，访问图像和音频文件，applet上下文
10.4 应用程序首选项存储
属性映射（Properties类）
Preferences API（数据以XML存储）
```

### 第十一章 异常、断言、日志和调试
```
11.1 处理错误
出现错误而无法执行：返回安全状态，进行保存作业的操作，可以以适当的方式终止程序。
用户输入错误（如输入不标准），设备错误（如设备脱机），物理限制（如磁盘满了），代码错误（如返回值处理）。Java中出现问题后会采用另一种路劲退出该方法，并抛出异常（throw），并停止继续执行，立即退出。异常处理机制（exception handler）会接着处理。
异常分类
Throwable:Error（无法控制的错误）+Exception（）
Exception:IOException+RuntimeException（逻辑错误，checked）+非	RuntimeException（unchecked）
声明已检查异常(throws)
如何抛出异常
throw new XXXException;
throws XXXException
创建异常类
派生一个Exception类，超类为Throwable
11.2 捕获异常(try/catch/finally)
捕获多个异常
多个catch语句，每一个catch语句内对应一个已知的错误类
再次抛出异常与异常链
try{
...
}catch(SQLException e){
throw new ServletException("throw again"+e.getMessage());
}
finally字句
当程序错误时，一般会无条件终止，这使得资源尚未释放，finally则无论发生还是没有发生错误都会执行，以便及时关闭资源。
带资源的try语句
open a res
try{
try{
work with the res
}finally{
close the res
}
}catch(XXXException e){
...
}
分析堆栈跟踪元素(stack trace)
方法调用过程的列表，包含程序执行过程中的方法调用的特定位置。
printStackTace();
11.3 使用异常机制的技巧
尽量避免错误的发生，不能用作测试，滥用会降低效率；
不要太过分的细化异常；
抛出的异常如果重要，需要利用异常的层次结构；
重要的异常需要去处理，而不是一直空着，捕获到却不处理；
早抛出，晚捕获
11.4 使用断言（assert）
启用和禁用断言
java -anableassertions XXXX
使用断言完成参数检查
Java语言中3种常见处理错误的机制：
抛出一个异常，日志，使用断言
使用断言时，断言失败是致命的，不可恢复的错误。断言检查用于开发和测试阶段。
为文档假设使用断言
assert i!=null:i;
11.5 记录日志
最原始的记录方式就是println出觉得有错误的地方。但是建议使用记录日志(Logger)的API。
可以方便的取消全部日志记录，打开关闭很方便。
可以一键禁止日志，减少系统开销。
日志记录可以设置不同的处理方式，如XML或文本。
日志记录应该添加上如包名等具有层次结构的名字。
基本日志
高级日志
修改日志管理器配置
本地化
处理器
过滤器
格式化器
日志记录说明
11.6 调试技巧
System.out.println("x=",x);
日志代理，logging proxy
Logger.getGlobal().info("x=",x);
每个类里可以放一个main()用于测试，事后可以注释，java只调用启动类的main()
JUnit
Throwable提供的printStackTrace()
代码的任意位置插入这条语句就可以堆栈跟踪，Thread.dumpStack()
可以将错误输出到文件中，比如printStackTrace(out)，或者命令行参数重定向输出
将日常调试的信息写入log文件
java -verbose 可以将类的加载信息输出
javac -Xlint:fallthrough 可以通知编译器检查某个特别注意点
jconsole监控管理虚拟机的内存消耗，线程使用，类加载情况
jmap获得堆的转储，然后在localhost:7000
如果使用-Xprof标志，则会剖析经常用到的方法。
11.7 GUI程序排错技巧
11.8 使用调试器
使用Eclipse的Debug模式
```

### 第十二章 泛型程序设计
```
在java5内加入，比强制使用类型转换的代码较安全，较C#而言，Java的泛型是“假泛型”。
12.1 为什么要使用泛型程序设计
在编译时提供安全检查，消除强制类型转换（隐式自动完成，运行时都是普通类，再从泛型容器里取出值时，编译器进行类型转换），最大限度重用代码。
但是，擦除使得无法与数组混合使用的很好，基本类型（int,float等）无法作为类型参数，需要使用包装类（Integer等），不能用于显示引用运行时的类型操作中（如instanceof和new表达式），重载不识别泛型，基类劫持了接口，try/catch语句不能捕获泛型类型的异常，泛型类不能使用Super，通配符不能盲目使用？等。
12.2 定义简单泛型类
类型变量T，用<>框住，放在类名后面。用具体的类型来替换。
class Generic<T>{}
12.3 泛型方法
public static <T> T getMiddle(T...a){
return a[a.length/2];
}
类型变量放在修饰符之后，返回类型之前
12.4 类型变量的限定
泛型需要绑定接口，比如int和Integer.
public static <T extends Comparable> T minmax(T[] a){}
修饰符+类型变量+返回类型+方法名（参数...）
T与绑定的类型可以是类也可以是接口，不过选择extends接近子类的概念。限定类型用&隔开，而逗号用来分割类型变量
12.5 泛型代码和虚拟机
虚拟机没有泛型类型对象，所有对象都属于普通类。无论何种泛型，都有一种相应的原始类型（raw type），原始类型的名字就是删除类型参数后的泛型类型名，擦除（erased）类型变量T，并替换为限定类型（无限定类型就使用Object），T是一个无限定的变量，所以可以直接用Object替换，但是这有区别。
翻译泛型表达式，擦除返回的是Object类型，编译器会自动插入强制类型转换。
翻译泛型方法，类型擦除和多态可以用桥方法解决，即
public void setSecond(Object second) { setSecond( (Date) second); }
泛型转换的事实，虚拟机中没有泛型，只有普通类和方法；所有类型参数都用它们的限定类型替换；桥方法被合成来保持多态；为保持类型安全性，必要时插入强制类型转换。
调用遗留代码，利用注释（annotation）@SuppreussWarnings("unchecked")，@SafeVarargs
12.6 约束与局限性
Java泛型的限制，大多由类型擦除引起的。
不能用类型参数代替基本类型，例如XXX<int>，只用XXX<Integer>，保持Java一切皆对象思想，如果要使用可以使用包装类实现，比如Integer或者自己定义的类。
不能使用Instanceof，所有泛型类运行时都是普通类而已，同样getClass()也是如此。
不能创建参数化类型的数组，比如Pair<String> table=new Pair<>[10]这样在擦除后。table类型是Pair[10]，而非Pair<String>[10]，如果需要收集参数化类型对象，可以使用ArrayList<Pair<String>>周转实现。或者：
@SafeVarargs static <T> T[] array(T...array){return array;}
Pair<String>[] table =array(pair1,pair2);
Object[] objary=table;
bojary[0]=new Pair<Employee>();  YES
table[0]=new Pair<Employee>();    ERR
不能实例化类型变量，如new T(...)，new T[...]，T.class，因为T会被擦除成Object，可以使用以下实现：
public static <T> Pair<T> makePair(Class<T> cl){
try{
return new Pair<>(cl.newInstance(),cl.newInstance());
}catch( Exception e ) {
return null;
}
}
接下来：
Pair<String> p=ArrayAlg.makeClass(String.class);
不能抛出或捕获泛型类的实例异常，
注意擦除后的冲突
泛型类型的继承规则，为了安全，其实泛型的继承互相没有联系
12.8 通配符类型
Pair<? extends Employee>
? super Manager
12.9 反射和泛型
```

### 第十三章 集合
```
见《数据结构与算法经典问题解析Java版》
数据结构（Data Structures）
JAVA最初：Vector、Stack、HashTable、BitSet、Enumeration
现在：Collection, List,LinkedList,ArrayList,Vector,Stack,Set 
C++标准模板库STL复杂，结合泛型算法，Java的集合类库（Collection）
接口interface、实现implementation在Java中分离

队列（queue）,FIFO先进先出：
循环数组，链表
Queue接口实现
ArrayQueue，LinkedList类
一般循环数组比链表要高效，但是数组有界
Java中还有一个AbstractQueue，抽象队列，可以自己实现扩展队列

集合接口（Collection）和迭代器接口（Iteration）
集合接口有两个基本的方法add，iterator
Java的迭代器位于两个元素之间，当调用next时，会越过下一个元素，并返回刚刚越过的那个元素的引用。需要配合hasNext()。remove方法会删除上次next方法返回的元素。next与remove相互依赖。
每次调用remove时需要在之前调用next
Java中有一个AbstractCollenction，基础方size和iterator抽象化，泛型接口。

Collection  
├List  
│├LinkedList  
│├ArrayList  
│└Vector  
│　└Stack  
└Set
│   ├ TreeSet
│   ├ EnumSet
│   ├ HashSet
│   ├ LinkedHashSet
│
└Queue
   ├ArrayDeque
   ├PriorityDeque

Map  
├Hashtable  
├HashMap  
├TreeMap
├EnumMap
├LinkedHashMap
├IdentityHashMap    
└WeakHashMap 




Set集,List链,Map（映射）表，Queue队列，Stack栈
Linked链接,Array数组,Hash哈希、散列,Tree树
Sorted,Unsorted
Vector

链表：LinkedList，List接口
数组列表ArrayList类，删除插入一个元素代价较大
链表LinkedList类，删除插入方便
Java中，链表是双向的（doubly linked）
C语言指针，Java则是引用
链表是有序集合（Ordered Collection）
add方法依赖迭代器
set是无序的
Java屏蔽了双向链表复杂的引用变更，add，remove,next,hasNext，很好的屏蔽了细节。
如果同时有两个迭代器最链表进行修改操作，将会抛出ConcurrentModificationException
但是一个只读，另一个可读可写的两个迭代器是允许的
链表不支持随机读取，所以get(i)方法不适合用来遍历链表
如果数组中元素较少，可以完全使用ArrayList，可以实现随机访问

数组列表：ArrayList，List接口
封装了一个可动态再分配的对象数组
Vector类所有方法是同步的，可由多个线程来访问Vector，进行同步操作
但是，单线程使用Vector就得不偿失了
ArrayList是非同步的。

散列表：HashTable，散列值：HashCode，Set接口，HashSet
链表和数组都是有一定顺序的，查找元素需要具体的位置或者从头遍历等方法，速度较慢
HashTable可以快速的找到元素，如果不在意数据的排列的话，数据是无序的
Java中，散列表是有链表数组产生的，每个列表被称为桶（bucket）
确定位置，先计算散列值，在与桶的总数取余，这个值就是桶的索引
有时有散列冲突（hash collision）,有对应的解决方法，比如向后沿找空位放或者链表串起来等
散列表太满的话会再散列（rehashed）
如果元素的散列码发生变化，则位置可能也会改变
散列的递归

树集：TreeSet，集Set接口
树集是一个有序集合（Sorted Collection）,红黑树（red-black tree）
每次添加元素时，都会添加到正确的结点上，这样迭代器访问时是按照排好序的顺序访问元素
如果不需要数据排序，不建议使用
排序要比散列困难的多
NavigableSet，增加了反向遍历和定位元素的方法

队列和双端队列：Deque
ArrayDeque、LinkedList
可动态增加队列的长度

优先级队列：Priority Queue
优先级队列采用了一个高雅数据结构，堆（Heap）
堆是一个可以自动调整的二叉树，add和remove操作会将最小的元素移动到树根，这样不用再花费时间排序
可以进行任务调度操作，优先级最高值也为最小1
与TreeSet不同，PriorityQueue是根据优先级排序

映射表：Map
HashMap，TreeMap
散列映射表对键进行散列，
树映射表用键的整体顺序进行排序，在组织成搜索树
散列和比较函数只能用于键，其关联的值不用于散列和比较
与集Set一样，如果不需要排序则散列
键值（key）必须是唯一的

专用集和映射表类
弱散列映射表WeakHashMap：删除表中一些无用的值
链接散列集LinkedHashSet和链接映射表LinkedHashMap：表面是随机排列的，实际会记住插入的顺序
枚举集EnumSet和映射表EnumMap：位序列
标识散列映射表IdentityHashMap：根据内存地址计算散列码

集合框架：


集合和数组之间的转化toArray()

算法：排序和查找
Collections类中的sort方法，shuffle方法，Collections是一个包装类
将所有元素转入一个数组，然后运用归并排序的一个变体进行数组排序，好了后在复制回去
归并排序比快速排序慢一点但是稳定
排序方法
查找方法

遗留的集合
HashTable和子类Properties：和HashMap类似，区别是HashTable是同步的，另一个是不同步的
Vector和子类Stack,Bitset
Enumeration：hasMoreElements和nextElement与Iterator的hasNext、next类似
属性映射表（property map）
位集BitSet：位序列，更类似与位向量或者位数组
栈Stack
```

### 第十四章 多线程
```
多任务（multitasking），在同一时刻运行多个程序的能力。
每一个任务可以称为线程（Thread），多线程程序（multithreaded）。
多进程和多线程：每一个进程拥有一整套变量，而线程是共享数据。
共享变量使得线程之间通信比进程之间更有效。
线程是轻量级的进程，启动、撤销一个线程比启动新进程容易少开销。
14.1 什么是线程
线程的sleep方法可能会抛出InterruptedException异常。当这个线程在sleep时，发出中断（Interrupt）请求。
线程使得程序能够执行多个任务，对短暂的任务可以使用线程，对耗时的复杂项目需要精心设计。
Runnable接口：
public interface Runnable{
void run();
}
由Runnable创建一个Thread对象：
Thread t=new Thread(t);
t.start();//创建一个新的线程
t.run();//执行一个已有的线程
但是为每一个任务创建一个独立的线程付出代价也不小，可以使用线程池。
14.2 中断线程
inerrupt()使用前isInterrupted()。
14.3 线程状态
New（新创建），Runnable（可运行，考虑优先级），Blocked（被阻塞），Waiting（等待），Time Waiting（计时等待），Terminated（被终止）。
一个在运行状态（State）的线程可能是正在Running也可能是没有Running。

14.4 线程属性
线程优先级，守护线程，线程组，处理未捕获异常的处理器。
线程优先级
可以使用setPriority设置优先级，但是Java的优先级是高度依赖系统的，即通过虚拟机映射到系统中的优先级，但不同操作系统中优先级不一。进程的优先级需要谨慎使用，比如使用不当，低优先级进程会“饿死”状态。
守护线程（daemon thread）
为其他线程提供服务，比如计时线程，定时地发送信号给其他线程或者清空过时的高速缓存项的线程。
未捕获异常处理器
Thread.UncaughtExceptionHandler，线程的run并不能抛出任何已检查的错误，但是这些异常会导致线程的终止。可以用线程集合ThreadGroup，还有线程组。
14.5 同步
多个线程同时访问共享数据，竞争条件（race condition）。
比如银行的转账系统。
两种机制防止并发访问的干扰：synchronized关键字，和ReentrantLock类。
A 锁对象：
myLock.lock();//进入锁状态
try{
critical section
}finally{//确保即使程序异常也能执行finally
myLock.unlock();//解除锁
}
锁是可以重入的，线程可以反复获得已经持有的锁，保持一个持有计数(hold count)来跟踪对lock方法的嵌套调用。线程每一次Lock()后必然调用Unlock()方法。所以，基于这一特性，被一个锁保护的代码可以调用另一个使用相同的锁的方法。留心临界区的代码。
条件对象
条件对象（条件变量,conditional variable）
程序进入临界区，但是发现在满足一定条件后才执行。
比如：
if(a>b){
//线程可能在判断完a>b后满足条件,，但在执行do sth时，线程时间到，被暂停，再次得到运行时间时，a>b可能已经被其他相关线程改变，而不满足，再do sth可能就有问题。比如订票系统。
do sth
}
可以使用锁来解决这个问题（在if前加锁），但是如果账户中没有足够的钱，此时程序通过判断后执行等待用户存钱的线程使得余额增加，但是花钱的线程却得到了线程锁，也就意味着别的相关线程无法执行，就一直卡在这。
此时就可以使用条件变量来使得花钱线程进入阻塞状态并放弃锁，这样存钱线程可以执行。await方法可以等待条件，但并不自动恢复执行。应当再在其他线程使用signAll来激活因为同一原因而阻塞等待的线程。从而从await()处调用返回。，从阻塞的地方继续执行。当然条件可能依然不满足，signAll知识通知而已，具体需要各个线程自行判断。
while(!(ok to process)){
condition.await();
}
如果没有其他线程来调用signAll那么这就会产生死锁(deadlock)现象。
同步互锁虽然解决了并发读取数据的冲突，但是速度肯定降了下来。
B synchronized关键字
总结Lock和Condition：
锁用来保护代码片段，保证任何时刻只用一个线程在运行被保护的代码。
锁可以管理试图进入被保护代码段的线程。
锁可以拥有一个或多个相关的条件对象。
每个条件对象管理那些已经进入被保护的代码段但还不能运行的线程
Lock和Condition的控制是高度的，但是大多数情况并不需要如此严格。
每个对象内部都有一个内部锁，如果使用synchronized关键字声明，那么对象锁将保护整个方法。
public synchronized void method(){}//隐式锁
等价于
public void method(){//显示锁
this.intrinsicLock.lock();//进入锁状态
try{
critical section
}finally{//确保即使程序异常也能执行finally
this.intrinsicLock.unlock();//解除锁
}
}
内部锁简单够用，但是有局限：
不能中断一个正在试图获得锁的线程。
试图获得锁时不能设定超时。
每个锁只有单一的条件，可能是不够的。

具体是用这两种方法中的哪一种：
这两种方法可以都不使用，使用阻塞对列来同步完成一个共同任务的线程。
减少代码编写量可以使用synchronized关键字，简单够用。
如果特别需要Lock/Condition方式再使用

同步阻塞
synchronized(obj){
critical section
}
监视器概念
锁和条件是线程同步的强大工具，但严格的讲，它们不是面对对象的。
那么，可以不添加锁来保证安全性吗，那就是监视器（monitor）
监视器是只包含私有域的类
每个监视器类的对象有一个相关的锁。
使用这种锁使得所有的方法都进行加锁，加锁是在方法开始之前自动获得，而方法返回时自动释放。域是私有的，所以又能确保只有一个线程能同时访问数据进行操作。
Volatile域
Java内存模型和线程规范。
volatile关键字为实例域的同步访问提供了一种免锁机制。即如果声明一个域为volatile，那么编译器和虚拟机就会知道该域是被另一个线程并发更新的。
使用synchronized可以使得保持同步，但方法也可能阻塞，再加上Condition。
但是，如果是这样：
public boolean isOk;
public synchronized boolean isOk() { return isOk; }
public synchronized void setOk() { this.isOk=true; }
不如这样：
public volatile boolean isOk;
public boolean isOk() { return isOk; }
public void setOk() { this.isOk=true; }
volatile不能保证原子性。
例如：public void flipDone() { done=!done; }//不是原子的，所以不能保证是能够翻转域中的值。
final变量
锁和volatile可以确保安全的读取一个域。
还有一种方法是域被声明为final时可以保障。
原子性
java.util.concurrent.atomic包中有很多高效的机器级指令来确定不操作的原子性。
原子性是指要么完整的一次性做完，要么就安全回滚到起点。
死锁
账户A:200 账户B:300
线程1：从A转500到B
线程2：从B转400到A
这两个线程都阻塞了，余额均不足。这就是互锁。
如果在运行中遇到死锁，在Java中目前没有什么解决方法，只能设计时避免。
线程的局部变量
SimpleDateFormat线程并不是安全的，因为如果两个线程同时调用同一个SimpleDateFormat的静态变量dateFormat的format方法时结果可能是混乱的。dateFormat的内部数据结构可能会因为并发处理而混乱。当然解决方法是每个线程一个单独的SimpleDateFormat对象，不过确实浪费。
ThreadLocal<> initalValue()
锁测试和超时
tryLock()方法可以试图申请一个锁，成功则返回true，不成功再等待，超时后返回false。
tryLock()在方法的等待时间时如果被中断会抛出InterruptedException并退出，这个可以反过来打破死锁，比如向死锁线程发送中断信号。
同理await也有超时等待设置。特别的awaitUninterruptibly可以对InterruptedException不予理会，继续等待。
读/写锁
ReentrantReadWriteLock类，readLock()和writeLock()。
适用于读的多写的少，或者写的多读的少情况。
stop，suspend阻塞，resume天生不安全，容易导致死锁和意外情况。
安全的挂起线程需要引入boolean变量来使得线程在suspend挂起的时候没有影响到其他线程需要对象的地方。
14.6 阻塞队列
上节内容是Java底层实现并发程序设计的技术。但是，正常使用中应当考虑阻塞队列。
比如根关键字查找某目录下包含子目录的文件及文件内容。
生产者线程负责枚举所有的文件，并添加到队列中去。
搜索线程负责取出队列的东东，然后匹配判断。
队列数据结构可以作为一种同步机制，但是显式的线程同步也是需要的。
14.7 线程安全的集合
映射表（HashMap），有序表（SkipListMap，SkipListSet），队列（LinkedQueue）。
这些集合往往采用复杂的算法，来确保并发访问的安全性。
写数组的拷贝CopyOnWriteArrayList，和CopyOnWriteArraySet。
较早的线程安全集合Vector和HashTable，以及后来的ArrayLIst和HashMap，不过他们并不是安全的，可以通过同步包装器（synchronization wrapper）来加锁等确保安全。
不过再此建议使用synchronizedList和synchronizedMap，而不是自己手动封装。
比如：List<E> synchArrayList=Collections.synchronizedList(new ArraryList<E>());
   Map<K,V> synchHashMap=Collections.synchronizedMap(new HashMap<K,V>());
14.8 Callbale和Future
Runnable封装一个异步运行的任务，是一个没有参数和返回值的异步方法。
而Callable是有返回值和参数的，只有一个方法call()
public interface Callable<V>{
V call() throws Exception;
}
类型参数是返回值的类型。
Future保存异步计算的结果，Future可以将计算结果保存好，可以稍后获取。
可以把结果放在ArrayList里，比如ArrayList<Future<Integer>>
14.9 执行器
线程池（thread pool）用来解决程序中大量生命期比较短的线程。当run方法结束后，线程不会死亡，而是再等待为下一个请求服务。同时创建大量的线程也大大降低性能，甚至使得虚拟机崩溃。所以一个固定数目的线程池是有必要的。
执行器（Executor）类用于构建线程池。
1调用newCachedThreadPool,newFixedThreadPool,newSingleThreadPool等静态方法
2调用submit提交Runnable或Callable对象
3如果要取消一个task，或提交Callable对象，需要保存好返回的Future对象，因为线程池别的进程也会使用。
4shutdown,shutdownNow
预定执行（scheduledExecutorService）或重复执行。允许线程池机制的Timer的泛化。
控制任务组invokeAny。比如因式分解求解RSA，可以一次提交很多任务，只要有一个得到答案，其他的可以结束。
Fork-Join框架
用以将一个大问题划分为小部分，同时计算。RecursiveTask
在后台，fork-jion框架使用了有效的智能方式来保持线程的工作负载。称为工作密取(work stealing)。每个工作线程都有一个双端队列(deque)来完成任务。
比如在100000000个随机0-1的小数中，统计符合要求的数字和，比如x>0.5。就可以采fork-join框架：在compute方法内，invokeAll接收很多小任务后阻塞，知道所有的小任务完成后再返回，而join方法获得各个小任务的返回值。
同步器（java.util.concurrent）
管理相互合作的线程集。
信号量，倒计时门栓（CountDownLatch，下一个任务等待当前的任务计数减为0再运行），障栅（barrier，将运行进度不一致的线程统一起来，到某个阶段都同步起来再继续执行），交换器（Exchanger，共用数据缓冲区，一个写，一个读，完事再相互交换），同步队列（synchronousQueue，数据沿着一个方向传递，一个只负责put生产，另一个就一直take消费，不相互交换）
14.11 线程与Swing
在界面中使用线程是为了提高响应的性能。当程序再做某一个耗时的任务时，应当启动另一个线程来继续与用户接口交互。Swing并不是线程安全的，也就是说，多线程会导致界面崩溃，而程序的逻辑功能可能继续运行。Swing不使用同步机制的原因在于，同步需要大量的时间，降低速度，另一方面，同步机制比较容易搞混，造成死锁。
运行耗时的任务
如果线程任务耗时，那把它通过分配任务线程扔到一个独立的线程吧
除了事件分配线程，其他的线程与界面无关，尽量分开他们。单一线程规则（single-thread rule）
事件分配线程不要进行IO操作等耗时或者资源操作，很可能会卡死，也不要调用sleep()方法，如果需要等待指定的时间，建议使用定时器事件。
这两条规则看起来彼此冲突，比如进行一个下载操作，启动下载线程，同时在界面上更行进度条。但是下载进程不能接触Swing组件。为此，可以将界面更新操作放在invokeLater和invokeAndWait里。
EventQueue.invokeLater(new Runnable(){
public void run(){
label.setText(percentage+"ok");
}
});
invokeLater会立即返回继续执行原来程序的接下来的部分，而run的方法会异步执行。例子读取长文件文本内容。
单一线程机制
main就是一个单一的线程。在Swing程序中，main的生命周期比较短暂，基本就是构建用户界面后退出，接下来由事件分配处理线程来接管。Swing中大部分方法是不安全的，只有很少但很有用的是安全的。
JTextComponent.setText
JTextArea.insert
JTextArea.append
JTextArea.replaceRange
JComponent.repaint
JComponent.revalidate
```

> 卷I - 终

