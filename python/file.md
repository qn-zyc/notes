<!-- TOC -->

- [1. 路径](#1-路径)
    - [1.1. join](#11-join)
    - [1.2. 绝对路径](#12-绝对路径)
- [2. 文件信息](#2-文件信息)
- [3. 创建](#3-创建)
    - [3.1. 创建一个可写的文件](#31-创建一个可写的文件)
- [4. 读取](#4-读取)
    - [4.1. 读取文件](#41-读取文件)
        - [4.1.1. 分批读取](#411-分批读取)
    - [4.2. 读取目录](#42-读取目录)
- [5. csv](#5-csv)
    - [5.1. 读取](#51-读取)
    - [5.2. 写入](#52-写入)
        - [5.2.1. windows下防止乱码](#521-windows下防止乱码)
- [6. open()](#6-open)
    - [6.1. 语法](#61-语法)
    - [6.2. 常用mode组合](#62-常用mode组合)

<!-- /TOC -->


# 1. 路径

## 1.1. join

```python
os.path.join(dir, file)
```

## 1.2. 绝对路径

```python
abs_path = os.path.abspath(file_name)
```


# 2. 文件信息

```py
import os, stat

file_stat = os.stat(file_name)
# 文件大小
file_stat[stat.ST_SIZE
# 创建时间
file_stat[stat.ST_CTIME]
# 最后修改时间
file_stat[stat.ST_MTIME]
# 最后访问时间
file_stat[stat.ST_ATIME]
```


# 3. 创建

## 3.1. 创建一个可写的文件

```python
>>> f = open('data.txt', 'w')
>>> f.write('hello\n')
>>> f.write('world\n')
>>> f.close()  # close to flush output buffers to disk
```

# 4. 读取

## 4.1. 读取文件

```python
try:
	f = open("data.txt", 'r')
	print f.read()  # 会读取所有内容
finally:
	if f:
		f.close()
```

如果文件不存在，open()函数就会抛出一个IOError的错误。

下面是简洁形式：

```python
with open("data.txt", "r") as f:
	print f.read()
```

```python
with open("data.txt", "r") as f:
	for line in f:
		print line
```


不需要调用close关闭。

### 4.1.1. 分批读取

- read(size)方法，每次最多读取size个字节的内容
- readline()可以每次读取一行内容
- readlines()一次读取所有内容并按行返回list

```python
with open("data.txt", "r") as f:
	while True:
		line = f.readline()
		if not line:
			break
		print line
```

## 4.2. 读取目录

```python
os.listdir('/path')  # 返回子目录或文件的名称
```



# 5. csv

## 5.1. 读取

```python
>>> import csv
>>> with open('eggs.csv', 'rb') as csvfile:
...     spamreader = csv.reader(csvfile, delimiter=' ', quotechar='|')
...     for row in spamreader:
...         print ', '.join(row)
Spam, Spam, Spam, Spam, Spam, Baked Beans
Spam, Lovely Spam, Wonderful Spam
```



## 5.2. 写入

```python
import csv
with open('eggs.csv', 'wb') as csvfile:
    spamwriter = csv.writer(csvfile, delimiter=' ',
                            quotechar='|', quoting=csv.QUOTE_MINIMAL)
    spamwriter.writerow(['Spam'] * 5 + ['Baked Beans'])
    spamwriter.writerow(['Spam', 'Lovely Spam', 'Wonderful Spam'])
```

### 5.2.1. windows下防止乱码

```python
    with open(filename, 'wb') as f:
        # windows下防止乱码
        f.write('\xEF\xBB\xBF')
        w = csv.writer(f)
        w.writerow(title)
        for d in data:
            w.writerow(d)
```



# 6. open()

## 6.1. 语法

```
open(file[, mode[, buffering[, encoding[, errors[, newline[, closefd=True]]]]]])
```

* `file`: 文件路径
* `mode`: 打开模式
	* `r`: 只读(默认值).
	* `w`: 只写, 会先清空文件。
	* `a`: 追加写。
	* `b`: 二进制模式。
	* `t`: 文本模式(默认).
	* `+`: 读写模式。
	* `U`: 通用换行符。
* `buffering`: 
	* 取值0: buffer关闭，只适用于二进制模式。
	* 取值1: line buffer, 只适用于文本模式。
	* 取值大于1: 初始化的buffer大小。
* `encoding`: 返回的数据使用何种编码，一般采用 utf8 或者 gbk.
* `errors`: 
	* `strict`: 字符编码出现错误时会报错。
	* `ignore`: 忽略错误，继续执行下面的程序。
* `newline`: 可选值有 `None`, `\n`, `\r`, `\r\n`, 区分换行符，只对文本模式有效。
* `closefd`: 默认为 True, 当为 False 时文件名需为文件描述符。

## 6.2. 常用mode组合

* `r` 或 `rt`: 默认模式，文本模式读.
* `rb`: 读二进制文件.
* `w` 或 `wt`: 文本模式写，打开前文件存储被清空
* `wb`: 二进制写，文件存储同样被清空
* `a`: 追加模式，只能写在文件末尾
* `a+`: 可读写模式，写只能写在文件末尾
* `w+`: 可读写，与a+的区别是要清空文件内容
* `r+`: 可读写，与a+的区别是可以写到文件任何位置