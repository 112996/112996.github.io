---
layout: post
title: "Android之launchMode与android:TaskAffinity"
date: 2018-11-12 11：00
description: "Android之launchMode与android:TaskAffinity"
tag: Android
---
最近在刷牛客网上面关于Android的基础题，涉及到android:launchMode属性的描述,里面有个选项是这么讲到：“android:taskAffinity与singleTask搭配使用可以使Activity运行在指定栈中”。

这个东西是个什么鬼？？虽然平时时不时就new activity，但是关于activity的启动模式仅限于字面上的理解，今天我就来搞一下这个launchMode以及taskAffinity。


Activity的任务栈是我们很熟悉的了，它是一种后进先出的结构。位于栈顶的Activity处于焦点状态,当按下back按钮的时候,栈内的Activity会一个一个的出栈,并且调用其onDestory()方法。如果栈内没有Activity,那么系统就会回收这个栈,每个APP默认只有一个栈,以APP的包名来命名.

首先，我们需要了解什么是Activity的启动模式（launchMode）：

Standard：

标准模式,每次启动Activity都会创建一个新的Activity实例,并且将其压入任务栈栈顶,而不管这个Activity是否已经存在。Activity的启动三回调(onCreate()->onStart()->onResume())都会执行。

SingleTop：

栈顶复用.这种模式下,如果新Activity已经位于任务栈的栈顶,那么此Activity不会被重新创建,所以它的启动三回调就不会执行,同时Activity的onNewIntent()方法会被回调.如果Activity已经存在但是不在栈顶,那么作用与standard模式一样.一般用于显示推送过来的消息。

SingleTask：

栈内复用.创建这样的Activity的时候,系统会先确认它所需任务栈已经创建,否则先创建任务栈.然后放入Activity,如果栈中已经有一个Activity实例,那么将其之上的Activity全部出栈，使得这个目标Activity被调到栈顶,onNewIntent(),并且singleTask会清理在当前Activity上面的所有Activity.(clear top)。一般用于程序入口等启动页面。

SingleInstance：

加强版的singleTask模式,这种模式的Activity只能单独位于一个任务栈内,由于栈内复用的特性,后续请求均不会创建新的Activity,除非这个独特的任务栈被系统销毁了。类似于闹钟那样的使用。

TaskAffinity

现在我们进入到TaskAffinity的学习，如果单独设置TaskAffinity属性的话是不会产生任何效果的，只有Activity的launchMode为singleTask的时候才会生效。下面我们就来验证：

```
注意：
1.TaskAffinity的值应该是xxx.xxx.xxx类似包名的，如果没有包括.的话是安装不了的；   
2.如果不指定TaskAffinity的话，默认的值是包名。
```

实验一：
<div>
  <img src = "/images/LaunchMode/manifest.png" width="720" height="500">
</div>

<div>
  <img src = "/images/LaunchMode/mainActivity.png" width="773" height="438">
</div>

<div>
  <img src = "/images/LaunchMode/log1.png" width="844" height="428">
</div>
这里可以看到，每个Activity的TaskId都是59，可见TaskAffinity属性并没有起作用。

实验二：

将Num2Activity的launchMode属性修改为singleTask
<div>
  <img src = "/images/LaunchMode/manifest2.png" width="720" height="500">
</div>

<div>
  <img src = "/images/LaunchMode/log2.png" width="844" height="428">
</div>

这里可以看到Num2Activity的TaskId改变了，启动Activity的时候会根据taskAffinity查找是否有存在的任务栈，没有的话就创建一个新的任务栈。同时在启动Num3Activity，也会加入到当前的任务栈中。

实验三：

将Num2Activity的launchMode属性修改为singleInstance
<div>
  <img src = "/images/LaunchMode/manifest3.png" width="720" height="500">
</div>

<div>
  <img src = "/images/LaunchMode/log3.png" width="844" height="428">
</div>

可以看到Num2Activity是创建在新的栈里，而Num3Activity却还是创建在原来的栈里面，这是因为singleInstance的特性造成的，它会创建一个新的栈并且里边就只有一个Activity实例，所以和singTask不同，之后启动的Num3Activity不会进入到该栈中。

其实这里Num2Activity中设置的taskAffinity是没意义的，就算不设置结果也是一样的，因为singleInstance会创建一个新的栈并只能保存唯一的Activity，所以其他的Activity就算设置了一样的taskAffinity也不起作用了。

这里讲一下启动过程：
MainActivity(62栈进入前台) -->Num2Activity(63栈进入前台，62栈进入后台) --> Num3Activity(62栈进入前台，63栈进入后台)  

最终栈里面的情况是：

前台：MianActivity、Num3Activity

后台：Num2Activity

那么我们再一次按返回键可以看到返回过程是：

Num3Activity --> MainAcitvity --> Num2Activity

总结：

到这里TaskAffinity的作用已经很明了了，通过这个属性可以把不同的Activity分在不同的任务栈中，这里总结一下重点：

1.TaskAffinity属性只有在launchMode=singleTask的时候才有作用(实验一、实验二)；

2.正常情况下启动的Activity会默认加到当前的前台栈中(实验二)；
