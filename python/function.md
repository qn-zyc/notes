- `def` 是可执行代码，创建一个对象并自动赋值给一个变量。
- `lambda` 创建一个对象并将其返回。
- `def` 语句是实时执行的。

# 定义和调用

```python
# -*- coding: utf-8 -*

def funcName(p1, p2):
    print p1, p2
    return 123


print funcName(1, 2)
```

调用时指定参数名:

```python
# -*- coding: utf-8 -*

def funcName(p):
    print p
    return


funcName(p="hello")
```

## 默认参数

```python
# -*- coding: utf-8 -*

def funcName(p=23):
    print p
    return


funcName()
```

## 不定参数(参数前加*)

```python
# -*- coding: utf-8 -*

def funcName(*p):
    print p  # (1, 2, 3)
    return


funcName(1, 2, 3)

# 把列表或元组当做可变参数传递
nums = [1, 2, 3]
funcName(*nums)  # 在nums前加*
```



## 关键字参数(参数前加**)

```python
def person(name, age, **kw):
	print 'name:', name, 'age:', age, 'other:', kw


if __name__ == '__main__':
	person('Michael', 30)  # name: Michael age: 30 other: {}
	person('Bob', 35, city='Beijing')  # name: Bob age: 35 other: {'city': 'Beijing'}
	person('Adam', 45, gender='M', job='Engineer')  # name: Adam age: 45 other: {'gender': 'M', 'job': 'Engineer'}

	# 传递一个字典
	kw = {'city': 'Beijing', 'job': 'Engineer'}
	person('Jack', 24, **kw)  # name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
```



## 参数组合

参数定义的顺序必须是：必选参数、默认参数、可变参数和关键字参数

```python
def func(a, b, c=0, *args, **kw):
    print 'a =', a, 'b =', b, 'c =', c, 'args =', args, 'kw =', kw
```


## 返回多个值

```python
def move(x, y, step, angle=0):
    nx = x + step * math.cos(angle)
    ny = y - step * math.sin(angle)
    return nx, ny


# 调用
x, y = move(100, 100, 60, math.pi / 6)
```

其实返回的是元组

```python
>>> r = move(100, 100, 60, math.pi / 6)
>>> print r
(151.96152422706632, 70.0)
```

# 匿名函数

格式：`lambda args: expression`

```python
# -*- coding: utf-8 -*

sum = lambda a1, a2: a1 + a2

print sum(1, 2)
```

默认参数：

```python
f = lambda c, a='a', b='b': a + b + c

print f('c')  # abc
```

`lambda` 可以用在 `def` 无法使用的地方：

```python
L = [
	lambda x: x ** 2,
	lambda x: x ** 3,
	lambda x: x ** 4
]

print L[0](3)  # 9
```





# 变量作用域

```python
# -*- coding: utf-8 -*

total = 1


def sum(a1, a2):
    total = a1 + a2
    return total


print sum(2, 3)  # 5
print total  # 1
```

函数内的变量属于局部变量。

函数内访问和修改全局变量（使用 `global` 声明）：

```python
# -*- coding: utf-8 -*

total = 1


def sum(a1, a2):
    global total
    total = a1 + a2
    return total


print sum(2, 3)  # 5
print total  # 5
```

## 嵌套函数

```python
def f1():
	x = 8

	def f2():
		print(x)

	return f2


action = f1()
action()  # 8
```

在较早版本的python中，`f2` 不能访问 `x`，可以通过为 `f2` 指定默认参数来达到同样的效果：

```python
def f1():
	x = 8

	def f2(x=x):
		print(x)

	return f2


action = f1()
action()  # 8
```

嵌套 `lambda`:

```python
def f1():
	x = 8
	action = (lambda: x)
	return action


f = f1()
print(f())  # 8
```

循环变量与嵌套`lambda`:

```python
acts = []
for i in range(2):
	acts.append(lambda: i)

for act in acts:
	print(act())

# 1
# 1
```

`lambda` 访问的都是相同的循环变量值，因为嵌套作用域中的变量在嵌套的函数被调用时才进行查找，可以使用默认参数避免这个问题：

```python
acts = []
for i in range(2):
	acts.append(lambda i=i: i)

for act in acts:
	print(act())

# 0
# 1
```



# 函数属性

函数也是一个对象，可以拥有自己的字段属性：

```python
def count(i):
	count.c += i
	return count.c


count.c = 0  # 必须先初始化
print(count(1))  # 1
print(count(2))  # 3
```



# 在序列中映射函数：map

`map` 会对一个序列对象中的每个元素应用被传入的函数，并且返回一个**列表**。

```python
def incr(x): return x + 1


L = (1, 2, 3, 4)
L = map(incr, L)
print L  # [2, 3, 4, 5]

L = map(lambda x: x * 2, L)
print L  # [4, 6, 8, 10]
```

多个序列：

```python
print pow(3, 4)  # 81

print map(pow, [1, 2, 3], [2, 3, 4])  # [1, 8, 81]
```



# reduce

```python
a = reduce((lambda x, y: x + y), [1, 2, 3, 4])
print a  # 10
```

每一步 `reduce` 将当前结果和下一个元素传入 `lambda` ，第一个元素为初始结果，最后返回整个结果。



# filter

函数如果返回真则放入序列中。

`filter` 返回一个迭代器。

```python
L = list(filter(lambda x: x > 0, (-1, 0, 1)))
print L  # [1]
```



# 生成器函数

```python
def gen_squares(n):
	for i in range(n):
		yield i ** 
```

当执行到 `yield` 时，函数直接返回结果并记住当前的状态，下次可以继续从 `yield` 后执行。

```python
x = gen_squares(3)
print x
print next(x)
print next(x)
print next(x)
print next(x)
# <generator object gen_squares at 0x10dc10a50>
# 0
# 1
# 4
# Traceback (most recent call last):
#   File "test.py", line 32, in <module>
#     print next(x)
# StopIteration
```

`x` 是一个生成器，`next` 方法会调用 `x.__next__` 从而执行 `gen_squares` 函数或者从上次 `yield` 值后的地方恢复，并且在得到最后一个值后产生 `StopIteration` 异常。

还可以使用 `for` 直接遍历生成器：

```python
for i in gen_squares(3):
	print i
```

