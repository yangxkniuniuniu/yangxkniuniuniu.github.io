---
layout:     post
title:      Python
subtitle:   Python学习笔记
date:       2020-07-08
author:     owl city
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - Python
    - 数据分析
---

> - Create Date: 2021-10-15
> - Update Date: 2021-10-15

## 快速入门
- 让脚本可以直接执行：
1.在脚本的第一行加上：`#!/usr/bin/env python`
2.将python设置为可执行：`chmod a+x hello.py`
3.`hello.py`即可直接执行

- python的注释;
直接使用 # 进行注释

- str和repr
```python
print(repr("Hello,\nworld!"))
# Hello,\nworld!
print(str("Hello,\nworld"))
# Hello,
# world！
```
- 删除List中的元素:`del arrs[2]`

- append:List末尾添加一个数据:`arrs.append(9)`

- insert将一个对象插入列表:`arrs.insert(3,'hello')`

- pop:从列表中删除一个元素，并返回这一元素: `arrs.pop(1)`

- remobe:从列表中删除一个元素，不返回任何值

- reverse:按相反顺序排列: `arrs.reverse` 修改列表，但是不返回任何值

- sort:排序: `arrs.sort()`  不返回任何值,要想有返回值，可以使用`y=sorted(arrs)`

- 高级排序(sort/sorted函数接受两个可选参数：key和reverse)
`arrs.sort(key=len)` 按照元素长度进行排序
`arrs.sort(reverse=True)` 按照相反的顺序对列表进行排序

## 元组
元组与列表的差别在于元组是不可修改的
函数`tuple()`可以将一个序列转换为元组
`x = 1, 2, 3` ,返回 `x = (1, 2, 3)`
或者`x = 1,` ,返回 `x = (1,)`

## 格式化
- `'you %s my %s' % ('are','prince') ` 输出 `you are my prince`
- `'you {yy} my {0}'.format(15,yy='are')` 输出 `you are my 15`
```
tt = ['hel','jop']
print('you {name} lll'.format(name=tt[1]))
# 输出 you jop lll
```
- 可以在字段后加转换标志来设置需要的格式
`!s -> str` `!r -> repr` `!a -> ascii`
- 也可以指定转换成什么类型,在字段后加`:f`->float `:b`->二进制
`:d`->十进制，默认  `:e`->科学表示法表示小数 `:o`->八进制
`:s`->字符串 `:%`->表示为百分比值
- 精度：`{pi:.2f}`表示保留两位小数 输出`3.14`
- 宽度：`{pi:7.2f}` 输出 `'   3.14'`
- 使用`{:,}`添加千位分隔符

## 字符串方法
- center
在字符串两边添加填充字符 `'hello'.center(10,'*')`
输出 `'**hello***'`
- find 在字符串中查找子串，如果找到就返回子串所在的第一个字符的索引，负责返回-1。也可以通过`string.find('hello',2)`设置查找起点
- join 用于合并序列的元素
```python
s = ['usr', 'local', 'data']
print('/'.join(s))
# usr/local/data
```
- lower 返回字符串的小写版本
- replace 指定子串替换为另外一个字符串
- split 拆分字符串，返回列表
- strip 删除字符串开头和末尾的空白
- translate 进行单字符替换
使用translate前必须创建一个转换表，str调用maketrans,表示第一个参数中的字符都会替换为第二个参数的相应位置的字符，第三个参数是可选的，可以指定删除某些字符。
```python
#表示将myStr中的'c'替换为'k'，'s'替换为'z'，并删除所有的't'
table = str.makrtrans('cs','kz','t')
myStr.translate(table)
```

## 字典
- 可以使用`dict()`函数从其他映射或者键值对创建字典。
- `clear()`  清空字典
- `copy()` 返回一个新的字典
当替换副本中的值，原件不会受到影响，但是如果修改副本中的值，原件也会发生变化。
为避免这种同时复制值以及其包含的所有值。使用模块copy中的函数`deepcopy()`
- `fromkeys()` 创建一个只有key的字典
```python
{}.fromkeys(['name','age'])
{'age':None,'name':None}
```
- `get()`
- `item()` 返回一个包含所有字典项的**列表**，每个元素都是KV对的形式。返回值为一种名为视图字典的特殊类型（可用于迭代）
- `keys()` 返回一个字典视图，其中包含指定字典中的健
- `values()` 返回一个字典中的值组成的字典视图
- `pop()` 删除指定键值对
- `popitem()` 随机删除一个键值对

## 语法
- 三目表达式：s
`status = "friends" if name.endswith("Goger") "yes" else "no"`
- `elif`语句与java中的`else if`一样
- while 循环
- for 循环 `for xx in xxx: ...`
- `range(`创建范围的内置函数 `range(start, end, [duration])`
- `zip()` 并行迭代
```python
names = ['codi', 'mray', 'kkk']
ages = [18, 31, 55]
print(list(zip(names, ages)))
# [('codi', 18), ('mray', 31), ('kkk', 55)]
# 在循环中将元组解包
for names, ages in zip(names, ages):
    print(names, 'is', ages, 'years old')
# codi is 18 years old
# mray is 31 years old
# kkk is 55 years old
```
- `enumerate()` 能够迭代索引-值对，其中索引是自动提供的
- break 跳出循环
- continue 结束当前迭代，并调到下一次迭代开头
- 列表推导
```python
[x * x for x in range(10) if x % 3 == 0]
# [0, 9, 36, 81]
```
- pass 什么都不做，解决在python中代码块不能为空的问题
- del 删除
删除对象可以直接赋值None,也可以使用del语句
- 使用exec和eval执行字符串及计算其结果
exec() 可以将字符串作为代码执行  
- eval
与exec类似，计算并返回字符串表示的表达式的结果

## 抽象(函数)
####自定义函数
- 给函数编写文档
```python
def square(x):
  'calculates the squa...'
  return x * x
# 可以使用square._doc_来调用文档字符串
```
- 收集参数
`def print_name(*params)`
星号代表收集余下的位置参数，如果没g有可收集的参数，params将是一个空元组
要想收集关键字参数，可以使用**

- 私有方法的定义使用两个下划线开头:`__`
```python
class Secretive:
    def __inaccessible(self):
        print("hello world~")
```
- 继承,在定义子类时，在类名后加`class 子类名(父类名)`
```python
class Filter:
    def init(self):
        self.blocked = []
    def filter(self,sequence):
        return ...
class SPAMFilter(Filter):
    def init(self):
        self.blocked = ['SPAM']
```
    * 可以使用`issubclass(SPAMFilter, Filter）`方法来确认是否为子类
    * 可以使用`SPAMFilter.__bases__`
    * 获得使用`s.__class__`查看对象s属于哪个类

- `**kwargs`的用法
`**kwargs`允许将不定长度的键值对作为一个参数传递给一个函数
```python
def greet_me(**kwargs):
    for key, value in kwargs.iterms():
        print("{0} == {1}".format(key, value))

>>> greet_me(name="yahoo")
```

### 构造方法 `_init_`
- 调用未关联的超类构造函数
```python
class SongBird(Bird):
    def __init__(self):
        Bird.__init__(self)
        self.sound = 'Squawk!'
    def sing(self):
        print(self.sound)
```
- 使用函数super
```python
class SongBird(Bird):
    def __init__(self):
        super().__init__()
        self.sound = 'Squawk!'
    def sing(self):
        print(self.sound)
```
**客观来看，使用super更加直观。再有多个超类的时候，也只需要调用一次super函数就可以了**

### 静态方法和类方法
静态方法和类方法创建方式：将它们分别包装在staticmethod和classmethod类的对象中。
- 静态方法的定义中没有参数self，可通过类直接调用。
- 类方法的定义中包含类似于self的参数，通常为cls

#### 使用装饰器指定方法类型，之后可以直接使用类名进行调用
```python
class MyClass:
    @staticmethod
    def sta_me():
        print("static method")

    @classmethod
    def clas_me(slef):
        print('class_method')

MyClass.clas_me()
```
### 函数property

### 迭代器
**实现了方法_iter_的对象是可迭代的，而实现了方法_next_的对象是迭代器**

##Python JSON
使用python编码和解码JSON对象
`import json`
#### 函数
- `json.dumps` 将python对象编码成json字符串
- `json.loads` 将已编码的JSON字符串解码为python对象

#### 模块导入
`from drawing import shapes` 这种导入方法最方便
#### yaml
```python
import yaml
logs = yaml.load(open("/Users/cody/Desktop/test/python/value1.yaml").read())
print(logs['image'])
```

#### fileinput模块
- `input` 迭代多个输入流中的行
参数列表`files=None, inplace=False, backup='', bufsize=0, mode='r', openhook=None`
- `filename()` 当前文件的名称
- `lineno()` 当前累计行号
- `filelineno()` 在当前文件中的行号
- `isfirstline()`
- `isstdin()` 检查最后一行是否来自sys.stdin
- `nextfile()` 关闭当前文件并移到下一个文件
- `close()` 关闭序列
#### random模块
- `random()` 返回一个(0,1]的随机实数
- `randrange()` 从`range(start, stop, step)`中随机选择一个数
#### shelve模块
#### 模块re
- `compile() ` 根据正则表达式的字符串创建模式对象
- `search()` 在字符串中查找模式
- `match()` 在字符串开头匹配模式
- `split()` 根据模式来分隔字符串
- `findall(pattern, string)` 返回一个列表,其中包含字符串中所有与模式匹配的子串
- `sub(pat, repl, string)` 将字符串中与模式pat匹配的子串都替换为repl
