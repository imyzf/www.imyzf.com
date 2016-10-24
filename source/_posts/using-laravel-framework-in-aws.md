---
title: 在AWS上安装laravel框架
id: 13
tags:
  - php
categories:
  - server
date: 2014-02-28 13:17:00
---

laravel是现在非常热门的php框架，这几天我试着在亚马逊aws的服务器上安装laravel，遇到很多问题，最后还是成功了。我的系统是amazon linux。

怎么在aws上建linux就不说了，自行百度吧。

### 1、获取laravel

首先获取laravel.phar安装器文件

```bash
wget http://laravel.com/laravel.phar
```

然后将laravel.phar移动到/usr/local/bin并重命名为laravel，方便调用

```bash
mv laravel.phar /usr/local/bin/laravel
```

检查一下是否有运行权限，没有的话要加上。

现在你可以用laravel new命令一件生成一个laravel目录了，里面包含了所需的全部文件。例如：

```bash
laravel new demo //demo是目标文件夹，只支持相对路径
```

**注意**：不要用github上下载的laravel_master.zip，这个只包含laravel的源文件，缺少依赖项。

### 2、软链接
（避开apache配置错误）

apache配置是非常麻烦的问题，用了alias虚拟目录后，一不小心就可能出现403等错误。
我一直找不到解决方法，后来有大神告诉我用一种非常简单的方法避开httpd.conf来配置虚拟目录，那就是——万能的软链接！

```bash
ln -s /yourlaravelpath /var/www/html/laravel
```

### 3、权限

用ll命令检查app/storage是否有写入权限，没有就用下面的命令增加（请确保目录所有者是apache的账户）

```bash
chomd u+w -r app/storage
```

### 4、安装扩展

如果访问public/index.php，提示“laravel requires the mcrypt php extension”，那就是没有安装php-mcrypt扩展了，用yum一键完成吧！

但是在没有安装rpmforge源的情况下还是不能搜索到的，所以先安装rpmforge再yum。。
地址：http://repoforge.org/use/

```bash
wget http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm //下载地址根据系统版本有所不同，见上面地址
sudo rpm -ivh rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm
sudo yum install php-mcrypt
```

当然，还有可能缺少其他扩展，不同人的情况不一样，laravel会给出错误提示的，请自行百度吧。例如我就提示“class 'pdo' not found”，然后我又用yum安装了php-pdo。

安装完扩展后需要重启apache:

```bash
sudo service httpd restart
```

### 5、配置laravel
关于如何配置，网上的教程很清楚，我就不多说了，见[http://www.golaravel.com/docs/4.1/configuration/](http://www.golaravel.com/docs/4.1/configuration/)

但是既然是在aws上安装，就应该充分利用aws的rds功能，另外建一台专门处理数据库的服务器。

在rds instance控制面板中，第一行有个endpoint,这个就是你的服务器地址了，在配置database.php的时候，将'host'=>'localhost'中的localhost改成endpoint中的地址即可。

### 6、完成安装
**在浏览器中访问你的地址/public/index.php，如果出现下图结果，那就恭喜你，安装成功了！

![](http://cdn.imyzf.com/img/blog/2015/using-laravel-framework-in-aws/1.jpg)
