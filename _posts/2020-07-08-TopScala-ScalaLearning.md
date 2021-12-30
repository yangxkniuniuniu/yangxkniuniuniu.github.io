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
> - Update Date: 2021-02-08

## scala部署
#### scala项目创建、编译、打包及运行
1. 使用idea基于sbt创建scala项目, 然后在src/main/scala目录下创建测试程序：
```java
object TestProject {
  def main(args: Array[String]): Unit = {
    print("hello Scala")
  }
}
```

2. 在项目根目录下执行`sbt compile`
![编译](https://tva1.sinaimg.cn/large/008i3skNgy1gvmy83gs96j613507ignp02.jpg)

3. 运行`sbt run`
![运行](https://tva1.sinaimg.cn/large/008i3skNgy1gvmy99fnnxj60nq03paat02.jpg)

4. 打包`sbt package`
![打包](https://tva1.sinaimg.cn/large/008i3skNgy1gvmy9r59c4j60md02cwex02.jpg)

5. 最后在target目录下可以看到打包好的jar包，可以使用`jar tvf target/scala-2.12/user_base_log_2.12-0.1.jar`查看目录结构

6. 运行程序`scala target/scala-2.12/user_base_log_2.12-0.1.jar`，或者直接用java来执行,需要引入scala的一个类库：`java -classpath /usr/local/Cellar/scala@2.12/2.12.12/libexec/lib/scala-library.jar:target/scala-2.12/user_base_log_2.12-0.1.jar TestProject`

#### 打包所有依赖
1. 增加`project/plugins.sbt`文件,文件内容
```
addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.14.7")
```

2. 项目根目录下执行`sbt clean compile assembly`

3. 如果报文件重复的异常，需要对重复文件进行处理，在`build.sbt`文件中增加如下内容：
```
assemblyMergeStrategy in assembly := {
  case PathList("LZ4BlockInputStream.class") => MergeStrategy.discard
  case PathList("META-INF", xs @ _*) => MergeStrategy.discard
  case x => MergeStrategy.first
}
```

#### flink-sql-client
例：通过flink-sql-client消费kafka中的json数据
1. 在`/usr/local/Cellar/apache-flink/1.12.0/libexec`下新建kafka-lib的目录用于存放jar包
2. 放入jar包：flink-connector-kafka_2.12-1.12.0.jar、flink-json-1.12.0.jar、kafka-clients-2.4.1.jar，注意版本
3. 启动cli: `./bin/sql-client.sh embedded -l kafka-lib`
4. 建表、查询，`select count(1) from xxx`

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


#### 基础二（类、对象、包、继承、特质）
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

- 包对象,每个包都可以有一个包对象,需要在父包中定义,且名称和子包一样
```java
package com.host.impatient
package object people {
	val defaultName = "Mark"
}
package people {
	class Person {
		var name = defaultName //可以直接从包对象拿到常量
	}
}
```

- 包引入
```java
// 引入包中的几个成员
import java.awt.{Color, Font}
// 重命名
import java.util.{HashMap => JavaHashMap}
// 隐藏包成员
import java.util.{HashMap => _, _}
```

- 特质构造顺序：1. 首先调用超类的构造器 2.特质构造器在超类构造器之后、类构造器之前执行 3.特质由左到右被构造 4.在每个特质中，父特质先被构造 5. 如果多个特质共有一个父特质，而父特质已经被构造，则该父特质不会被再次构造 6.所有特质构造完毕，子类被构造

- 特质的自身类型
```java
// 此特质只能被混入指定类型的子类
trait LoggerException extends ConsoleLogger {
	this: Exception =>
		def log() {log(getMessage())}
}
```

#### 基础三（正则、高阶函数）

- 读取文件
scala并没有提供读取二进制文件和写文件的方法，需要通过java类库
```java
import scala.io.Source
val source = Sorce.fromFile("aa.txt", "UTF-8")
val lines = source.getLines()
for (line <- lines) println(line)
source.close()
```

- 与shell交互
```java
import scala.sys.process._
"ls -al .".!
```

- 在scala中使用正则表达式
```java
import scala.util.matching.Regex
val numPattern = "[0-9]+".r
val matchedArr = numPatten.findAllIn("99 human").toArray
```

- scala闭包
闭包是指在变量不再处于作用域内的时候被调用，上示例
```java
def mulBy(factor: Double) = (x: Double) => factor * x
val test1 = mubBy(1)
val test2 = mubBy(2)
// mubBy函数在两次被调用的时候，factor均会被赋值而产生新的函数，在新的函数体内被调用，像mulBy这样的函数称为闭包(closure)
```

- scala柯里化
是指将原来接收两个参数的函数编程新的接收一个参数的函数的过程，新的函数返回一个以原有的第二个参数作为参数的函数

- 控制抽象
在scala中，可以将一系列语句归组成不带参数也没有返回值的函数，示例：
```java
def until(condition: => Boolean)(block: => Unit) {
	if (!condition) {
		block
		until(condition)(block)
	}
}
// 下面是使用until的示例
var x = 10
until (x == 0) {
	x -= 1
	println(x)
}
```

#### 基础四（集合、模式匹配、案例类）
- Mutable集合继承关系图
![mutable_map](https://tva1.sinaimg.cn/large/008eGmZEgy1gnyvvmk0nbj30qa0j6gma.jpg)

- 集合用来添加和移除元素的操作符
![集合操作符](https://tva1.sinaimg.cn/large/008eGmZEgy1gnyw89fvw3j30ta0uyneu.jpg)

- 模式匹配
```java
// 类型模式匹配
obj match {
	case x: Int => x
	case s: String => s
	case _: Bigint => Int.MaxValue
	case _ => 0
}
// 数组模式匹配
arr match {
	case Array(0) => "0"
	case Array(x, y) => s"$x, $y"
	case Array(0, _*) => s"${_.min}"
	case _ => "else"
}
// 如果要将匹配到的_*可边长参数绑定到变量，可以使用@来表示
case Array(x, rest @ _*) => rest.min
```

- 变量声明中的模式
```java
val arr = Array(0, 1, 2, 3, 4)
val Array(x, y, rest @ _*) = arr
// 返回
val x: Int = 1
val y: Int = 2
val rest: Seq[Int] = ArraySeq(3, 4, 5)
```

- 偏函数
被包在花括号内的一组case语句是一个偏函数--一个并非对所有输入值都有定义的函数。它是PartitalFuntion[A, B]类的一个示例（A是参数类型，B是返回类型）。该类有两个方法: apply方法从匹配到的模式计算函数值，而isDefinedAt方法在输入至少匹配其中一个模式时返回true
例如：
```java
val f:PartitionFunction[Char, Int] = { caxse '+' => 1; case '-' => 1 }
f('-') // 调用f.apply('-') 返回-1
f.isDefinedAt('0') // false
f('0') // 抛出MatchError
```

#### 基础五（类型参数）
- 泛型： 类型边界限定分为上边界和下边界来对类进行限制
上边界：泛型的类型必须是某种类型或者某种类型的**子类**，语法为`<:`
下边界：泛型的类型必须是某种类型或者某种类型的**父类**，语法为`>:`

- 视图界定(未来被弃用，可用类型约束替换，见下文)：
`T <& V`要求必须存在一个从T到V的隐式转换
```java
// 使用<%意味着T可以被隐式转换为Comparable[T]
class Pair[T <% Comparable[T]]
```

- 上下文界定: `T : M`其中M是另一个泛型类，它要求必须存在一个类型为M[T]的隐式值
如:
例1：
```java
class Pair(T : Ordering)
```
上述定义要求必须存在一个类型为`Ordering[T]`的隐式值，该隐式值可以被用在该类的方法中。当声明一个使用隐式值的方法时，需要添加一个隐式参数，如下：
```java
class Pair[T : Ordring](val first: T, val second: T) {
	def smaller(implicit ord: Ordring[T]) = if (ord.compare(first, second) < 0 first else second)
}
```
例2：
```java
object MyTest {
	def main(args: Array[String]): Unit = {
		implicit val c = new Comparator[Int] {
			override def compare(o1: Int, o2: Int): Int = o1 - o2
		}
		// 如果上下文中存在Comparator[Int]这样的隐式值，下面的语句就可以正常执行
		println(cusMax2(10, 4))
	}
	def cusMax1[T](a: T, b: T)(implicit cp: Comparator[T]): T = {
		if (cp.compare(a, b) > 0) a else b
	}
	// 上面的方法也可以写成
	def cusMax2[T : Comparator](a: T, b: T): T = {
		val cp = implicitly[Comparator[T]]
		if (cp.compare(a, b) > 0) a else b
	}
}
```

- ClassTag上下文界定
ClassTag[T]保存着在运行时被JVM擦除的类型T的信息，可以在运行时获得传递的类型参数的信息
> jvm类型擦除
Java中的泛型基本上都是在编译器这个层次来实现的。在生成的Java字节码中是不包含泛型中的类型信息的。使用泛型的时候加上的类型参数，会在编译器在编译的时候去掉，保留原始类型。这个过程就称为类型擦除。原始类型（raw type）就是擦除去了泛型信息，最后在字节码中的类型变量的真正类型

```java
import scala.reflect._
def mkArray[T : ClassTag](ele: T*) = Array[T](ele: _*)
```

#### code练习
- 如何使用reduceLeft得到数组中的最大元素?
```java
val aa = List(10, 23, 12, 34, 7, 54, 3, 23, 77, 12, 4, 5)
aa.reduceLeft((a, b) => if (a>b) a else b) // 返回77
```

- 编写函数largest(fun: (Int)=> Int, inputs : Seq[Int])，输出在给定 输入序列中给定函数的最大值。 举例来说， largest(x => 10 * x - x * x,
1 to 10)应该返回25。 不得使用循环或递归。
```java
def largest(fun: (Int) => Int, inputs: Seq[Int]): Int = {
	val maxValue = inputs.reduceLeft((a, b) => if (fun(a) > fun(b)) a else b)
	fun(maxValue)
 }
```

#### scala泛型


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
