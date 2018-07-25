---
layout: post
title: "Kotlin学习笔记（一）"
date: 2018-7-24 17：13
description: "在Android Studio里面使用Kotlin语言"
tag: Android、Kotlin
---
什么是Kotlin？
Kotlin是JVM和Android的实用编程语言，结合OO和功能特性，专注于互操作性，安全性，清晰度和工具支持，作为通用语言，Kotlin可以在java工作的地方工作：服务器端应用程序，移动应用程序（Android），桌面应用程序。
关键重点在于混合Java+Kotlin项目的互操作性和无缝支持，采用更容易，从而减少样板代码和更多的类型安全性。

 首先的是在Android Studio里面搭建Kotlin的环境：

 （一）创建全新的Kotlin项目：
 在Android Studio里面创建新的项目，然后勾选下方的 Include Kotlin support

 <div>
  <img src = "/images/Kotlin_learn/create_Kotlin_pro.png" width = "370" height = "250">
 </div>


创建成功之后，MainActivity的后缀名由.java变化成了.kt，然后里面的语法我在代码中做了注释：

<div>
  <img src = "/images/Kotlin_learn/MainActivity.png" width = "370" height = "250">
</div>


那么gradle里面会自动添加以下依赖：

<div>
  <img src= "/images/Kotlin_learn/gradle.jpg" width="370" height = "250">
</div>


（二）在java代码下转换为Kotlin模式

选择Tools -> Kotlin -> Configure Kotlin Plugin Updates, 更新完插件后重新启动Android Studio

<div>
  <img src = "/iamges/Kotlin_learn/mode_change.png" width="300" height = "370">
</div>

然后选中想要转换的.java文件，选择Code -> Convert Java File to Kotlin File 后变成.kt文件：

<div>
  <img src = "/images/Kotlin_learn/java_to_kotlin.png" width="250" height = "370">
</div>


第一个Kotlin程序 —— Hello World：
```
import java.text.SimpleDateFormat
import java.util.*

fun main(args: Array<String>){
  println("Hello World!")
  println(SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(Date()))
}

```
这里我使用的编译环境是云平台：https://try.kotlinlang.org/

<div>
  <img src = "/images/Kotlin_learn/hello_world.png" width = "310" height = "370">

</div>

下面我们进行一个稍微带逻辑的Kotlin函数代码：
```
data class Person(val name :String, val age: Int? = null) //创建一个数据类Person
fun main(args: Array<String>){
  val persons = listOf(Person("Alice"), Person("Bob", age = 20))
  val oldest = persons.maxBy{it.age?：0}
  println ("The oldest is : $oldest")
}
```

运行效果：

<div>
  <img src = "/images/Kotlin_learn/first_function.png" width="370" height="250">
</div>

现在来分解这几句代码：
1、首先创建一个带有两个参数的数据类Person，第一个参数是name，类型是不可变量 val String ，第二个参数是可为空的Int类型（Int?）age，意思是这里年龄可以为空值。

2、fun main ()是顶层函数，相当于Java代码当中的static函数。

3、val persons = listOf(Person("Alice"), Person("Bob", age = 20)) 是命名声明，放入数据:Alice，年龄未知；Bob，年龄为20。

4、val oldest = persons.maxBy{it.age?：0}，这个是lambda表达式：elvis操作符。

Lambda 表达式的完整语法形式，即函数类型的字面值如下：

val sum = { x: Int, y: Int -> x + y }

lambda 表达式总是被大括号括着， 完整语法形式的参数声明放在括号内，并有可选的类型标注， 函数体跟在一个 -> 符号之后。如果推断出的该 lambda 的返回类型不是 Unit，那么该 lambda 主体中的最后一个（或可能是单个）表达式会视为返回值。

而里面的it.age? : 0 表达的意思是 ？：左边表达式非空elvis操作符返回左边的结果，否则返回0.
