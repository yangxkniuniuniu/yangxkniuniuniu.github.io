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

## Scala语法学习要点记录
- println 要点:
```Java
val word = "world"
println(s"Hello $word")
```

- Any, AnyVal, AnyRef
 - Any: Any是abstract类，它是Scala类继承结构中最底层的。所有运行环境中的Scala类都是直接或间接继承自Any这个类，它就是其它语言（.Net，Java等）中的Object。
 - AnyVal: 所有值类型的基类, 它描述的是值, 而不是代表一个对象。它包括9个AnyVal子类型：Double, Float, Long, Int, Char, Short, Byte, Unit, Boolean
 - AnyRef: 是所有引用类型的基类。除了值类型，所有类型都继承自AnyRef 。

- 多参数列表（柯里化）： 方法可以定义多个参数列表，当使用较少的参数列表调用多参数列表的方式时，会产生一个新的函数，该函数接受剩余的参数列表作为其参数，这被称为柯里化。

- 案例列 case class: 
	- 在实例化案例累的时候，不需要使用new关键字，因为案例类有一个默认的apply方法来负责对象的创建。且在创建包含参数的案例类时，这些参数是public val（即不能被再次赋值）
	- 案例类在比较的时候是按值比较而非按引用比较

- 协变： 使用注释`+A`，可以使一个泛型类的类型参数`A`成为协变。对于某些类`class List[+A]`，使A成为协变意味着对于两种类型`A`和`B`，如果`A`是`B`的子类型，那么`List[A]`就是`List[B]`的子类型

- 逆变： 与协变相反，如果`A`是`B`的子类型，那么`Writer[B]`是`Writer[A]`的子类型
