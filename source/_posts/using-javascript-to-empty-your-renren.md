---
title: 用javascript快速清空你的人人时间轴、状态和分享
tags:
  - 人人
  - 删除
  - 日志
  - 状态
id: 7
categories:
  - Javascript
date: 2014-10-22 05:39:00
---

现在玩人人的人越来越少了，很多人担心不玩以后东西放上面不安全。。我也有同样的想法，但是手动删除上百条东西，太累了，于是写了些javascript来帮忙删除

<span style="line-height: 1.5;">首先提醒一下：</span>

*   删除前可以用浏览器网页另存为功能，把你的东西保存在本地（不保存下来真的太可惜了呀）
*   这些代码是我花了几分钟临时写的，仅仅是为了删除我自己的，不保证一直都有效
其实代码的原理很简单，根据class名称找到全部删除按钮，然后去click，再去click确定按钮

**使用方法：**

进入相应的页面，按f12，出现审查元素，直接在底部粘贴以下代码，按下回车即可。（若底部没有输入的地方，请按esc键）

![](http://www.imyzf.com/wp-content/uploads/2015/03/221339000279754.jpg)

这些代码不是一件删除全部的，只是删除当前页面的，**请多次输入运行**，直到全部删除。

&nbsp;

删除状态（进入状态页面）

[js]

var btn = document.getelementsbyclassname('del-status');
for (var i=0; i&lt;btn.length; i++)  {
    btn[i].click(); 
    document.getelementsbyclassname('ui-button-text')[2].click();
}
location.reload(true);
[/js]

删除分享（进入分享页面）
<div>

[js]
var btn = document.getelementsbyclassname('share-item-delete');
for (var i=0; i&lt;btn.length; i++) {
    btn[i].click();
    document.getelementsbyclassname('ui-button-blue')[0].click();
}
[/js]

清空时间轴（进入个人主页）

[js]
var btn = document.getelementsbyclassname('del');
for (var i=0; i&lt;btn.length; i++) {
    btn[i].click(); 
    document.getelementsbyclassname('input-submit')[0].click();
}
[/js]

ok，大功告成，突然有点悲伤，竟然就这么全部删完了。。

</div>