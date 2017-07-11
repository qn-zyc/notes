
<!-- TOC -->

- [简介](#简介)
- [re模块](#re模块)
    - [re.compile](#recompile)
        - [Pattern](#pattern)
    - [re.match](#rematch)
    - [re.search](#research)
    - [search() vs. match()](#search-vs-match)
    - [re.escape](#reescape)
    - [re.split](#resplit)
    - [re.findall](#refindall)
    - [re.finditer](#refinditer)
    - [re.sub](#resub)
    - [re.subn](#resubn)
- [参考](#参考)

<!-- /TOC -->

# 简介

* 文档可以在 `re` 模块中查看.
* 数量词默认是贪婪的, 总是尝试匹配尽可能多的字符.


# re模块

```py
import re

pattern = re.compile(r'hello')
match = pattern.match('hello world')
if match:
    print match.group()  # hello
```

## re.compile

将字符串编译为 `Pattern` 对象.

第二个参数是指定匹配模式, 取值可以使用 `|` 相连, 比如 `re.I | re.M`, 另外还可以在字符串中指定模式:

`re.compile('pattern', re.I|re.M)` 与 `re.compile('(?im)pattern')` 等价.

模式可选值:
* `re.I` (`re.IGNORECASE`): 忽略大小写(括号内是完整写法, 下同)
* `re.M` (`re.MULTILINE`): 多行模式，改变'^'和'$'的行为
* `re.S` (`re.DOTALL`): 点任意匹配模式，改变'.'的行为
* `re.L` (`re.LOCALE`): 使预定字符类 `\w \W \b \B \s \S` 取决于当前区域设定
* `re.U` (`re.UNICODE`): 使预定字符类 `\w \W \b \B \s \S \d \D` 取决于unicode定义的字符属性
* `re.X` (`re.VERBOSE`): 详细模式。这个模式下正则表达式可以是多行，忽略空白字符，并可以加入注释。以下两个正则表达式是等价的：
    ```py
    a = re.compile(r"""\d +  # the integral part
                    \.    # the decimal point
                    \d *  # some fractional digits""", re.X)
    b = re.compile(r"\d+\.\d*")
    ```


### Pattern

Pattern 提供了几个属性用于获取表达式相关信息:
* `pattern`: 编译时用的表达式字符串。
* `flags`: 编译时用的匹配模式。数字形式。
* `groups`: 表达式中分组的数量。
* `groupindex`: 以表达式中有别名的组的别名为键、以该组对应的编号为值的字典，没有别名的组不包含在内。

示例:

```py
import re

p = re.compile(r'(\w+) (\w+)(?P<sign>.*)', re.DOTALL)

print "p.pattern:", p.pattern  # p.pattern: (\w+) (\w+)(?P<sign>.*)
print "p.flags:", p.flags  # p.flags: 16
print "p.groups:", p.groups  # p.groups: 3
print "p.groupindex:", p.groupindex  # p.groupindex: {'sign': 3}
```



## re.match

```py
match(string[, pos[, endpos]])
或者: re.match(pattern, string[, flags])
```

* 这个方法将从string的pos下标处起尝试匹配pattern, 如果可匹配, 则返回一个Match对象, 如果无法匹配, 或者匹配未结束就已到达endpos, 则返回None。 
* pos和endpos的默认值分别为0和len(string)；re.match()无法指定这两个参数，参数flags用于编译pattern时指定匹配模式。 
* 注意：这个方法并不是完全匹配。当pattern结束时若string还有剩余字符，仍然视为成功。想要完全匹配，可以在表达式末尾加上边界匹配符'$'。 

`result = re.match(pattern, string)` 等同于如下代码:

```py
prog = re.compile(pattern)
result = prog.match(string)
```

result 是一个 Match 对象 (`re.MatchObject`), 包含了一些匹配信息:

属性:
* `string`: 匹配时使用的文本。
* `re`: 匹配时使用的Pattern对象。
* `pos`: 文本中正则表达式开始搜索的索引, 值与 `Pattern.match()` 和 `Pattern.seach()` 方法的同名参数相同。
* `endpos`: 文本中正则表达式结束搜索的索引, 值与 `Pattern.match()` 和 `Pattern.seach()` 方法的同名参数相同。
* `lastindex`: 最后一个被捕获的分组在文本中的索引。如果没有被捕获的分组，将为None。
* `lastgroup`: 最后一个被捕获的分组的别名。如果这个分组没有别名或者没有被捕获的分组，将为None。

方法:
* `group([group1, …])`: 
    获得一个或多个分组截获的字符串；指定多个参数时将以元组形式返回。group1可以使用编号也可以使用别名；编号0代表整个匹配的子串；不填写参数时，返回group(0)；没有截获字符串的组返回None；截获了多次的组返回最后一次截获的子串。
* `groups([default])`: 
    以元组形式返回全部分组截获的字符串。相当于调用group(1,2,…last)。default表示没有截获字符串的组以这个值替代，默认为None。
* `groupdict([default])`: 
    返回以有别名的组的别名为键、以该组截获的子串为值的字典，没有别名的组不包含在内。default含义同上。
* `start([group])`: 
    返回指定的组截获的子串在string中的起始索引（子串第一个字符的索引）。group默认值为0。
* `end([group])`: 
    返回指定的组截获的子串在string中的结束索引（子串最后一个字符的索引+1）。group默认值为0。
* `span([group])`: 
    返回 `(start(group), end(group))`。
* `expand(template)`: 
    将匹配到的分组代入template中然后返回。template中可以使用 `\id` 或 `\g<id>`、`\g<name>` 引用分组，但不能使用编号0。`\id` 与 `\g<id>` 是等价的；但 `\10` 将被认为是第10个分组，如果你想表达 `\1` 之后是字符'0'，只能使用 `\g<1>0`。


示例:

```py
import re

m = re.match(r'(\w+) (\w+)(?P<sign>.*)', 'hello world!')

print "m.string:", m.string  # m.string: hello world!
print "m.re:", m.re  # m.re: <_sre.SRE_Pattern object at 0x105a0c8b0>
print "m.pos:", m.pos  # m.pos: 0
print "m.endpos:", m.endpos  # m.endpos: 12
print "m.lastindex:", m.lastindex  # m.lastindex: 3
print "m.lastgroup:", m.lastgroup  # m.lastgroup: sign

print "m.group(1,2):", m.group(1, 2)  # m.group(1,2): ('hello', 'world')
print "m.groups():", m.groups()  # m.groups(): ('hello', 'world', '!')
print "m.groupdict():", m.groupdict()  # m.groupdict(): {'sign': '!'}
print "m.start(2):", m.start(2)  # m.start(2): 6
print "m.end(2):", m.end(2)  # m.end(2): 11
print "m.span(2):", m.span(2)  # m.span(2): (6, 11)
print r"m.expand(r'\2 \1\3'):", m.expand(r'\2 \1\3')  # m.expand(r'\2 \1\3'): world hello!
```



## re.search

```py
search(string[, pos[, endpos]])
或者: re.search(pattern, string[, flags])
```

这个方法用于查找字符串中可以匹配成功的子串. 从string的pos下标处起尝试匹配pattern，如果pattern结束时仍可匹配，则返回一个Match对象；若无法匹配，则将pos加1后重新尝试匹配；直到pos=endpos时仍无法匹配则返回None。 

pos和endpos的默认值分别为0和len(string))；re.search()无法指定这两个参数，参数flags用于编译pattern时指定匹配模式。 

示例:

```py
import re

pattern = re.compile(r'world')
match = pattern.search('hello world!')  # 使用 match() 无法成功
if match:
    print match.group()  # world
```


## search() vs. match()

`match()` 从字符串开始位置匹配, 如果字符串开始位置不匹配的话则匹配失败.

`search()` 只要从字符串任意位置能匹配就算成功.

例如:

```py
>>> re.match("c", "abcdef")    # No match
>>> re.search("c", "abcdef")   # Match
```

使用 `^` 字符可以让 `search()` 从字符串开始位置开始匹配:

```py
>>> re.match("c", "abcdef")    # No match
>>> re.search("^c", "abcdef")  # No match
>>> re.search("^a", "abcdef")  # Match
```

当使用 `MULTILINE` 模式时 `match()` 仅仅从字符串开始位置匹配, 而使用 `^` 的 `search()` 会从每行的开始位置开始匹配:

```py
>>> re.match('X', 'A\nB\nX', re.MULTILINE)  # No match
>>> re.search('^X', 'A\nB\nX', re.MULTILINE)  # Match
```



## re.escape

用于将非字母数字的符号加上斜杠(Return string with all non-alphanumerics backslashed).

```py
print re.escape('*/+/abc? &')  # \*\/\+\/abc\?\ \&
```


## re.split

```py
split(string[, maxsplit]) | re.split(pattern, string[, maxsplit])
```

按照能够匹配的子串将string分割后返回列表。maxsplit用于指定最大分割次数，不指定将全部分割。

```py
import re
 
p = re.compile(r'\d+')
print p.split('one1two2three3four4')
 
### output ###
# ['one', 'two', 'three', 'four', '']
```

## re.findall

```py
findall(string[, pos[, endpos]]) | re.findall(pattern, string[, flags])
```

搜索string，以列表形式返回全部能匹配的子串。 

```py
import re
 
p = re.compile(r'\d+')
print p.findall('one1two2three3four4')
 
### output ###
# ['1', '2', '3', '4']
```

## re.finditer

```py
finditer(string[, pos[, endpos]]) | re.finditer(pattern, string[, flags])
```

搜索string，返回一个顺序访问每一个匹配结果（Match对象）的迭代器。

```py
import re
 
p = re.compile(r'\d+')
for m in p.finditer('one1two2three3four4'):
    print m.group(),
 
### output ###
# 1 2 3 4
```


## re.sub

```py
sub(repl, string[, count]) | re.sub(pattern, repl, string[, count])
```

使用repl替换string中每一个匹配的子串后返回替换后的字符串。 

当repl是一个字符串时，可以使用 `\id` 或 `\g<id>`、`\g<name>` 引用分组，但不能使用编号0。 

当repl是一个方法时，这个方法应当只接受一个参数（Match对象），并返回一个字符串用于替换（返回的字符串中不能再引用分组）。 

count用于指定最多替换次数，不指定时全部替换。

```py
import re
 
p = re.compile(r'(\w+) (\w+)')
s = 'i say, hello world!'
 
print p.sub(r'\2 \1', s)
 
def func(m):
    return m.group(1).title() + ' ' + m.group(2).title()
 
print p.sub(func, s)
 
### output ###
# say i, world hello!
# I Say, Hello World!
```


## re.subn

```py
subn(repl, string[, count]) |re.sub(pattern, repl, string[, count])
```

返回元组 `(sub(repl, string[, count]), 替换次数)`.

```py
import re
 
p = re.compile(r'(\w+) (\w+)')
s = 'i say, hello world!'
 
print p.subn(r'\2 \1', s)
 
def func(m):
    return m.group(1).title() + ' ' + m.group(2).title()
 
print p.subn(func, s)
 
### output ###
# ('say i, world hello!', 2)
# ('I Say, Hello World!', 2)
```




# 参考

* [Python正则表达式指南](http://www.cnblogs.com/huxi/archive/2010/07/04/1771073.html)
