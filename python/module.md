<!-- TOC -->

- [1. 简介](#1-简介)
    - [1.1. 3种方式](#11-3种方式)
    - [1.2. 查看已经导入的模块](#12-查看已经导入的模块)
    - [1.3. 搜索路径](#13-搜索路径)
    - [1.4. import & from](#14-import--from)
    - [1.5. 列举模块的属性](#15-列举模块的属性)
    - [1.6. 判断对象是否是模块](#16-判断对象是否是模块)
- [2. 模块的编写](#2-模块的编写)
    - [2.1. import](#21-import)
    - [2.2. from](#22-from)
        - [2.2.1. from *](#221-from-)
    - [2.3. 重载模块](#23-重载模块)
        - [2.3.1. 过渡性模块重载](#231-过渡性模块重载)
    - [2.4. 包导入](#24-包导入)
        - [2.4.1. `__init__.py` 中的 `__all__`](#241-__init__py-中的-__all__)
    - [2.5. 相对导入](#25-相对导入)
        - [2.5.1. 为什么使用相对导入](#251-为什么使用相对导入)
- [3. 在模块中隐藏数据](#3-在模块中隐藏数据)
- [4. 修改模块搜索路径](#4-修改模块搜索路径)
- [5. 用字符串导入模块](#5-用字符串导入模块)
    - [5.1. 构造python代码执行](#51-构造python代码执行)
    - [5.2. 使用内置的`__import__`函数](#52-使用内置的__import__函数)

<!-- /TOC -->

# 1. 简介

* 每个文件是一个模块。

## 1.1. 3种方式

* `import`     以一个整体获取模块。
* `from`       从一个模块中获取特定变量名。
* `imp.reload` 不终止程序的情况下重新载入模块。

## 1.2. 查看已经导入的模块

已导入的模块保存在 `sys.modules` 字典中。

```python
print sys.modules.keys()
```

## 1.3. 搜索路径

搜索路径保存在 `sys.path` 中。程序启动时设置这个变量，主要从下面这些路径中查找需要导入的路径：

1. 程序的主目录。顶层文件所在的目录。
2. PYTHONPATH 环境变量指定的目录。
3. 标准库目录。
4. 任何 .pth 文件的内容。

## 1.4. import & from

被导入的模块 const.py:

```python
A = "hello"
B = "world"
```

hello.py:

```python
import const

print(const.A, const.B)
```

输出：`('hello', 'world')`

只导入特定属性:

```python
from const import A

print(A)  # hello
```

## 1.5. 列举模块的属性

```python
import const

print(dir(const))
```

```python
['A', 'B', '__builtins__', '__doc__', '__file__', '__name__', '__package__']
```

## 1.6. 判断对象是否是模块

```py
type(obj) == types.ModuleType
```


# 2. 模块的编写

* 模块名只能包含字母、数字、下划线。
* import 和 from 是赋值语句。

## 2.1. import

```python
import module1

module1.function()
```

多级路径使用 `.` 相隔：

```py
import dir1.dir2.mod
```

使用 as 扩展：

```py
import modulename as name
```

相当于：

```py
import modulename
name = modulename
del modulename
```

## 2.2. from

```python
from module1 import function
function()
```

多级路径：

```py
from dir1.dir2 import mod
```

使用 as 扩展：

```py
from modulename import attrname as name
```

### 2.2.1. from *

取得模块中所有赋了值的变量的拷贝。

```python
from module1 import *
function()
```

## 2.3. 重载模块

* reload 是内置函数，而不是语句。
* 传给 reload 的是已经存在的模块对象。
* 重载会影响所有使用 import 的客户端。
* 重载之前的 from 不受影响，之后的 from 会受影响。

```python
import module
... use module.attributes...

from imp import reload # in 3.0
reload(module)
... use module.attributes...
```

### 2.3.1. 过渡性模块重载

例如：A导入模块B和C，重载A时，B和C不会重载，只会重新执行导入语句，获取的是已经载入的B和C的模块对象。



## 2.4. 包导入

比如：

```py
import dir1.dir2.mod
```

在 `dir1` 和 `dir2` 目录内都必须包含 `__init__.py` 文件。上面的语句执行时会先执行 `dir1` 下的 init 文件，在执行 `dir2` 下的 init 文件，在初始化 mod 文件。

`__init__.py` 可以防止这样的情况：导入路径 `a/b` 和 `c/d` 中下都有目录 `dir`，但是只有第二个 `dir` 是有效的，为了防止 python 自动使用第一个 `dir`，可以只在第二个 `dir` 下放置 `__init__.py`。

### 2.4.1. `__init__.py` 中的 `__all__`

当使用 `from mod import *` 语句导入 mod 时，如果在 mod 目录下的 `__init__.py` 文件中：

1. 定义了 `__all__`  : 导入指定的子模块。
2. 没有定义 `__all__`: 只导入赋值语句定义的变量名和 `__init__.py` 导入的模块。


## 2.5. 相对导入

* 适用于只在包内导入。
* 只适用于 from 语句。

```py
from . import spam
```

在当前目录下导入 spam 模块，找不到的话到 `sys.path` 中找(python 2.x)。

```py
from .spam import name
```

在当前目录下的spam文件中导入name.

```py
from .. import spam # 从当前包的父目录的相对导入
```

```py
from ..E import X   # 从当前目录下的父目录的E目录下查找X模块
```

### 2.5.1. 为什么使用相对导入

加入你的项目中有 `string.py` 模块，但是你想导入标准库的 string, 这时候Python总是导入你项目下的 string 模块而无法导入标准库里的 string 模块。

Python 3.x 里的解决方案是：

> `import string` 时总是通过 `sys.path` 的绝对导入搜索，而不搜索当前项目。使用 `from . import string` 则只在当前项目下查找。


# 3. 在模块中隐藏数据

* 数据隐藏是一种惯例，而非语法约束。没有办法阻止别人修改模块内的变量。
* 变量名前加下划线（比如 `_X`）防止 `from *` 的导入。
* 通过 `__all__` 指明需要导出的变量。（仅对 `from *` 有效）

# 4. 修改模块搜索路径

在程序中修改 `sys.path` 列表，对之后的导入都有影响。


# 5. 用字符串导入模块

## 5.1. 构造python代码执行

```py
modname = "string"
exec("import " + modname)
string
```

## 5.2. 使用内置的`__import__`函数

```py
modname = "string"
string = __import__(modname)
string
```

