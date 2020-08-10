---
title: 开发和学习中积累的常用工具类
date: 2017/09/23
categories:
- 工具
tags:
- 工具
- Java
---

开发和学习中积累的常用工具类
==============================
在开发和学习中，往往有很多方法会重复编写，也会碰到一些比较好的工具类。我选择其中一些并将它们整理、优化、完善，打成jar包，方便日后使用。

--------------------------------

* 本工具类名称：__chain-utils__

* 查看[GitHub源码](https://github.com/ChainGit/chain-utils)

* 点击[下载历史版本](https://github.com/ChainGit/chain-utils/blob/master/Chain-Utils/out)

* 点击[下载最新版](https://github.com/ChainGit/chain-utils/raw/master/Chain-Utils/web/latest/chain-utils-latest.jar)

* 点击[下载文档](https://github.com/ChainGit/chain-utils/raw/master/Chain-Utils/web/latest/chain-utils-javadoc.jar)

如果您在使用中存在问题，或者有改进建议，欢迎留言或提出[Issues](https://github.com/ChainGit/chain-utils/issues)。

__注：__
1、jar包内包含__源码__，使用时建议使用最新的JDK。
2、根据具体情况，有的类可以通过new和getInstance方式获得实例后使用方法，有的也可以直接使用静态方法。
3、对于异常处理，均会做log，且作为一个工具类，异常均会继续往上层抛出，但会选择对异常做适当的重新包装以减少纷繁的各种checked exception。

> 更新jar包：
1、点击上面的下载链接。
2、在命令行界面输入 java -jar chain-utils-xxxx.jar 就会自动下载最新版到当前目录。
3、双击jar包运行就会自动下载最新版到当前目录。

__版本信息：__

[version](https://github.com/ChainGit/chain-utils/blob/master/Chain-Utils/web/latest/version.txt)

<div id="util_version"></div>

---------------------------------

<div id="util_history"></div>

<script type="javascript" src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
<script>
window.onload=function(){
    $("#util_version").empty().load('https://raw.githubusercontent.com/ChainGit/chain-utils/master/Chain-Utils/web/latest/version_web.txt');
    $("#util_history").empty().load('https://raw.githubusercontent.com/ChainGit/chain-utils/master/Chain-Utils/web/latest/history.txt');
}
</script>
