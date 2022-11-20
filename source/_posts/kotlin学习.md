---
title: kotlin基础
date: 2021-10-06 21:51:30
tags:
categories: Kotlin
---

## 1.Kotlin 是什么？ 

Kotlin 是一种在 Java 虚拟机上运行的静态类型编程语言，被称之为 Android 世界的Swift，由 JetBrains 设计开发并开源。

在Google I/O 2017中，Google 宣布 Kotlin 成为 Android 官方开发语言。



## 2.为什么选择 Kotlin？

- 简洁: 大大减少样板代码的数量。
- 安全: 避免空指针异常等整个类的错误。
- 互操作性: 充分利用 JVM、Android 和浏览器的现有库。
- 工具友好: 可用任何 Java IDE 或者使用命令行构建。

<!--more -->

## 3.我的第一个 Kotlin 程序

```kotlin
fun main(args: Array<String>) {
    println("hello word!")
}
```

> Kotlin 程序文件以 **.kt** 结尾，如：hello.kt 、app.kt。

## 4.基础语法

- 函数定义

格式：

```kotlin
fun 方法名(参数A : 类型A, 参数B : 类型B): 返回值类型(可以为空) {
   函数体
}
```

如：

```kotlin
fun sum(a: Int, b: Int): Int {   // Int 参数，返回值 Int
    return a + b
}
```

```kotlin
fun main() {
   var result = add(3,5)
    println(result)
}

//fun add(x: Int, y: Int): Int {
//  return x+ y
//}
fun add(x: Int, y: Int): Int = x+ y
```

- 可变长参数

```kotlin
fun vars(vararg v:Int){
    for(vt in v){
        print(vt)
    }
}
fun main(args: Array<String>) {
    vars(1,2,3,4,5)  // 输出12345
}
```

- 默认参数和具名参数

```kotlin
fun calThePerimeterOfCircle(pi: Float = Pi, r: Float): Float{
    return 2* pi * r
}
fun main() {
    println("圆的周长为：${calThePerimeterOfCircle(r = 2.0f)}")
}
```

- 函数表达式

```kotlin
fun main() {
   var i = {x: Int, y: Int -> x+ y}
    println(i(3,5))

    // 输入类型是两个int ，返回值是int，输入参数为x,y 表达式为x+y
    var j:(Int,Int) -> Int = {x,y -> x+y}
    println(i(4,5))
}
```

- 条件控制 if

```kotlin
fun returnBigValue(a:Int,b:Int):Int{
    return if (a>b) a else b
}
fun main(args: Array<String>) {
    var a= 3
    var b = 5
    println("${a}和${b}中较大的数是${returnBigValue(1,2)}")
}
```

- When 表达式

```kotlin
fun gradeStudent(score:Int){
    when(score){
        10 -> println("优秀！")
        1 -> println("差劲！")
        else -> println("努力努力！")
    }
}
fun main() {
    println(gradeStudent(9))
}
```

- list

```kotlin
fun main() {
    var items = listOf<String>("语文","数学","英语");
    for (i in items){
        println("科目有：${i}")
    }
    for ((i,index) in items.withIndex()){
        println("科目为：${i}，序号为${index}")
    }
}
```

- map

```kotlin
fun main() {
   var map = TreeMap<String,String>()
    map["好"] = "good"
    map["坏"] = "bad"
    println(map["好"])
}
```

- 字符串和数字的转换

```kotlin
fun main() {
    var a = "string"
    var b = 1
    println("字符串转数字：${a.toString()}")
    println("数字转字符串：${b.toInt()}")
}
```

- 控制台输入变量

```kotlin
fun main() {
    println("请输入第一个数字：")
    var str1 = readLine()
    println("请输入第二个数字：")
    var str2 = readLine()

    var num1 = str1!!.toInt()
    var num2 = str2!!.toInt()
    println("${num1}+${num2}=${num1+num2}")
}
```

- 处理异常-> try...catch()

```kotlin
fun main() {
    println("请输入第一个数字：")
    var str1 = readLine()
    println("请输入第二个数字：")
    var str2 = readLine()
    try{
        var num1 = str1!!.toInt()
        var num2 = str2!!.toInt()
        println("${num1}+${num2}=${num1+num2}")
    }catch (e: Exception){
        println("输入的数据有误！")
    }
}
```

- 字符串模板

  $ 表示一个变量名或者变量值

  $varName 表示变量值

  ${varName.fun()} 表示变量的方法返回值:

```kotlin
fun main(args: Array<String>) {
    var a = 1

// 模板中的简单名称：
    val s1 = "a is $a"

    a = 2
// 模板中的任意表达式：
    val s2 = "${s1.replace("is", "was")}, but now is $a"
    println(s2)
}
```

- 空值处理

  Kotlin的空安全设计对于声明可为空的参数，在使用时要进行空判断处理，有两种处理方式。

  - 字段后加!!像Java一样抛出空异常

  - 字段后加?可不做处理返回值为 null或配合?:做空判断处理

```kotlin
//类型后面加?表示可为空
var age: String? = "23" 
//抛出空指针异常
val ages = age!!.toInt()
//不做处理返回 null
val ages1 = age?.toInt()
//age为空返回-1
val ages2 = age?.toInt() ?: -1
```

## 5.类

### 5.1 类的创建

Kotlin 中没有 new 关键字

```kotlin
class Rectangular{
    var width:Int = 10
    var height:Int = 5
}

fun main() {
    var rectangular = Rectangular()
    println(rectangular.height)
}
```

### 5.2 枚举类

```kotlin
enum class Color{
    RED,BLACK,BLUE,GREEN,WHITE
}
```

### 6. Kotlin 继承

Kotlin 中所有类都继承该 Any 类，它是所有类的超类，对于没有超类型声明的类是默认超类：

```
class Example // 从 Any 隐式继承
```


Any 默认提供了三个函数：

```
equals()

hashCode()

toString()
```

注意：Any 不是 java.lang.Object。

如果一个类要被继承，可以使用 open 关键字进行修饰。

```
open class Base(p: Int)           // 定义基类

class Derived(p: Int) : Base(p)
```

# 7.Kotlin 接口

Kotlin 接口与 Java 8 类似，使用 interface 关键字定义接口，允许方法有默认实现：

```kotlin
interface MyInterface {
    fun bar()    // 未实现
    fun foo() {  //已实现
      // 可选的方法体
      println("foo")
    }
}
```

```kotlin
class Child : MyInterface1,MyInterface2 {
    override fun bar1() {
        // 方法体
    }
   override fun bar2() {
        // 方法体
    }
}
```

