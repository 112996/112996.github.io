---
layout: post
title: "Kotlin 学习笔记（七）"
date: 2018-10-22 14：28
description: "异常处理"
tag: Kotlin
---
Koltlin的异常处理机制可以让程序具有极好的错性，让程序更加健壮。当程序出现以意外情形时，系统会自动生成一个Exception对象来通知程序，从而实现将“业务功能实现代码”和“错误处理代码”分离，提供更好的可读性。

一、使用try...catch捕获异常
```
try {
  //业务实现代码
}catch(e: Exception){
  //异常处理代码
}
finally{
  //可选的finally块
}
```
与java的异常处理流程相同，可包含1个try块、0-N个catch块、0-1个finally块，但后两块至少出现其中之一。

注意：不管程序代码块是否处于try块中，甚至包括catch块中的代码，只要执行该代码块时出现异常，系统总会自动生成一个异常对象。如果程序没有为这段代码定义任何catch块，则运行时环境无法找到处理该异常的catch块，程序就在此退出。

有些时候，程序在try块中打开了一些物理资源（例如连接数据库、网络连接和磁盘文件等），这些物理资源都必须在finally块中显示回收。不管try块中的代码是否出现异常，也不管哪一个catch块被执行，甚至在try或catch块中执行了return语句，finally块都会被执行。
```
import java.io.*
fun main(args: Array<String>){
    var fis: FileInputStream ?= null
    try{
        fis = FileInputStream("a.txt")
    }catch(ioe: IOException){
        println(ioe.message)
        return //虽然return语句会强制语法结束，但是一定会先执行finally块
    }finally{
        fis?.close()
        println("执行finally块里的资源回收！")
    }
}
```

通常情况下，不在finally块里面使用return或throw语句，一旦使用会导致try块、catch块中的return、throw失效。
```
fun main(args: Array<String>){
    var a = test()
    println(a)
}
fun test(): Boolean{
    try{
        return true
    }finally{
        return false //使得try块里面的return true失效，将打印出false结果
    }
}
```
注意：当程序执行try、catch块遇到return、throw语句时，这两句都会导致该方法立即结束，但是系统执行这两条语句并不会结束该方法，而是去找寻该异常处理流程是否包含finally块，若没有，程序立即执行return、throw语句，方法中止；若有，系统立即执行finally块——只有当finally块执行完成后，系统才会再次跳回来执行try块、catch块的return或throw语句；如果finally块中也使用了导致方法中止的语句，那么系统将不再跳回执行try、catch块。

二、异常体的继承体系
```
fun main(args: Array<String>){
    try{
        var a = Integer.parseInt(args[0])
        var b = Integer.parseInt(args[1])
        val c = a / b
        println("您输入的两个数的想加结果是：${c}")
    }catch(ie: IndexOutOfBoundsException){
        println("数组越界：运行程序输入的参数个数不够")
    }catch(ne:NumberFormatException){
        println("数字格式异常：程序只能接收整数格式参数")
    }catch(ae: ArithmeticException){
        println("算术异常")
    }catch(e: Exception){
        println("未知错误")
    }
}
```
注意：在进行捕获异常时不仅应该把Exception类对应的catch块放在最后，而且应该把用来捕获所有父类异常的catch块排在用来捕获子类异常的catch块后面（先处理小异常，再处理大异常）

三、访问异常信息

所有的异常对象都包含了以下常用属性和方法：
message：该属性返回该异常的详细描述字符串。
stackTrace:该属性返回该异常的跟踪栈信息。
printStackTrace():将该异常的跟踪栈信息输出到标准错误输出。
printStackTrace(printStream s):将该异常的跟踪栈信息输出到指定输出流。

```
import java.io.*
fun main(args: Array<String>){
    var fis: FileInputStream ?= null
    try{
        fis = FileInputStream("a.txt")
    }catch(ioe: IOException){
        println(ioe.message)  //打印出异常对象的详细信息
        ioe.printStackTrace()//打印该异常的跟踪栈信息
}
```

四、使用throw抛出异常

与java类似，Kotlin也允许程序使用throw语句自行抛出异常。语法为：throw ExceptionInstance
```
fun main(args: Array<String>){
    throwRuntime(3)
}
fun throwRuntime(a: Int){
    if(a > 0){
        throw RuntimeException("a的值大于0，不符合要求")
    }
}
```
