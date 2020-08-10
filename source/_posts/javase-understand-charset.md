---
title: 理解字符集和编码以及Java中的乱码解决
date: 2016/10/02
categories:
- 学习
tags:
- Java
- 学习
- 知识点
---

理解字符集和编码以及Java中的乱码解决
================================
回顾复习一下字符集和编码，以及如何解决Java中乱码的问题，也查阅整理了Java中char的含义。


## 基本概念

1、字符、字节、字

__字符__：人类使用的记号，抽象意义上的一个符号。比如阿拉伯数字“1”，英文字母“A”，中文汉字“中”。

__字节__：计算机中的一个存储单位，8个比特为一个字节。

> 在C语言中，用char来表示一个字节，如果用于存储字符的话则对应的为ASCII字符集编码表，而Java中byte表示一个字节，而char表示的是Unicode字符集，UTF-16BE(Big Edition)编码表。

__字__：这个也是一个计算机中的存储单位，英文为WORD，表示两个字节。对应的还有DWORD,QWORD。

> Java中WORD-short(两个字节),DWORD-int(四个字节),QWORD-long(八个字节)。

注：有些教科书中讲“一个汉字占两个字节”是不全面的，准确的说“在有些汉字编码（比如GB2312）中，一个汉字占两个字节，而再其他编码中，汉字不一定占两个字节，比如UTF系列”。

2、字符集、编码

__字符集__：首先是一个集合，集合内包含了一些被选择的字符。是一种__规范__。

> 比如GBK包含了GB2312的字符，但是GB2312内没有GBK的一些字符。

__编码__：字符如何在计算机中存储，每个字符用一个字节还是多个字节，以及存储的具体格式。是一种__方式__，用于传播和存储。

> 比如ASCII均用一个字节来存储字符，而GB2312中一个字节且序号低于127的表示原来ASCII内的字符，两个字节且每个字节序号都高于127的表示汉字等，也是GB2312与ASCII兼容的由来；在GBK中，高字节序号低于127表示源来的ASCII内的字符，高字节序号高于127则连同低字节一起两个字节表示汉字等。（高字节可以理解为第一个字节，低字节为第二个字节）

注：不少编码和字符集是同时制定的，如GB2312既表示字符集也表示编码。而Unicode则表示字符集，UTF-8，UTF-16等表示编码。

3、ASCII、ANSI、Unicode

这三者其实是字符集发展的历史历程。ASCII -> ANSI -> Unicode

__ASCII__：American Standard Code for Information Interchange。这个是最早在DOS系统中使用,包括一些控制字符，数字，英文字母大小写，特殊字符，对于英文环境足够使用。字符集中的字符序号和实际编码格式是一样的。

__ANSI__：American National Standards Institute。美国国家标准协会。这不是一个编码，而是一种标准。比如在简体中文的Windows操作系统中ANSI指的是GB2312，在繁体中文的Windows操作系统中ANSI指的是BIG5，在日文操作系统中ANSI指的是JIS。其实这个很蛋疼，比如日本的网友发了一个ANSI标准(JIS编码)的文本，中国伙伴也以ANSI(GB2312)打开，却是乱码。

__Unicode__：万国码。主要是将全世界的字符放到一个字符集合中，解决跨平台和跨语言的问题。Unicode有很多编码方案，常用的是UTF-8，还有UTF-16,UTF-32等。UTF( UCS Transformation Format)是Unicode的具体编码实现方式。

补充：
1、ANSI转其他编码比如Unicode是人为规定的，而Unicode转其他编码是可以算出来的。
2、ISO-8859-1是最简单的单字节编码。
3、这里再介绍一下UTF-8：

UTF-8是变长编码，编码长度和Unicode编码有关，UTF-8（一到四个字节）兼容ASCII，UTF-16（两个或四个字节）不兼容ASCII。

| Unicode序号(十六进制) | UTF-8 字节流(二进制)                |
| --------------------- | ----------------------------------- |
| 000000-00007F         | 0xxxxxxx                            |
| 000080-0007FF         | 110xxxxx 10xxxxxx                   |
| 000800-00FFFF         | 1110xxxx 10xxxxxx 10xxxxxx          |
| 010000-1FFFFF         | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx |

摘自百度的例子：“汉”这个字的Unicode编码是0x6C49。0x6C49在0x0800-0xFFFF之间，使用__三字节__模板：1110xxxx 10xxxxxx 10xxxxxx。将0x6C49写成二进制是：0110 1100 0100 1001， 用这个比特流依次代替模板中的x，得到：11100110 10110001 10001001，即E6 B1 89。汉字在UTF-8中范围是[0x4e00,0x9fbb]，三字节编码。

__注意__：在编程中，源文件的编码格式也很重要，一个项目中的源文件编码应当统一，一般使用UTF-8。否则，会出现字符显示异常，甚至莫名其妙的编译问题。

## 常见现象

1、存储的数据是对的，打开文档时选错编码格式。
现象：以GBK编码写了一篇文档，打开时的软件却错误的选择了UTF-8的编码，从而导致中文显示乱码，英文和数字正常。
分析：GBK和UTF-8都是兼容ASCII的，所以英文和数字正常。而中文在GBK和UTF-8这两个编码方式中是不一样的，换句话说，存储的数据是不一样的。
解决：这个时候只要设置打开文本的软件编码格式为GBK，再重新打开文本就行，此时千万不要修改和保存文档。一般以什么编码保存，就以什么编码再打开最好。

2、将文本进行转码
解决：如果原先文本是以GBK编码存储，想转为UTF-8，则将原文本以GBK格式先打开，然后复制文本的内容到剪贴板，删除原文本文件内的所有内容，更改文本的存储编码为UTF-8，再粘贴刚才复制的内容保存即可。

3、打开文本发现乱码后，又修改了文本，并保存。
解决：这个解决起来很麻烦，而且恢复成功率不高，取决于文本编辑器如何对文本进行保存。

## Java中的char

前面已经说到，Java中的char是采用Unicode字符集，UTF-16BE的编码。UTF-16使用两个或四个字节表示一个字符，Unicode编号范围在65536以内的占两个字节，超出范围的占四个字节。
BE (Big Endian)的意思大端，即是先输出高位字节，再输出低位字节，这与整数的在内存中的表示是一样的。
换句话说__char只能表示编号在65536以内的字符，超过部分需要用两个char凑四个字节表示__。

Java的源文件如果是GBK编码，那么打开这个源文件也需要用GBK编码。如果IDE不支持GBK，比如使用英文操作系统，那么就会默认以其他如UTF-8编码打开，这样就会显示乱码。因此，最好统一项目源码为UTF-8编码。

在C语言中,char是一个字节，只能表示256个字符，而Java中的char就可表示的字符大了不少。在Java中，用一个char就可以表示汉字。另外，也可以直接赋数值，或者赋Unicode编码序号值。

```Java
char c1 = '汉';
char c2 = '\u6c49';
int i3 = c1;// 27721
char c3 = (char) i3;
char c4 = 0x6c49;
System.out.println(c1);
System.out.println(c2);
System.out.println(i3);
System.out.println(c3);
System.out.println(c4);
System.out.println(Integer.toBinaryString(c1));

if ((c1 >= 0x4e00) && (c1 <= 0x9fbb)) {
    System.out.println("中文");
}
```

输出：
> 汉
汉
27721
汉
汉
110110001001001
中文

char本质上也是数值，它和short一样是占两个字节，只不是它是无符号数，范围和short不一样，char可以参与数值运算、位运算，但是作为数值赋值和运算时会自动转型为int。

char的运算主要运用在判断某个字符是否是[0-9A-Za-z]，转换字母大小写，这点和学C语言时char的作用是类似的。

## Java中的转码

在WEB项目中，字符编码的问题一直很头疼，很容易就遇到乱码的问题。

先说一下历史背景：
```
RFC 1738规定：只有字母和数字[0-9a-zA-Z]，一些特殊符号如
$  (美元)、
-  (负号，减号)、
_  (下划线)、
.  (点)、
+  (加号)、
!  (英文感叹号)、
*  (星号)、
'  (英文单引号)、
(  (英文左括弧)、
)  (英文右括弧)、
,  (英文逗号)，
以及某些保留字，才可以不经过编码直接用于URL
```
也就是说，除规定之外的其他字符不能出现在URL中。这个规定是好的，但是RFC 1738没有规定具体的编码方法，而是交给各个浏览器厂家自行制定方案。
这也就是如今令人头疼的乱码问题的由来，不同的操作系统、不同的浏览器、不同的字符集，将导致完全不同的编码结果。

```
URL格式：schema://host[:port#]/path/.../[;url-params][?query-string][#anchor]
```

那么有没有什么比较好的办法解决这个问题呢？
解决：本文写于2016年，现在的字符编码问题已经好了很多。主流的Chrome浏览器默认对URL的query部分和URL的path中的非RFC 1738的字符采用UTF-8编码，而有些老浏览器编码则是与操作系统甚至页面有关。
如果需要兼容这一部分老浏览器，最好的做法是在前端使用JS中encodeURL方法对URL__进行UTF-8编码__。也就是说页面中的超链接href，URL中的参数，AJAX.GET中的url，最好都使用encodeURL方法来屏蔽差异。
界面的显示字符编码可以在meta中注明使用的charset。

GET：
```
1、GET /JavaWeb-Test/charSet?req=123你好abc HTTP/1.1
Host: localhost:8080
Cache-Control: no-cache
Postman-Token: 82b3c5f8-8463-be94-f1ff-b81b9ef5400d

2、GET /JavaWeb-Test/charSet?req=123%E4%BD%A0%E5%A5%BDabc HTTP/1.1
Host: localhost:8080
Cache-Control: no-cache
Postman-Token: bec648b3-3733-2b4f-7b85-edd226708e91
```
```java
protected void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
    request.setCharacterEncoding("UTF-8");
    response.setCharacterEncoding("UTF-8");
    response.setContentType("text/html;charset=utf-8");

    PrintWriter pw = response.getWriter();
    String req = request.getParameter("req");
    System.out.println(req);
    pw.println(req);
    pw.println("<br/>");
    byte[] bytes = req.getBytes("ISO-8859-1");
    String tras = new String(bytes, "UTF-8");
    System.out.println(tras);
    pw.println(tras);
    pw.println("<br/>");

    //tras为最终的正确参数
}
```

POST：
```
1、POST /JavaWeb-Test/charSet HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded
Cache-Control: no-cache
Postman-Token: 54d9777a-e4c3-305a-9385-0c0941875e09

req=123%E4%BD%A0%E5%A5%BDabc

2、POST /JavaWeb-Test/charSet HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded
Cache-Control: no-cache
Postman-Token: cc92d9c9-d864-8b39-4b05-8ec9828b74dd

req=123你好ABC
```
```java
protected void doPost(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
    request.setCharacterEncoding("UTF-8");
    response.setCharacterEncoding("UTF-8");
    response.setContentType("text/html;charset=utf-8");

    PrintWriter pw = response.getWriter();
    String req = request.getParameter("req");
    System.out.println(req);
    pw.println(req);
    pw.println("<br/>");

    //req为最终的正确参数
}
```


补充：在项目中到底使用UTF-8还是GBK:
现在项目最好使用UTF-8，特别是面向多终端，不同的操作系统，不同的国家。因为有些英文操作系统并不默认携带有GBK的编码，而UTF-8被广泛的支持。
如果仅仅是中文的网站，可以使用GBK，因为GBK每个字符只占两个字节，UTF-8中汉字占三个字节，是变长编码，使用GBK节省空间。
当然，无论是GBK还是UTF-8，对英文和数字支持都是没有问题的。

## 其他知识点

整理自[这里](http://polaris.blog.51cto.com/1146394/377468)

### UTF字节序和BOM

__字节序（大小端）__：

UTF-8以字节为编码单元，没有字节序的问题。
UTF-16以两个字节为编码单元，在解释一个UTF-16文本前，首先要弄清楚每个编码单元的字节序。

问题：例如收到一个“奎”的Unicode编码是594E，“乙”的Unicode编码是4E59。如果我们收到UTF-16字节流“594E”，那么这是“奎”还是“乙”？
解决：在UCS编码中有一个叫做"ZERO WIDTH NO-BREAK SPACE"的字符，它的编码是FEFF。而FFFE在UCS中是不存在的字符，所以不应该出现在实际传输中。UCS规范建议我们在传输字节流前，先传输字符"ZERO WIDTH NO-BREAK SPACE"。这样如果接收者收到FEFF，就表明这个字节流是Big-Endian的；如果收到FFFE，就表明这个字节流是Little-Endian的。因此字符"ZERO WIDTH NO-BREAK SPACE"又被称作BOM。

__BOM__：

UTF-8没有字节序的问题，所以不需要BOM来表明字节顺序。但是可以用BOM来表明编码方式。字符"ZERO WIDTH NO-BREAK SPACE"的UTF-8编码是EF BB BF（读者可以用我们前面介绍的编码方法验证一下）。所以如果接收者收到以EF BB BF开头的字节流，就知道这是UTF-8编码了。

__BOM与XML__:

XML解析读取XML文档时，W3C定义了3条规则：
 
①如果文档中有BOM，就定义了文件编码；
> 如果是FF FE则为UTF-16LE
如果是FE FF则为UTF-16BE
如果是EF BB BF则为UTF-8

②如果文档中没有BOM，就查看XML声明中的编码属性；
比如：
```
<?xml version="1.0" encoding="UTF-8"?>
```

③如果上述两者都没有，就假定XML文档采用UTF-8编码。

### Windows记事本支持的编码

（1）ANSI编码 
记事本默认保存的编码格式是：ANSI，即本地操作系统默认的内码，简体中文操作系统一般为GB2312。
这个怎么验证呢？用记事本保存后，使用EmEditor、EditPlus和UltraEdit之类的文本编辑器打开。打开后，在右下角会显示编码：GB2312。
 
（2）Unicode编码 
用记事本另存为时，编码选择“Unicode”，用EmEditor打开该文件，发现编码格式是：UTF-16LE+BOM（有签名）。
用十六进制方式查看，发现开头两字节为：FF FE。这就是BOM。
 
（3）Unicode big endian 
用记事本另存为时，编码选择“Unicode”，用EmEditor打开该文件，发现编码格式是：UTF-16BE+BOM（有签名）。
用十六进制方式查看，发现开头两字节为：FE FF。这就是BOM。
 
（4）UTF-8 
用记事本另存为时，编码选择“UTF-8”，用EmEditor打开该文件，发现编码格式是：UTF-8（有签名）。
用十六进制方式查看，发现开头三个字节为：EF BB BF。这就是BOM。


### 编辑软件自动判断文件的编码方式

> 首先说一个奇特的例子：
在简体中文Windows环境下，新建记事本文档为test.txt,写入"联通"两个字，默认ANSI编码保存并关闭，再打开test.txt文本文件后后发现变成一个方块黑点（Win10中为两个菱形）。

软件通常有三种途径来决定文本的字符集和编码。
 
（1）对于Unicode文本最标准的途径是检测文本最开头的几个字节。如：
 
| 开头字节    | Charset/encoding                      |
| ----------- | ------------------------------------- |
| EF BB BF    | UTF-8                                 |
| FE FF       | UTF-16/UCS-2, little endian(UTF-16LE) |
| FF FE       | UTF-16/UCS-2, big endian(UTF-16BE)    |
| FF FE 00 00 | UTF-32/UCS-4, little endian           |
| 00 00 FE FF | UTF-32/UCS-4, big-endian              |
 
（2）采取一种比较安全的方式来决定字符集及其编码，那就是弹出一个对话框来询问用户。
 
然而MBCS文本（ANSI）没有这些位于开头的字符集标记，现在很多软件保存文本为Unicode时，可以选择是否保存这些位于开头的字符集标记。
因此，软件不应该依赖于这种途径。这时，软件可以采取一种比较安全的方式来决定字符集及其编码，那就是弹出一个对话框来询问用户。
 
（3）采取自己“猜”的方法。
 
如果软件不想麻烦用户，或者它不方便向用户请示，那它只能采取自己“猜”的方法，软件可以根据整个文本的特征来猜测它可能属于哪个CharSet，这就很可能不准了。

> 现在再来解释一下之前的奇特的例子：
使用UltraEdit打开那个test.txt,可以看到十六进制下为C1 AA CD A8，二进制则为‭11000001 10101010 11001101 10101000，观察前文的UTF-8编码，‬符合__模板二__，因而记事本会认为这是UTF-8编码存储的，打开后自然是乱码了。可以设置UltraEdit右下角打开的编码为GB2312或GBK，就可以正常显示。

### 谈谈“烫烫烫”和“棍斤拷”
[参考1](https://zhuanlan.zhihu.com/p/27253604) [参考2](http://blog.csdn.net/mig_davidli/article/details/37507731)

在学习C/C++语言时，调试状态下有时会出现“烫烫烫”和“屯屯屯”。而在学习Java时，在CMD调试或者WEB项目时，会碰到“浣犲ソ”、“涓 浗”和“钝斤拷”，前端则会遇到“锘锘锘”和“锘匡豢”。

前者主要原因是C语言调试器会默认将栈内的未初始化数据初始化为"0xcc"，堆内的未初始化数据初始化为"0xcd"。0xcc对应的是中断向量表中的3号中断（INT 3)。 微软的调试器之所以会写入一些特定的字符，主要是为了便于检查出程序在调试过程中发生错误的原因，就像flag一样。因此一定要注意数组的初始化、数组的越界检查，字符串末尾的0，出现了“烫烫烫”总比程序崩溃要好很多。

中者主要是字符编码和转码的问题。Unicode尽管被成为万国码，但是和其他编码转化时仍然会有一些字符是用Unicode没法表示的，Unicode转为其他编码可以算出来，而其他编码转为Unicode只能人工写一个转换表，因此Unicode使用了一个占位符来表示这些没有被收编的字符，这就是：U+FFFD REPLACEMENT CHARACTER。另一方面，U+FFFD使用UTF-8编码结果是0xef 0xbf 0xbd。如果这个0xef 0xbf 0xbd被重复多次，例如 0xef 0xbf 0xbd 0xef 0xbf 0xbd，然后放到GBK环境中显示的话，一个汉字占2个字节，那么最终的显示就变成了“锟斤拷”。因此在涉及到转码的时候要格外注意转码。

后者主要是前文讲的BOM问题。BOM在UTF-8表示为EF BB BF。两个EF BB BF就总共是六个字节，在GBK编码中刚好是三个汉字，也就是“锘匡豢”。因此在编写HTML等页面时不使用BOM，而在页面中注明charset=utf-8，页面本身以UTF-8编码保存即可。

下面在简体中文操作系统中复现这些现象：

1、“烫烫烫”和“屯屯屯”
```c
#include <stdio.h>
#include <stdlib.h>

int main(){
    //栈：烫烫烫
    char a[10];
    printf("%s\n",a);

    //堆：屯屯屯
    int* b = (int* )malloc(sizeof(int)*10);
    printf("%s\n",b);

    free(b);
    b = NULL;

    return 0;
}
```

2、“浣犲ソ”和“钝斤拷”
```java
@Test
public void test6() throws UnsupportedEncodingException {
    /* 错误的转码 */

    // 浣犲ソ
    String str = "你好";
    byte[] bytes = str.getBytes("UTF-8");
    String s = new String(bytes, "GBK");
    System.out.println(s);

    // 涓 浗
    String str2 = "中国";
    byte[] bytes2 = str2.getBytes("UTF-8");
    String s2 = new String(bytes2, "GBK");
    System.out.println(s2);

    // 锟斤拷
    String str3 = "联通";
    byte[] bytes3 = str3.getBytes("UTF-8");
    String s3 = new String(bytes3, "GBK");
    // System.out.println(s3);
    byte[] bytes4 = s3.getBytes("UTF-8");
    for (byte b : bytes4)
        System.out.print(Integer.toHexString(b) + " ");
    System.out.println();
    String s4 = new String(bytes4, "GBK");
    System.out.println(s4);
}
```

3、“锘锘锘”和“锘匡豢”
现在基本遇不到了。

4、使用Java汇总模拟
```java
@Test
public void test5() throws UnsupportedEncodingException {

    byte[] bytes = new byte[100];
    // 烫烫烫
    bytes[0] = (byte) 0xcc;
    bytes[1] = (byte) 0xcc;
    bytes[2] = (byte) 0xcc;
    bytes[3] = (byte) 0xcc;
    bytes[4] = (byte) 0xcc;
    bytes[5] = (byte) 0xcc;
    // 屯屯屯
    bytes[6] = (byte) 0xcd;
    bytes[7] = (byte) 0xcd;
    bytes[8] = (byte) 0xcd;
    bytes[9] = (byte) 0xcd;
    bytes[10] = (byte) 0xcd;
    bytes[11] = (byte) 0xcd;
    // 锘匡豢
    bytes[12] = (byte) 0xef;
    bytes[13] = (byte) 0xbb;
    bytes[14] = (byte) 0xbf;
    bytes[15] = (byte) 0xef;
    bytes[16] = (byte) 0xbb;
    bytes[17] = (byte) 0xbf;

    System.out.println(new String(bytes, "GBK"));
}
```

最后附上一首诗：
手持两把锟斤拷, (GBK与UTF-8)
口中疾呼烫烫烫。(VC++)
脚踏千朵屯屯屯, (VC++)
笑看万物锘锘锘。(HTML)

> 完
