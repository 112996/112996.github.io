---
layout: post
title: "Kotlin 学习笔记（四）"
date: 2018-7-31 11：45
description: "函数的声明和调用"
tag: Kotlin
---

一、对集合、字符串和常规表达式有效的函数

创建集合：setOf函数
```
val set = setOf(1,3,5)
```
创建列表：listOf函数
```
val list = listOf(1,3,5)
```
创建映射：mapOf函数
```
val map = mapOf(1 to "one", 3 to "three", 5 to "five")
```
注意：to不是特殊结构，是一个常规函数
例（可以简单获得列表或者集合的最后一个数或者最大值）：
```
fun main(args:Array<String>){
    val strings = listOf("你","好","啊")
    println(strings.last())             //字符串最后一位
    val numbers = setOf(1,3,5)
    println(numbers.max())              //最大一位数
}
```
 打印输出添加前缀、后缀、分隔符的列表数字：
```
fun <T> joinToString(
	collection:Collection<T>,
    separator:String,         //分隔符
    prefix:String,            //前缀
    postfix:String            //后缀
):String{
    val result = StringBuilder(prefix)
    for((index, element) in collection. withIndex()){
        if(index > 0) result. append(separator)       //第一个元素前不添加分隔符
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
fun main(args: Array<String>){
    val list = listOf(1,2,3)
	println(joinToString(list,";","(",")"))
}
```

现在我们将上面的代码写成最符合Kotlin风格的代码：
```
fun <T>Collection<T>.joinToString(                  //用Collection<T>声明一个扩展函数
  separator: String = ",",                          //为参数声明默认值
  prefix: String = "",
  postfix: String = ""
  ): String{
    val result = StringBuilder(prefix)
    for((index, element) in this. withIndex()){     //"this"指向接收器对象：T类型的集合
      if(index > 0) result. append(separator)
      result.append(element)
    }
    result.append(postfix)
    return result.toString()
  }
  fun main(args: Array<String>){
    val list = arrayListOf(1,2,3)       
    println(list.joinToString(" "))                  //像类的成员那样调用
  }
```

在上面的代码中，为一个集合声明了一个扩展，同时为所有的参数提供了默认值。这样我们就可以直接像类的成员调用。
扩展的静态类型意味着静态函数不能被子类覆盖，覆盖对于扩展函数不起作用，Kotlin以静态解析代码。

扩展属性(能通过属性语法来访问API的扩展类,尽管被叫做属性，但是却不能拥有任何状态)：

二、使用命名参数，默认参数和中缀调用语法

可变参数（Varargs）：可以接受任意个参数的函数

使用元组（pairs）：中缀调用和析构声明
当我们创建映射时，使用mapOf函数：
```
val map = mapOf(1 to "one", 2 to "two")
```
这里的to就是中缀调用（infix call），在一个中缀调用中，方法的名字被放在目标对象名和参数之间，而且没有其他的分隔符，下面两个调用就是等价的：
```
to("one")  //以常规的方式来调用
to "one"  //使用中缀标记来调用函数
```
中缀调用可以用于带一个参数的常规方法和扩展函数，为了使得函数可以使用中缀标记来调用，需要用infix修饰符标记：
```
infix fun Any.to(other: Any) = Pair(this, other)  //to函数返回一个Pair实例。Pair是Kotlin一个标准库类，表示一对元素
val (number, name) = 1 to "one"                    //为两个变量直接分配一对元素
```

三、通过扩展函数和属性来适配Java库到Kotlin中
拆分字符串：
```
fun main(args: Array<String>){
  println("12.345-6.A".split("\\.|-".toRegex()))  //通过扩展函数toRegex来显示的创建一个正则表达式
  prinln("12.345-6.A".split(".","-"))             //Kotlin中其他重载split的扩展函数接收任意数量的分隔符作为普通的字符串
}
```
以上量两种方式均可以得到想要的被拆分的字符串.

将一个文件的完整路径名解析成它的组件：一个路径、一个文件名、一个扩展名:
1、通过扩展的方式：
```
fun parsePath(path: String){
  val directory = path.substringBeforeLast("/")
  val fullName = path. substringAfterLast("/")
  val fileName = fullName.substringBeforeLast(".")
  val extension = fullName.substringAfterLast(".")
  println("Dir:$directory, name: $fileName, ext: $extension")
}
fun main(args: Array<String>){
  parsePath("/Users/yole/Kotlin-book/chapter.adoc")
}
```
2、通过正则表达式的方式:
```
fun parsePathRegex(path: String){
  val regex = """(.+)/(.+)\.(.+)""".toRegex()
  val matchResult = regex.matchEntire(path)
  if(matchResult != null){
    val (directory, fileName, extension) = matchResult.destructured
    println("Dir: $directory, name: $fileName, ext: $extension")
  }
}
fun main(args: Array<String>){
  parsePathRegex("Users/yole/Kotlin-book/chapter.adoc")
}
```
正则表达式被写在一个三字引号字符串(作用：避免转义字符b )里面，不需要转义任何字符，包括反斜杠。通过斜杠和点号把路径分割成三组
通过substringBeforeLast、substringAfterLast把文件路径拆分成一个路径、一个文件名、一个文件扩展名
四、使用顶层代码和本地函数跟属性来结构化代码
```
class User(val id: Int, val name: String, val address: String)
fun User.validateBeforeSave(){
  fun validate(value: String, fileName: String){
    if(value.isEmpty()){
      throw IllegalArgumentException(
        "Can't save user $id: empty $fieldName")      //可以直接访问User对象的属性
    }
  }
  validate(name, "Name")
  validate(address, "Address")
}
fun saveUser(user: User){
  user.validateBeforeSave()       //调用扩展函数
}
```
函数主要处理一个单独的对象并不需要访问它的私有数据，可以访问它的成员却没有额外的限制.扩展函数也可以声明为本地函数，所以下一步可以把User.validateBeforeSave()放在saveUser()中作为本地函数。但是多重嵌套的本地函数往往是难以阅读的，原则上不推荐使用超过一个层级以上的嵌套。

总结：
1、Kotlin没有定义它自己的集合类，而是使用更加丰富的API来扩展Java的集合类。

2、为函数参数定义默认值在极大程度上减少定义重载函数的需求，同时命名参数语法使得有许多参数的函数变得更加可读。

3、函数和属性可以直接在文件中声明，而不是仅仅作为类的成员， 这为代码结构提供更多灵活性。

4、扩展函数和属性允许扩展任意类的API，包括定义在扩展中的类，而无需修改它的源代码以及运行时消耗。

5、中缀调用为单一参数调用类操作符函数提供了清晰的语法。

6、三重引号字符串提供了一种清晰的方式来编写在Java中需要一大堆无聊的转移和字符拼接操作的表达式。

7、本地函数有助于清晰的组织代码并消除重复。
