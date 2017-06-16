# 字典map

## 创建

```python
D = {
    'food': 'Spam',
    'quantity': 4,
    'color': 'pink'
}
```

或者：

```python
D = {}
D['food'] = 'Spam'
D['quantity'] = 4
```

或者(这种方式下键必须是字符串)：

```python
>>> D = dict(name='bob', age=20)
>>> D
{'age': 20, 'name': 'bob'}
```

嵌套

```python
res = {
    'name': {'first': 'Bob', 'last': 'Smith'},
    'job': ['dev', 'mgr'],
    'age': 40.5
}
print(res)
print(res['name']['first'])
```

### 元组可以作为字典的key

```python
matrix = {}
matrix[(2, 3, 4)] = 88
matrix[(7, 8, 9)] = 99
print matrix[(2, 3, 4)]
```



## 访问

```python
D['food']
```

## 修改值

```python
D['quantity'] += 1
```

## 遍历

```python
for k in res.keys():
	print(k, res[k])
```

```python
>>> D = {'spam': 2, 'ham': 1, 'eggs': 3}
>>> for it in D.items():
...     print(it[0], it[1])
...
('eggs', 3)
('ham', 1)
('spam', 2)
```

```python
m = {
	'a': 'A',
	'b': 'B'
}
for (key, value) in m.items():
	print(key, '=>', value)
```

## 判断键是否存在

访问不存在的键将会报错。

```python
if 'name' in res:
	print("has name")

if not 'a' in res:
	print("hasn't a")
```

或者使用带默认值的get:

```python
v = res.get('a', 0)
```

或者通过if-else:

```python
v = res['a'] if 'a' in res else 0
```

## update合并

```python
>>> D1 = {'a':1, 'b':2}
>>> D2 = {'a':3, 'c':4}
>>> D1.update(D2)
>>> D1
{'a': 3, 'c': 4, 'b': 2}
>>> D2
{'a': 3, 'c': 4}
```

## 排序

```python
m = {"a": 1, "c": 2, "b": 3}
items = m.items()
items.sort()
print m  # {'a': 1, 'c': 2, 'b': 3}
print items  # [('a', 1), ('b', 3), ('c', 2)]
```

```python
import operator

t = {"a":1, "c":2, "b":3}
sorted(t.iteritems(), key=operator.itemgetter(1), reverse=True)

# 输出（按value由大到小排序），注意这是元组的列表
[('b', 3), ('c', 2), ('a', 1)]
```

`iteritems()` 返回一个迭代器，遍历字典的所有元素

`operator.itemgetter(1)` 返回一个函数，这个函数接收一个列表，并且返回第1个元素，如果传入0就是按key排序了。

```python
In [8]: m
Out[8]: {'a': 1, 'b': 10, 'c': 3, 'd': 2}

In [9]: [{k: m[k]} for k in sorted(m.keys())]
Out[9]: [{'a': 1}, {'b': 10}, {'c': 3}, {'d': 2}]
```



## 拷贝

```python
d = {"name": "abc"}
d2 = d.copy()
```





# 集合set

字符集合：

```python
>>> x = set('abcde')
>>> x
set(['a', 'c', 'b', 'e', 'd'])
```

- 集合只能包含不可变对象类型（因为要比较两个元素是否相等），列表、字典不能嵌入到集合中。

## 集合的操作

```python
x = set('abcde')
y = set('bdxyz')
print('e' in x)  # True
print(x - y)  # set(['a', 'c', 'e'])
print(x | y)  # set(['a', 'c', 'b', 'e', 'd', 'y', 'x', 'z'])
print(x & y)  # set(['b', 'd'])
print(x ^ y)  # set(['a', 'c', 'e', 'y', 'x', 'z'])
print(x > y, x < y)  # (False, False)
```

