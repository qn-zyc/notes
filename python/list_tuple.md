# 列表list

* 不固定大小
* 类型可以不一样

## 长度和索引访问和字符串一样

```python
>>> L = ['a', 'b', 123]
>>> L
['a', 'b', 123]
>>> len(L)
3
>>> L[0:-1]
['a', 'b']
```


## 嵌套

```python
M = [[1, 2, 3],
     [2, 3, 3],
     [3, 4, 5, 6],
     ]
```

## 列表解析

```python
M = [[1, 2, 3],
     [2, 3, 3],
     [3, 4, 5, 6],
     ]
col2 = [row[1] for row in M]

print(col2)  # [2, 3, 4]
```

遍历M,每一行赋值给row，取出每个row中的[1]组成列表。

第二列的每个值加1：

```python
M = [[1, 2, 3],
     [2, 3, 3],
     [3, 4, 5, 6],
     ]
col2 = [row[1] + 1 for row in M]

print(col2)  # [3, 4, 5]
```

取出第二列是偶数的：

```python
M = [[1, 2, 3],
     [2, 3, 3],
     [3, 4, 5, 6],
     ]
col2 = [row[1] for row in M if row[1] % 2 == 0]

print(col2)  # [2, 4]
```

取出矩阵对角线上的值：

```python
M = [[1, 2, 3],
     [4, 5, 6],
     [7, 8, 9],
     ]
diag = [M[i][i] for i in [0, 1, 2]]

print(diag)  # [1, 5, 9]
```

重复字符串中的字符：

```python
doubles = [s * 2 for s in 'hello']

print(doubles)  # ['hh', 'ee', 'll', 'll', 'oo']
```

输出两个字符串的交集：

```python
s1 = "SPAM"
s2 = "SCAM"
intersect = [x for x in s1 if x in s2]
print(intersect) # ['S', 'A', 'M']
```



### 列表解析中嵌套for

```python
res = [x + y for x in [0, 1, 2] for y in [100, 200, 300]]
print res  # [100, 200, 300, 101, 201, 301, 102, 202, 302]
```

等效于下面的写法：

```python
res = []
for x in [0, 1, 2]:
	for y in [100, 200, 300]:
		res.append(x + y)
print res
```

**嵌套for从句中附加if**

```python
res = [(x, y) for x in range(5) if x % 2 == 0 for y in range(5) if y % 2 == 1]
print res  # [(0, 1), (0, 3), (2, 1), (2, 3), (4, 1), (4, 3)]
```

等效于：

```python
res = []
for x in range(5):
	if x % 2 == 0:
		for y in range(5):
			if y % 2 == 1:
				res.append((x, y))
print res
```





## 遍历

```python
list = [2, 3, 1]
for item in list:
	print item  # 2 3 1

for i in range(len(list)):
	print i, ":", list[i]  # 0:2, 1:3, 2:1

for i, item in enumerate(list):
	print i, ":", item  # 0:2, 1:3, 2:1
```

### 遍历并修改列表值

```python
L = [1, 2, 3]
for i in range(len(L)):
	L[i] += 1
```



## 拷贝

```python
>>> L1 = [2, 3, 4]
>>> L2 = L1[:]
>>> L1[0] = 24
>>> L1
[24, 3, 4]
>>> L2
[2, 3, 4]
```

和Go不同的是，`L1[:]`是复制了L1, L1和L2指向了不同的内存区域。


## list与字符串的转换：

```python
>>> s = "abc"
>>> L = list(s)
>>> L
['a', 'b', 'c']
>>> L[0] = 'd'
>>> L
['d', 'b', 'c']
>>> s = ''.join(L)
>>> s
'dbc'
```

## 增加元素

### 合并两个列表

`L1 + L2`

```python
>>> L1 = [0, 1]
>>> L2 = [3, 4]
>>> L1 + L2
[0, 1, 3, 4]
```

### 重复列表元素n次

`L1 * 3`

```python
>>> L1 = [1, 2]
>>> L1 * 3
[1, 2, 1, 2, 1, 2]
```

重复复制的是引用而不是深拷贝：

```python
>>> L1 = [1, 2, 3]
>>> X = [L1] * 3
>>> X
[[1, 2, 3], [1, 2, 3], [1, 2, 3]]
>>> L1[0] = 4
>>> X
[[4, 2, 3], [4, 2, 3], [4, 2, 3]]
```

当 `L1` 被改变时，`X` 中所有元素都受影响。

### append

```python
>>> L = [1, 2]
>>> L.append([3, 4])
>>> L
[1, 2, [3, 4]]
```

用分片模拟append():

`L[len(L):] = [X]` 在后端插入

`L[:0] = [X]` 在前端插入

### extend

```python
>>> L = [1, 2]
>>> L.extend([3, 4])
>>> L
[1, 2, 3, 4]
```



## 判断

### 是否包含某个元素

```python
>>> 1 in L1
True
```

## 分片赋值

```python
>>> L = [1, 2, 3]
>>> L[1:3] = [4, 5, 6]
>>> L
[1, 4, 5, 6]
>>> L[3:] = []
>>> L
[1, 4, 5]
>>> L[1:2] = [6, 7, 8]
>>> L
[1, 6, 7, 8, 5]
```

对于`L[1:3] = [4, 5, 6]`首先删除`L[1:3]`,然后将`[4, 5, 6]`插入到`L[0]`之后，类似链表。

如果等号前后出现重叠也是可以的，python会先计算出右边的值然后再删除左边的值：

```python
>>> L = [1, 2, 3, 4]
>>> L[0:3] = L[1:4]
>>> L
[2, 3, 4, 4]
```

分片是复制了一遍：

```python
t = [1, 2, 3]
t2 = t[:]
t2[0] = 4
# 此时t[0]没有变
```

## 排序

```python
list = [2, 3, 1]
list.sort()
print list  # [1, 2, 3]
```


## 某个元素出现的次数

```python
t = [1, 1, 3, 4, 3]
t.count(1)  # 2
```



# 元组tuple

* 不可变
* 任意类型

```python
>>> T = (1, 2, 3)
>>> len(T)
3
>>> T[1]
2
>>> T + (4, 5)
(1, 2, 3, 4, 5)
>>> T[1] = 9  # 不能改变元组中元素的值
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
```

```python
>>> (1, 2) + (3, 4)
(1, 2, 3, 4)
>>> (1, 2) * 4
(1, 2, 1, 2, 1, 2, 1, 2)
>>> T = (1, 2, 3, 4)
>>> T[1:3]
(2, 3)
```

元组也可以不加括号：

```python
>>> x = 1,2,3
>>> x
(1, 2, 3)
```

加括号的不一定是元组：

```python
>>> x = (1)  # 这是数字1
>>> x
1
>>> x = (1,)  # 加 ',' 是为了避免歧义，强制认为这是元组
>>> x
(1,)
```



## 排序

方法1：先将元组转换成列表，排序，然后转换成元组

```python
t = ('c', 'b', 'd', 'a')
tmp = list(t)
tmp.sort()
t = tuple(tmp)
print t
```

方法2：使用 `sorted`

```python
t = ('c', 'b', 'd', 'a')
t = sorted(t)
print t
```

