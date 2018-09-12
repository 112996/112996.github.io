---
layout: post
title: "Kotlin 学习笔记（六）"
date: 2018-8-3 10：12
description: "类、对象和接口"
tag: Kotlin
---
二、重要属性和构造函数

1、初始化类：主构造器和初始化器
首先，我们来声明一个简单的类：
```
class User(val name: String)
```
通常情况下，类中所有的声明都会在闭合大括号的内部，而上面的例子中仅仅是一个小括号，在这里圆括号里面的代码叫做主构造函数，它有两个目的：指定构造函数参数，同时定义由这些参数初始化的属性。
当我们编写的同时实现同样功能的最直接的代码：
```
class User constructor(_name: String){  //带有一个参数的主构造器
  val name: Stirng
  init{                                 //初始化块
    name = _name
  }
}
```
上面的例子中有两个Kotlin关键字：construstor和init。constructor开启了一个主构造器或次构造器的声明；init引入了一个initializer块。这样的块包含了当这个类通过主构造函数创建时执行的初始化代码。那我们为什么需要初始化块呢？是因为主构造器有一个限制语法，不能包含初始化代码。
那我们用另一种方法来声明一个同样的类：
```
class User(_name: String){  //带有一个参数的主构造器
  val name = _name          //属性被参数初始化
}
```
前面的两个例子在类主体中使用val关键词声明的属性，如果属性被初始化为对应的构造函数参数，那么代码就可以简化为在参数面前添加val关键字，替代类中的属性定义：
```
class User(val name: String)    //“val”意味着对应的属性是为构造函数参数生成的
```
同样，我们可以为构造函数参数声明默认值：
```
class User(val name: Stirng, val isSubscribed: Boolean = true)  //为构造函数参数提供默认值
```
创建一个类的实例，可以直接调用构造函数：
```
fun main(args: String){
  val alice = User("Alice")       //为isSubscribed参数使用默认值“true ”
  println(alice.isSubscribed)
}
```
