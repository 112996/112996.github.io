---
layout: post
title: "Android之混淆"
date: 2018-12-4 20:17
description: "Android之混淆"
tag: Kotlin
---
一、基本概念
混淆，字面上来说就是把项目中的包名、类名、方法名和变量名等进行更改，用来迷惑别人。但是实际上混淆包含了代码压缩、优化、校验等过程，把混淆称作ProGuard更加合适。

ProGuard就是Java对Class文件进行“混淆”的工具。

<div>
  <img src = "/images/Android_ProGuard/proguard.png" width="675" height="168">
</div>
这个图就好比设计模式中的责任链模式。下面我们解释以下名词：

1、shrink（压缩）：ProGuard会递归地确定哪些类和类成员被使用，而其他的则被丢弃。

2、optimize（优化）：ProGuard会进一步分析和优化方法。比如一些无用的参数会被丢弃，一些方法会做内联。

3、obfuscate（混淆）：这个过程就是进行重命名，把原来包含注释意义的类名、方法名等进行无意义的重命名。

4、preverify（预校验）：这个步骤就是将预校验信息添加到类中。

这四个步骤都是可选的，当然Android中一般情况下我们都会保留前三个步骤，而忽略preverify过程，这样可以加快混淆速度。

二、Android中的ProGuard

Android中默认集成了ProGuard工具，在sdk目录的/tools/proguard中。混淆开启方法：
```
android{
  。。。。。
  //AS自动生成
  buildTypes{
    release{
      //混淆开关
      minifyEnabled true
      //移除无用的resource文件
      shrinkResources true
      //proguard-android.txt文件表示默认的混淆规则，proguard-rules.pro表示自定义的混淆规则（文件名和后缀名可更改）
      proguardFiles getDefaultFile('proguard-android.txt'),'proguard*rules.pro'
    }
  }
}
```
默认的proguard-android.txt文件在sdk目录下/tools/proguard中。这个目录下还有proguard-android-optimize.txt文件。我们自定义的proguard-rules.pro文件中有部分基础混淆规则就是来自这个optimize.txt文件。

三、混淆语法

混淆语法都可以在ProGuard官网上面找到，这里简单介绍一些常用的语法规则：

1、保留类和类成员
<div>
  <img src = "/images/Android_ProGuard/keepclass.png" width="748" height="138">
</div>
2、类成员中的一些符号
<div>
  <img src = "/images/Android_ProGuard/symbol.png" width = "680" height = "180">
</div>
3、一些常用通配符
<div>
  <img src = "/images/Android_ProGuard/wildcard.png" width = "" height ="">
</div>
其他有需要的话可以自行查询。

四、自定义混淆的模板
```
# 优化算法，一般不用修改，来自Google
-optimizations !code/simplification/arithmetic,!code/simplification/cast,!field/*,!class/merging/*
# 代码混淆压缩比，在0~7之间，默认为5，一般不做修改
-optimizationpasses 5
# 混合时不使用大小写混合，混合后的类名为小写
-dontusemixedcaseclassnames
# 不去忽略非公共库的类
-dontskipnonpubliclibraryclasses
# 优化时允许访问并修改有修饰符的类和类的成员
-allowaccessmodification
# 项目混淆后产生映射文件
-verbose
# 不做预校验
-dontpreverify

# 保留Annotation不混淆
-keepattributes *Annotation*,InnerClasses
# 避免混淆泛型
-keepattributes Signature
# 抛出异常时保留代码行号
-keepattributes SourceFile,LineNumberTable

# 保留四大组件，自定义的Application等
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Appliction
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.preference.Preference
-keep public class * extends android.view.View
-keep public class com.google.vending.licensing.ILicensingService
-keep public class com.android.vending.licensing.ILicensingService

# 保留native方法
-keepclasseswithmembernames class * {
    native <methods>;
}

# 保留在Activity中的方法参数是view的方法，
# 这样以来我们在layout中写的onClick就不会被影响
-keepclassmembers class * extends android.app.Activity{
    public void *(android.view.View);
}

# 保留自定义View
-keepclassmembers public class * extends android.view.View {
   void set*(***);
   *** get*();
   public <init>(android.content.Context);
   public <init>(android.content.Context, android.util.AttributeSet);
   public <init>(android.content.Context, android.util.AttributeSet, int);
}

# 保留枚举
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# 保留Parcelable序列化对象
-keepclassmembers class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator CREATOR;
}

# 保留R文件中的成员
-keepclassmembers class **.R$* {
    public static <fields>;
}

# 保留Serializable序列化的类不被混淆
-keepnames class * implements java.io.Serializable
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    !static !transient <fields>;
    !private <fields>;
    !private <methods>;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}

# -------------------------------------------------------------------------------
# webView处理，项目中没有使用到webView忽略即可
-keepclassmembers class fqcn.of.javascript.interface.for.webview {
    public *;
}
-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.WebView, java.lang.String, android.graphics.Bitmap);
    public boolean *(android.webkit.WebView, java.lang.String);
}
-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.webView, jav.lang.String);
}

# --------------------------保留JS接口-------------------------------------------

# --------------------------保留反射类-------------------------------------------

# --------------------------保留实体类-------------------------------------------

# --------------------------第三方库的混淆规则-----------------------------------

```
上面贴出的都是一些常用的混淆规则，其中部分规则都在Android默认混淆文件中声明过了。所以具体的自定义混淆规则还是得根据项目进行细化。

五、混淆产物

混淆后一般会产生以下几个文件：

1、dump.txt: 描述APK文件中所有类的内部结构。

2、mapping.txt文件：提供混淆前后类、方法、类成员等的对照表。
3、seeds.txt：列出没有被混淆的类和成员。

4、usage.txt：列出被移除的代码。

当遇到混淆问题时，通常需要通过查看mapping.txt文件来分析原因。
如果在进行Crash追踪中遇到可困难，可以使用sdk目录下/tools/proguard/bin中的progurardgui.bat可视化工具进行混淆定位。
<div>
  <img src = "/images/Android_ProGuard/proguardgui_bat.png" width = "963" height = "653">
</div>

六、混淆引发过的问题

场景一

问题描述：业务方集成了我们的一个SDK，SDK的一个页面中会显示Banner图，而Banner图所用的是SDK中自定义的ViewPager组件，结果业务方那边无法显示Banner图。

原因分析：通过反编译apk，比对mapping.txt文件，发现Banner图的自定义Adapter中关键方法都被清除了。查看混淆文件，发现只keep了support v4的ViewPager，并没有keep住PagerAdapter。而SDK中又是通过compileOnly（provided）方式依赖v4包的，导致自定义的Adapter在工程编译的时候找不到被引用的关系，然后就被混淆给优化掉了。

经验：针对compileOnly（provided）依赖的库，一定要注意相关类的混淆，特别是SDK开发者，因为混淆导致的问题一般较难定位。

场景二

问题描述：业务方又集成了我们的一个SDK（没错，是“又”），SDK有一个换肤功能，业务方可以通过构造特定格式的资源文件来实现皮肤替换。但在业务通过公司编译工具编译后的apk中无法实现换肤功能。

原因分析：机智的我很快想到了混淆问题，于是让业务方加上保留所有R文件的规则，结果还是失败。于是开始了长时间的打log看log过程，还是确定问题出在资源名的混淆上。最后发现业务方虽然在本地编译环境中keep了资源名，但在公司的编译环境中还是对资源进行了混淆，导致换肤失败。

经验：对于和资源相关的功能，一定要注意R文件的混淆。不要轻易相信业务，因为他们经常有你预料不到的操作。

七、总结

1、混淆其实不是新鲜的话题，但是对于Java和Android开发者来说是必不可少的，还是要保存敬畏之心。

2、混淆的好处不言而喻，增加了反编译的难度，优化了代码，但是仅仅是难度增加而已，对于有心之人还是能从反编译中找到蛛丝马迹。所以，对于敏感代码和数据使用更安全的保护措施是非常必要的。这就是我们后面要提到的加固了。
