---
title: 在Ubuntu服务器中安装和配置VirtualBox
date: 2016/07/03
categories: 
- 运维
tags:
- Linux
- Ubuntu
- 服务器 
- VirtualBox
- 运维 
---

在Ubuntu服务器中安装和配置VirtualBox
===================

实验室购置了两台服务器，原本是将所有的业务都装在了物理机上，也供其他的成员学习使用。时间长了后发现，这样的“一篮子鸡蛋”做法不可取，存在安全问题。
为此学习了服务器安装虚拟机的方法，这之中也走了不少弯路。最终，实现将业务分离在不同的虚拟机上，且当有成员需要服务器进行学习时，也能提供独立的平台和故障恢复功能，管理起来很方便。
> 配置环境和软件：
Ubuntu 14.04 64位
Oracle VirtualBox 5.x

------------

## 安装和配置过程
### 准备环境
#### 安装Ubuntu
安装Ubuntu服务器版，这里不做阐述。实验室服务器安装的时Ubuntu 14.04 LTS 64位版本，且服务器需要支持虚拟化（一般支持）。
#### 安装VirtualBox
安装Linux版的VirtualBox，并做简单的配置
进入[Oracle VirtualBox](https://www.virtualbox.org/)官网，下载与系统对应的[Linux版本](https://www.virtualbox.org/wiki/Linux_Downloads)的VirtualBox。我这里时Ubuntu 14.04，因此下载__Ubuntu 14.04 ("Trusty") / 14.10 ("Utopic") / 15.04 ("Vivid")__中AMD64版本，下载好deb包即可。
当然也可以使用apt安装方式，官网有详细说明。

> 安装VirtualBox：
> &gt; sudo apt-get update
> &gt; sudo dpkg -i virtualbox-5.xxx-Ubuntu-trusty_amd64.deb
> 如果安装出现错误
> &gt; sudo at-get -f install
> 再执行dpkg即可

另外需要下载好 VirtualBox 5.xxx Oracle VM VirtualBox Extension Pack 包。建议也下载好官网上的帮助手册（[下载](/uploads/use-vbox-on-linux/vboxmanage.txt)）。
#### 设置VirtualBox
因为服务器是纯命令行界面，不能像桌面VirtualBox那样方便鼠标点击，所以设置均需要使用命令，好在VBox的命令也很强大。

1、安装一同下载好的VIrtualBox的扩展包。

> &gt; vboxmanage extpack install Oracle_VM_VirtualBox_Extension_Pack-5.xxx.vbox-extpack

2、可以修改虚拟机的安装路径。
> &gt; vboxmanage setproperty machinefolder "存放虚拟机安装位置的目录"

3、设置VRDE auth library。
虚拟机安装系统时，可以对其进行远程登陆显示，就好像安装了一个远程显示器一样，方便操作。这里建议在局域网中安装虚拟机系统，因为VRDE并不是很安全，且安装好后也需要关闭对应的虚拟机的VRDE的功能。
> &gt; vboxmanage setproperty vrdeauthlibrary "VBoxAuthSimple"

4、 设置VBox开启自启动
这样可以使得Ubuntu物理机服务器启动时，虚拟机也能定时启动。
> &gt; vboxmanage setproperty autostartdbpath /etc/vbox

### 使用VirtualBox
#### 在VirtualBox中安装Ubuntu Server
注：安装部分 [参考1](http://www.linuxidc.com/Linux/2016-04/129728.htm) [参考2](http://beiersi.iteye.com/blog/1266645) [参考3](http://www.sijitao.net/1712.html )

1、进入到VirtualBox的虚拟机安装目录。
默认是__/home/用户名__下的VirtualBox VMs 目录，如果更改了VirtualBox的设置，则进入设置中的Default machine folder配置的虚拟机安装目录。
> 确保安装目录的可读写权限。

2、新建目录，设置为安装系统的存储路径，比如ubuntu-vm-server-01
> &gt; mkdir ubuntu-vm-server-01
> &gt; cd ubuntu-vm-server-01

3、新建一个大小为100G的vdi磁盘（--size 单位 M）。
默认使用动态增加的方式，简单讲就是虚拟机系统用多少就占多少实际物理硬盘空间（只可增不会减），而虚拟机系统会认为磁盘大小为100G。
> &gt; vboxmanage createmedium disk --filename ubuntu-vm-server-01.vdi --size 102400

4、新建vbox虚拟机文件并注册（--ostype 设置系统格式）。
可以使用__vboxmanage list ostypes__查看vbox支持的系统格式。
> &gt; vboxmanage createvm --name ubuntu-vm-server-01.vdi--ostype "Ubuntu_64" --register

5、新建SATA磁盘控制器并将3中新建的vdi磁盘绑定到该虚拟机。
> &gt; vboxmanage storagectl ubuntu-vm-server-01--name "SATA Controller" --add sata --controller IntelAHCI
> &gt; vboxmanage storageattach ubuntu-vm-server-01 --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium ubuntu-vm-server-01.vdi

6、新建IDE控制器，设置它为dvd，并绑定系统ios文件到该dvd。
注：--medium为你的iso路径。
> &gt; vboxmanage storagectl ubuntu-vm-server-01--name "IDE Controller" --add ide
> &gt; vboxmanage storageattach ubuntu-vm-server-01--storagectl "IDE Controller" --port 0 --device 0 --type dvddrive --medium /home/downloads/ubuntu-14.04.4-server-amd64.iso

7、查看自己的网卡信息，并设置虚拟机的网卡为桥接方式（bridged）。
> &gt; ifconfig
输出：
eth0      Link encap:Ethernet  HWaddr f4:4d:30:1b:34:e1
          inet addr:192.168.100.8  Bcast:192.168.100.255  Mask:255.255.255.0

将虚拟机的第一块“网卡”nic1和eth0绑定。
> &gt; vboxmanage modifyvm ubuntu-vm-server-01--nic1 bridged --bridgeadapter1 eth0

8、设置虚拟机的IO控制，启动项顺序，内存大小，CPU数量，显存大小等其他设置（可以参考[手册](/uploads/use-vbox-on-linux/vboxmanage.txt)）。
注：安装系统时先设置dvd为第一启动项，disk为第二启动项，安装好后再掉过来。
> &gt; vboxmanage modifyvm ubuntu-vm-server-01 --ioapic on
> &gt; vboxmanage modifyvm ubuntu-vm-server-01 --boot1 dvd  --boot2 disk  --boot3 none  --boot4 none
> &gt; vboxmanage modifyvm ubuntu-vm-server-01 --memory 2048  --vram 128

9、设置远程桌面访问。
可以使用Windows自带的远程桌面连接程序连接虚拟机，也可以使用Ubuntu桌面版自带的Remmina remote desktop client。vrde模式的认证库是VBoxAuth，使用系统的用户来认证。官方文档还提供了一个VBoxAuthSimple认证库。
这里设置成VBoxAuthSimple认证库：
> \# 设定vrdeauthtype为external
> &gt; vboxmanage modifyvm ubuntu-vm-server-01 --vrdeauthtype external
> \# 设定vrdeauthlibrary 为 VBoxAuthSimple
> &gt; vboxmanage modifyvm ubuntu-vm-server-01 --vrdeauthlibrary VBoxAuthSimple
> \# 生成加密的密码字串，比如我要设定一个密码为 1234
> &gt; vboxmanage internalcommands passwordhash "1234"
> \# 输出，复制加密的密码字串
> &gt; Password hash: XXXXXXX
> \# 添加一个VBoxAuthSimple用户，用户名：test 密码：1234
> &gt; vboxmanage setextradata ubuntu-vm-server-01 "VBoxAuthSimple/users/test" XXXXXXX
> \# 开启vrde服务
> &gt; vboxmanage controlvm ubuntu-vm-server-01 vrde on vrdeport 3389

vrde服务可以在虚拟机运行时动态开启和关闭。

这样可以设置一些与系统用户无关的用户和密码用于远程桌面登陆。
> 另外，Windows使用远程登陆时需要注意，如果登陆一直卡在准备阶段，需要在连接设置里勾选上__允许我保存凭证__。

10、启动虚拟机。
万事具备，启动虚拟机开始安装。因为服务器没有桌面程序，所以需要使用headless方式启动。
> &gt; vboxmanage startvm ubuntu-vm-server-01 --type=headless

11、现在就可以用rdesktop远程连接虚拟机了
连接的 IP为服务器IP，端口为3389，需要确保服务器的防火墙3389可访问。
> &gt; rdesktop localhost:3389

12.安装完成后需要修改启动项顺序，并退出dvd上的iso，可以关闭vrde。
> &gt; vboxmanage storageattach ubuntu-vm-server-01 --storagectl "IDE Controller" --port 0 --device 0 --type dvddrive --medium __none__

#### 在VirtualBox中安装其他操作系统
大致安装过程和安装Ubuntu Server类似，我的建议是可以参照桌面版的VirtualBox安装其他操作系统的方式来反推命令的设置。

## 常用命令
> vboxmanage controlvm  &lt;uuid|vmname&gt; 
> pause | resume | reset | poweroff | savestate 
> | acpipowerbutton | acpisleepbutton |


## 其他事项
### 实现VirtualBox开机自启动
1、 在/etc/default中新建virtualbox文件
> &gt; sudo vim /etc/default/vbox

添加如下内容
> VBOXAUTOSTART_DB=/etc/vbox
> VBOXAUTOSTART_CONFIG=/etc/vbox/autostart.conf

2、 设置vbox目录的权限，添加vbox用户到vboxusers用户组
> &gt; sudo chgrp vboxusers /etc/vbox
> &gt; sudo chmod 1775 /etc/vbox
> &gt; sudo adduser vbox vboxusers

3、 新建虚拟机自动启动配置文件autostart.conf并添加如下内容
> &gt; sudo vim /etc/vbox/autostart.conf
> 
> \# default policy is to deny starting a VM, the other option is “allow”.
> default_policy = deny
> \# user vbox is allowed to start virtual machines but starting them
> \# will be delayed for 120 seconds
> vbox= {
>        allow = true
>        startup_delay = 120
> }

4、 设置开启自动启动
设置成功后会在/etc/vbox目录下生成vbox.start文件。
> &gt; vboxmanage setproperty autostartdbpath /etc/vbox
> &gt; vboxmanage modifyvm ubuntu-vm-server-01 –autostart-enabled on

如果最后一步出现以下类似的错误，那么需要用这个命令groups vbox确认vbox用户已经添加到vboxusers组中，而且vbox对/etc/vbox有写权限。
> VBoxManage: error: Adding machine ‘ubuntu-vm-server-01’ to the autostart database failed with VERR_ACCESS_DENIED

### 设置虚拟机的快照
快照是一个很好的功能，可以备份系统的状态，在虚拟机因病毒或者黑客破坏后，也能迅速恢复到备份时的状态。
新建快照
> &gt; vboxmanage snapshot ubuntu-vm-server-01 take "snapshot-001";

删除快照
> 会将虚拟机的修改写入磁盘
>  &gt; vboxmanage snapshot ubuntu-vm-server-01 delete "snapshot-001";

回滚快照
> 恢复虚拟机到备份时的状态
> &gt; vboxmanage snapshot ubuntu-vm-server-01 restore "snapshot-001";

### 复制虚拟机
安装好虚拟机系统后，再开一个虚拟机新安装系统很麻烦，可以使用虚拟机复制。
> 
方法一：
使用vboxmanage clonevm 将现有的系统克隆。
&gt; vboxmanage clonevm ""ubuntu-vm-server-02" --name "ubuntu-vm-server-01" --register
使用vboxmanage list vms查看到新的虚拟机已经复制成功。
> 
方法二：
使用vboxmanage clonehd 复制vdi磁盘。
首先：
&gt; vboxmanage clonehd ubuntu-vm-server-01.vdi ubuntu-vm-server-02..vdi
会生成一个新的UUID但内容一样的磁盘，复制到ubuntu-vm-server-02目录下，进入ubuntu-vm-server-02目录，然后就和使用VirtualBox部分差不多，区别是不用再安装系统，使用磁盘即可。

 
> 完

 
