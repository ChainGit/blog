---
title: 操作系统原理之IO管理
date: 2017/07/29
categories: 学习
tags:
- 学习
- 操作系统
- 转载
---

操作系统原理之IO管理
===============================
转载整理自[这篇博客](http://c.biancheng.net/cpp/u/xitong_5/)。

[操作系统原理之基础知识](https://www.leechain.top/blog/2017/07/15-operation-system-basic-knowledge.html)
[操作系统原理之死锁](https://www.leechain.top/blog/2017/08/24-operation-system-deadlock.html)
[操作系统原理之处理机调度](https://www.leechain.top/blog/2017/08/23-operation-system-job-schedule.html)
[操作系统原理之信号量](https://www.leechain.top/blog/2017/08/22-operation-system-semaphore.html)
[操作系统原理之进程和线程管理](https://www.leechain.top/blog/2017/08/25-operation-system-multithreads.html)
[操作系统原理之内存管理](https://www.leechain.top/blog/2017/07/25-operation-system-memory.html)
[操作系统原理之文件管理](https://www.leechain.top/blog/2017/07/21-operation-system-file.html)

## I/O设备分类

I/O设备管理是操作系统设计中最凌乱也最具挑战性的部分。由于它包含了很多领域的不同设备以及与设备相关的应用程序，因此很难有一个通用且一致的设计方案。

所以在理解设备管理之前，应该先了解具体的I/O设备类型。计算机系统中的I/O设备按使用特性可分为以下类型：

1) 人机交互类外部设备：用于同计算机用户之间交互的设备，如打印机、显示器、鼠标、键盘等。这类设备数据交换速度相对较慢，通常是以字节为单位进行数据交换。
2) 存储设备：用于存储程序和数据的设备，如磁盘、磁带、光盘等。这类设备用于数据交换，速度较快，通常以多字节组成的块为单位进行数据交换。
3) 网络通信设备：用于与远程设备通信的设备，如各种网络接口、调制解调器等。其速度介于前两类设备之间。网络通信设备在使用和管理上与前两类设备也有很大不同。

除了上面最常见的分类方法，I/O设备还可以按以下方法分类：

1) 按传输速率分类：
低速设备：传输速率仅为每秒几个到数百个字节的一类设备，如键盘、鼠标等。
中速设备：传输速率在每秒数千个字节至数万个字节的一类设备，如行式打印机、 激光打印机等。
高速设备：传输速率在数百个千字节至千兆字节的一类设备，如磁带机、磁盘机、 光盘机等。

2) 按信息交换的单位分类：
a. 块设备：由于信息的存取总是以__数据块__为单位，所以存储信息的设备称为块设备。
它属于有结构设备，如磁盘等。磁盘设备的基本特征是__传输速率较高，以及可寻址，即对它可随机地读/写任一块__。
b. 字符设备：因为其传输的基本单位是__字符__，所以该类设备被称为字符设备。
它属于无结构类型，如交互式终端机、打印机等。它们的基本特征是__传输速率低、不可寻址，并且在输入/输出时常釆用中断驱动方式__。

## I/O设备控制方式

> 设备管理的主要任务之一是控制设备和内存或处理机之间的数据传送。

外围设备和内存之间的输入/输出控制方式有四种，下面分别介绍：

![image](/uploads/multithreads-basic-knowledge/81.jpg)

在Java中，NIO下就支持了DMA和通道方式传输数据。

### 程序直接控制方式（主动询问）

如下图所示，计算机从外部设备读取数据到存储器，每次读一个字的数据。对读入的每个字，CPU需要对外设状态进行循环检查，直到确定该字已经在I/O控制器的数据寄存器中。在程序直接控制方式中，由于CPU的高速性和I/O设备的低速性，致使CPU的绝大部分时间都处于等待I/O设备完成数据I/O的阻塞状态中，无法再做其他的事情，造成了CPU资源的极大浪费。

在该方式中，CPU之所以要不断地测试I/O设备的状态，就是因为在CPU中没有釆用中断机构，使I/O设备无法向CPU报告它已完成了一个字符的输入操作。

程序直接控制方式虽然简单易于实现，但是其缺点也是显而易见的，由于CPU和I/O设备只能串行工作，导致CPU的利用率可能会相当低。

### 中断驱动方式

中断驱动方式的思想是，允许I/O设备主动打断CPU的运行并请求服务，从而能__部分“解放”CPU__，使得CPU在向I/O控制器发送读命令之后可以继续做其他有用的工作。

我们从I/O控制器和CPU两个角度分别来看中断驱动方式的工作过程：

1）从I/O控制器的角度来看：I/O控制器从CPU接收一个读命令，然后从外围设备读数据。一旦数据读入到该I/O控制器的数据寄存器，便通过控制线给CPU发出一个中断信号，表示数据已准备好，然后等待CPU请求该数据。I/O控制器收到CPU发出的取数据请求后，将数据放到数据总线上，传到CPU的寄存器中。至此，本次I/O操作完成，I/O控制器又可幵始下一次I/O操作。

2）从CPU的角度来看：CPU发出读命令后，保存当前运行作业的上下文（现场，如程序计数器及处理机寄存器），接着转去执行其他作业。在每个指令周期的末尾，CPU检查中断。当有来自I/O控制器的中断时，CPU保存当前正在运行作业的上下文，转去执行中断处理程序处理该中断。这时，CPU从I/O控制器读一个字的数据传送到寄存器，并存入主存，同时，CPU恢复出当时发出I/O命令对应的的上下文，将这个作业继续运行下去。

中断驱动方式比程序直接控制方式有效，但由于数据中的每个字在存储器与I/O控制器之间的传输都必须经过CPU，这就导致了中断驱动方式仍然需要CPU来处理IO请求。

### DMA方式

在中断驱动方式中，I/O设备与内存之间的数据交换必须要经过CPU中的寄存器，所以速度还是受限的，而DMA（直接存储器存取）方式的基本思想是在I/O设备和内存之间开辟直接的数据交换通路，__基本上“解放”了CPU（还是有一点占有的）__。

DMA方式的特点是：
1、基本单位是数据块。
2、所传送的数据，是从设备直接送入内存的，或者相反。

> 仅在传送一个或多个数据块的开始和结束时，才需CPU干预，接下来整块数据的传送是在__DMA控制器__的控制下完成的。

为了实现在主机与控制器之间成块数据的直接交换，必须在DMA控制器中设置如下四类寄存器：
1、命令/状态寄存器(CR)：用于接收从CPU发来的I/O命令或有关控制信息，或设备的状态。
2、内存地址寄存器(MAR)：在输入时，它存放把数据从设备传送到内存的起始目标地址；在输出时，它存放由内存到设备的内存源地址。
3、数据寄存器(DR)：用于暂存从设备到内存，或从内存到设备的数据。
4、数据计数器(DC)：存放本次CPU要读或写的字（节）数。

DMA控制器的组成：

![image](/uploads/multithreads-basic-knowledge/82.jpg)

如上图所示，DMA方式的工作过程是：
1、CPU读写数据时，它给I/O控制器发出一条命令，启动DMA控制器，然后继续其他工作。
2、之后CPU就把控制操作委托给DMA控制器，由该控制器负责处理。DMA控制器直接与存储器交互，传送整个数据块，每次传送一个字，这个过程不需要CPU参与。
3、当传送完成后，DMA控制器发送一个中断信号给CPU，告知工作完成。

> DMA模式下只有在传送开始和结束时才需要CPU的参与。

DMA控制方式与中断驱动方式的主要区别：
1、中断驱动方式在每个数据需要传输时中断CPU，而DMA控制方式则是在所要求传送的一批数据全部传送结束时才中断CPU。
2、中断驱动方式数据传送是在中断处理时由CPU控制完成的，而DMA控制方式则是在DMA控制器的控制下完成的。

### 通道控制方式

I/O通道是指专门负责输入/输出的处理机。I/O通道方式是对DMA方式的发展，它可以进一步减少CPU的干预，即把对一个数据块的读（或写）为单位的干预，减少为对一组数据块的读（或写）及有关的控制和管理为单位的干预。同时，又可以实现CPU、通道和I/O设备三者的并行操作，从而更有效地提高整个系统的资源利用率。

例如，当CPU要完成一组相关的读（或写）操作及有关控制时，只需向I/O通道发送一条I/O指令，以给出其所要执行的通道程序的首地址和要访问的I/O设备，通道接到该指令后，通过执行通道程序便可完成CPU指定的I/O任务，数据传送结束时向CPU发中断请求。

I/O通道与一般处理机的区别是：
通道指令的类型单一，没有自己的内存，通道所执行的通道程序是放在主机的内存中的，也就是说通道与CPU共享内存。

I/O通道与DMA方式的区别是：
DMA方式需要CPU来控制传输的数据块大小、传输的内存位置，而通道方式中这些信息是由通道控制的，更能解放CPU。
另外，每个DMA控制器对应一台设备与内存传递数据，而一个通道可以控制多台设备与内存的数据交换。

## I/O子系统层次结构

I/O软件涉及的面非常广，往下与硬件有着密切的联系，往上又与用户直接交互，它与进程管理、存储器管理、文件管理等都存在着一定的联系，因为后三者都需要I/O来实现一些操作。

为了使复杂的I/O软件具有清晰的结构，良好的可移植性和适应性，在I/O软件中普遍釆用了__层次式结构__，将系统输入/输出功能组织成一系列的层次，__每一层都利用其下层提供的服务，完成输入/输出功能中的某些子功能，并屏蔽这些功能实现的细节，向高层提供服务__。在层次式结构的I/O软件中，只要层次间的接口不变，对某一层次中的软件的修改都不会引起其下层或高层代码的变更，仅最底层才涉及硬件的具体特性。

一个比较合理的层次划分如下图所示：
![image](/uploads/multithreads-basic-knowledge/83.png)

整个I/O系统可以看成具有五个层次的系统结构，各层次及其功能如下：

### 用户层I/O软件

1) 用户层I/O软件：实现与用户交互的接口，用户可直接调用在用户层提供的、与I/O操作有关的库函数，对设备进行操作。
一般而言，大部分的I/O软件都在操作系统内部，但仍有一小部分在用户层，包括与用户程序链接在一起的库函数，以及完全运行于内核之外的一些程序。
用户层软件必须通过一组系统调用来获取操作系统服务。   

### 设备独立性软件

2) 设备独立性软件：用于实现用户程序与设备驱动器的统一接口、设备命令、设备保护、以友设备分配与释放等，同时为设备管理和数据传送提供必要的存储空间。
设备独立性也称设备无关性，使得应用程序独立于具体使用的物理设备。为了实现设备独立性而引入了__逻辑设备和物理设备__这两个概念。
在应用程序中，使用逻辑设备名来请求使用某类设备；在系统实际的执行过程中，将逻辑设备名映射成物理设备名使用。

这种使用逻辑设备名的好处是：
1、增加设备分配的灵活性；
2、易于实现I/O重定向，所谓I/O重定向，是指用于I/O操作的设备可以更换（即重定向），而不必改变应用程序，比如原本输入到屏幕上的文字输入到硬盘文件中去。

为了实现设备独立性，必须在驱动程序之上设置一层__设备独立性软件__。

总的来说，设备独立性软件的主要功能可分以为以下两个方面：
1、执行所有设备的公共操作。
包括：对设备的分配与回收；将逻辑设备名映射为物理设备名；对设备进行保护，禁止用户直接访问设备；缓冲管理；差错控制；提供独立于设备的大小统一的逻辑块，屏蔽设备之间信息交换单位大小和传输速率的差异。
2、向用户层（或文件层）提供统一接口。
无论何种设备，它们向用户所提供的接口应该是相同的。例如，对各种设备的读/写操作，在应用程序中都统一使用read/write命令等。

### 设备驱动程序

3) 设备驱动程序：与硬件直接相关，负责具体实现系统对设备发出的操作指令，驱动I/O设备工作的驱动程序。

通常，每一类设备配置一个设备驱动程序，它是I/O进程与设备控制器之间的通信程序，常以进程形式存在。
一方面设备驱动程序向上层用户程序提供一组标准接口，设备具体的差别被设备驱动程序所封装，用于接收上层软件发来的抽象I/O要求，如read和write命令，转换为具体要求后，发送给设备控制器，控制I/O设备工作。
另一方面它也将由设备控制器发来的信号传送给上层软件。从而为I/O内核子系统隐藏设备控制器之间的差异。

### 中断处理程序

4)中断处理程序：用于保存被中断进程的CPU环境，转入相应的中断处理程序进行处理，处理完后恢复被中断进程的现场后，返回到被中断进程。

中断处理层的主要任务有：进行进程上下文的切换，对处理中断信号源进行测试，读取设备状态和修改进程状态等。由于中断处理与硬件紧密相关，对用户而言，应尽量加以屏蔽，__故应放在操作系统的底层，系统的其余部分尽可能少地与之发生联系。__

### 硬件设备

5) 硬件设备：I/O设备通常包括一个机械部件和一个电子部件。为了达到设计的模块性和通用性，一般将其分开：电子部件称为设备控制器（或适配器），在个人计算机中，通常是一块插入主板扩充槽的印刷电路板（被称为主板），机械部件则是设备本身（使用PCI插槽、USB接口等接入）。

__设备控制器__通过寄存器与CPU通信，在某些计算机上，这些寄存器占用内存地址的一部分，称为内存映像I/O；另一些计算机则釆用I/O专用地址，寄存器独立编址。操作系统通过向控制器寄存器写命令字来执行I/O功能。控制器收到一条命令后，CPU可以转向进行其他工作，而让设备控制器自行完成具体的I/O操作。当命令执行完毕后，控制器发出一个中断信号，操作系统重新获得CPU的控制权并检查执行结果，此时，CPU仍旧是从控制器寄存器中读取信息来获得执行结果和设备的状态信息。

设备控制器的主要功能为：
1、接收和识别CPU或通道发来的命令，如磁盘控制器能接收读、写、查找等命令。
2、实现数据交换，包括设备和控制器之间的数据传输；通过数据总线或通道，控制器和主存之间的数据传输。
3、发现和记录设备及自身的状态信息，供CPU处理使用。
4、设备地址识别。

为实现上述功能，设备控制器必须包含以下组成部分：
1、设备控制器与CPU的接口。该接口有三类信号线：数据线、地址线和控制线。数据线通常与两类寄存器相连接：数据寄存器（存放从设备送来的输入数据或从CPU送来的输出数据）和控制/状态寄存器（存放从CPU送来的控制信息或设备的状态信息)。
2、设备控制器与设备的接口。设备控制器连接设备需要相应数量的接口，一个接口连接一台设备。每个接口中都存在数据、控制和状态三种类型的信号。
3、I/O控制逻辑。用于实现对设备的控制。它通过一组控制线与CPU交互，对从CPU收到的I/O命令进行译码。CPU启动设备时，将启动命令发送给控制器，同时通过地址线把地址发送给控制器，由控制器的I/O逻辑对地址进行译码，并相应地对所选设备进行控制。

![image](/uploads/multithreads-basic-knowledge/84.jpg)

## I/O子系统概述
由于I/O设备种类繁多，功能和传输速率差异巨大，需要多种方法来进行设备控制。这些方法共同组成了操作系统内核的I/O子系统，它将内核的其他方面从繁重的I/O设备管理中解放出来。I/O核心子系统提供的服务主要有I/O调度、缓冲与高速缓存、设备分配与回收、假脱机、设备保护和差错处理等。

## I/O调度概念
I/O调度就是按照确定好的顺序来执行这些I/O请求。应用程序所发布的系统调用的顺序不一定总是最佳选择，所以需要I/O调度来改善系统整体性能，使进程之间公平地共享设备访问，减少I/O完成所需要的平均等待时间。

操作系统开发人员通过为每个设备维护一个请求队列来实现调度。当一个应用程序执行阻塞I/O系统调用时，该请求就加到相应设备的队列上，I/O调度会__重新安排队列顺序__以改善系统总体效率和应用程序的平均响应时间。

I/O子系统还可以使用主存或磁盘上的存储空间的技术，如缓冲、高速缓冲、假脱机等，来改善计算机效率。

## 高速缓存与缓冲区

利用缓存技术来提高I/O速度。

### 磁盘高速缓存(Disk Cache)

操作系统中使用磁盘高速缓存技术来提高磁盘的I/O速度，对高速缓存复制的访问要比原始数据访问更为高效。
例如，正在运行的进程的指令既存储在磁盘上，也存储在物理内存上，也被复制到CPU的二级和一级高速缓存中。

不过，磁盘高速缓存技术不同于通常意义下的介于CPU与内存之间的小容量高速存储器，而是指利用内存中的存储空间来暂存从磁盘中读出的一系列盘块中的信息。

> 磁盘高速缓存在逻辑上是属于磁盘的，但是在物理上却是驻留在内存中的数据。

高速缓存在内存中分为两种形式：
1、一种是在内存中开辟一个单独的存储空间作为磁速缓存，大小固定；
2、另一种是把未利用的内存空间作为一个缓沖池，供请求分页系统和磁盘I/O时共享。

### CPU缓冲区(Buffer)

在设备管理子系统中，引入缓冲区的目的主要有：
1、缓和CPU与I/O设备间速度不匹配的矛盾。
2、减少对CPU的中断频率，放宽对CPU中断响应时间的限制。
3、解决基本数据单元大小（即数据粒度）不匹配的问题。
4、提高CPU和I/O设备之间的并行性。

其实现方法有：
1、釆用硬件缓冲器，但由于成本高所以除一些关键部位外，一般不釆用硬件缓冲器。
2、釆用缓冲区（位于内存区域）。

根据系统设置缓冲器的个数，缓冲技术可以分为：

#### 单缓冲

在设备和处理机之间设置一个缓冲区。设备和处理机交换数据时，先把被交换数据写入缓冲区，然后需要数据的设备或处理机从缓冲区取走数据。

如下图所示，在块设备输入时，假定从磁盘把一块数据输入到缓冲区的时间为T，操作系统将该缓冲区中的数据传送到用户区的时间为M，而CPU对这一块数据处理的时间为C。
由于T和C是可以并行的，当T>C时，系统对每一块数据的处理时间为M+T，反之则为M+C，故可把系统对每一块数据的处理时间表示为Max{C,T}+M。

![image](/uploads/multithreads-basic-knowledge/85.jpg)

#### 双缓冲

根据单缓冲的特点，CPU在传送时间M内处于空闲状态，由此引入双缓冲。I/O设备输入数据时先装填到缓冲区1，在缓冲区1填满后才开始装填缓冲区2，与此同时处理机可以从缓冲区1中取出数据放入用户进程处理，当缓冲区1中的数据处理完后，若缓冲区2已填满，则处理机又从缓冲区2中取出数据放入用户进程处理，而I/O设备又可以装填缓冲区1。

双缓冲机制提高了处理机和输入设备的并行操作的程度。

如下图所示，系统处理一块数据的时间可以粗略地认为是Max{C,T}。如果C<T，可使块设备连续输入（上图中所示情况)；如果C>T，则可使CPU不必等待设备输入。

对于字符设备，若釆用行输入方式，则釆用双缓冲可使用户在输入完第一行之后，在CPU执行第一行中的命令的同时，用户可继续向第二缓冲区输入下一行数据。而单缓冲情况下则必须等待一行数据被提取完毕才可输入下一行的数据。

![image](/uploads/multithreads-basic-knowledge/86.jpg)

如果两台机器之间通信仅配置了单缓冲，如下图(a)所示。那么，它们在任一时刻都只能实现单方向的数据传输。例如，只允许把数据从A机传送到B机，或者从B机传送到A机，而绝不允许双方同时向对方发送数据。为了实现双向数据传输，必须在两台机器中都设置两个缓冲区，一个用做发送缓冲区，另一个用做接收缓冲区，如下图(b)所示。

![image](/uploads/multithreads-basic-knowledge/87.jpg)

#### 循环缓冲

包含多个大小相等的缓冲区，每个缓冲区中有一个链接指针指向下一个缓冲区，最后一个缓冲区指针指向第一个缓冲区，多个缓冲区构成一个环形。

循环缓冲用于输入/输出时，还需要有两个指针in和out。对输入而言，首先要从设备接收数据到缓冲区中，in指针指向可以输入数据的第一个空缓冲区；当运行进程需要数据时，从循环缓冲区中取一个装满数据的缓冲区，并从此缓冲区中提取数据，out指针指向可以提取数据的第一个满缓冲区。输出则正好相反。

#### 缓冲池

由多个系统公用的缓冲区组成，缓冲区按其使用状况可以形成三个队列：空缓冲队列、装满输入数据的缓冲队列（输入队列）和装满输出数据的缓沖队列（输出队列）。
还应具有四种缓冲区：用于收容输入数据的工作缓冲区、用于提取输入数据的工作缓冲区、用于收容输出数据的工作缓冲区及用于提取输出数据的工作缓冲区，如下图所示。

![image](/uploads/multithreads-basic-knowledge/88.jpg)

当输入进程需要输入数据时，便从空缓冲队列的队首摘下一个空缓冲区，把它作为收容输入工作缓冲区，然后把输入数据输入其中，装满后再将它挂到输入队列队尾。当计算进程需要输入数据时，便从输入队列取得一个缓冲区作为提取输入工作缓冲区，计算进程从中提取数据，数据用完后再将它挂到空缓冲队列尾。当计算进程需要输出数据时，便从空缓冲队列的队首取得一个空缓冲区，作为收容输出工作缓冲区，当其中装满输出数据后，再将它挂到输出队列队尾。当要输出时，由输出进程从输出队列中取得一个装满输出数据的缓冲区，作为提取输出工作缓冲区，当数据提取完后，再将它挂到空缓冲队列的队尾。

### 高速缓存与缓冲区的比较

高速缓存是可以保存数据拷贝的高速存储器，访问高速缓存比访问原始数据更高效速度更快。其对比见下表。

![image](/uploads/multithreads-basic-knowledge/89.png)

## 设备分配与回收

### 设备分配概述

设备分配是指根据用户的I/O请求分配所需的设备。分配的总原则是__充分发挥设备的使用效率，尽可能地让设备忙碌，又要避免由于不合理的分配方法造成进程死锁__。

从设备的特性来看，釆用下述三种使用方式的设备分别称为__独占设备、共享设备和虚拟设备__三类。

1) 独占式使用设备。指在申请设备时，如果设备空闲，就将其独占，不再允许其他进程申请使用，一直等到该设备被释放才允许其他进程申请使用。例如，打印机，在使用它打印时，__只能独占式使用__，否则在同一张纸上交替打印不同任务的内容，无法正常阅读。

2) 分时式共享使用设备。独占式使用设备时，设备利用率很低，当设备没有独占使用的要求时，可以通过分时共享使用，提高利用率。例如，对磁盘设备的I/O操作，各进程的每次I/O操作请求可以__通过分时来交替进行__。

3) 以SPOOLing方式使用外部设备。SPOOLing技术是在批处理操作系统时代引入的，即假脱机I/O技术。这种技术用于对设备的操作，实质上就是对I/O操作进行__批处理__。

### 设备分配的数据结构

设备分配依据的主要数据结构有设备控制表(DCT)、控制器控制表(COCT)、通道控制表(CHCT)和系统设备表(SDT)。

各数据结构功能如下：
设备控制表DCT：系统为每一个设备配置一张DCT，如下图所示。
它用于记录设备的特性以及与I/O控制器连接的情况。DCT包括设备标识符、设备类型、设备状态、指向控制器控制表COCT的指针等。其中，设备状态指示设备是忙还是空闲，设备队列指针指向等待使用该设备的进程组成的等待队列，控制表指针指向与该设备相连接的设备控制器。

设备控制表：
![image](/uploads/multithreads-basic-knowledge/90.jpg)

控制器控制表COCT：每个控制器都配有一张COCT，如图5-10a所示。它反映设备控制器的使用状态以及和通道的连接情况等。
通道控制表CHCT：每个通道配有一张CHCT，如图5-10b所示。
系统设备表SDT：整个系统只有一张SDT，如图5-10c所示。它记录已连接到系统中的所有物理设备的情况，每个物理设备占一个表目。

COCT、CHCT和SDT：
![image](/uploads/multithreads-basic-knowledge/91.jpg)

由于在多道程序系统中，__进程数一般多于资源数，会引起资源的竞争__。因此，要有一套合理的分配原则，主要考虑的因素有：I/O设备的固有属性，I/O设备的分配算法，设备分配的安全性以及设备独立性。

### 设备分配的策略

1) 设备分配原则：设备分配应根据设备特性、用户要求和系统配置情况。分配的总原则既要充分发挥设备的使用效率，又要避免造成进程死锁，还要将用户程序和具体设备隔离开。

2) 设备分配方式：设备分配方式有静态分配和动态分配两种。

静态分配主要用于对独占设备的分配，它在用户作业开始执行前，由系统一次性分配该作业所要求的全部设备、控制器（和通道)。一旦分配后，这些设备、控制器（和通道）就一直为该作业所占用，直到该作业被撤销。静态分配方式不会出现死锁，但设备的使用效率低。因此，静态分配方式弁不符合分配的总原则。

动态分配是在进程执行过程中根据执行需要进行。当进程需要设备时，通过系统调用命令向系统提出设备请求，由系统按照事先规定的策略给进程分配所需要的设备、I/O控制器，一旦用完之后，便立即释放。动态分配方式有利于提高设备的利用率，但如果分配算法使用不当，则有可能造成进程死锁。

3) 设备分配算法：常用的动态设备分配算法有__先请求先分配、优先级高者优先__等。

对于独占设备，既可以釆用动态分配方式也可以静态分配方式，往往釆用静态分配方式，即在作业执行前，将作业所要用的这一类设备分配给它。共享设备可被多个进程所共享，一般釆用动态分配方式，但在每个I/O传输的单位时间内只被一个进程所占有，通常釆用先请求先分配和优先级高者先分的分配算法。

### 设备分配的安全性

设备分配的安全性是指设备分配中应防止发生进程死锁。

1) 安全分配方式：每当进程发出I/O请求后便进入阻塞状态，直到其I/O操作完成时才被唤醒。这样，一旦进程已经获得某种设备后便阻塞，不能再请求任何资源，而且在它阻塞时也不保持任何资源。i点是设备分配安全；缺点是CPU和I/O设备是串行工作的（对同一进程而言)。

2) 不安全分配方式：进程在发出I/O请求后继续运行，需要时又发出第二个、第三个I/O请求等。仅当进程所请求的设备已被另一进程占用时，才进入阻塞状态。优点是一个进程可同时操作多个设备，从而使进程推进迅速；缺点是这种设备分配有可能产生死锁。

### 逻辑设备名到物理设备名的映射

为了提高设备分配的灵活性和设备的利用率、方便实现I/O重定向，因此引入了__设备独立性__。

> 设备独立性是指应用程序独立于具体使用的物理设备。

为了实现设备独立性，在应用程序中使用逻辑设备名来请求使用某类设备，在系统中设置一张逻辑设备表(Logical Unit Table, LUT)，用于将逻辑设备名映射为物理设备名。LUT 表项包括逻辑设备名、物理设备名和设备驱动程序入口地址；当进程用逻辑设备名来请求分配设备时，系统为它分配相应的物理设备，并在LUT中建立一个表项，以后进程再利用逻辑设备名请求I/0操作时，系统通过查找LUT来寻找相应的物理设备和驱动程序。

在系统中可釆取两种方式建立逻辑设备表：
在整个系统中只设置一张LUT。这样，所有进程的设备分配情况都记录在这张表中，故不允许有相同的逻辑设备名，主要适用于单用户系统中。
为每个用户设置一张LUT。当用户登录时，系统便为该用户建立一个进程，同时也为之建立一张LUT，并将该表放入进程的PCB中。

## SPOOLing技术(假脱机技术)

> 为了缓和CPU的高速性与I/O设备低速性之间的矛盾而引入了脱机输入/输出技术。

该技术是利用专门的外围控制机，将低速I/O设备上的数据传送到高速磁盘上；或者相反。 

SPOOLing的意思是外部设备同时联机操作，又称为假脱机输入/输出操作，是操作系统中釆用的一项__将独占设备改造成共享设备的技术__。

SPOOLing系统组成如下图所示：
![image](/uploads/multithreads-basic-knowledge/92.jpg)

### 输入井和输出井

输入井和输出井是在__磁盘__上开辟出的两个存储区域。

输入井模拟脱机输入时的磁盘，用于收容I/O设备输入的数据。
输出井模拟脱机输出时的磁盘，用于收容用户程序的输出数据。

### 输入缓冲区和输出缓冲区

输入缓冲区和输出缓冲区是在__内存__中开辟的两个缓冲区。

输入缓冲区用于暂存由输入设备送来的数据，以后再传送到输入井。
输出缓冲区用于暂存从输出井送来的数据，以后再传送到输出设备。

### 输入进程和输出进程

输入进程模拟脱机输入时的外围控制机，将用户要求的数据从输入机通过输入缓冲区再送到输入井。当CPU需要输入数据时，直接将数据从输入井读入内存。
输出进程模拟脱机输出时的外围控制机，把用户要求输出的数据先从内存送到输出并，待输出设备空闲时，再将输出井中的数据经过输出缓冲区送到输出设备。

共享打印机是使用SPOOLing技术的一个实例，这项技术已被广泛地用于多用户系统和局域网络中。
当用户进程请求打印输出时，SPOOLing系统同意为它打印输出，但并不真正立即把打印机分配给该用户进程，而只为它做两件事：
1、由输出进程在输出井中为之申请一个空闲磁盘块区，并将要打印的数据送入其中。
2、输出进程再为用户进程申请一张空白的用户请求打印表，并将用户的打印要求填入其中，再将该表挂到请求打印队列上。

SPOOLing系统的主要特点有：__提高了I/O的速度；将独占设备改造为共享设备；实现了虚拟设备功能__。

## 总结思考

1) 分配设备。首先根据I/O请求中的物理设备名查找系统设备表（SDT)，从中找出该设备的DCT，再根据DCT中的设备状态字段，可知该设备是否正忙。
若忙，便将请求I/O进程的PCB挂在设备队列上；空闲则按照一定算法计算设备分配的安全性，安全则将设备分配给请求进程，否则仍将其PCB挂到设备队列。

2) 分配控制器。系统把设备分配给请求I/O的进程后，再到其DCT中找出与该设备连接的控制器的COCT，从COCT中的状态字段中可知该控制器是否忙碌。
若忙，便将请求I/O进程的PCB挂在该控制器的等待队列上；空闲便将控制器分配给进程。

3) 分配通道。在该COCT中又可找到与该控制器连接的通道的CHCT，再根据CHCT内的状态信息，可知该通道是否忙碌。
若忙，便将请求I/O的进程挂在该通道的等待队列上；空闲便将该通道分配给进程。

只有在上述三者（设备、控制器、通道）都分配成功时，这次设备的分配才算成功，然后便可启动该I/O设备进行数据传送。

为使独占设备的分配具有更强的灵活性，提高分配的成功率，还可以从以下两方面对基本的设备分配程序加以改进：

1、增加设备的独立性。
进程使用逻辑设备名请求I/O。这样，系统首先从SDT中找出第一个该类设备的DCT。若该设备忙，又查找第二个该类设备的DCT。仅当所有该类设备都忙时，才把进程挂在该类设备的等待队列上；只要有一个该类设备可用，系统便进一步计算分配该设备的安全性。

2、考虑多通路情况。
为防止I/O系统的“瓶颈”现象，通常釆用多通路的I/O系统结构。此时对控制器和通道的分配同样要经过几次反复，即若设备（控制器）所连接的第一个控制器（通道）忙时，应查看其所连接的第二个控制器（通道)，仅当所有的控制器（通道）都忙时，此次的控制器（通道）分配才算失败，才把进程挂在控制器（通道）的等待队列上。而只要有一个控制器（通道）可用，系统便可将它分配给进程。