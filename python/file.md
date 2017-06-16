# 路径

## join

```python
os.path.join(dir, file)
```

## 绝对路径

```python
abs_path = os.path.abspath(file_name)
```


# 文件信息

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


# 创建

## 创建一个可写的文件

```python
>>> f = open('data.txt', 'w')
>>> f.write('hello\n')
>>> f.write('world\n')
>>> f.close()  # close to flush output buffers to disk
```

# 读取

## 读取文件

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

### 分批读取

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

## 读取目录

```python
os.listdir('/path')  # 返回子目录或文件的名称
```



# csv

## 读取

```python
>>> import csv
>>> with open('eggs.csv', 'rb') as csvfile:
...     spamreader = csv.reader(csvfile, delimiter=' ', quotechar='|')
...     for row in spamreader:
...         print ', '.join(row)
Spam, Spam, Spam, Spam, Spam, Baked Beans
Spam, Lovely Spam, Wonderful Spam
```



## 写入

```python
import csv
with open('eggs.csv', 'wb') as csvfile:
    spamwriter = csv.writer(csvfile, delimiter=' ',
                            quotechar='|', quoting=csv.QUOTE_MINIMAL)
    spamwriter.writerow(['Spam'] * 5 + ['Baked Beans'])
    spamwriter.writerow(['Spam', 'Lovely Spam', 'Wonderful Spam'])
```

### windows下防止乱码

```python
    with open(filename, 'wb') as f:
        # windows下防止乱码
        f.write('\xEF\xBB\xBF')
        w = csv.writer(f)
        w.writerow(title)
        for d in data:
            w.writerow(d)
```

