---
title: 解决laravel中环境配置不起作用的方法
id: 12
categories:
  - server
date: 2014-04-06 03:23:00
tags:
---

laravel有个环境配置选项很好用，在bootstrap/start.php中，曾经百度到这里面加入域名，就可以自动选择环境

```php
$env = $app->detectenvironment(array(
    'development' => array('localhost'),
    'production' => array('www.example.com'),
));
```

本以为这样就好了，结果不论放在服务器上还是本机，都自动选择了production环境配置。

经过google以后，找到了答案：

> using hostname for environment detection is bad design and can open up your app to attacks. laravel 4.1 no longer allows using hostnames for environment detection. you should be using machine names or use a closure which makes use of server vars to detect current environment.

从laravel 4.1开始，不支持通过域名来判断环境了，解决方法很简单，把上面代码中localhost改成你的**计算机名**就是了。（win7里右键“计算机”就可以看到，点属性就可以到计算机名）

```php
$env = $app->detectenvironment(array(
    'development' => array('pc_name'),
    'production' => array('server_name'),
));
```
