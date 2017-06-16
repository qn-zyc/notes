# 赋值

- python不支持 `i++` 或 `++i`。

## 序列赋值

```python
>>> x, y = 1, 2
>>> x, y
(1, 2)
>>> [a, b] = [3, 4]
>>> a, b
(3, 4)
```

互换值：（先计算右侧的值，然后保存在临时变量中，再赋值给左侧变量）

```python
>>> a, b = 1, 2
>>> a, b
(1, 2)
>>> a, b = b, a
>>> a, b
(2, 1)
```

右侧可以是任何可迭代的对象，只要长度相等就行：

```python
>>> [a, b] = (4, 5)
>>> a, b
(4, 5)

>>> a, b = "ha"
>>> a, b
('h', 'a')
>>> a, b = "haha"
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: too many values to unpack
    
>>> a, b, c = range(3)
>>> a, b, c
(0, 1, 2)
```

赋值嵌套序列：

```python
>>> ((a, b), c) = ('SP', 'AM')
>>> a, b, c
('S', 'P', 'AM')
```

## 其他例子

```python
spam = 'Spam'
spam, ham = 'yum', 'YUM' # 元组赋值，省略了括号
[spam, ham] = ['yum', 'YUM'] # 列表赋值
a, b, c = 'spa' # 序列赋值 a='s' b='p' c='a' 左右个数应相等
a = b = 'lunch'
[a, b, c] = (1, 2, 3) # 通用化，右侧可以是任意类型的序列
```





# 比较运算

## 连续比较

```python
x < y > z
等效于： x < y and y > z

1 < 2 < 3.0 < 4
1 == 2 < 3  # same as: 1 == 2 and 2 < 3
```

## 是否相等

```python
l1 = [1, ['a', 'b']]
l2 = [1, ['a', 'b']]
print l1 == l2  # True
print l1 is l2  # False
```

`==` 递归地比较所有内嵌对象

`is` 判断是否是同一内存

另外 `<` `>` 也会递归地比较内嵌对象。



# 位运算

```python
x << 2  # 左移
x | 2   # 或
x & 2   # 与
```



# 逻辑运算

- 逻辑运算遵循短路运算

```python
x and y
x or y
not x
```

`and` 和 `or` 返回的是对象，而不是 `True` 或 `False`:

```python
>>> 2 and 3
3
>>> 3 and []
[]
>>> [] and 3
[]

>>> 2 or 3
2
>>> 2 or []
2
>>> [] or 2
2
>>> [] or {}
{}
```

`and` 返回第一个为假的对象，如果都是真时返回最后一个对象。

`or` 返回第一个为真的对象，如果都是假时返回最后一个对象。



```python
x and y
not x
x in y, x not in y
x is y, x is not y
x < y, x <= y, x > y, x >= y
x == y, x != y
x | y
x ^ y  # 异或
x & y
x << y, x >> y
x + y, x - y
x * y, x % y, x / y, x // y
-x, +x
x ** y
```





# if-else

* 以冒号结尾
* 括号是可选的
* python中没有switch-case语句

```python
if <test1>:
	<statements1>
elif <test2>:
	<statements2>
else:
	<statements3>
```

```python
>>> if 1:
...     print('true')
...
true
>>>
>>> if not 1:
...     print('true')
... else:
...     print('false')
...
false
```

## if-else三元表达式

```python
A = Y if X else Z
```

等同于：

```python
if X:
	A = Y
else:
	A = Z
```

这是短路运算，只有X为真才会执行Y,同样X为假才会执行Z



# while

## 一般格式

```python
while <test1>:
    <statements1>
    if <test2>: break     # exit loop now, skip else
    if <test3>: continue  # go to top of loop now, to test1
else:
    <statements2>         # run if we didn't hit a 'break'
```



```python
i = 10
while i > 0:
	print('hello')
	i -= 1
	
while True:
	print('hello')
```

什么也不做：

```python
while True: pass
```

## while-else

```python
while x:
	if match:
		break
	x = x[1:]
else:
	print('not found')
```

当while中没有执行break语句时就会执行else部分。



# for

```python
for <target> in <object>:
	statements
    if <test>: break     # exit loop now, skip else
    if <test>: continue  # go to top of loop now
else:
	statements           # if we didn't hit a 'break'
```

`in` 是必须的

`else` 也是在 `for` 中没有执行 `break` 时调用的。

```python
for x in ["a", "b"]:
	print(x)

prod = 1
for item in [1, 2, 3, 4]: prod *= item
```



# range

- `range` 是一个迭代器

```python
print range(5)  # [0, 1, 2, 3, 4]
print(range(1, 5))  # [1, 2, 3, 4]
print(range(5, -3, -2))  # [5, 3, 1, -1]
```

产生的整数列范围是[`$1`, `$2`), 步进为`$3`. (`$n`表示第`n`个参数)

```python
for i in range(5):
	print(i)
```

带索引的迭代：

```python
x = 'hello'
for i in range(len(x)):
	print(x[i])
```

或者：

```python
x = 'hello'
for (offset, item) in enumerate(x):
	print(offset, item)
```



# zip

- 同时遍历**多个**可迭代对象
- `zip` 返回一个迭代器

```python
L1 = [1, 2, 3]
L2 = [4, 5, 6]

print list(zip(L1, L2))  # [(1, 4), (2, 5), (3, 6)]

for (x, y) in zip(L1, L2):
	print (x, y)

# (1, 4)
# (2, 5)
# (3, 6)
```

当参数长度不同时，以最短序列的长度为准：

```python
L1 = [1, 2, 3]
L2 = [4, 5, 6, 7, 8]

print list(zip(L1, L2))  # [(1, 4), (2, 5), (3, 6)]
```

## 使用zip构建字典

```python
keys = ["a", "b", "c"]
vals = [1, 2, 3]

print zip(keys, vals)  # [('a', 1), ('b', 2), ('c', 3)]

d = {}
for (k, v) in zip(keys, vals): d[k] = v
print d  # {'a': 1, 'c': 3, 'b': 2}

# 或者
d2 = dict(zip(keys, vals))
print d2  # {'a': 1, 'c': 3, 'b': 2}
```



# 产生偏移和元素：enumerate

```python
s = 'Spam'
for (i, v) in enumerate(s):
	print i, v

# 0 S
# 1 p
# 2 a
# 3 m
```

`enumerate` 是一个迭代器，每次返回一个元组，包含偏移和元素。

