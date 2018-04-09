<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [array](#array)
	- [创建](#创建)
		- [zeros](#zeros)
	- [shape 维度](#shape-维度)
	- [tile 重复](#tile-重复)
	- [差值](#差值)
	- [平方](#平方)
	- [sum](#sum)
	- [argsort](#argsort)
	- [分片赋值](#分片赋值)
	- [min, max](#min-max)
- [参考](#参考)

<!-- /TOC -->


# array

## 创建

```py
>>> np.array([1, 2, 3])
array([1, 2, 3])
```

Upcasting:

```py
>>> np.array([1, 2, 3.0])
array([ 1.,  2.,  3.])
```

More than one dimension:

```py
>>> np.array([[1, 2], [3, 4]])
array([[1, 2],
       [3, 4]])
```

Minimum dimensions 2:

```py
>>> np.array([1, 2, 3], ndmin=2)
array([[1, 2, 3]])
```

Type provided:

```py
>>> np.array([1, 2, 3], dtype=complex)
array([ 1.+0.j,  2.+0.j,  3.+0.j])
```

Data-type consisting of more than one element:

```py
>>> x = np.array([(1,2),(3,4)],dtype=[('a','<i4'),('b','<i4')])
>>> x['a']
array([1, 3])
```

Creating an array from sub-classes:

```py
>>> np.array(np.mat('1 2; 3 4'))
array([[1, 2],
       [3, 4]])
>>> np.array(np.mat('1 2; 3 4'), subok=True)
matrix([[1, 2],
        [3, 4]])
```

### zeros

`zeros(shape, dtype=float, order='C')`: 返回零矩阵.

* shape 为列表时从后向前分别表示低维度到高维度的数量.

```py
>>> np.zeros(5)
array([ 0.,  0.,  0.,  0.,  0.])
>>> np.zeros((5,), dtype=np.int)
array([0, 0, 0, 0, 0])
>>> np.zeros((2, 1))  # 两行一列
array([[ 0.],
       [ 0.]])
>>> s = (2,2)
>>> np.zeros(s)
array([[ 0.,  0.],
       [ 0.,  0.]])
>>> np.zeros((2,), dtype=[('x', 'i4'), ('y', 'i4')]) # custom dtype
array([(0, 0), (0, 0)],
      dtype=[('x', '<i4'), ('y', '<i4')])
```


## shape 维度

返回维度的元组(Tuple of array dimensions). 从右向左分别表示低维度到高纬度的长度.

```py
>>> x = np.array([1, 2, 3, 4])
>>> x.shape
(4,)
>>> y = np.zeros((2, 3, 4))
>>> y.shape
(2, 3, 4)
>>> y.shape = (3, 8)
>>> y
array([[ 0.,  0.,  0.,  0.,  0.,  0.,  0.,  0.],
       [ 0.,  0.,  0.,  0.,  0.,  0.,  0.,  0.],
       [ 0.,  0.,  0.,  0.,  0.,  0.,  0.,  0.]])
>>> y.shape = (3, 6)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: total size of new array must be unchanged
```


## tile 重复

`numpy.tile(A, reps)`: 将 A 各维度重复 reps 次.

如果 A 的维度大于 reps 的长度, `A.ndim > len(reps)`, 在 reps 前补1, 比如 `A = np.array([[1,2], [2,3]])`, A 的维度是2, 如果 reps 是 2, 则扩充至 `(1, 2)`.

重复时按照先低维度后高维度计算, 比如 `A = np.array([[1,2], [2,3]])`, `resp = (2, 3)`, 先将 A 中最低维度重复3次, 变成:

```py
[
    [1, 2, 1, 2, 1, 2],
    [3, 4, 3, 4, 3, 4]
]
```

把 A 中低维度的数组当做一个元素, `A = [a1, a2]`, 再重复2次:

```py
[a1, a2, a1, a2]
```

替换下变成:

```py
[
    [1, 2, 1, 2, 1, 2],
    [3, 4, 3, 4, 3, 4],
    [1, 2, 1, 2, 1, 2],
    [3, 4, 3, 4, 3, 4],
]
```

文档示例:

```py
>>> a = np.array([0, 1, 2])

>>> np.tile(a, 2)
array([0, 1, 2, 0, 1, 2])

>>> np.tile(a, (2, 2))
array([[0, 1, 2, 0, 1, 2],
       [0, 1, 2, 0, 1, 2]])

>>> np.tile(a, (2, 3))
array([[0, 1, 2, 0, 1, 2, 0, 1, 2],
       [0, 1, 2, 0, 1, 2, 0, 1, 2]])

>>> np.tile(a, (2, 1, 2))
array([[[0, 1, 2, 0, 1, 2]],
       [[0, 1, 2, 0, 1, 2]]])

>>> b = np.array([[1, 2], [3, 4]])
>>> np.tile(b, 2)
array([[1, 2, 1, 2],
       [3, 4, 3, 4]])

>>> np.tile(b, (2, 1))
array([[1, 2],
       [3, 4],
       [1, 2],
       [3, 4]])

>>> c = np.array([1,2,3,4])
>>> np.tile(c,(4,1))
array([[1, 2, 3, 4],
       [1, 2, 3, 4],
       [1, 2, 3, 4],
       [1, 2, 3, 4]])
```


## 差值

相同维度:

```py
>>> a = np.array([[1, 2], [3, 4]])
>>> b = np.array([[0, 1], [2, 3]])
>>> a - b
array([[1, 1],
       [1, 1]])
```


二维减一维: (每一维都减)

```py
>>> a = np.array([[1, 2], [3, 4]])
>>> b = np.array([0, 1])
>>> a - b
array([[1, 1],
       [3, 3]])
```


## 平方

```py
>>> a = np.array([[1, 2], [3, 4]])
>>> a**2
array([[ 1,  4],
       [ 9, 16]])
```


## sum

```py
>>> np.sum([0.5, 1.5])
2.0
>>> np.sum([0.5, 0.7, 0.2, 1.5], dtype=np.int32)  # 每个元素先类型转换再求和
1
>>> np.sum([[1, 2], [3, 4]])  # 所有的都加起来
10
>>> np.sum([[0, 1], [0, 5]], axis=0)    #axis=0是按列求和
array([0, 6])
>>> np.sum([[0, 1], [0, 5]], axis=1)    #axis=1 是按行求和
array([1, 5])
```


## argsort

`argsort(a, axis=-1, kind='quicksort', order=None)`: 返回排序后的**索引值**.

```py
One dimensional array:一维数组

>>> x = np.array([3, 1, 2])
>>> np.argsort(x)
array([1, 2, 0])

Two-dimensional array:二维数组

>>> x = np.array([[0, 3], [2, 2]])
>>> np.argsort(x, axis=0) #按列排序
array([[0, 1],
        [1, 0]])

>>> np.argsort(x, axis=1) #按行排序
array([[0, 1],
        [0, 1]])
>>> np.argsort(x)  # 默认升序排列
array([[0, 1],
       [0, 1]])

>>> x = np.array([3, 1, 2])
>>> np.argsort(x) #按升序排列
array([1, 2, 0])

>>> np.argsort(-x) #按降序排列
array([0, 2, 1])

>>> x[np.argsort(x)] #通过索引值排序后的数组, 二维数组不适合
array([1, 2, 3])

>>> x[np.argsort(-x)]
array([3, 2, 1])

# 另一种方式实现按降序排序：
>>> a = x[np.argsort(x)]
>>> a
array([1, 2, 3])
>>> a[::-1]
array([3, 2, 1])
```


## 分片赋值

二维数组分片赋值:

```py
>>> a = np.zeros([3, 2])
>>> a
array([[ 0.,  0.],
       [ 0.,  0.],
       [ 0.,  0.]])

>>> a[0, :] = [1, 2]
>>> a
array([[ 1.,  2.],
       [ 0.,  0.],
       [ 0.,  0.]])
```


## min, max

```py
numpy.amin(a, axis=None, out=None, keepdims=<class numpy._globals._NoValue>)
numpy.amax(a, axis=None, out=None, keepdims=<class numpy._globals._NoValue>)
```

* `axis == 0` 表示返回每一列的极值.
* `axis == 1` 表示返回每一行的极值.

```py
>>> a = np.arange(4).reshape((2,2))
>>> a
array([[0, 1],
       [2, 3]])
>>> np.amax(a)           # Maximum of the flattened array
3
>>> np.amax(a, axis=0)   # Maxima along the first axis
array([2, 3])
>>> np.amax(a, axis=1)   # Maxima along the second axis
array([1, 3])

# 也可以 a.min(axis=0)
```



# 参考

- [CS231n课程笔记翻译：Python Numpy教程](https://zhuanlan.zhihu.com/p/20878530?refer=intelligentunit)
