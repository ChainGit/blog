---
title: 树莓派3搭建cups或p910nd打印服务器
date: 2018/01/20
categories:
- 其他
tags:
- 其他
- Linux
---

树莓派3搭建cups或p910nd打印服务器
==============
放假前实验室搭建出一套在线的浏览器WEB打印平台，共享打印机使用。放假回家记录一下踩到的坑，方便学习。

## 系统概述

[某宝类似产品](https://item.taobao.com/item.htm?spm=a230r.1.14.20.7535cd0c4xb2xz&id=37311711672&ns=1&abbucket=10#detail)

上面的某宝上有专门的厂家卖打印服务器，我这里使用树莓派搭建一个类似的平台。

![打印示意图](/uploads/0-common-images/19.gif)

大致的原理如上图。其他计算机可以通过这个打印服务器将需要打印的文件打印到打印机上。

实现的方式大致有两种：
> 一种是通过CUPS方法
> 一种是通过p910d方式

当然还有蓝牙等其他方式，我没有做研究。

## cups方式

首先将树莓派刷入纯净的Linux操作系统，比如2017-11-29-raspbian-stretch-lite.img，下载地址可以在[raspberrypi官网](https://www.raspberrypi.org/downloads/raspbian/)找到。

刷入系统的方法大致是先清除SD卡上的数据和分区，然后写入img镜像，插入树莓派后通电启动，则默认会加载内存卡中的系统。树莓派可以理解为一个裸机，系统和数据均是在内存卡中存储的，只需要更换内存卡即可方便的更换操作系统。

清除分区可以使用SDFormatter或者Windows命令diskpart，写入镜像可以使用Win32DiskImager。具体教程参考[树莓派入门之装系统](http://blog.csdn.net/u011388550/article/details/49981703)。

初次启动树莓派有很多方式，我的方式是通电前，插入网线连接到家用路由器上，然后再使用HDMI线连接到显示器，插入USB鼠标和键盘。通电后，可以在路由器配置界面(通常是192.168.1.1)查看到树莓派的IP。路由器可以设置IP和MAC绑定，防止树莓派的IP有变化。也可以设置好新的IP和MAC绑定项，比如192.168.1.188，然后关闭树莓派，重启路由器，再打开树莓派。这样树莓派就可以自动获得指定的IP。

树莓派启动后，还不能直接SSH连接，需要使用键盘和显示器的情况下预先配置一下。如果显示器没有输出图像的话，那么最简单的方式就是重新刷写SD卡，先确保HDMI线接入到通电状态下的显示器上，然后再启动树莓派。

使用如下命令可以配置树莓派的一些预先设置：
```bash
sudo raspi-config
```
在设置中可以开启SSH。然后重启树莓派，使用winscp+putty就可以登陆树莓派，此时键盘和显示器也可以撤了。

接着可以先安装一些常用的软件：
```bash
sudo apt-get update
sudo apt-get -f install vim tree openssh-server
```

接着就万事俱备，可以安装cups打印服务了。

先输入下面的命令安装hplip：
```bash
sudo apt-get -f install hplip
```
安装好hplip后通常cups也会一并安装好。

将当前树莓派用户pi加入到lp管理员组，用于之后的web配置，也可以单独新建一个用户加入。
```bash
sudo usermod -a -G lpadmin pi
```

备份配置文件（这是一个好习惯）：
```bash
sudo mv /etc/cups/cupsd.conf /etc/cups/cupsd.conf.bak
```

然后为了便于cups配置界面在其他电脑上也能直接访问，需要修改配置文件：
```bash
sudo vim /etc/cups/cupsd.conf
```
将
```
Listen localhost:631  
```
改成
```
Listen 0.0.0.0:631  
```
再将
```
<Location /> 
<Location /admin> 
<Location /admin/conf>
```
三个节点均添加
```
Allow from @Local  
```

保存后重启cups服务或重启树莓派，连接上打印机的USB口。

重启后在浏览器中输入：

> http://（树莓派IP，比如192.168.1.188）:631

即可打开cups的配置界面。

点击Add printer可以添加打印机，添加打印机需要https访问，并输入之前设置的管理用户和密码（比如pi，密码raspberry）。

注意在windows中添加打印机时需要将https改成http。

具体操作步骤参考[将树莓派变成网络打印机服务器](http://ju.outofmemory.cn/entry/21571)。

可以安装samba让windows能直接在网络邻居中看到这台打印服务器，并共享打印机的驱动文件。参考[CUPS&samba](http://www.natbbs.com/thread-9178-1-1.html)。

cups需要打印机在linux下的驱动才能完美支持，但是很多打印机在linux下没有驱动（比如联想），这就很麻烦，所以p910nd方式兼容性会好些。

联想激光和喷墨打印机可以参考brother打印机，在linux中，使用lsusb可以看出来，联想的打印机显示的是brother公司，brother打印机在cups的驱动列表中可以找到部分驱动。如果不确定，可以比较生产日期和外观，确定具体的型号，但是驱动也并不能确保完美支持。惠普打印机在linux驱动方面做的很好，基本都能找到对应的驱动，hplip也是惠普主导的，这值得点赞。

如果实在没有驱动，那么可以使用驱动列表中的RAW，再选择queue即可，在windows中安装对应的打印机驱动也能支持。
有些windows打印驱动下有ppd文件，如果有那么可以通过上传ppd的方式添加打印机。

针式打印机很少有linux下的驱动，cups下无法正常使用（自己尝试时，偶然有一次打印出了Print Test Page部分信息，但是之后再也没有进展）。

## p910nd方式

很多智能路由器装有基于openwrt的操作系统，这也是一个基于linux的操作系统，曾经在学校用过的天翼客户端破解路由器也是基于openwrt。

树莓派作为一个强大的开源设备（确实牛的不行），也支持openwrt系统，不过是其一个分支lede。某宝上卖的那款打印服务器推测也基本上基于这个系统实现的。

p910nd方式是基于TCP/IP方式连接打印机，传输协议采用RAW，服务器只是做打印数据的转发，并不需要像cups那样需要打印机的驱动，因此兼容性更好，也能支持部分老式的针式打印机。

下面就来折腾p910nd方式打印机服务器吧。

将树莓派的内存卡取出，使用另一张新的sd卡，这样可以实现方便的更换系统。

我所使用的是：树莓派3代B型，1.2GHz，四核64位ARMv8 CortexA53 CPU（bcm2837）。
在lede官网上下载对应CPU的img系统镜像：
```
http://downloads.lede-project.org/releases/17.01.4/targets/brcm2708/bcm2710/
```
如果镜像下载的不对，是不能启动的，或者支持的有问题。树莓派3就下载bcm2708下的bcm2710即可。

我下载的镜像文件是：lede-17.01.4-brcm2708-bcm2710-rpi-3-ext4-sdcard.img.gz

官网会不断的更新，需要自己选择最新和最稳定的版本，[版本选择](http://downloads.lede-project.org/releases)。

下载后，按照cups写的安装系统的方式，将img写入内存卡。

不一样的是，树莓派的lede启动后默认是当路由器使用，而我们需要作为打印服务器。

电脑网口与树莓派直连，然后在浏览器中输入192.168.1.1，即可访问lede配置界面。

![lede配置界面](/uploads/0-common-images/20.jpg)

初始账号root，密码为空，首次使用需要修改。并开启SSH便于远程访问。

登录后修改Network中的Interfaces，修改LAN口的配置，将IP改为其他，比如192.168.1.188（需要在路由器的同一网段下），修改网关为192.168.1.1（路由器网关），并设置DNS服务器（因地区而异，可以在家用路由器主界面中看到），可以关闭树莓派自己的DHCP服务器，也可以不关。应用并保存（Save&Apply）后关闭树莓派。

将树莓派插入到家用路由器上（无需清除之前设置的IP和MAC绑定，因为树莓派硬件没有变化，MAC地址也没有变化，IP和MAC绑定仍然有效），接通树莓派的电源。

在浏览器中输入192.168.1.188，打开lede配置界面，进入System的Software下，点击update list，更新软件列表。内部的软件源是预先设置好的，主要如下：
```
src/gz reboot_core http://downloads.lede-project.org/releases/17.01.4/targets/brcm2708/bcm2710/packages
src/gz reboot_base http://downloads.lede-project.org/releases/17.01.4/packages/arm_cortex-a53_neon-vfpv4/base
src/gz reboot_luci http://downloads.lede-project.org/releases/17.01.4/packages/arm_cortex-a53_neon-vfpv4/luci
src/gz reboot_packages http://downloads.lede-project.org/releases/17.01.4/packages/arm_cortex-a53_neon-vfpv4/packages
src/gz reboot_routing http://downloads.lede-project.org/releases/17.01.4/packages/arm_cortex-a53_neon-vfpv4/routing
src/gz reboot_telephony http://downloads.lede-project.org/releases/17.01.4/packages/arm_cortex-a53_neon-vfpv4/telephony
```
可以Configuration下修改。注意源需要和树莓派系统版本对应，否则容易出现valid architectance提示。
如果更新列表失败，则很可能是网络不通，需要重新检查LAN口配置。

在Available Packages下__依次__搜索安装：
> kmod-lp、kmod-usb-printer、usbutils、p910nd、luci-app-p910nd

也可以在命令行下使用：
```bash
opkg update
opkg install kmod-lp kmod-usb-printer usbutils p910nd luci-app-p910nd
```

kmod-lp用于并行口的打印机，kmod-usb-printer用于USB打印机。安装usbutils以支持lsusb命令。p910nd为服务器，luci-app-p910nd为luci界面下的图形配置界面。

安装好后重启树莓派，插入打印机的USB口。重启后进入Services下新出现的p910nd - Printer server。

使用SSH连接到树莓派，输入lsusb，查看是否有打印机连接，再进入/etc/usb下，查看是否lp0字样。

可以使用echo "XXX" > /etc/usb/lp0，测试打印机是否打印。

配置界面如下：

![打印机配置界面](/uploads/0-common-images/21.jpg)

之后就可以在电脑上连接打印机了，具体连接步骤参考[网络打印机TCP/IP共享设置教程](http://blog.csdn.net/guanfangtao2/article/details/47190505)。

如果是针式打印机使用Serial-USB转换器，基于PL2305芯片，插入到USB口，即可在/etc/serial中看到打印机的id或path，复制路径到Device栏中即可使用。但是并不是所用的针式打印机都能支持的。可以使用Add，分配不同的端口，比如9101，可以实现支持多个打印机。

## 在线打印

（这是额外的内容）

实验室还搭建了一个浏览器上传文件打印方式的WEB服务。WEB服务主要是采用利用Linux的lpr命令和libreoffice软件实现。

lpr是linux的命令行打印命令，具体的学习资料[链接](http://man.linuxde.net/sub/%E6%89%93%E5%8D%B0)。
![打印机配置界面](/uploads/0-common-images/22.jpg)
部分命令需要root权限，如果不考虑安全性的话，可以将web服务以root账号运行。

libreoffice是linux下的一个开源软件，能支持命令行文件格式转换，比如doc->pdf，但是支持的不是很好。
比如常用的一个使用案例如下：
```bash
libreoffice --headless --convert-to pdf --outdir xxx.pdf xxx.doc
```

web服务后台服务采用java实现，调用系统命令可以使用：
```java
private String exec(String e, boolean r) {
    try {
        logger.info("exec: " + e);
        Process process = Runtime.getRuntime().exec(e);
        process.waitFor();
        if (!r)
            return null;
        InputStream is = process.getInputStream();
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        byte[] bytes = new byte[1024 * 8];
        int len = -1;
        while ((len = is.read(bytes)) != -1) {
            baos.write(bytes, 0, len);
            baos.flush();
        }
        return baos.toString();
    } catch (IOException e1) {
        logger.error("printer", e1);
    } catch (InterruptedException e1) {
        logger.error("printer", e1);
    }
    return null;
}
```

前端静态界面采用bootstrap和fileinput组件。

大致流程图如下：[png](/uploads/0-common-images/24.png)
![流程图](/uploads/0-common-images/23.svg)

由于打印一般是在client端（即要打印的本地客户端）进行转换（这也是为什么要驱动的原因），转换后再将数据流发送给打印机打印，所以linux下这种在线打印平台经过测试，对pdf支持的较好，对其他格式支持的不佳（尤其是doc，会出现乱版）。

市面上也有专门的公司制作云打印平台，支持的最好的方式个人猜测还是windows下底层编程比较好，而不是像我实现的这种利用系统命令的方式。

ubuntu中添加打印机最好是在desktop下进行，命令行下比较麻烦。

## 思考总结

p910nd没有cups稳定，但是它的兼容性更好。如果打印机是hp的，那么推荐使用cups，如果需要更好的兼容设备，那么选择p910nd。

某宝上卖的打印服务器可能还有直接的底层打印指令程序编写，支持上会更好。

教程写的不是很详细，只是记录了关键步骤，需要帮助的可以留言。


