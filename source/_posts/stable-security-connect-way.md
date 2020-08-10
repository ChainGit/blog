---
title: 建立稳定、安全的通信连接
date: 2019/01/21
categories: 
- 学习
tags: 
- 学习
---

建立稳定、安全的通信连接
=========================
连接阿里云、腾讯云服务器时，有时网络不稳定，经常出现断网情况。有很多能增强连接稳定性与安全性的方式。
比如PPTP、L2TP、IPSec、IKEv2等，这里介绍使用PPTP、IKEv2方式来有效解决这类问题。

## 环境准备

1、新装 Ubuntu 16.04.5 (其余操作系统类似，除了安装软件方式有区别外)

ubuntu16开始使用apt代替apt-get，在使用上没有什么差别。

## 安装准备

执行

```
apt update
apt -f install tree lrzsz git-core
```

可以安装上nginx，也可以不装。

```
apt -f install nginx
```

## 安装步骤

### 前提准备

1、防火墙、IP转发等

ubuntu默认使用ufw。

```
1、切换到root账户
sudo su

2、切换到/dev
cd /etc

3、修改ufw默认配置
vim /etc/default/ufw
修改
DEFAULT_FORWARD_POLICY="DROP"
为
DEFAULT_FORWARD_POLICY="ACCEPT"

4、修改sysctl.conf
vim /etc/ufw/sysctl.conf
反注释为：
# Uncomment this to allow this host to route packets between interfaces
net/ipv4/ip_forward=1
net/ipv6/conf/default/forwarding=1
net/ipv6/conf/all/forwarding=1

5、修改ufw防火墙配置
vim /etc/ufw/before.rules
在指定位置插入：
#
# rules.before
#
# Rules that should be run before the ufw command line added rules. Custom
# rules should be added to one of these chains:
#   ufw-before-input
#   ufw-before-output
#   ufw-before-forward
#

##############################################################################
# 指定位置加入nat配置
*nat
:PREROUTING ACCEPT [0:0]
# 在有的服务提供商是这两条生效，注意eth0不同机器可能不一样，使用ifconfig可以判断实际是哪一个
# -A PREROUTING -i eth0 -p tcp --dport 1723 -j DNAT --to xx.xx.xx.xx
# -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j SNAT --to xx.xx.xx.xx
# PPTP使用
-A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
# IKEv2使用
-A POSTROUTING -s 10.31.2.0/24 -o eth0 -j MASQUERADE
COMMIT
##############################################################################

# Don't delete these required lines, otherwise there will be errors
*filter
:ufw-before-input - [0:0]
:ufw-before-output - [0:0]
:ufw-before-forward - [0:0]
:ufw-not-local - [0:0]
# End required lines

6、打开ufw的特定入端口
# IKEv2基于UDP，使用500、4500
ufw allow 500
ufw allow 4500
# PPTP基于TCP，使用1723
ufw allow 1723
# 申请证书使用
ufw allow 80

其余端口比如SSH视自身情况而定。

7、重启服务器

```

### 配置PPTP

```
1、安装pptpd
apt -f install pptpd

2、配置pptpd.conf
反注释：
#          be set to the given one. You MUST still give at least one remote
#          IP for each simultaneous client.
#
# (Recommended)
##############################################################################
localip 192.168.0.1
remoteip 192.168.0.234-238,192.168.0.245
##############################################################################
# or
#localip 192.168.0.234-238,192.168.0.245
#remoteip 192.168.1.234-238,192.168.1.245

3、配置ppp
vim /etc/ppp/pptpd-options 
修改为：
# If pppd is acting as a server for Microsoft Windows clients, this
# option allows pppd to supply one or two DNS (Domain Name Server)
# addresses to the clients.  The first instance of this option
# specifies the primary DNS address; the second instance (if given)
# specifies the secondary DNS address.
# Attention! This information may not be taken into account by a Windows
# client. See KB311218 in Microsoft's knowledge base for more information.
##############################################################################
#ms-dns 8.8.8.8
#ms-dns 8.8.4.4
ms-dns 223.5.5.5
ms-dns 223.6.6.6
##############################################################################

# If pppd is acting as a server for Microsoft Windows or "Samba"
# clients, this option allows pppd to supply one or two WINS (Windows
# Internet Name Services) server addresses to the clients.  The first
# instance of this option specifies the primary WINS address; the
# second instance (if given) specifies the secondary WINS address.
##############################################################################
#ms-wins 8.8.8.8
#ms-wins 8.8.4.4
ms-wins 223.5.5.5
ms-wins 223.6.6.6
##############################################################################

4、修改chap验证
vim /etc/ppp/chap-secrets
增加pptp服务使用的chap账号
# Secrets for authentication using CHAP
# client        server  secret                  IP addresses

##############################################################################
user pptpd pass *
##############################################################################

5、启用pptpd服务开启自启动
systemctl enable pptpd

6、重启服务器，配置完毕
netstat -ant
可以看到1723端口已经监听
```

PPTP协议简单，配置方便，无需证书，兼容性好。但是连接有时也会受到干扰，建议使用IKEv2方式。

### 配置IKEv2

#### 签发自有证书

IKEv2建议配置证书方式+账号密码验证，这样安全性很高。

证书可以选择rsa方式自己生成一个，但是缺点是客户端需要安装证书并信任，比较麻烦，这里不推荐。

使用letsencrypt，结合自己的域名，签署一个自然信任的证书，这样客户端不用安装额外自定义证书，很方便。

```
1、下载letsencrypt
切换到自己的home目录下，下载letsencrypt
git clone https://github.com/letsencrypt/letsencrypt
然后
cd letsencrypt
执行
./letsencrypt-auto --help
会安装一些必须环境，比如python环境。
在配置过程中，可能会遇到如下提示：
Creating virtual environment...
Traceback (most recent call last):
  File "/usr/lib/python3/dist-packages/virtualenv.py", line 2363, in <module>
    main()
  File "/usr/lib/python3/dist-packages/virtualenv.py", line 719, in main
    symlink=options.symlink)
  File "/usr/lib/python3/dist-packages/virtualenv.py", line 988, in create_environment
    download=download,
  File "/usr/lib/python3/dist-packages/virtualenv.py", line 918, in install_wheel
    call_subprocess(cmd, show_stdout=False, extra_env=env, stdin=SCRIPT)
  File "/usr/lib/python3/dist-packages/virtualenv.py", line 812, in call_subprocess
    % (cmd_desc, proc.returncode))
OSError: Command /opt/eff.org/certbot/venv/bin/python2.7 - setuptools pkg_resources pip wheel failed with error code 2

执行可以解决：
pip install setuptools
pip install --upgrade setuptools
pip install virtualenv
pip install --upgrade virtualenv
如果还是不行，强制重装pip，比如重新安装成最新版pip，再执行以上步骤。

执行成功后会提示如下：
Creating virtual environment...
Installing Python packages...
Installation succeeded.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

  letsencrypt-auto [SUBCOMMAND] [options] [-d DOMAIN] [-d DOMAIN] ...

Certbot can obtain and install HTTPS/TLS/SSL certificates.  By default,
it will attempt to use a webserver both for obtaining and installing the
certificate. The most common SUBCOMMANDS and flags are:

...

2、签发证书
./letsencrypt-auto certonly --server https://acme-v01.api.letsencrypt.org/directory --agree-dev-preview

会有以下选择：
How would you like to authenticate with the ACME CA?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: Nginx Web Server plugin (nginx)
2: Spin up a temporary webserver (standalone)
3: Place files in webroot directory (webroot)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

因为已经安装了nginx，最简单的就是选择3。

Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): aaa@bbb.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v01.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: a

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: y
Please enter in your domain name(s) (comma and/or space separated)  (Enter 'c'
to cancel): xxx.yyy.zzz
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for xxx.yyy.zzz
Input the webroot for xxx.yyy.zzz: (Enter 'c' to cancel): /var/www/html
Waiting for verification...
Cleaning up challenges
Use of --agree-dev-preview is deprecated.

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/xxx.yyy.zzz/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/xxx.yyy.zzz/privkey.pem
   Your cert will expire on 2019-04-21. To obtain a new or tweaked
   version of this certificate in the future, simply run
   letsencrypt-auto again. To non-interactively renew *all* of your
   certificates, run "letsencrypt-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

至此，就成功生成了证书。

注意：证书三个月后会过期，可以使用定时任务脚本去定期更新

./letsencrypt-auto --nginx renew ----force-renew

```

#### 配置strongswan

```
1、安装strongswan
apt -f install strongswan*
安装所有的stringswan插件，避免缺失插件造成功能不全。

2、复制letsencrypt生成的证书
cp /etc/letsencrypt/live/xxx.yyy.zzz/chain.pem /etc/ipsec.d/cacerts/ca.cert.pem    
cp /etc/letsencrypt/live/xxx.yyy.zzz/cert.pem /etc/ipsec.d/certs/server.cert.pem  
cp /etc/letsencrypt/live/xxx.yyy.zzz/privkey.pem /etc/ipsec.d/private/server.pem   

结构如下：
.
├── aacerts
├── acerts
├── cacerts
│   └── ca.cert.pem
├── certs
│   └── server.cert.pem
├── crls
├── ocspcerts
├── policies
├── private
│   └── server.pem
└── reqs

注意pem文件权限。

3、编辑strongswan.conf
vim /etc/strongswan.conf

# strongswan.conf - strongSwan configuration file
#
# Refer to the strongswan.conf(5) manpage for details
#
# Configuration changes should be made in the included files

charon {
        load_modular = yes
        duplicheck.enable = no
        compress = yes
        plugins {
                include strongswan.d/charon/*.conf
        }
		#dns1 = 8.8.8.8
        #dns2 = 8.8.4.4
        #nbns1 = 8.8.8.8
        #nbns2 = 8.8.4.4
        dns1 = 223.5.5.5
        dns2 = 223.6.6.6
        nbns1 = 223.5.5.5
        nbns2 = 223.6.6.6
}

include strongswan.d/*.conf

4、编辑ipsec.secrets
 vim /etc/ipsec.secrets 

# This file holds shared secrets or RSA private keys for authentication.

# RSA private key for this host, authenticating it to any other host
# which knows the public part.

: RSA server.pem
#: PSK "myPSKkey"
#: XAUTH "myXAUTHPass"
user %any : EAP "pass"

5、编辑ipsec.conf
vim /etc/ipsec.conf

config setup
    uniqueids=never 

conn ikev2
    keyexchange=ikev2
    ike=aes256-sha256-modp2048,3des-sha1-modp2048,aes256-sha1-modp2048!
    esp=aes256-sha256,3des-sha1,aes256-sha1!
    rekey=no
    left=%defaultroute
    leftid=@xxx.yyy.zzz
    leftsendcert=always
    leftsubnet=0.0.0.0/0
    leftcert=server.cert.pem
    right=%any
    rightauth=eap-mschapv2
    rightsourceip=10.31.2.0/24
    rightsendcert=never
    eap_identity=%any
    dpdaction=clear
    fragmentation=yes
    auto=add

6、重启服务器

7、查看ipsec
ipsec statusall

Status of IKE charon daemon (strongSwan 5.3.5, Linux 4.4.0-141-generic, x86_64):
  uptime: 4 seconds, since Jan 21 13:07:17 2019
  malloc: sbrk 1765376, mmap 532480, used 1025520, free 739856
  worker threads: 11 of 16 idle, 5/0/0/0 working, job queue: 0/0/0/0, scheduled: 0
  loaded plugins: charon test-vectors unbound ldap pkcs11 aes rc2 sha1 sha2 md4 md5 rdrand random nonce x509 revocation constraints acert pubkey pkcs1 pkcs7 pkcs8 pkcs12 pgp dnskey sshkey dnscert ipseckey pem openssl gcrypt af-alg fips-prf gmp agent chapoly xcbc cmac hmac ctr ccm gcm ntru bliss curl soup mysql sqlite attr kernel-netlink resolve socket-default connmark farp stroke updown eap-identity eap-sim eap-sim-pcsc eap-aka eap-aka-3gpp2 eap-simaka-pseudonym eap-simaka-reauth eap-md5 eap-gtc eap-mschapv2 eap-dynamic eap-radius eap-tls eap-ttls eap-peap eap-tnc xauth-generic xauth-eap xauth-pam xauth-noauth tnc-imc tnc-imv tnc-tnccs tnccs-20 tnccs-11 tnccs-dynamic dhcp whitelist lookip error-notify certexpire led radattr addrblock unity
Virtual IP pools (size/online/offline):
  10.31.2.0/24: 254/0/0
Listening IP addresses:
  172.xx.xx.xx
Connections:
       ikev2:  %any...%any  IKEv2, dpddelay=30s
       ikev2:   local:  [xxx.yyy.zzz] uses public key authentication
       ikev2:    cert:  "CN=xxx.yyy.zzz"
       ikev2:   remote: uses EAP_MSCHAPV2 authentication with EAP identity '%any'
       ikev2:   child:  0.0.0.0/0 === dynamic TUNNEL, dpdaction=clear
Security Associations (0 up, 0 connecting):
  none

```

## 建立连接

1、安卓手机

直接在系统设置里，新建VPN连接，选择PPTP协议或IKEv2，输入服务器域名、账号、密码即可。

2、苹果手机

在系统设计里，新建VPN，IOS最近版本只能选择IKEv2，远程ID输入上述的xxx.yyy.zzz，再输入服务器域名、账号、密码即可。

3、Windows系统

系统设置里，新建VPN连接，选择PPTP协议或IKEv2，输入服务器域名、账号、密码即可。
另外，在控制面板，找到"你的VPN连接名"，右键属性，选择网络，打开"Internet协议"属性，在高级设置里，勾选"在远程网络上使用默认网关"。

由于ipsec配置使用了强加密，因此需要修改windows的默认注册表设置：
[修改为强加密](/_files/NoRasManParameters_IKEv2CustomPolicy.reg)
[修改为弱加密](/_files/RasManParameters_IKEv2CustomPolicy.reg)

4、Mac系统

系统设置里，新建IKEv2连接，远程ID输入上述的xxx.yyy.zzz，再输入服务器域名、账号、密码即可。

## 经验技巧

```
tail -200f /var/log/syslog
```

查看连接日志，能有效排除一些配置中遇到的问题。

如果发现连接成功没有成功上网：
1、检查DNS是否可用（可以本地客户端设置自己的DNS服务器地址）
2、检查服务器防火墙配置

如果发现突然连接不上：
1、更换服务器IP

