<!-- TOC -->

- [1. First Class](#1-first-class)
- [2. 继承](#2-继承)
    - [2.1. 抽象类](#21-抽象类)
        - [2.1.1. 显示声明抽象方法](#211-显示声明抽象方法)
- [3. 特殊方法](#3-特殊方法)
    - [3.1. `__getitem__` 和 `__setitem__`](#31-__getitem__-和-__setitem__)
        - [3.1.2. 什么情况下会调用 `__getitem__`](#312-什么情况下会调用-__getitem__)
    - [3.2. `__iter__` 和 `__next__`](#32-__iter__-和-__next__)
        - [3.2.1. 有多个迭代器的对象](#321-有多个迭代器的对象)
    - [`getattr` 和 `setattr`](#getattr-和-setattr)
        - [实现私有属性](#实现私有属性)
- [4. 最简单的类](#4-最简单的类)
- [5. 属性](#5-属性)
    - [5.1. 实例属性和类属性](#51-实例属性和类属性)
- [6. 方法](#6-方法)

<!-- /TOC -->

# 1. First Class

定义：

```python
class FirstClass:
	def setData(self, value):
		self.data = value

	def display(self):
		print(self.data)
```

创建：

```python
x = FirstClass()
x.setData("hello")
x.display()
```

可以对未定义的字段赋值：

```python
x.data2 = "未定义的字段"
```


# 2. 继承

```python
class FirstClass:
	def setData(self, value):
		self.data = value

	def display(self):
		print(self.data)


class SecondClass(FirstClass):
	def display(self):  # 重载
		FirstClass.display(self)  # 注意需要传递self
		print('current value = "%s"' % self.data)


x = SecondClass()
x.setData("second")
x.display()
```

通过 `FirstClass.display(self)` 的方法来调用父类的方法，调用父类的构造方法可以用 `FirstClass.__init__(self, 其余参数)`


## 2.1. 抽象类

```py
class Super(object):
    def delegate(self):
        self.action()  # 抽象方法


class Sub(Super):
    def action(self):
        print("action in Sub")
```

### 2.1.1. 显示声明抽象方法

```py
from abc import ABCMeta, abstractmethod

class Super():
    __metaclass__ = ABCMeta

    @abstractmethod
    def action(self):
        pass

    def delegate(self):
        self.action()  # 抽象方法
```

通过类属性和装饰器来声明一个类属于抽象类和抽象方法，这样就不能对 Super 进行实例化了。



# 3. 特殊方法

```python
class ThirdClass(SecondClass):
	def __init__(self, value):
		self.data = value

	def __add__(self, other):
		return ThirdClass(self.data + other)

	def __str__(self):
		return "[ThirdClass: %s]" % self.data

x = ThirdClass('abc')
y = x + 'def'
print(y)
```

* `__init__` 是构造方法。
* `__del__` 是析构函数。
* `__add__` 是加法运算。 
* `__radd__` 右侧加法(Other + X)
* `__iadd__` 实地加法(X += Y)
* `__sub__` 是减法运算。
* `__str__`, `__repr__` 打印、转换。
* `__len__` 长度
* `__call__` 函数调用。
* `__getattr__` 点号运算(X.field)
* `__getattribute__` 属性获取(X.any)
* `__setattr__` 属性赋值(X.any = value)
* `__delattr__` 属性删除(del X.any)
* `__getitem__` 索引运算(X[key])
* `__setitem__` 索引赋值(X[key] = value, X[i:j] = sequence)
* `__delitem__` 索引和分片删除(del X[key], del X[i:j])
* `__bool__` 布尔测试(bool(X))
* `__lt__`, `__gt__`, `__le__`, `__ge__`, `__eq__`, `__ne__` : `<, >, <=, >=, ==, !=`
* `__iter__`, `__next__` 迭代
* `__contains__` 包含测试(item in X)
* `__index__` 整数值(hex(X), bin(X), oct(X), O[X], O[X:])
* `__enter__`, `__exit__` 环境管理器
* `__get__`, `__set__` 描述符属性
* `__delete__`
* `__new__` :在 `__init__` 之前创建对象。


## 3.1. `__getitem__` 和 `__setitem__`


```py
class Indexer(object):
    def __getitem__(self, index):
        return index**2


def main():
    X = Indexer()
    print X[2]

    for i in range(5):
        print X[i]
```

对于 `X[0]` 这样不包含分片的调用，index表示的就是`[]`中的整数，对于分片操作来说 index 就是一个分片对象：

```py
class Indexer(object):
    data = [5, 6, 7, 8, 9]

    def __getitem__(self, index):
        print('getitem:', index)
        return self.data[index]


def main():
    X = Indexer()
    print X[2]  # 7 (index=2)
    print X[-1]  # 9 (index=-1)
    print X[1:3]  # [6,7] (index=slice(1,3,None))
    print X[1:]  # [6,7,8,9] (index=slice(1,None,None))
    print X[:-1]  # [5,6,7,8] (index=slice(None,-1,None))
    print X[::2]  # [5,7,9] (index=slice(None,None,2))
```

### 3.1.2. 什么情况下会调用 `__getitem__`

```py
class Indexer(object):
    data = "hello"

    def __getitem__(self, index):
        print('getitem:', index)
        return self.data[index]


def main():
    X = Indexer()
    print 'e' in X
    print [c for c in X]  # 迭代
    print list(map(str.upper, X))  # 应用于map，这应该也是属于迭代
    (a, b, c, d, e) = X
    print a, b, c, d, e
    print list(X), tuple(X), ' '.join(X)
```


## 3.2. `__iter__` 和 `__next__`

* 尽管 `__getitem__` 也可以用于迭代环境，但是 `__iter__` 优先于 `__getitem__`
* 迭代通过调用内置函数 iter 来尝试寻找 `__iter__`, `__iter__` 应该返回一个迭代器对象，Python 会重复调用迭代器对象的 next 方法，直到 StopIteration 异常被抛出。
* 没有 `__iter__` 时会尝试 `__getitem__`，直到引发 IndexError 异常。

```py
class Squares(object):
    def __init__(self, start, stop):
        self.value = start - 1
        self.stop = stop

    def __iter__(self):
        return self  # 自己就是一个迭代器

    def next(self):  # Python 3.0 上应该是 __next__
        if self.value == self.stop:
            raise StopIteration
        self.value += 1
        return self.value**2


def main():
    for i in Squares(1, 3):
        print i
```

下面展示的是手动迭代：

```py
    squares = Squares(1, 3)
    i = iter(squares)  # 调用 __iter__
    next(i)  # 调用 i.next()
    next(i)
    next(i)
    next(i)  # raise StopIteration
```

迭代器没有重载索引表达式，所以调用 `squares[0]` 这样的会报错。

### 3.2.1. 有多个迭代器的对象

Squares 类只能循环一次，想再次循环就得重新创建一个类。如果想每次循环都重新开始，可以让 `__iter__` 每次都返回新对象，而不是 self.

```py
class SquaresIterator(object):
    def __init__(self, value, stop):
        self.value = value
        self.stop = stop

    def next(self):
        if self.value == self.stop:
            raise StopIteration
        self.value += 1
        return self.value**2


class Squares(object):
    def __init__(self, start, stop):
        self.value = start - 1
        self.stop = stop

    def __iter__(self):
        return SquaresIterator(self.value, self.stop)  # 每次都返回一个新对象


def main():
    squares = Squares(1, 3)
    for i in squares:
        for j in squares:
            print i, j
```


## `getattr` 和 `setattr`

* 只有在Python找不到该属性时才调用 `__getattr__`。
* `__getattribute__` 拦截所有属性访问，必须避免循环。

```py
class Empty(object):
    def __getattr__(self, attrname):
        if attrname == "age":
            return 40
        raise AttributeError, attrname


def main():
    x = Empty()
    print x.age
    print x.name  # raise Error
```

* `__setattr__` 会拦截所有属性的赋值语句。在 `__setattr__` 中对 self 任何属性赋值时都会再次调用 `__setattr__`，导致无穷递归。
* 可以使用 `self.__dict__[attrname] = value` 的方式避免无穷递归。

```py
class Empty(object):
    def __setattr__(self, attr, value):
        if attr == "age":
            self.__dict__[attr] = value
            return
        raise AttributeError, attr + ' not allowed'


def main():
    x = Empty()
    x.age = 40
    print x.age
```

### 实现私有属性

下面是通过继承的方式实现私有属性，还可以通过类装饰器的方式。

```py
class Privacy(object):
    def __setattr__(self, attrname, value):
        if attrname in self.privates:
            raise AttributeError, attrname
        self.__dict__[attrname] = value


class Test(Privacy):
    privates = ['name', 'pay']

    def __init__(self):
        self.__dict__['name'] = 'Tom'


def main():
    x = Test()
    x.name = 'Bob'  # fails
    x.age = 33  # fails
```






# 4. 最简单的类

```python
class rec: pass


rec.name = "test"
rec.age = 13
```

name 和 age 是类rec的属性，对于rec的实例来说(比如x=rec())所有的实例都继承这些属性（像是java中的静态属性），当`x.name=""`时x才拥有自己的属性，此时x和其他实例就不再共享相同的name属性。

```python
def upperName(self):
	return self.name.upper()


print(upperName(rec))

rec.method = upperName

x = rec()
x.name = 'x'
print(x.method())
print(rec.method(x))  # 通过类调用时需要传实例
```

在类定义之外的方法`upperName`可以当做普通函数使用，也可以赋值成类的方法。


```python
rec = {}
rec['name'] = "test"
rec['age'] = 12
print(rec)  # {'age': 12, 'name': 'test'}


class rec: pass


pers = rec()
pers.name = 'test'
pers.age = 12
print(pers)  # <__main__.rec instance at 0x110022200>
```

实现字典的功能。（打印的话还是需要定义`__str__`方法）




# 5. 属性

## 5.1. 实例属性和类属性

```py
class Data(object):
    shared_field = "shared" # 所有实例共享

    def __init__(self):
        self.instanse_field = "instance" # 实例专属


def main():
    d = Data()
    print d.shared_field
    print Data.shared_field
    print d.instanse_field
```

# 6. 方法

```py
class Hello(object):
    def printer(self):
        print("hello world")


def main():
    hello = Hello()
    hello.printer()

	Hello.printer(hello)  # 通过类来调用实例的方法
```

方法调用：`instance.method(args...)`

这会自动翻译成以下形式：`class.method(instance, args...)`

