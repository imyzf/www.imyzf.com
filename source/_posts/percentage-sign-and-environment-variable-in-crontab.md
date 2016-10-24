---
title: crontab的两大坑：百分号和环境变量
tags:
  - linux
id: 99
categories:
  - server
date: 2015-04-03 21:07:59
---

今天想给服务器加个自动备份mysql数据库的功能（别怪我这么久才加，阿里云每天全盘备份的，不怕丢数据库），本以为只要5分钟就能搞定的，结果入了两个大坑。

我的crontab是这样写的：

```bash
0 3 * * * mysqldump -u user -pxxxx database > "/alidata/backup/imyzf.com/$(date +%F\ %T).sql"
```

首先，是百分号（%）。

在crontab -e中输入的命令里，第一个%会被认为是标准输入的开始，接下来的%都会被认为是换行。所以在这里原本只是格式化日期的%被当成了标准输入，命令就出问题了。（详见：http://www.hcidata.info/crontab.htm）
解决方案有两种，一是用上面链接里提到的sed；我采用了另外一种方法，把命令写到了sh文件里（为什么？因为还有一个大坑）。

然后，是环境变量。

即使解决了上面的问题，还是不能正常执行任务，因为crontab的环境变量是另外定义的。通过`cat /etc/crontab`你会发现默认的PATH是`/sbin:/bin:/usr/sbin:/usr/bin`，而我们的mysqldump是在/alidata/server/mysql/bin里的。

[![contab系统设置](http://cdn.imyzf.com/img/blog/2015/percentage-sign-and-environment-variable-in-crontab/1.png)](http://cdn.imyzf.com/img/blog/2015/percentage-sign-and-environment-variable-in-crontab/1.png)

所以要修改默认设置，或者简单点，在sh文件里另外加上一行修改PATH，最后成了这样：

```bash
#!/bin/bash
PATH="$PATH:/alidata/server/mysql/bin"
mysqldump -u user -pxxxx database > "/alidata/backup/www.imyzf.com/$(date +%F\ %T).sql"
```

然后，我的crontab写成了这样（使用/dev/null是为了丢弃mysqldump使用标准输出的提示）：

```bash
0 3 * * * /alidata/backup/www.imyzf.com/backup.sh > /dev/null 2>&1
```

好了，终于搞定了！
