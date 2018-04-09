<!-- TOC -->

- [1. 注释](#1-注释)
    - [1.1. 中文注释](#11-中文注释)
- [2. 在命令行加载文件](#2-在命令行加载文件)
- [3. 启动参数](#3-启动参数)
    - [argparse](#argparse)
        - [基本使用](#基本使用)
        - [positional arguments](#positional-arguments)
        - [optional arguments](#optional-arguments)
- [4. 文档](#4-文档)
    - [4.1. 通过 `__doc__` 查看各种文档](#41-通过-__doc__-查看各种文档)
- [5. 检测变量类型](#5-检测变量类型)
- [6. pip](#6-pip)
    - [6.1. 安装到指定目录](#61-安装到指定目录)
- [7. 打印错误堆栈信息](#7-打印错误堆栈信息)

<!-- /TOC -->


# 1. 注释

## 1.1. 中文注释

在开头加上 `# -*- coding: utf-8 -*`



# 2. 在命令行加载文件

假设当前文件有一个trees.py

```py
import sys
sys.path.append("./")

import trees

# 修改文件后重新加载
reload(trees)
```


# 3. 启动参数

```python
import sys

print sys.argv
```

执行 `python test.py hello ad ds "dd"`
输出: `['test.py', 'hello', 'ad', 'ds', 'dd']`


## argparse

### 基本使用

基本使用, 什么参数也没有:
```py
import argparse

parser = argparse.ArgumentParser()
parser.parse_args()
```

运行: `python test.py --help` 将看到下面的输出:

```
usage: test.py [-h]

optional arguments:
  -h, --help  show this help message and exit
```

### positional arguments

```py
import argparse

parser = argparse.ArgumentParser()
parser.add_argument('echo', help='echo the string you use here')
args = parser.parse_args()
print args.echo
```

`python test.py --help`:
```
usage: test.py [-h] echo

positional arguments:
  echo        echo the string you use here

optional arguments:
  -h, --help  show this help message and exit
```

`python test.py hello`: 得到输出 hello. (注意不是 `python test.py --echo=hello`)


指定多个参数并指定类型:
```py
import argparse

parser = argparse.ArgumentParser()
parser.add_argument('echo', help='echo the string you use here')
parser.add_argument('num', type=int)  # 默认是 string, 这里需要指明是 int.
args = parser.parse_args()
print args.echo
print args.num**2
```

`python test.py hello 3`: `hello 9`



### optional arguments

```py
import argparse

parser = argparse.ArgumentParser()
parser.add_argument(
    '--verbose', help='increase output verbosity',
    action='store_true')  # 如果指定了就置为 True.
args = parser.parse_args()
if args.verbose:
    print 'verbosity truned on'
```

```
➜  python test.py --verbose
verbosity truned on
➜  python test.py --verbose true
usage: test.py [-h] [--verbose]
test.py: error: unrecognized arguments: true
```

只需要指定 `--verbose`, 不能设置值.

**short options**

```py
parser.add_argument(
    '-v', '--verbose', help='increase output verbosity',
    action='store_true')  # 如果指定了就置为 True.
```

```
➜  python test.py -v
verbosity truned on
```

**指定类型**

```py
import argparse

parser = argparse.ArgumentParser()
parser.add_argument(
    '-v', '--verbose', help='increase output verbosity', type=int)
args = parser.parse_args()

print args.verbose
# print args.v # 没有这个
```

```
➜  python test.py -v 2
2
➜  python test.py -v=2
2
➜  python test.py -v = 2
usage: test.py [-h] [-v VERBOSE]
test.py: error: argument -v/--verbose: invalid int value: '='
```

**添加可选值**

```py
import argparse

parser = argparse.ArgumentParser()
parser.add_argument(
    '-v',
    '--verbose',
    help='increase output verbosity',
    type=int,
    choices=[1, 2, 3])
args = parser.parse_args()

print args.verbose
```

```
➜  python test.py -v 2
2
➜  python test.py -v 4
usage: test.py [-h] [-v {1,2,3}]
test.py: error: argument -v/--verbose: invalid choice: 4 (choose from 1, 2, 3)
```


**默认值**

```py
parser = argparse.ArgumentParser()
parser.add_argument(
    '-v', '--verbose', help='increase output verbosity', type=int, default=3)
args = parser.parse_args()
```

```
➜  python test.py
3
➜  python test.py -v
usage: test.py [-h] [-v VERBOSE]
test.py: error: argument -v/--verbose: expected one argument
➜  python test.py -v 2
2
```







# 4. 文档

## 4.1. 通过 `__doc__` 查看各种文档

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





# 5. 检测变量类型

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



# 6. pip

## 6.1. 安装到指定目录

用户目录下面，.pip目录下建立pip.conf文件

```
[install]

install-option=--prefix=~/.local
```



# 7. 打印错误堆栈信息

```python
import traceback

except Exception as e:
    print traceback.format_exc()
```
