---
layout: post
title: "Android——smallestWidth限定符适配方案学习"
date: 2018-10-29 14：43
description: "smallestWidth限定符适配方案"
tag: Android
---
首先，我们了解一下几个简单的公式：

px * density = dp

DPI / 160 = density、

px / (DPI / 160) = dp

px ：  传统计算机语言中描述的像素，在Android中代表绝对像素。（比如手机分辨率 320 * 480 表示宽有320像素，高有480像素）

DPI :  “屏幕密度”，每英寸所打印的点数，也就是每一英寸的屏幕所包含的像素数。

density ：  就单词本身意思而言为“密度”，但是在Android中，我们指的是通过代码"content.getResources().getDisplayMetrics().density()"获取的“density”值。而通过该方法获取到的值实际上等价于 “DPI / 160 ”的结果

如果屏幕信息为：1920 * 1080，480dpi，那么这个设备的最小宽度值为360dp。

values-sw360dp文件夹下，dimens.xml的生成原理

因素1： 最小宽度基准值

因素2： 项目需要适配哪些最小宽度（需要生成多少个values-se<N>dp文件夹）

最小宽度基准值:

需要把设备的屏幕宽度分为多少份。假设把项目的最小宽度基准值定为360，那么就会在dimens.xml文件中生成1到360的dimens引用，例如values-sw360dp文件夹中的dimens.xml：
```
<resource>
    <dimen name = "dp_1">1dp</dimen>
         ......
    <dimen name = "dp_360">360dp</dimen>
</resource>

```
当我们将最小宽度基准值定为360，values-sw400dp中的dimens.xml：
```
<resource>
    <dimen name = "dp_1">1.1111dp</dimen>
         ......
    <dimen name = "dp_360">400.0000dp</dimen>
</resource>

```

总结dimens.xml生成原理：

1、先确定最小宽度基准值；

2、将每个values-sw<N>dp中的dimens.xml文件都分配与最小宽度基准值相同的份数；

3、根据公式 屏幕最小宽度 / 份数（最小宽度基准值）求出每份占多少dp。

所以在份数不变的情况下，根据屏幕宽度在不同设备上动态调整每份占的dp值，就完成适配（前提：项目提供当前设备所对应的values-sw<N>dp）

当没有对用的values-sw<N>dp时，设备会寻找与之最近的文件，这时就会存在误差，这个误差的大小就和第二个因素密切相关。

需要适配哪些最小宽度：

    如果想适配的最小宽度有：320dp、360dp、400dp、411dp、480dp等。那么项目就需要生成values-sw320dp、values-sw360dp、values-sw400dp、values-sw411dp、values-sw480dp这几个资源文件。

    如前面所说，当没有为某个设备提供对应的values-sw<N>dp文件夹时，若相近的values-sw<N>dp与期望的values-sw<N>dp差距太大，那么我们得到的就会是一个失败的适配。

    当然也不是说values-sw<N>dp越多越好，那样apk体积也会很大。

注意：可能某些设备的宽高一致（xxx * xxx） ，但DPI不同，由于smallestWidth方案并没有如今日头条适配方案一样去自行修改density，那么系统就会默认使用 DPI / 160 求出的density，而density又会影响到dp和px的换算。所以DPI的变化会影响到smallestWidth限定符适配。

若生成的values-sw<N>dp与设备的最小宽度差别不大，那么误差也是在可以接受范围内的，适配方案即是可行的。

smallestWidth限定符适配优点：

1、稳定，基本不会出现意外。

2、没有性能耗损。

3、适配范围可自由控制，不影响其他三方库。

4、学习成本低。

smallestWidth限定符适配缺点：

1、引用dimen方式，日常修改维护麻烦。

2、侵入性高，若项目想切换其他屏幕适配方案，因为每个Layout文件中都有大量dimens引用，修改起来工作量巨大。

3、无法覆盖所有机型（想要覆盖的越多，就会生成越多的资源文件，那么apk体积就会越大）。在未覆盖的机型上会出现误差，需要在适配效果和占用空间上做抉择。

4、若需要使用SP，则另外生成一系列的dimens文件。

5、不能自动支持横竖屏切换时的适配（若想自动适配，则需要使用values-sw<N>dp或者屏幕方向限定符再生成一套资源文件，再次增加apk的体积）。

6、只能以宽度作为基准进行适配（不支持高度为基准的适配）。方案本身就是“最小宽度限定符适配方案”
