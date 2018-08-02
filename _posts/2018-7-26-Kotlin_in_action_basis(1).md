---
layout: post
title: "Kotlin 学习笔记（二）"
date: 2018-7-26 10:24
description: "Kotlin基础"
tag: Kotlin
---
声明函数、变量、类、枚举类型和属性

1、  简单的无返回值的打印“Hello World！”程序：
```
fun main(args:Array<String>){
  println("Hello World!")
}
```

(1)、fun 是一个关键词，用来声明一个函数；

(2)、参数类型写在参数名后面；

(3)、函数可以声明在文件的顶层；

(4)、数组只是一个类，Kotlin没有特殊语法来声明数组类型；

(5)、末行不加分号；

2、有返回值的函数：
```
fun max(a:Int, b:Int):Int{
  return if(a>b)a else b
}
println(max(1,2))
```
(1)、fun关键字声明一个名为max的函数，括号里面为参数列表，返回值类型以冒号为分隔符跟在参数列表后面

(2)、if是一个带结果值的表达式(不是声明)，类似于Java中的三元操作符：（a>b）? a:b

(3)、Java中，控制结构都属于声明，而在Kotlin中循环以外的大多控制结构都为表达式，可以作为另一表达式的一部分

3、表达式主体：
```
fun max(a:Int, b: Int): Int = if(a>b) a else b
```
若函数有大括号，则这个函数有一个块主体，如果直接返回一个表达式，则说明这个函数有一个表达式主体。

4、变量声明：
```
val name = "Jack"
val age = 20
```
以关键词val开始定义不可变变量，忽略类型，编译器会自行推断变量的类型，或者也可以在变量名后面添加类型（显示声明指定类型）：
```
val age: Int = 20
```
以关键词var定义可变变量：
```
var age: Int = 20
```
但是默认在Kotlin中我们应该尽量使用val声明所有的变量，仅在必要的时候将val改为var。一个val便令必须在定义块执行费时被有且只能初始化一次，可以根据情况用不同的值初始化变量：
```
val message: String
if(canPerformOperation()){
  message = "Success"
  //...
}
else{
  message = "Failed"
}
```
当然尽管一个val引用本身是不可变的，但是它指向的对象对象却可以是可变的：
```
val languages = arraylistOf("Java") //声明一个不可变的引用
languages.add("Kotlin") //引用指向可变的对象
```
虽然var声明的是可变的变量，但是一开始就定义了变量的类型，只能是值的改变，类型不能变：
```
var num = 20
num = "hello"  //错误:类型匹配错误
```

6、声明枚举类（enum class）：
```
enum class Color(
  val r:Int, val g: Int, val b: Int               //声明枚举常量的属性
  ){
    RED(255, 0, 0), ORANGE(255, 255, 0),          //当每个常量被创建时指定属性值
    YELLOW(255, 255, 0), GREEN(0, 255, 0), BLUE(0, 0, 255),
    INDIGO(75, 0, 130), VIOLET(238, 130, 238);    //这里必须是分号，这是Kotlin语法中唯一一处需要使用分号的地方
    fun rgb() = (r * 256 + g) * 256 + b           //在枚举类中定义了一个方法
  }
  println(Color.BLUE.rgb())
```
1、Kotlin中关键字较多，enum（或称为关键软词），

2、如果在枚举类当中定义了任何方法，则必须使用 “;” 来将枚举常量列表从函数定义中间那个分隔开。
