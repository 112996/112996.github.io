---
layout: post
title: "Android逆向之反编译APK和安装包漏洞解析"
date: 2018-10-24 14：30
description: "反编译APK"
tag: Android
---
最近在学习如何对一个APK进行反编译，当然首要前提就是的得到应用的APK，这次我想要学习的是布卡漫画，那么我在酷安上面获取布卡漫画的APK：
<div>
  <img src = "/images/android_reverse/apk_get.png" width="720" height="270" />
</div>
成功获取到APK后，我们就开始正式反编译了。这里我们用到的工具是：
APKTool：将Android应用程序反编译成Smali文件。
Dex2Jar-2.0：将Android应用程序反编译为Java文件。
JD_GUI_windows-1.4.0:读取Java源文件。
Drozer-2.3.4：Android应用程序评估框架。

第一步：直接解压APK文件（我将apk的后缀名直接改为.zip，在解压打开）：
<div>
  <img src = "/images/android_reverse/apk_unzip.png" width="720" height="270" />
</div>
在这里我们可以看到经过解压后得到的文件里面有了很多东西，下面我们先补充一下大多APK都有的文件的内容里面都是些什么：
assets：配置文件，放置一些本地资源，例如本地html，可以通过使用AssetManager检索。
kotlin：这个文件里面包括了注解、反射等。
lib：存放特定平台使用的编译后的.so文件，每一个子文件夹对应一个特特定的平台，例如armeabi，armeabi-v7a,armeabi-v8a,x86,x86_64,和mips等。
res：存放没有被编译到的resourse.arsc文件中的资源文件。
META-NIF:包含CERT,RSA,CERT.SF,MANIFEST.MF,存放的签名信息，用于保证系统的安全性和APK文件的完整性。
resources.arsc:编译后的二进制资源文件，存放res/values目录下的所有XML内容，打包工具对XML内容进行提取，编译为二进制，斌进行分类，其中，xml内容包含了不同语言的string和styles，同时还包含没有被直接打包至resource.arsc文件的其他文件的路径，例如布局文件和图片文件。
classes.dex/classes2.dex：编译后的字节码文件，能够被Dalvik/ART理解。

第二步：得到AndroidManifest.XML文件
保证上面所用软件都安装好的情况下，在apk所在文件路径下打开命令行窗口，输入命令：
<div>
  <img src = "/images/android_reverse/apktool_apk.png" width="616" height="136" />
</div>

那么我们就得到了一个新的文件：
<div>
  <img src = "/images/android_reverse/apktool_finish.png" width="754" height="279" />
</div>

下面打开新得到的文件：
<div>
  <img src = "/images/android_reverse/buka.png" width="658" height="304" />
</div>

我使用Notepad++打开AndroidManifest.xml文件：
<div>
  <img src = "/images/android_reverse/androidmanifest.png" width="1000" height="567" />
</div>
这样我们就可以看到布卡漫画这个应用的清单文件了，可以看到里面的那个是启动activity，这个应用所申请的权限等等，当然我们对一个apk的安全性测试也包括权限是否得当。

第三步：查看源代码
首先我们需要将第一步解压后得到的classes.dex文件、classes2.dex文件通过工具Dex2Jar-2.0来转换为java文文件：
1、将classes.dex文件、classes2.dex复制到Dex2Jar-2.0文件目录下。
2、在Dex2Jar-2.0文件目录下打开命令行窗口。
3、通过如下命令（这里两个文件就操作两次）：
<div>
  <img src = "/images/android_reverse/dex2jar.png" width="720" height="270" />
</div>
4、3操作后我们看到Dex2Jar-2.0文件目录下多了classes-dex2jar.jar、classes2-dex2jar.jar两个文件
5、通过工具JD_GUI_windows-1.4.0打开classes-dex2jar.jar、classes2-dex2jar.jar文件
<div>
  <img src = "/images/android_reverse/jd-gui_java_code.png" width="749" height="204" />
</div>
当然我们可以看到这里的代码是经过Android Studio混淆打包的，很多都是以简易字符替换。

第四步：对APK进行安全检测
我们通过第二步得到的AndroidManifest.xml查看到应用的入口activity是cn.ibuka.manga.ui.ActivityStartup：
<div>
  <img src = "/images/android_reverse/enter_activity.png" width="1076" height="175" />
</div>
 同时我们看到 application里面 android:allowBackup="true"，这个属性设置为true将会有很大的问题，外部可以通过adb backup 或者abd restore 进行备份。所以安全起见，allowBackup应设置为false。

 下面我们通过Drozer-2.3.4对APK进行基本的框架评估：
 1、将apk安装到手机或者模拟器上。
 2、将Drozer里面的agent.apk安装到手机或者模拟器上。
 3、通过adb进入模拟器的root权限下：
 <div>
   <img src = "/images/android_reverse/adb_shell.png" width="628" height="253" />
 </div>
4、打开终端上的agent.apk，并开启：
<div>
  <img src = "/images/android_reverse/agent.png" width="333" height="581" />
</div>
5、我们去到Drozer目录下，打开命令行窗口，通过命令adb forward tcp:31415 tcp:31415，./drozer.bat console connect去连接终端：
<div>
  <img src = "/images/android_reverse/enter_drozer.png" width="742" height="406" />
</div>
6、我们先查看布卡漫画APK的攻击面，使用的命令是 run app.apckage.attacksurface 包名
<div>
  <img src = "/images/android_reverse/attacksurface.png" width="720" height="220" />
</div>
