# 注释

## 中文注释

在开头加上 `# -*- coding: utf-8 -*`



# 在命令行加载文件

假设当前文件有一个trees.py

```py
import sys
sys.path.append("./")

import trees

# 修改文件后重新加载
reload(trees)
```


# 启动参数

```python
import sys

print sys.argv
```

执行 `python test.py hello ad ds "dd"`
输出: `['test.py', 'hello', 'ad', 'ds', 'dd']`


# 文档

## 通过 `__doc__` 查看各种文档

```py
""" for test something """


def func():
    """ I am func.doc """
    pass


class Spam(object):
    """ I am spam.doc """

    def method(self):
        """ I am spam.method.doc """
        pass


def main():
    """ the main function """
    print __doc__
    print func.__doc__
    print Spam.__doc__
    print Spam.method.__doc__
```





# 检测变量类型

```python
>>> type(f)
<type 'file'>
```

```python
L = []
if type(L) == type([]):
	print('yes')

if type(L) == list:
	print('yes')

if isinstance(L, list):
	print('yes')
```



# pip

## 安装到指定目录

用户目录下面，.pip目录下建立pip.conf文件

```
[install]

install-option=--prefix=~/.local
```



# 打印错误堆栈信息

```python
import traceback

except Exception as e:
    print traceback.format_exc()
```

