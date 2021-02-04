---
layout:     post
title:      Scala
subtitle:   Scala学习笔记
date:       2020-07-08
author:     owl city
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - Scala
    - 大数据
---

> - Create Date: 2020-07-08
> - Update Date: 2020-07-08

## 快学scala

#### 基础一
- 反编译.scala代码为java代码
```sh
scalac Person.scala
# 运行scala代码并查看耗时
scala -Dscala.time Person
# 反编译为java代码
javap Person
```

- println 要点:
```Java
val word = "world"
println(s"Hello $word")
```

- Any, AnyVal, AnyRef
 - Any: Any是abstract类，它是Scala类继承结构中最底层的。所有运行环境中的Scala类都是直接或间接继承自Any这个类，它就是其它语言（.Net，Java等）中的Object。
 - AnyVal: 所有值类型的基类, 它描述的是值, 而不是代表一个对象。它包括9个AnyVal子类型：Double, Float, Long, Int, Char, Short, Byte, Unit, Boolean
 - AnyRef: 是所有引用类型的基类。除了值类型，所有类型都继承自AnyRef 。

- 异常捕获搭配模式匹配
```java
// 当不需要使用捕获的异常对象时，可以使用_来代替变量名
// finally不论有没有捕获到异常都会被执行
val url = new URL(".....")
try {
	process(url)
} catch {
	case _: MalformedURLException => print(s"Bad URL: $url")
	case ex: IOException => ex.printStackTrace()
} finally {
	url.close()
}
```

- 对数组进行排序
`newArr.sorted`: 不会改变原始版本
`newArr.sortWith(_>_)`: 降序排序

- 多参数列表（柯里化）： 方法可以定义多个参数列表，当使用较少的参数列表调用多参数列表的方式时，会产生一个新的函数，该函数接受剩余的参数列表作为其参数，这被称为柯里化。

- 案例列 case class: 
	- 在实例化案例类的时候，不需要使用new关键字，因为案例类有一个默认的apply方法来负责对象的创建。且在创建包含参数的案例类时，这些参数是public val（即不能被再次赋值）
	- 案例类在比较的时候是按值比较而非按引用比较


#### 基础二（类）
- 类的getter与setter
在scala中方法名表示为`age`和`age_`，调用时直接`val myAge = person.age; person.age = 10`

- 类的私有字段
类的私有字段不会生成getter和setter，用private[this]来声明字段。`private[this] var vallue = 0`

- 类构造器
scala可以有一个主构造器和任意多个辅助构造器
每个类都有主构造器，并不是以`this`方法定义的，而是与类定义在一块。如
```java
class Person(val name: String, age:Int) {
	// 括号中的内容就是主构造器的参数
}
```
辅助构造器的名称为`this`，每一个辅助构造器都必须以一个对一个先前已经定义的其他辅助构造器或者主构造器的调用开始.
```java
class Person{
	private var name = ""
	private var age = 0

	def this(name: String) {
		this() //调用主构造器
		this.name = name
	}

	def this(name: String, age: Int) {
		this(name)
		this.age = age
	}
}
```

#### scala泛型
- 类型边界限定分为上边界和下边界来对类进行限制

> 上边界：泛型的类型必须是某种类型或者某种类型的**子类**，语法为`<:`
下边界：泛型的类型必须是某种类型或者某种类型的**父类**，语法为`>:`


- 使用`<%`可以对某种类型进行转换，多上下边界的补充，即为使用implicit进行隐式转换，语法为`<%`

- 协变： 使用注释`+A`，可以使一个泛型类的类型参数`A`成为协变。对于某些类`class List[+A]`，使A成为协变意味着对于两种类型`A`和`B`，如果`A`是`B`的子类型，那么`List[A]`就是`List[B]`的子类型

- 逆变： 与协变相反，如果`A`是`B`的子类型，那么`Writer[B]`是`Writer[A]`的子类型

#### scala内置函数
- `flodLeft`:
```Java
// 方法源码
def flodLeft[B](z: B)(op: (B, A) => B): B = {
	var result = z
	this.seq foreach (x => result = op(result, x))
	result
}
// 使用举例
val stgList = List(1, 2, 3)
val result = stgList.flodLeft(0)(_+_)
// 返回result = 6
```
