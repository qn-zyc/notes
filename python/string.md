* 字符串不可变

# 引号

单双引号都一样：

```python
s = "hello"
s = 'hello'
```

单双引号互相嵌套

```python
s = 'hello "world" !'
s = "hello 'world' !"
```

多行文本（常用于包含像HTML或XML这样的内容）：

```python
>>> s = """aaaaa
... bbbbb'a'a'a
... ccccc"""
>>> s
"aaaaa\nbbbbb'a'a'a\nccccc"
```



# 长度和下标访问

```python
>>> s = "hello"
>>> len(s)
5
>>> s[0]
'h'
>>> s[10]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: string index out of range
```

不适用于中文字符串：

```python
>>> s = "你好"
>>> s[0]
'\xe4'
>>> s[1]
'\xbd'
>>> len(s)
6
```

# 反向索引

```python
>>> s = "hello"
>>> s[-1]
'o'
>>> s[-2]
'l'
>>> s[len(s)-1]
'o'
```

# 分片

```python
>>> s = "abcdefg"
>>> s[1:3]
'bc'
```

包括左边界1但不包括右边界3.

默认值：左边界默认为0，右边界默认为字符串长度。

还可以有第三个参数，表示步进，步进的默认值为1：

```python
>>> s = 'abcdefg'
>>> s[1:len(s):2]
'bdf'
```

步进还可以是负值：

```python
>>> s[len(s):0:-2]
'gec'
```


# 字符串连接

```python
>>> s1 = "abc"
>>> s2 = "def"
>>> s1 + s2
'abcdef'
>>> s1 * 3
'abcabcabc'
```

`+` 表达式不能混合字符串和数字：

```python
s = 'hello'
s + 9  # TypeError: cannot concatenate 'str' and 'int' objects
```



```python
>>> title = "Meaning " 'of' " Life"
>>> title
'Meaning of Life'
```

# 字符串查找

```python
'a' in x
s.find("")  # 返回下标，如果找不到，返回-1
s.index("")  # 返回下标，如果找不到，报错
```

## 在utf8字符串里查找

```python
    if not isinstance(content, unicode):
        content = content.decode('utf8')
    c = len(content)
    if content.find(u'【娱票儿】') >= 0:
        return c
```



# 不转义

使用原始(raw)字符串常量，去掉反斜线转义机制。

```python
>>> path = r'C:\new\text.data'
>>> path
'C:\\new\\text.data'
```

# 转换

```python
>>> int("42"), str(42)
(42, '42')
>>> float("1.5"), str(3.14)
(1.5, '3.14')
```

ASCII码转换：

```python
>>> ord('s')
115
>>> chr(115)
's'
```

# 格式化

```python
>>> 'That is %d %s bird!' % (1, 'dead')
'That is 1 dead bird!'
```

`%[(name)][flags][width][.precision]typecode`

typecode取值如下：

| 代码   | 意义               |
| ---- | ---------------- |
| s    | 字符串或任意对象         |
| r    | s, 但使用repr而不是str |
| c    | 字符               |
| d    | 十进制整数            |
| i    | 整数               |
| u    | 无符号（整数）          |
| o    | 八进制整数            |
| x    | 十六进制整数           |
| X    | x，但打印大写          |
| e    | 浮点指数             |
| E    | e,大写             |
| f    | 浮点数              |
| F    | 浮点数              |
| g    | e或f              |
| G    | E或f              |
| %    | 常量%              |



flags取值：

* 左对齐 `'%-6d' % 1234  => '1234  '`
* 正负号 `'%+6d' % 1234 => ' +1234'`
* 0 补零 `'%06d' % 1234 => '001234'`

浮点数：

```python
>>> x = 1.23
>>> '%-6.2f | %05.2f | %+06.1f' % (x, x, x)
'1.23   | 01.23 | +001.2'
```

使用 * 指定从参数列表中获取width和precision:

```python
>>> '%f, %.2f, %.*f' % (1/3.0, 1/3.0, 4, 1/3.0)
'0.333333, 0.33, 0.3333'
```

使用字典填充：

```python
>>> '%(n)d %(x)s' % {"n":1, "x":"spam"}
'1 spam'
```





# 字符串help

```python
>>> dir(s)
['__add__', '__class__', '__contains__', '__delattr__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getnewargs__', '__getslice__', '__gt__', '__hash__', '__init__', '__le__', '__len__', '__lt__', '__mod__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmod__', '__rmul__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '_formatter_field_name_split', '_formatter_parser', 'capitalize', 'center', 'count', 'decode', 'encode', 'endswith', 'expandtabs', 'find', 'format', 'index', 'isalnum', 'isalpha', 'isdigit', 'islower', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'partition', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill']
```

具体某个属性的作用：

```python
>>> help(s.find)

Help on built-in function find:

find(...)
    S.find(sub [,start [,end]]) -> int

    Return the lowest index in S where substring sub is found,
    such that sub is contained within S[start:end].  Optional
    arguments start and end are interpreted as in slice notation.

    Return -1 on failure.
```


# 模式匹配

```python
import re

match = re.match("Hello[ \t]*(.*)world", "Hello python world")

print(match.groups())  # ('python ',)

print(match.group(1))  # python 0是整个字符串
```


# 编码

## 判断是否是unicode编码

```python
    if not isinstance(content, unicode):
        content = content.decode('utf8')
```



## 编码转换

```python
In [27]: s = '中'

In [28]: s
Out[28]: '\xe4\xb8\xad'  # 这是python自己的编码
    
In [30]: s = s.decode('utf8')  # 把Python的编码转成utf8编码
Out[30]: u'\u4e2d'  # 这是unicode编码
    
In [33]: s.encode('utf8')  # 把utf8编码转换成Python自己的编码
Out[33]: '\xe4\xb8\xad'
```

