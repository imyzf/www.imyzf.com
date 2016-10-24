---
title: 'C# WinForm动态控件实例：口算训练'
id: 15
categories:
  - others
date: 2013-09-29 05:01:00
tags:
---

昨天晚上回寝室看到室友正在被一个c#课的作业苦恼，作业的内容是编写一个口算训练程序，能够实现随意添加题目数量。于是，喜欢写c#的我就决定解救一下他们。

### 创建动态控件

既然要动态添加，那就必须使用动态控件了。在c#中，控件也是类，除了在画窗体的时候添加固定的控件外，还可以在代码中用实例化类的方法添加。

具体操作是，我们先定义一个控件变量，然后对控件设置size, location这些属性，最后，再把控件添加的一个panel中。而且我们只要定义一次控件变量，之后用new不停的添加，就可以获得很多控件了。

部分代码如下

```csharp
txtbox = new textbox();
txtbox.size = new size(50, 50);     //设置大小
txtbox.location = new point(x, y);  //设置位置坐标
txtbox.name = "txt" + convert.tostring(i); //设置控件名（可重名）
panelquestion.controls.add(txtbox);
```

### 访问动态控件

在窗体中手动绘制的控件，我们可以通过控件名直接访问，但是动态添加的控件就不可以了，只能在panel中查找对应name属性的控件。

```csharp
string str = ((textbox)panelquestion.controls.find("txtbox" ,true)[0]).text;
```

find方法中的第一个参数为控件名称，第二个参数为是否搜索所以子控件。由于可以重名，所以返回的是一个控件数组，上面的[0]表示取第一个返回结果。由于返回的类型是control，还需要强制转换为具体的控件类型，所以前面加了(textbox)，强制转换为textbox类型，这样才能当做textbox使用。

### 具体实现

[![](http://cdn.imyzf.com/img/blog/2013/a-csharp-winform-dynamic-component-example/1.jpg)](http://cdn.imyzf.com/img/blog/2013/a-csharp-winform-dynamic-component-example/1.jpg)

窗体设计如上图，控件名称分别为txttotal, btnadd, btnjudge, panelquestion

在出题按钮事件中，进行进行动态添加textbox和label，每行3个textbox，显示两个加数和一个空白框填写结果，name都为txt+行号；还有三个label，从左到右为&ldquo;+&rdquo;、&ldquo;=&rdquo;和空白的用来显示对错。

在批改按钮事件中，访问已经动态建立的控件，获取textbox里的值，然后进行批改，把对错写入每行最后一个label中。

运行结果如下：

[![](http://cdn.imyzf.com/img/blog/2013/a-csharp-winform-dynamic-component-example/1.jpg)](http://cdn.imyzf.com/img/blog/2013/a-csharp-winform-dynamic-component-example/1.jpg)

其他的不废话了，贴代码！

```csharp
using system;
using system.collections.generic;
using system.componentmodel;
using system.data;
using system.drawing;
using system.linq;
using system.text;
using system.windows.forms;

namespace addprogram
{
    public partial class form1 : form
    {
        public form1()
        {
            initializecomponent();
        }

        private void btnadd_click(object sender, eventargs e)
        {
            if (txttotal.text == &quot;&quot;)
            {
                messagebox.show(&quot;请输入题目数！&quot;, &quot;错误&quot;);
                return;
            } //判断题目数是否未填
            panelquestion.autoscroll = true;  //为panel添加滚动条
            panelquestion.controls.clear();  //清空已有题目
            int total = int.parse(txttotal.text);  //题目总数
            textbox txtbox = new textbox();
            label label = new label();
            random rand = new random();  //随机数
            for (int i = 0; i &amp;lt; total; i++)
            {
                for(int j = 0; j &amp;lt; 3; j++)
                {
                    txtbox = new textbox();
                    txtbox.size = new size(50, 50);   //textbox大小
                    txtbox.location = new point(10 + 70 * j, 30 * i);  //textbox坐标
                    txtbox.name = &quot;txt&quot; + convert.tostring(i);  //设定控件名称
                    if (j &amp;lt; 2)
                    {
                        txtbox.text = convert.tostring(rand.next(100));
                        txtbox.readonly = true;
                    }  //产生随机数,作为加数
                    panelquestion.controls.add(txtbox); //把控件加入到panel中
                    label = new label();
                    label.size = new size(12, 12);
                    label.location = new point(64 + 70 * j, 30 * i);
                    switch (j)
                    {
                        case 0: label.text = &quot;+&quot;; break;
                        case 1: label.text = &quot;=&quot;; break;
                        case 2: label.name = &quot;labelresult&quot; + convert.tostring(i); break;
                    }
                    panelquestion.controls.add(label);
                }
            }
        }

        private void btnjudge_click(object sender, eventargs e)
        {
            if (txttotal.text == &quot;&quot;)
            {
                messagebox.show(&quot;请输入题目数！&quot;, &quot;错误&quot;);
                return;
            }
            int total = convert.toint32(txttotal.text);
            for (int i = 0; i &amp;lt; total; i++)
            {
                textbox[] txtbox = new textbox[3];  //用控件数组来定义每一行的textbox,总共3个textbox
                for (int j = 0; j &amp;lt; 3; j++)   //获取原来已经动态添加的textbox,这样才能访问
                    txtbox[j] = (textbox)panelquestion.controls.find(&quot;txt&quot; + convert.tostring(i), true)[j];
                label label = (label)panelquestion.controls.find(&quot;labelresult&quot; + convert.tostring(i), true)[0];
                if (txtbox[2].text == &quot;&quot;)  //如果未填答案,直接批改为错误
                {
                    label.text = &quot;&amp;times;&quot;;
                    continue;
                }
                int add = convert.toint32(txtbox[0].text) + convert.toint32(txtbox[1].text);
                if (add == convert.toint32(txtbox[2].text))
                    label.text = &quot;&amp;radic;&quot;;
                else
                    label.text = &quot;&amp;times;&quot;;
            }
        }

    }
}
```
