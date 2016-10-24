---
title: 手动将MySQL服务安装到windows中
id: 11
categories:
  - 服务器
date: 2014-04-08 03:47:00
tags:
---

我的系统是win7 x64，mysql版本5.6 （网上的老教程很多都没用了，版本太旧，这是我自己总结出来的方法）

下载zip包的mysql可以获得最新版本，还可以免安装，好处多多，但是要把mysql安装到windows的服务中才可以开机启动，不然每次都要手动启动mysqld.exe。

安装过程其实简单：

<span style="line-height: 1.5;"><span style="line-height: 1.5;">打开cmd，cd到mysql文件的bin目录，我的是解压到了c:\program files\mysql，于是输入</span></span>

[shell]
cd &quot;c:\program files\mysql\bin&quot;
[/shell]

使用mysqld.exe安装服务

[shell]
mysqld --install
[/shell]

提示安装成功了，但是这时候还是不能使用的，需要修改路径，这里使用sc命令，binpath表示路径（请修改为你的mysqld所在路径），路径后面跟了一个mysql是mysqld.exe本身的参数（安装时默认添加的）。

[shell]
sc config mysql binpath= &quot;c:\program files\mysql\bin\mysqld mysql&quot;
[/shell]

为什么要做这一步呢，网上很多人说安装后不能启动服务，打开计算机管理——服务，查看mysql服务属性，就会发现，默认的路径是不正确的，需要修改为正确的路径。

ok，至此mysql服务就安装完成了。