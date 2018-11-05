---
layout: post
title: "Android之APP启动速度优化"
date: 2018-11-5 14：06
description: "Android之APP启动速度优化"
tag: Android
---
APP启动方式分为三类，分别为：冷启动、热启动以及温启动。

冷启动：

  当启动应用时，后台没有该应用的进程，这时系统会重新创建一个新的进程分配给该应用，这个启动方式就是冷启动。

热启动：

  当启动应用时，后台已有该应用的进程（例：按back键、home键，应用虽然会退出，但是该应用的进程是依然会保留在后台，可进入任务列表查看），所以在已有进程的情况下，这种启动会从已有的进程中来启动应用，这个方式叫热启动。

温启动：

  App进程存在，当时Activity可能因为内存不足被回收。这时候启动App不需要重新创建进程，但是Activity的onCrate还是需要重新执行的。场景类似打开淘宝逛了一圈然后切到微信去聊天去了，过了半小时再次回到淘宝。这时候淘宝的进程存在，但是Activity可能被回收，这时候只需要重新加载Activity即可。

AI与启动方式的关系：

  AI在进程管理方面可谓是大有可为。MIUI10发布了进程AI唤醒功能，是APP启动速度远超友商。这其中的道理简单说就是学习用户的使用习惯，提前将App进程创建好，当用户打开APP时不会出去冷启动。比如你是微信重度用户你发现用了MIUI10就再也见不到微信启动页面的那个地球了，这就是AI唤醒的功劳。

从点击APP图标到主页显示出现需要经过的步骤
  这里我们讨论冷启动的过程，进程启动原则上有四种途径，就是通过其他进程对该APP的四大组件的调用实现。
  <div>
    <img src = "/images/app_start-up/start-up_app.png" width="637" height="486" />
  </div>
  这里我们重点讨论用户点击桌面后的APP启动，通过startActivity方式启动。调用startActivity，这个方法经过层层调用，最终会调用ActivityStackSupervisor.java中的startSpcificActivirtyLocked，当activity所属进程还没启动的情况下，则需要创建相应的进程。
  <div>
    <img src = "/images/app_start-up/startSpecificActivityLocked.png" width="589" height="217">
  </div>
  最终进程由 Zygote Fork进程:
  <div>
    <img src = "/images/app_start-up/zygote.png" width ="635" height="421">
  </div>
  进程启动后系统还有一个工作：进程启动后立即显示应用程序的空白启动窗口。

  一旦系统创建应用程序进程，应用程序进程就会负责下一些阶段：

  1、创建应用程序对象；

  2、启动主线程；

  3、创建主要Activity；

  4、绘制视图（View）；

  5、布局屏幕；

  6、执行初始化绘制；

  当App进程完成了第一次的绘制之后，系统进程就会用MainActivity替换已经展示的Background Window，这个时候用户就可以正式使用App了。
  <div>
    <img src = "/images/app_start-up/use_app.png" width="874" height="391">
  </div>

  这个地方有两个优化点：

  1、Application OnCreate()优化。当App启动时，空白的启动窗口将保留在屏幕上，直到系统首次完成绘制应用程序。这个时候，系统进程会交换应用程序的启动窗口，允许用户开始与应用程序进行交互。如果应用程序中重载了Application.onCreate()，系统会调用onCreate()方法。之后，应用程序会生成主线程（也称为UI线程），并通过创建MainActivity来执行任务。

  2、Activity onCreate()优化。onCreate()方法对加载时间的影响最大，因为它以最高的开销执行工作：加载并绘制视图，以及初始化Activity运行所需的对象。

如何对启动时间进行量化?

    通过shell命令
    ```
    adb shell am start -W [packageName]/[packageName. MainActivity]
    ```
    执行成功后会返回三个测量到的时间：

    ThisTime:一般和TotalTime时间一样，除非在应用启动时开了一个透明的Activity预先处理一些事再显示出主Activity，这样将比TotalTime小。

    TotalTime:应用的启动时间，包括创建进程+Application初始化+Activity初始化到界面显示。

    WaitTime:一般比TotalTime大点，包括系统影响的耗时。

Application onCreate()优化

    1、第三方SDK初始化的处理:

    Application是程序的主入口，很多三方SDK示例程序中都要求自己在Application OnCreate时做初始化操作。这就是增加Application OnCreate时间的主要元凶，所以需要尽量避免在Application onCreate时同步做初始化操作。比较好的解决方案就是对三方SDK就行懒加载，不在Application OnCreate()时初始化，在真正用到的时候再去加载。
    下面实例对比下ImageLoader在采用懒加载后启动速度优化。
    <div>
      <img src="/images/app_start-up/application.png" width="884" height="295">
    </div>

    然后我们检查应用启动时间，每次执行命令时需要杀死进程：
    <div>
      <img src="/images/app_start-up/application_time.png" width="1083" height="230">
    </div>

    当我们封装一个懒加载ImageLoader的工具类后，可以明显看到应用启动时间的缩短。所以，在Applocation onCreate()方法中，避免在主线程做大量耗时操作，跟I/O有关的逻辑操作都会对应用的启动时间产生极大的影响。

Activity onCreate()优化：

  减少LaunchActivity的View层级，减少View测量绘制时间。避免主线程做耗时操作。

1、用户体验优先
  消除启动时的白屏或黑屏：

  为什么启动时会出现短暂黑屏或白屏的现象？当用户点击你的app那一刻到系统调用Activity.onCreate()之间的这个时间段内，WindowManager会先加载app主题样式中的windowBackground做为app的预览元素，然后再真正去加载activity的layout布局。

  很显然，如果你的application或activity启动的过程太慢，导致系统的BackgroundWindow没有及时被替换，就会出现启动时白屏或黑屏的情况（取决于你的主题是Dark还是Light）。
  <div>
    <img src="/images/app_start-up/use_app_activity.png" width="835" height="374">
  </div>

2、解决方案

主题替换

1、在style中自定义一个样式Lancher，在里面放置一张背景图片或者广告图片之类，
```
<style name="AppTheme.Launcher">
  <item name="android:windowBackground">@drawable/bg</item>
</style>
```

2、再把这个样式设置给启动的Activity，
```
<activity
    android:name=".activity.SplashActivity"
    android:screenOrientation="portrait"
    android:theme="@style/AppTheme.Launcher"/>
```
3、再在Activity的onCreate()方法，把Activity设置回原来的主题，这样就可以防止在启动时有黑白屏的尴尬
```
@Override
    protected void onCreate(Bundle savedInstanceState) {
        //替换为原来的主题，在onCreate之前调用
        setTheme(R.style.AppTheme);
        super.onCreate(savedInstanceState);
    }
```
