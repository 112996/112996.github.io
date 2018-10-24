---
layout: post
title: "Kotlin 学习笔记（八）"
date: 2018-10-24 10：01
description: "注解学习"
tag: Kotlin
---
一、定义注解

Kotlin使用annotation class 关键字（好比使用emnu class 关键字定义枚举类），相当于定义一个注解接口，这个接口继承Kotlin.Anootation,定义注解非常简单，Kotlin不允许为注解定义注解体，注解后面不需要花括号。
定义一个简单的注解：
```
annotatiaon class AAA
```
定义之后可以在程序的任何地方使用，使用语法类似于使用public、finally修饰符。通常把注解放在所有修饰符之前。
```
//使用@ AAA修饰类定义
@AAA class Myclass{
  ...
}
```
注解可用于修饰任何程序元素，包括类、接口、属性、方法等。
```
@AAA class Myclass{
  @AAA var name: String = "Tom"
  @AAA fun info(){
    ...
  }
  ...
}
```

二、注解的属性和构造器

注解还可以带属性，由于注解没有注解体，因此注解的属性只能在注解声明部分指定。相当于在注解的构造器中指定注解的属性。且注解的属性只能为只读属性。
定义一个带属性和构造器的注解
```
annotatiaon class MyTag(val name: String, val age: Int)
//或者在定义注解时为其设置初始值
//annotatiaon class MyTag(val name: String = "Tina", val age: Int = 18)
```
注意：注解的属性不能使用可空类型（不能在类型后添加"?"），因为JVM本身不允许使用null作为注解的属性值。
使用上面定义的MyTag注解：
```
class Item{
  @MyTag(name = "Tina", age = 18)
  fun info(){
      ...
  }
  ...
}
```
由此我们得出注解分为两类：
标记注解：未定义属性。这种注解仅利用自身的存在与否来提供信息，如前面的@AAA。
元数据注解：包含属性的注解。可以提供更多的配置信息，如@MyTag。

注意：与Java注解相同，如果属性名为value，则指定属性值时可省略属性名：
```
annotatiaon class HHH(val value: String)
```
```
class B{
  @HHH("nihao")
  fun info(){
    ...
  }
  ...
}
```
使用vararg修饰需要指定多个属性值的属性，且在使用时不需要指定属性名：
```
annotatiaon class @FFF(vararg val infos: String)
```
```
class V{
  @FFF("C", "Java", "Kotlin")]
  fun info(){
    ...
  }
  ...
}
```

当一个注解为另一个注解的属性时，那么在使用注解时则不需要“@”前缀：
```
annotatiaon class MyTag(avl value: Stirng)
```
```
annotatiaon class ShowTag(val message: Stirng, val target: MyTag)
```
```
@ShowTag("message 属性值", target = MyTag("nihao"))
```

三、元注解
@Retention：只能修饰注解定义，用于指定被修饰的注解可以保留多长时间。包含一个AnnotationRetention类型的value属性。
@Target：只能修饰注解定义，用于指定被修饰的注解能修饰哪些程序单元。包含一个AnnotationRetention类型的allowedTargets属性。
@MustBeDocumented：修饰的注解将被文档工具提取到API文档中。
@Repeatable：修饰可重复注解。定义一个@FKTag注解：
```
@Retention(AnnotationRetention.SOURCE)
@Target(AnnotationRetention.CLASS)
@Repeatable
annotatiaon class FKTag(val name: String = "你好"， val age: Int)
```
```
@FKTag(age = 5)
@FKTag(name = "Kotlin", age = 9) //FKTag可重复
class FKTag
```
