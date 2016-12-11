---
title: Scala学习笔记(一)
date: 2016-11-04 13:03:57
categories:
- Scala
tags:
- Scala
---

## 初识Scala
Scala是纯面对对象语言:**every value is an object and every operation is a method call**
For example, when you say `1 + 2` in Scala, you are actually invoking a method named `+` defined in class Int.

![](http://www.scala-lang.org/resources/img/smooth-spiral.png)

<!-- more -->

## 交互式Scala

### 使用 `var` 定义变量

```scala
scala> var msg1 = "Hello"
msg1: String = Hello
```

### 交互式环境下的回退

多行代码情况下，scala左边会有竖杠表示代码仍在继续，如果发现代码有问题，可以按下两次回车开始新的命令

```scala
scala> var msg1 =
     |
     |
You typed two blank lines.  Starting a new command.
```

### 使用`def`定义函数

```scala
scala> def max(x: Int, y: Int): Int = {
		 if (x > y) x
		 else y
       }
max: (x: Int, y: Int)Int
```

如果函数没有返回值，并且函数体可以简写为一行，那么可以如下简写:

```scala
scala> def max(x: Int, y: Int) = if (x > y) x else y
max: (x: Int, y: Int)Int
```

没有返回值的函数,也就是函数返回`void`，在Scala中会被成为`Unit`类型

```scala
scala> def greet() = println("Hello, world!")
greet: ()Unit
```

使用`:q`或者`:quit`退出交互式环境

```scala
scala> :q

C:\WINDOWS\system32>
```

## 初识循环

Scala不仅可以在交互环境运行，也可以以脚本运行,我们创建`test.scala`文件，并写入:

```scala
args.foreach(arg => println(arg))
```

`args`为用户输入的参数列表,`foreach`语法的使用也很好理解

当我们执行:

```powershell
$ scala test.scala Concise is nice
```

将会看到一下输出:

```
Concise
is
nice
```

上述方式中,Scala解释器能够根据`args`的元素基础类型猜出`arg`的类型，更明确表示类型的方式如下:

```scala
args.foreach((arg: String) => println(arg))
```

当然也有更简单的写法:

```scala
args.foreach(println)
```

也可以使用`for`循环:

```scala
for (arg <- args)
  println(arg)
```

## 初识`Array`

`Array`是可变的,使用`update`方法对元素进行修改

```scala
val numNames = Array("zero", "one", "two")
numNames.update(0, "Hello")
```

## 初识`List`

与java中的`List`不同，Scala中的`List`是不可改变的

Scala使用`:::`进行`List`连接

```scala
val oneTwo = List(1, 2)
val threeFour = List(3, 4)
val oneTwoThreeFour = oneTwo ::: threeFour
println(oneTwo + " and " + threeFour + " were not mutated.")
println("Thus, " + oneTwoThreeFour + " is a new list.")
```

将会输出:

```scala
List(1, 2) and List(3, 4) were not mutated.
Thus, List(1, 2, 3, 4) is a new list.
```

`::`操作符在`List`前插入元素:

```scala
val twoThree = List(2, 3)
val oneTwoThree = 1 :: twoThree
println(oneTwoThree)
```

输出:

```scala
List(1, 2, 3)
```

**If a method is used in operator notation, such as `a * b`, the method is invoked on the left operand, as in `a.*(b)`—unless the method name ends in a colon. If the method name ends in a colon, the method is invoked on the right operand. Therefore, in `1 :: twoThree`, the `::` method is invoked on `twoThree`, passing in `1`, like this: `twoThree.::(1)`.  **

空`List`用`Nil`表示

## 初识元组

虽然元组和`List`一样，都是不可改变的，但是元组可以包含不同类型的元素:

```scala
val pair = (99, "Luftballons")
println(pair._1)
println(pair._2)
```

The actual type of a tuple depends on the number of elements it contains and the types of those elements. Thus, the type of `(99, "Luftballons")` is `Tuple2[Int, String]`. The type of`('u', 'r', "the", 1, 4, "me")` is `Tuple6[Char, Char, String, Int, Int, String]` 

**注意元组中元素的索引方式**

## 初识`Set`

`Set`有不可变的也有可变的，不导入包时，默认是不可变的`Set`

```scala
scala> Set(1,2,3)
res0: scala.collection.immutable.Set[Int] = Set(1, 2, 3)
```

To add a new element to a set, you call `+` on the set, passing in the new element. On both mutable and immutable sets, the `+` method will create and return a new set with the element added. Although mutable sets offer an actual `+=` method, immutable sets do not. 

![](http://i.imgur.com/hKFF7H4.png)

可变`Set`和不可变`Set`虽然叫同一个名字，但是属于不同的包

```scala
import scala.collection.mutable

val movieSet = mutable.Set("Hitch", "Poltergeist")
movieSet += "Shrek"
println(movieSet)
```

```scala
import scala.collection.immutable.HashSet

val hashSet = HashSet("Tomatoes", "Chilies")
println(hashSet + "Coriander")
```

## 初识`Map`

`Map`与`Set`类似,有可变和不可变的,不导入包时，默认是不可变的`Map`

```scala
import scala.collection.mutable

val treasureMap = mutable.Map[Int, String]()
treasureMap += (1 -> "Go to island.")
treasureMap += (2 -> "Find big X on ground.")
treasureMap += (3 -> "Dig.")
// val romanNumeral = Map(
//	 1 -> "I", 2 -> "II", 3 -> "III", 4 -> "IV", 5 -> "V"
// )
println(treasureMap(2))
```

## 初识文件读取

```scala
import scala.io.Source

if (args.length > 0) {
	for (line <- Source.fromFile(args(0)).getLines())
		println(line.length + " " + line)
	}
else
	Console.err.println("Please enter filename")
```

## 重点

- Traits in Scala are like interfaces in Java, but they can also have method implementations and even fields.
- Scala中的数组使用`()`索引，而不是`[]`

## 参考文档

- 《Programming in Scala, 3rd Edition》
