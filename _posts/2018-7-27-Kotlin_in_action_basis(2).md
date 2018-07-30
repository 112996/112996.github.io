---
layout: post
title: "Kotlin 学习笔记（三）"
date: 2018-7-26 10:24
description: "Kotlin基础"
tag: Kotlin
---

结构控制

一、智能类型转换（求和示例）:
```
interface Expr                                        //定义一个接口，不声明任何方法
class Num(val value: Int) : Expr                      //带一个属性、值且实现了Expr接口的简单值对象
class Sum(val left: Expr, val right: Expr) : Expr     //求和操作的参数可以是任意的Expr；Num对象或者其他的Sum对象
fun eval(e: Expr): Int{                                
    if(e is Num){
        val n = e as Num                               //显示的Num类型转换是多余的，通过as关键词表达
        return n.value
    }
    if(e is Sum){
        return eval(e.right) + eval(e.left)           //变量e是智能类型转换
    }
    throw IllegalArgumentException("Unknown expression")
}
fun main(args: Array<String>){
    println(eval(Sum(Sum(Num(1),Num(2)),Num(4))))      //打印输出结果
}
```
1、通过is检查变量是否为一个指定的类型，与Java中的instanceof相似;当检查到了变量的特定类型，后续编译器会自行执行类型转换。

2、当且仅当一个变量在is检查时候不再改变时，智能类型转换才会发生作用。

3、一个指定类型的显示转换通过as关键词表达：val n = e as Num

下面我们将上面的代码通过when重写：
```
fun eval(e: Expr): Int =
  when (e){
    is Num ->                       //用于检查参数类型的when分支
      e.value                       //智能类型转换
    is Sum ->                       //检查参数类型的when分支
      eval(e.right) + eval(e.left)  //智能类型转换
    else ->
      throw IllegalArgumentException("Umknown expression")
  }
```
与上一个代码不同的是，when表达式不强制要求检查值的等价性。当分支逻辑复杂时，可以使用表达式块作为分支主体。

二、while循环

语法和Java相同，
```
while(condition){   
  //...
}
do{
  //...
}while(condition)  
```
三、遍历：范围和数列
Kotlin中没有Java for循环那样初始化一个变量，在循环的每一步更新它的值，当值到达边界时退出循环，未来替代这样的循环，Kotlin提出了ranges。范围本质上就是两个值之间的一个间隔，通常是成员：一个起始值个一个结束值，操作符“..”：
```
val oneToTen = 1..10
```
注意：这里的范围值为全闭

例：
```
fun fizzBuzz(i: Int) = when{
    i % 15 ==0 -> "FizzBuzz"	//被15整除
    i % 3 ==0 -> "Fizz"			//被3整除
    i % 5 == 0 -> "Buzz"		//被3整除
    else -> "$i"				//否则返回原始值
}
fun main(args: Array<String>){
    for(i in 1..100){			//遍历整数返回1..100
        println(fizzBuzz(i))
    }
}
```

通过索引遍历：
```
val list = arrayListOf("10","11","1001")
fun main(arges:Array<String>){
    for((index,element) in list.withIndex()){  //通过索引遍历集合
    println("$index:$element")
  }
}
```

使用in检查（“in”在范围内，“!in”不在范围内）:
```
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'
fun main(arges: Array<String>){
    println(isLetter('q'))             // 返回true
    println(isNotDigit('x'))           //返回true
}

注意：“in”和“!in”在when表达式里面同样可以用
