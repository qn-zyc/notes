<!-- TOC -->

- [简介](#简介)
- [多行文本变成单行](#多行文本变成单行)
- [替代字符](#替代字符)
- [打印要执行的命令](#打印要执行的命令)
- [参考](#参考)

<!-- /TOC -->


# 简介
* xargs命令是给其他命令传递参数的一个过滤器，也是组合多个命令的一个工具。
* 它擅长将标准输入数据转换成命令行参数，xargs能够处理管道或者stdin并将其转换成特定命令的命令参数。
* xargs也可以将单行或多行文本输入转换为其他格式，例如多行变单行，单行变多行。
* xargs的默认命令是echo，空格是默认定界符。这意味着通过管道传递给xargs的输入将会包含换行和空白，不过通过xargs的处理，换行和空白将被空格取代。
* xargs是构建单行命令的重要组件之一。

# 多行文本变成单行
* 换行符被替换成空格.

测试文件 test.txt:

```
a b c d e f g
h i j k l m n
o p q
r s t
u v w x y z
```

```shell
▶ cat test.txt | xargs
a b c d e f g h i j k l m n o p q r s t u v w x y z
```

`-n` 指定每行的个数:

```shell
▶ cat test.txt | xargs -n 3
a b c
d e f
g h i
j k l
m n o
p q r
s t u
v w x
y z
```


# 替代字符
* `-i` 使用默认的 `{}` 作为替代字符.
* `-I` 指定自定义的替代字符.

```shell
cat test.txt | xargs -n 1 -i echo "hello {}" # mac下不加 -i
cat test.txt | xargs -n 1 -I ^ echo "hello [^]" # 指定 ^ 作为替代字符, 网上资料可以使用 [], mac下没成功.
```


# 打印要执行的命令
* `-t` 打印将要执行的命令.

```
▶ cat test.txt | xargs -n 1 -t echo "hello {}"
echo hello {} aaa
hello {} aaa
echo hello {} bbb
hello {} bbb
echo hello {} ccc
hello {} ccc
```





# 参考
* [xargs命令](http://man.linuxde.net/xargs)