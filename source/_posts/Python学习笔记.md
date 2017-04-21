---
title: Python学习笔记
categories:
  - Python
tags:
  - 语法
date: 2017-04-21 14:39:34
---

<Excerpt in index | 摘要> 
教练，我要写爬虫！<!-- more -->
<The rest of contents | 余下全文>

> Python是动态语言，本身变量类型不固定 

## 基础

### 输入、输出： 
- input()函数：`name = input(“请输入...")` 
- print()函数：输出括号中的指定字符串，可以输出多个字符串，用逗号”,”隔开 
格式化输出：%d:整数; %f:浮点数; %s:字符串; %x:十六进制数 

### 常量
通常全部用大写的变量名表示常量，只是习惯，不是规定，仍然可以改 
`PI = 3.14159265359` 

### 列表
- list, 有序可变列表: l = [’Tom’, ‘Micheal’] 
- tuple, 有序不可变元组: t = (’Tom’, ‘Jhon') 

### 函数定义 
```
def my_abs(x): 
if x >= 0: 
        return x 
    else: 
        return -x 
```

- 位置参数：`def power(x, n):` 

- 默认参数：`def power(x, n=2):` 

- 可变参数：`def calc(*numbers):`  接收的参数是tuple 

- 关键字参数：`def person(name, age, **kw):`  接收的参数是dict  

- 命名关键字参数：`def person(name, age, *, city, job):`  限制关键字参数名字为city和job 

```
 print(name, age, city, job) 
>>> person(‘Jack’, 24, city=‘Beijing’, job=‘Engineer’) 
```

> Python的函数具有非常灵活的参数形态，既可以实现简单的调用，又可以传入非常复杂的参数。 

>默认参数一定要用不可变对象，如果是可变对象，程序运行时会有逻辑错误！ 

> 要注意定义可变参数和关键字参数的语法： 

> *args是可变参数，args接收的是一个tuple； 

> **kw是关键字参数，kw接收的是一个dict。 

> 以及调用函数时如何传入可变参数和关键字参数的语法： 

> 可变参数既可以直接传入：func(1, 2, 3)，又可以先组装list或tuple，再通过*args传入：func(*(1, 2, 3))； 

> 关键字参数既可以直接传入：func(a=1, b=2)，又可以先组装dict，再通过**kw传入：func(**{'a': 1, 'b': 2})。 

> 使用*args和**kw是Python的习惯写法，当然也可以用其他参数名，但最好使用习惯用法。 

> 命名的关键字参数是为了限制调用者可以传入的参数名，同时可以提供默认值。 

> 定义命名的关键字参数在没有可变参数的情况下不要忘了写分隔符*，否则定义的将是位置参数。 

## 高级特性

### 切片
```
>>>L[:10]        前10个数 
>>>L[:10:2]     前10个数，每2个取一个 
```

### 列表生成式
```
>>> list(range(1, 11)) 
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10] 
>>> [x * x for x in range(1, 11)] 
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100] 
>>> [m + n for m in ‘ABC’ for n in ‘XYZ’] 
[‘AX’, ‘AY’, ‘AZ’, ‘BX’, ‘BY’, ‘BZ’, ‘CX’, ‘CY’, ‘CZ’] 
```

### 斐波那契数列：(列表生成式+函数) 
```
def fib(max): 
    n, a, b = 0, 0, 1 
while n < max: 
        print(b) 
        a, b = b, a + b 
        n = n + 1 
return 'done' 
```

## 函数式编程
### map()
map将传入的函数依次作用到序列的每个元素，并把结果作为新的Iterator返回
map()传入的第一个参数是f，即函数对象本身。由于结果r是一个Iterator，Iterator是惰性序列，因此通过list()函数让它把整个序列都计算出来并返回一个list。 

### reduce()
reduce()把一个函数作用在一个序列[x1, x2, x3, ...]上，这个函数必须接收两个参数，reduce把结果继续和序列的下一个元素做累积计算，其效果就是： 
`reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4) `

#### 结合起来写个字符串转数字的函数： 
```
from functools import reduce 

def str2int(s): 
def fn(x, y): 
return x * 10 + y 
    def char2num(s): 
return {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}[s] 
    return reduce(fn, map(char2num, s)) 
```

### fliter()
filter()也接收一个函数和一个序列。和map()不同的是，filter()把传入的函数依次作用于每个元素，然后根据返回值是True还是False决定保留还是丢弃该元素
#### 生成1000以内的素数
```
def _odd_iter():  #生成一个从3开始的奇数序列
    n = 1
    while True:
        n = n + 2
        yield n

def _not_divisible(n):  ##定义一个筛选函数
    return lambda x: x % n > 0

def primes():
    yield 2
    it = _odd_iter() #初始序列
    while True:
        n = next(it) #返回序列的第一个数
        yield n
        it = filter(_not_divisible(n), it)

for n in primes():
    if n < 1000:
        print(n)
    else:
        break
```

### sorted()
内置排序算法，默认是升序
> key指定的函数将作用于list的每一个元素上

```
>>> sorted([36, 5, -12, 9, -21], key=abs)
[5, 9, -12, -21, 36]
```
> 要进行反向排序，不必改动key函数，可以传入第三个参数`reverse=True`

```
>>> sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower, reverse=True)
['Zoo', 'Credit', 'bob', 'about']
```
### 匿名函数lambda
`lambda x: x * x `等价于：
```
def f(x):
    return x * x
```

### 装饰器
> 假设我们要增强now()函数的功能，比如，在函数调用前后自动打印日志，但又不希望修改now()函数的定义，这种在代码运行期间动态增加功能的方式，称之为“装饰器”（Decorator）

#### 利用decorator打印出函数调用前和调用后的日志
```
import functools

def log(text):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kw):
            print("%s begin call %s()" % (text, func.__name__))
            ret = func(*args, **kw)
            print("%s end call %s()" % (text, func.__name__))
            return ret
        return wrapper
    if isinstance(text, str):
        return decorator
    else:
        f = text
        text = ''
        return decorator(f)

@log#('excute')
def f():
    print("soso!")
f()
```
### 偏函数
> Python的functools模块提供了很多有用的功能，其中一个就是偏函数(Partial function).functools.partial就是帮助我们创建一个偏函数的.

```
>>> import functools
>>> int2 = functools.partial(int, base=2)
>>> int2('1000000')
64
>>> int2('1010101')
85
```
## 模块
> 在Python中，一个.py文件就称之为一个模块（Module）

### 模块标准模版
```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

' a test module '

__author__ = 'Longxr'
```

### 安装第三方模块
> Python Imaging Library，这是Python下非常强大的处理图像的工具库

#### 安装PIL库
`pip install Pillow`
#### 生成缩略图
```
>>> from PIL import Image
>>> im = Image.open('test.png')
>>> print(im.format, im.size, im.mode)
PNG (400, 300) RGB
>>> im.thumbnail((200, 100))
>>> im.save('thumb.jpg', 'JPEG')
```

##  面向对象编程
### 类的定义
> 在类中定义的函数只有一点不同，就是第一个参数永远是实例变量self，并且，调用时，不用传递该参数

```
class Student(object):

    def __init__(self, name, score):
        self.name = name
        self.score = score

    def print_score(self):
        print('%s: %s' % (self.name, self.score))
```

### 访问限制
如果要让内部属性不被外部访问，可以把属性的名称前加上两个下划线__，在Python中，实例的变量名如果以__开头，就变成了一个私有变量（private）
> 在Python中，变量名类似__xxx__的，也就是以双下划线开头，并且以双下划线结尾的，是特殊变量，特殊变量是可以直接访问的，不是private变量，所以，不能用__name__、__score__这样的变量名。
有些时候，你会看到以一个下划线开头的实例变量名，比如_name，这样的实例变量外部是可以访问的，但是，按照约定俗成的规定，当你看到这样的变量时，意思就是，“虽然我可以被访问，但是，请把我视为私有变量，不要随意访问”。双下划线开头的实例变量是不是一定不能从外部访问呢？其实也不是。不能直接访问__name是因为Python解释器对外把__name变量改成了_Student__name，所以，仍然可以通过_Student__name来访问__name变量

### 使用property
```
class Student(object):

    @property
    def score(self):
        return self._score

    @score.setter
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value
```
不能直接访问属性，会调用对应的getter()、setter()方法
```
>>> s = Student()
>>> s.score = 60 # OK，实际转化为s.set_score(60)
>>> s.score # OK，实际转化为s.get_score()
60
>>> s.score = 9999
Traceback (most recent call last):
  ...
ValueError: score must between 0 ~ 100!
```

## 错误、调试、测试
### 错误处理
try...except...finally...
> 当我们认为某些代码可能会出错时，就可以用try来运行这段代码，如果执行出错，则后续代码不会继续执行，而是直接跳转至错误处理代码，即except语句块，执行完except后，如果有finally语句块，则执行finally语句块，至此，执行完毕

```
try:
    print('try...')
    r = 10 / 0
    print('result:', r)
except ZeroDivisionError as e:
    print('except:', e)
finally:
    print('finally...')
print('END')
```
### 调试
#### print()
打印出变量的信息

#### 断言
凡是用print()来辅助查看的地方，都可以用断言（assert）来替代。在启动Python解释器时可以用-o参数来关闭assert，`assert`语句会被当成`pass`看
#### logging
把print()替换为logging是第3种方式，和assert比，logging不会抛出错误，而且可以输出到文件
#### 单步调试
- qdb
- IDE，比如[PyCharm](http://www.jetbrains.com/pycharm/)

### 测试
- 单元测试是用来对一个模块、一个函数或者一个类来进行正确性检验的测试工作
- Python内置的“文档测试”（doctest）模块可以直接提取注释中的代码并执行测试。
doctest严格按照Python交互式命令行的输入和输出来判断测试结果是否正确。只有测试异常的时候，可以用...表示中间一大段烦人的输出


