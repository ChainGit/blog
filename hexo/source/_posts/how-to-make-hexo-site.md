---
title: 如何搭建免费又好用的静态博客
date: 2017/11/1
categories:
- 其他
tags:
- 其他
- Hexo
---

如何搭建免费又好用的静态博客
=============================

最近有小学弟咨询如何搭建一个自己的博客，大家的学习热情都很高，这是好事。

博客主要有静态博客和动态博客两种。这里我主要介绍静态博客搭建的注意事项。

目前我采用的方法：

> Hexo + GitHub + Coding.net

## 历史方案

介绍一下我自己的博客搭建历程：

1）使用WordPress搭建于阿里云学生版服务器
2）使用Hexo搭建于阿里云学生版服务器
3）使用本地虚拟机+GitHub+CDN
4）使用本地虚拟机+GitHub+阿里云学生版服务器
5）使用本地虚拟机+GitHub+Coding.net

搭建博客最好有一个自己的域名，我的域名是在阿里云上申请并备案的。

## 目前做法

1）博客采用Hexo平台，是安装在本地VirtualBox虚拟机里，虚拟机的好处是不用担心任何安全问题，且即使自己更换操作系统或新的电脑，可以随时备份虚拟机存储文件vdi，而不用担心重新配置的问题。

Hexo的安装可以参阅[官网](https://hexo.io/)。
使用的主题Next可以参阅[官网](http://theme-next.iissnan.com/)。

2）由Hexo生成静态网页后可以在本地虚拟机先进行预览，再修改。

3）通过Hexo的部署命令可以将生成的网站push到github和coding.net中。

4）github pages 和 coding pages 可以参阅这两篇博客：

[github pages](https://pages.github.com/)。
[coding pages](https://coding.net/help/doc/pages)。

当然也有一些其他的更详细的教程。

## 补充说明

1）github的静态网页在国内访问不是很稳定，因此采用coding来等效实现github pages的功能。而且两者都是免费的，coding的功能更完善。

2）github pages 可以通过添加CNAME文件来使得访问xxx.github.io时能跳转到自己的域名上；
同样的，coding pages 也可以在设置中直接添加自己的域名，而且支持https。

3）阿里云服务器还是需要支付一定的费用的，不过我本人更倾向于使用阿里云服务器做一些项目开发和学习。

4）至于如何将自己的域名添加到搜索引擎，谷歌和必应的添加是很简单的，只要提交一下自己的域名即可。百度的提交比较麻烦，而且不一定能添加成功。

Hexo支持各种sitemap。





