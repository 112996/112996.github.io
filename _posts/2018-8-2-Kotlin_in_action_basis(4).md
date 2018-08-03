---
layout: post
title: "Kotlin 学习笔记（五）"
date: 2018-8-2 9：45
description: "类、对象和接口"
tag: Kotlin
---
一、类和接口

1、定义接口，使用关键词interface，声明一个只带一个click()抽象方法的接口:
```
interface a{
  fun click()
}
```
实现这个接口：
```
class Button: a{
  override fun click() = println("I was clicked")
}
```
Kotlin在类名后面使用“:”代替Java中的extends和implements关键词,override修饰符用来标记覆盖来自父类或接口的方法和属性

一个接口可以有一个默认实现方法,下面我们添加一个有默认实现的方法来改变a接口：
```
interface a{
  fun click()                            //常规的方法声明
  fun showOff() = println("I'm a !")     //带有默认实现的方法
}
```

下面我们假定另一个接口也定义了showOff方法并为它添加以下实现：
```
interface b{
  fun setFocus(b: Boolean) =
    println("I ${if (b) "got" else "loat"}focus.")
  fun showOff() = println("I'm b !")
}
```
如果需要在类中同时实现这两个接口，两个接口都包含带有实现的showOff方法，若你没有显示的实现showOff方法，会出现编译错误：类必须覆盖public open fun showOff(),同时它集成了多个showOff实现。

所以我们必须提供自己的实现：
```
class Button : a, b{
  override fun click() = println("I was clicked")
  override fun showOff(){                         //如果继承的成员函数有一个以上的实现，就必须提供一个显示的实现
    super<a>.showOff()                            //“super”限制尖括号中指定的超类名
    soper<b>.showOff()
  }
}
```
Button类实现了两个接口，通过调用从超类继承的两个实现，使得showOff()生效了，为了调用一个继承的实现，使用super关键词，把基类名放在尖括号中。如果只需要一个继承的实现：
```
override fun showOff() = super<a>.showOff()
```
验证实例：
```
fun main(args: Array<String>){
  val btn = Button()
  btn.showOff()
  btn.setFocus(true)
  btn.click()
}
```
结果：
<div>
  <img src = "/images/Kotlin_learn/interface_verify.png" width = "834" height = "469"/>
</div>

2、覆盖基类中定义的成员（修饰符:open、 abstract、 final[默认]）

Kotlin中的类和方法默认是final的，若想允许一个类创建子类，需要使用open、修饰符标记这个类，另外需要添加open修饰符到每个可以被覆盖的属性或者方法。
```
open class RichButton: a{   //这个类是开放的，其他的子类可以继承它
  fun disable(){}           //这个类是不可修改的（final），不可以在子类中覆盖它
    open fun animate(){}    //这个函数是开放的，可以在子类中覆盖
    override fun click(){}   //这个函数覆盖了一个开放的函数同时它也是开放的
}
```
当覆盖了一个接口或者基类的成员，覆盖的方法将会是默认的open，若想禁止子类覆盖，则可以将覆盖后的成员标记为final：
```
final override fun click(){}   //被final修饰后变成不开放的
```

同样，我们可以把类声明为abstract：
```
abstract class  Animated{     //这个类是抽的，不能创建它的实例
  abstract fun animate()      //这个函数是抽象的，并没有实现同时必须在子类中被覆盖
  open fun stopAnimating(){}  //抽象类中的非抽象方法默认是final的，但是这里可以标记为open
  fun animateTwice(){}        //这里就是抽象类中非抽象方法默认的final
}
```
类中的访问修饰符的含义：
<div>
  <img src = "/images/Kotlin_learn/access_modifier.png" width= "786" height = "216"/>
</div>
注意：在接口中，不能使用final或者abstract，接口中的成员总是为open属性。

3、可见性修饰符：默认是公开的（public）可见性修饰符有助于访问你的代码中的声明，通过限制类实现细节的可见性，可以确保你百变实现细节但不会有破坏依赖代码的风险。
Kotlin可见性修饰符：
<div>
  <img src = "/images/Kotlin_learn/visibility_modifier.png" width = "785" height = "192"/>
</div>

4、内部类和嵌套类（默认为嵌套）

我们声明实现了Serializable接口的State接口，View接口声明了可以被用于保存一个视图状态的getCurrentState和restoreState方法：
```
interface State；Serializable
interface View{
  fun getCurrentState(): State
  fun restoreState(state: State){}
}
```
在Button类中定义一个保存按钮状态的类：
```
class Button: View{
  override fun getCurrentState(): State = ButtonState()
  override fun restoreState(state: State){}
  class ButtonState: State{}                              //对等与Java中静态嵌套
}
```
Kotlin中没有显示修饰符的嵌套类与Java中的静态嵌套相同。

Kotlin访问外部类实例语法：
```
class Outer{
  inner class Inner{
    fun getOuterReference(): Oyter = this@Outer
  }
}

```

5、密封类（定义受限的类层级）修饰符sealed

未使用sealed：
```
interface Expr
class Num(val value: Int): Expr
class Sum(val left: Expr, val right: Expr): Expr
fun eval(e: Expr): Int =
  when(e){
    is Expr.Num ->e.value
    is Expr.Sum ->eval(e.right) + eval(e.left)
    else ->                                                   //必须检查else分支
      throw IllegalArgumentException("Unknown expression")
  }
```
使用sealed修饰的类：
```
sealed class Expr{                                    //把一个基类标记为密封类
  class Num(val value: Int): Expr()
  class Sum(val left: Expr, val right: Expr(): Expr)  //列出所有可能的子类作为嵌套类
}
fun eval(e: Expr): Int =
  when(e){                                            //“when”表达式覆盖了所有的可能的情况，所以不再需要“else”
    is Expr.Num ->e.value
    is Expr.Sum ->eval(e.right) + eval(e.left)
  }
```
密封类不能有定义在外部的继承者：
<div>
  <img src = "/images/Kotlin_learn/sealed_class.png" width = "808" height="166"/>
</div>
