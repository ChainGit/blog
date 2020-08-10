---
title: Java的异常分类和处理原则
date: 2017/05/04
categories: 学习
tags:
- 学习
- Java
- 知识点
---

Java的异常分类和处理原则
==========================
刚刚学习Java时，感觉异常处理很累赘，在发生异常时就随便的try-catch或者throw，也不理解为什么要设立这个复杂的处理机制。直到真正着手做一些练习或者项目时，才日益发觉异常处理的重要性，胡乱处理既给开发的程序留下隐患，也对开发调试不利。为此，整理了一下异常相关的知识，也感谢《老马说编程》的文章。

## 基础概念

Java的异常处理机制底层比较复杂，作为开发者，详细了解底层机制并不是首要的，而是如何去正确的使用它。

Java的异常分类基本结构如下：

Throwable
    |— Exception
       |— RuntimeException
    |— Error

这里还有一个摘自微信《老马说编程》的一张图片：

![image](/uploads/javase-exception/exception.jpg)

Throwable是基类，定义了stackTrace，detailMessage，在1.4后添加了cause异常链支持，异常链能使得程序在发生异常处打印出Cause by，更能准确定位错误发生的位置和起因。Throwable里有常用的printStackTrace()方法，用以遍历打印出错误栈信息和异常链信息，了解一下这个方法对异常处理的流程也能有个基本认识。

Error往往是不可处理的，不可挽救的错误，比如VirtualMacheError，OutOfMemoryError，StackOverFlowError等等。程序中遇到这些错误，往往只能被虚拟机干掉。

Exception分为checked和unchecked两种，unchecked均是继承自RuntimeException，在代码种遇到unchecked异常可以不用catch或者throw，而checked必须要求程序员处理。

在Java中，方法执行return代表正常退出，throw代表异常退出。这和学习c语言时有不同，c语言中-1是程序执行异常结束，而0是正常执行结束。c语言中这种方法比较方便，但是程序中异常判断的标准和业务代码混在一起，如果业务结果确实是-1，那么还能简单的认为异常退出吗？换句话说，在程序中如果一个函数返回了-1，有时是处理的数据结果是-1，有时确实时异常，比如文件读写出错，程序终止，EXIT_CODE为-1。比如在做除法时，C语言中可以这样写：
```c
int div(int a, int b) {
    if (b == 0)
        return -1;
    int res = a / b;
    return res;
}
```
如果调用的参数时div(2,-2)，那么执行的返回结果是-1，又如果是div(2,0)，执行的返回结果是也是-1。这就有不足的地方，此时如果将b==0做一个特殊处理，能够使得程序员知道这是异常，那个是正确的结果，会好很多，因此异常机制存在是有道理的。

在C#中，异常均为unchecked，而java中，区分了为checked和unchecked，这两者的讨论一直没有到底哪个好，哪个不好的结果。

异常栈：在Java中，一个方法的正确执行后会执行return语句，而执行期间发生错误，则立即停止执行该方法，将异常返回到调用者，调用者可以获得异常栈信息。异常栈信息包括了从异常发生点到最上层调用的轨迹，也还包括行号，在异常分析中起到关键作用。不过这是默认的程序执行的规则，如果代码中加入catch则会由细微变化。

## 异常处理

1、unchecked异常可以不用catch或者throw，而checked必须要求程序员处理，程序员可以选择将错误在方法内catch处理掉，也可以选择直接throws出去，当然也可以在catch内做一些其它操作后再throw出去。

2、无论是checked还是unchecked，如果在方法中，对异常做catch操作，则只是try-catch代码块中的剩余代码不会执行，之外的代码正常执行；若对发生的异常不做catch处理，而是仅仅在方法上加上throws抛出，那么一旦方法出现异常，则异常点之后往下的剩余代码均不执行，相当于整个方法的代码都在try代码块中。

3、finally总能在方法（正常或异常）返回到上一层调用者前执行，除非finally有System.exit(0)，此时程序会整个终止，如果再做返回没有任何意义。在finally内，不建议对方法中的值做修改，因为发生异常时，原来的方法中值会被保护，finally中修改的值对原方法中的变量无效；也不要有return，因为finally主要是处理异常的，存在return会使得finally下面的代码无法执行，另一方面也会造成原来的异常信息丢失；也不要在finally里再抛出异常，原因和return差不多。因此，finally主要用于关闭在try中打开着的资源，无论try中的代码是否正确执行。

4、就算方法中什么异常也没有，方法上仍然是可以有throws异常的。当覆盖方法时，只能抛出在基类方法的异常说明里列出的那些异常，有些基类的方法声明抛出异常其实并没有抛出异常，这是因为可能在其子类的覆盖方法中会抛出异常。

5、构造器可以抛出任何异常而不必理会基类构造器所抛出的异常，派生类构造器异常说明必须包含基类构造器异常说明，因为构造派生类对象时会调用基类构造器。此外，派生类构造器不能捕获基类构造器抛出的异常。

6、异常不能代替正常的条件判断，比如在数组遍历时，应当认真判断数组的下标，不应该让虚拟机来抛出ArrayOutOfIndexException；反过来，真正出现异常的时候，应该抛出异常，而不是返回特殊值，比如在计算除法时，对除数为0时应该抛出异常，而不是返回-1；另外，异常不能假装当正常处理，发生异常时除非特殊情况，一定要处理，不能catch了但什么也没做，printStackTrace()不能算作正确的异常处理，但是一种处理的方式。比如在伤心时，哭不能解决问题，但是能够让别人了解到，也能够释放情绪。

7、如果发生了异常，程序最终的一定会做的操作是printStackTrace()。就算程序中没有体现，虚拟机也会调用这个方法，打印出异常信息。

8、根据实际情况选择抛出unchecked异常，checked异常，和返回特殊值，这三种处理都不是完全排他的，而是相互融合的，__这也是异常处理的困难的地方__。

下面看几段代码：
```java
// unchecked
public void test8() {
    System.out.println("111");

    try {
        System.out.println("222");
        int a = 1 / 0;
        System.out.println("333");
    } catch (Exception e) {// ArithmeticException - RuntimeException
        e.printStackTrace();
    } finally {
        System.out.println("444");
    }

    System.out.println("555");
}

// unchecked
public void test9() {
    System.out.println("111");

    int a = 1 / 0;// 隐含的有一个ArithmeticException - RuntimeException

    System.out.println("555");
}

// checked
public void test10(String dataStr) {
    DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    System.out.println("111");

    try {
        System.out.println("222");
        Date date = df.parse(dataStr);
        System.out.println(date);
        System.out.println("333");
    } catch (Exception e) {// ParseException - Exception
        e.printStackTrace();
    } finally {
        System.out.println("444");
    }

    System.out.println("555");
}

// checked
public void test11(String dataStr) throws ParseException {
    System.out.println("111");
    DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    df.parse(dataStr);
    System.out.println("222");
}
```
测试代码：
```java
@Test
public void test3() {
    MyTest t = new MyTest();
    t.test8();
}

@Test
public void test4() {
    MyTest t = new MyTest();
    t.test9();
}

@Test
public void test5() {
    MyTest t = new MyTest();
    t.test10("2016年11月2日 12时08分29秒");
}

@Test
public void test6() {
    MyTest t = new MyTest();
    t.test10("2016-11-02 12:08:29");
}

@Test
public void test7() {
    MyTest t = new MyTest();
    try {
        t.test11("2016年11-02 12:08:29");
    } catch (ParseException e) {
        e.printStackTrace();
    }
}
```
执行结果：
```java
test3:
111
222
java.lang.ArithmeticException: / by zero
	at com.chain.javase.test.day08.MyTest.test8(MyTest.java:84)
	at com.chain.javase.test.day08.ExceptionTest.test3(ExceptionTest.java:41)
    ...
444
555

test4:
111
但junit抛出：java.lang.ArithmeticException: / by zero

test5:
111
222
java.text.ParseException: Unparseable date: "2016年11月2日 12时08分29秒"
	at java.text.DateFormat.parse(Unknown Source)
	at com.chain.javase.test.day08.MyTest.test10(MyTest.java:111)
	at com.chain.javase.test.day08.ExceptionTest.test5(ExceptionTest.java:53)
	...
444
555

test6:
111
222
Wed Nov 02 12:08:29 CST 2016
333
444
555

test7:
111
java.text.ParseException: Unparseable date: "2016年11-02 12:08:29"
	at java.text.DateFormat.parse(Unknown Source)
	at com.chain.javase.test.day08.MyTest.test7(MyTest.java:127)
	at com.chain.javase.test.day08.ExceptionTest.test7(ExceptionTest.java:66)
    ...

```

## 异常典例

1、(Unchecked)RuntimeException - NullPointerException：
```java
@Test
public void test(Person person) {
    String name = person.getName();
    System.out.println(name);
}
```

2、(checked)Exception - ParseException
```java
public void test7(String dateStr) throws ParseException {
    DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    Date date = df.parse(dateStr);
    System.out.println(date);
}
```

由1，2可以看出，unchecked主要是指运行过程中容易发生的异常，比如person可能为null，根本无法调用getName方法，就会报出空指针异常，这个是编译器无法避免的异常；再看ParseException，如果传入的dateStr字符串不是一个符合pattern的时间日期，而是比如"2017年05月01日 12时08分28秒"，那么就无法转换成Date类，就会抛出异常。对比这两个异常，一个checked，一个unchecked，区别主要在于checked异常一定要程序员做处理，而unchecked不强制要求处理。对于checked和unchecked后文会继续讨论。

3、一段使用异常链的代码：
```java
public void f() throws MyException {
    throw new MyException("自定义异常");
}

public void g() throws Exception {
    try {
        f();
    } catch (MyException e) {
        e.printStackTrace();//第一次打印栈轨迹
        throw new Exception("重新抛出的异常1", e);
    }
}

public void h() throws IOException {
    try {
        g();
    } catch (Exception e) {
        // TODO Auto-generated catch block
        e.printStackTrace();//第二次打印栈轨迹
        IOException io = new IOException("重新抛出异常2");
        io.initCause(e);
        throw io;
    }
}

@Test
public void test2() {
    MyTest t = new MyTest();
    try {
        t.h();
    } catch (IOException e) {
        e.printStackTrace();//第三次打印栈轨迹
    }
}
```

调用h方法后打印出的精简信息：
[完整信息点击此处查看](/uploads/javase-exception/example.txt)
```java
第一次打印：
com.chain.javase.test.day08.MyException: 自定义异常
    at com.chain.javase.test.day08.MyTest.f(MyTest.java:50)
    at com.chain.javase.test.day08.MyTest.g(MyTest.java:55)
    at com.chain.javase.test.day08.MyTest.h(MyTest.java:64)
    at com.chain.javase.test.day08.ExceptionTest.test2(ExceptionTest.java:31)
    ...

第二次打印：
java.lang.Exception: 重新抛出的异常1
    at com.chain.javase.test.day08.MyTest.g(MyTest.java:58)
    at com.chain.javase.test.day08.MyTest.h(MyTest.java:64)
    at com.chain.javase.test.day08.ExceptionTest.test2(ExceptionTest.java:31)
    ...
Caused by: com.chain.javase.test.day08.MyException: 自定义异常
    at com.chain.javase.test.day08.MyTest.f(MyTest.java:50)
    at com.chain.javase.test.day08.MyTest.g(MyTest.java:55)
    ... 25 more

第三次打印：
java.io.IOException: 重新抛出异常2
    at com.chain.javase.test.day08.MyTest.h(MyTest.java:68)
    at com.chain.javase.test.day08.ExceptionTest.test2(ExceptionTest.java:31)
    ...
Caused by: java.lang.Exception: 重新抛出的异常1
    at com.chain.javase.test.day08.MyTest.g(MyTest.java:58)
    at com.chain.javase.test.day08.MyTest.h(MyTest.java:64)
    ... 24 more
Caused by: com.chain.javase.test.day08.MyException: 自定义异常
    at com.chain.javase.test.day08.MyTest.f(MyTest.java:50)
    at com.chain.javase.test.day08.MyTest.g(MyTest.java:55)
    ... 25 more
```

由上面可以清晰的看出错误的代码具体在那个位置，尤其是加入了cause之后。

## 异常底层

看一下Throwable的常用的printStackTrace()：
```java
private void printStackTrace(PrintStreamOrWriter s) {
    // Guard against malicious overrides of Throwable.equals by
    // using a Set with identity equality semantics.
    Set<Throwable> dejaVu =
        Collections.newSetFromMap(new IdentityHashMap<Throwable, Boolean>());
    dejaVu.add(this);

    synchronized (s.lock()) {
        // Print our stack trace
        s.println(this);
        StackTraceElement[] trace = getOurStackTrace();
        for (StackTraceElement traceElement : trace)
            s.println("\tat " + traceElement);

        // Print suppressed exceptions, if any
        for (Throwable se : getSuppressed())
            se.printEnclosedStackTrace(s, trace, SUPPRESSED_CAPTION, "\t", dejaVu);

        // Print cause, if any
        Throwable ourCause = getCause();
        if (ourCause != null)
            ourCause.printEnclosedStackTrace(s, trace, CAUSE_CAPTION, "", dejaVu);
    }
}
```
由代码可见，printStackTrace()这个方法先打印出自身this的toString信息，然后再依次打印stacktrace、suppressed exceptions、cause。

## 健壮代码

1、一个典型错误的处理方式：
```java
    try {
        OutputStreamWriter out = ...
        Connection conn = ...
        Statement stat = conn.createStatement();
        ResultSet rs = stat.executeQuery("select uid, name from user");
        while (rs.next())
            out.println("ID：" + rs.getString("uid") + "，姓名：" + rs.getString("name"));
        conn.close();
        out.close();
    }
    catch(Exception e) {
        e.printStackTrace();
    }
```

上面的代码在正常情况下，功能的实现是没有问题的，但是如果遇到异常情况，那么可能就有各种各样的问题了。

2、结合上文的一些内容，这是我认为的处理方式（每个人对异常处理的见解不一样）：
```java
//try-catch内的代码不要太多，不利于定位错误发生的地点
OutputStreamWriter out = ...
Connection conn = ...
try {
    Statement stat = conn.createStatement();
    ResultSet rs = stat.executeQuery("select uid, name from user");
    while (rs.next())
        out.println("ID：" + rs.getString("uid") + "，姓名: " + rs.getString("name"));
//分开catch错误，便于分别处理
} catch(SQLException sqlex) {
    //当发生SQL错误时，之前打印的信息可能就无效了，需要打印告知用户数据无效。
    //另外，也可以将输出放置在流中，程序执行到末尾时再调用一次性输出。
    System.err.println("系统出现错误，原先打印信息无效.");
    //记录log
    logger.error("读取数据时出现SQL错误",sqlex);
    //分类复杂的SQL错误一般让调用者再逐一catch会降低代码可读性，可以抛出RuntimeException
    throw new MySQLRuntimeException("读取数据时出现SQL错误", sqlex);
} catch(IOException ioex) {
    logger.error("写入数据时出现IO错误",sqlex);
    throw new MySQLRuntimeException("写入数据时出现IO错误", ioex);
//使用finally无论程序执行正确与否，会保证关闭资源
} finally {
    if (conn != null) {
        try {
            conn.close();
        } catch(SQLException sqlex2) {
            logger.error("不能关闭数据库连接",sqlex2);
            //对于无法关闭的异常，调用者也无法处理，不如内部直接处理掉，或者重新包装成RuntimeException
            System.err(this.getClass().getName() + ".mymethod - 不能关闭数据库连接: " + sqlex2.toString());
        }
    }

    if (out != null) {
        try {
            out.close();
        } catch(IOException ioex2) {
            logger.error("不能关闭输出文件",ioex2);
            System.err(this.getClass().getName() + ".mymethod - 不能关闭输出文件" + ioex2.toString());
        }
    }
}
```

## 异常讨论

一种普遍的说法是：

RuntimeException(unchecked exception)表示编程的逻辑错误，编程时应该检查以避免这些错误，比如说像空指针异常，如果真的出现了这些异常，程序退出也是正常的。程序员应该检查程序代码的bug，比如完善边界检查，长度判断，非空判断等等，而不是想办法处理这种异常。

checked exception表示程序代码本身没问题，但由于I/O、网络、数据库等其他不可预测的外在错误导致的异常，这类异常不是由于代码bug引起的，而是外部因素导致的，那么本方法的调用者应该想办法去处理异常。

由上面的说法，换句话说，RuntimeException其实是提醒程序员，代码写的不过健壮，要完善代码，而其他的异常就不是程序员能够控制的异常了，比如数据库连接的释放，IO的操作，好的健壮代码不应该抛出RuntimeException，但是可以抛出checked Exception。

另一种被认同的观点是：

Java中的这个区分是没有太大意义的，可以统一使用RuntimeException即unchecked exception来代替。

这个观点的基本理由是，无论是checked还是unchecked异常，无论是否出现在throws声明中，我们都应该在合适的地方以适当的方式进行处理，而不是只为了满足编译器的要求，盲目处理异常，既然都要进行处理异常，checked exception的强制声明和处理就显得啰嗦，尤其是在调用层次比较深的情况下。

个人观点：

对于有些checked exception，比如再创建String时，那个charset参数，如果传入肯定能支持的编码，那么抛出的UnsupportedEncodingException的checked exception就没有太大的再上抛给调用者的意义，所以可以包装成RuntimeException，类似的还有NoSuch异常；还有在编写工具类时，遇到关闭流的操作时的IOExcption等无法再得到有效解决的问题，也可以进行包装；再比如对于开发者而言，复杂的SQL异常的有很多种类的Exception，也可以包装成一个统一的Exception做处理，减少累赘，当然使用复杂的种类繁多的Exception也是可以的。

其实观点本身并不太重要，更重要的是一致性，一个项目中，应该对如何使用异常达成一致，按照约定使用即可。Java中已有的异常和类库也已经在哪里，我们还是要按照他们的要求进行使用。

## 参考资料

[1、Java中的常用异常处理方法](http://blog.sina.com.cn/s/blog_4ae187490100xj00.html)
[2、Java异常机制](http://www.webpiaoliang.com/xuexi/wysj/java/20170711436331.html)
[3、Java异常链](http://zy19982004.iteye.com/blog/1975986)
[4、Java的异常](http://mp.weixin.qq.com/s/8XiwZGn8djtO7TvtonAe8w)
[5、Java异常类学习](http://blog.csdn.net/touch_2011/article/details/6860043)

## 文后感想

异常处理细细学来，和黑客技术一样，感觉就像一门艺术。

异常处理的目的是在于让程序出现错误，不至于一律都选择退出，也不至于带“病”运行，而是能够让程序遇到故障能尽可能的恢复正常运行。

完善自己的代码编写才是最重要的，尽量较少不必要的异常的发生。

最简单细微的东西往往容易被忽略，最微小的细节可能会酿成不可挽回的损失。