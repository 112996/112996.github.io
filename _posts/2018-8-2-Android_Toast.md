---
layout: post
title: "Android中使用Kotlin语言写Toast"
date: 2018-8-2 16：28
description: "不同种弹出Toast信息的方法"
tag: Kotlin in Android
---
在学习了一周的Kotlin基础之后，今天想要在Android Studio里面写一个弹出Toast信息的小案例。
方法一：
使用最普通的Toast方法，需要在弹的点击监听事件里面定义Toast方法：
```
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        btn.setOnClickListener{
            toast("")
        }
    }
    fun toast(message: String){   
        Toast.makeText(this,message,Toast.LENGTH_SHORT).show() //这里语句跟我们平时写Toast一致
    }
}
```
方法二：
它实际是Anko的一部分，是Kotlin的延伸，需要在build.gradle里面添加依赖：org.jetbrains.anko:anko-common:0.8.3
<div>
  <img src = "/images/Kotlin_learn/add_jetbrains_anko.png" width = "818" height = "212"/>
</div>

然后就直接可以在activity里面import
<div>
  <img src ="/images/Kotlin_learn/import_anko.png" width = "660" height = "240"/>
</div>
就可以直接使用：
```
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        btn.setOnClickListener{
            toast("你好!")  
            toast(R.stirng.message)
            longToast("wow!")          

        }
    }
```
