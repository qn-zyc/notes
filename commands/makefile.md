<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [简介](#简介)
- [Makefile 规则](#makefile-规则)
	- [伪目标](#伪目标)
	- [前置条件](#前置条件)
	- [命令](#命令)
	- [注释](#注释)
	- [回声](#回声)
	- [通配符](#通配符)
	- [模式匹配](#模式匹配)
	- [变量和赋值符](#变量和赋值符)
		- [自动变量](#自动变量)
- [Makefile 示例](#makefile-示例)
	- [执行多个目标](#执行多个目标)
	- [获取当前路径](#获取当前路径)
	- [设置环境变量](#设置环境变量)
	- [执行 golang 程序](#执行-golang-程序)
- [参考](#参考)

<!-- /TOC -->


# 简介

make 主要用于 C 语言, 但是实际上, 只要某个文件有变化就要重新构建的项目, 都可以用 make 构建.

make 命令执行时需要一个 Makefile 文件.

如果 make 命令后没有指定 target, 默认运行的是第一个.




# Makefile 规则

Makefile 可以写作 makefile, 也可以使用其他文件, 比如 `make -f rules.txt` 或者 `make --file=rules.txt`.

Makefile 由一系列规则构成, 每条规则的形式如下:

```Makefile
<target> : <prerequisites>
[tab]  <commands>
```

- `target` 是最终要生成的文件名或者只是一个标签, 如果是文件名可以有多个, 用空格分隔.
- `prerequisites` 是生成 target 需要依赖的文件或者标签, 叫前置条件, 多个之间用空格分隔.
- 前置条件和命令都是可选, 但是二者至少需要存在一个.


## 伪目标

```makefile
clean:
    rm *.o
```

标签是一个操作名, 执行时使用 `make clean`, 但是如果当前目录中正好有一个叫 clean 的文件则命令不会被执行, 因为 make 命令发现文件已经存在了.


```makefile
.PHONY: clean others
clean:
    rm *.o temp
others:
```

可以明确声明 clean 不是一个文件名而是伪目标, 这样 make 命令就不会去检查是否有文件存在, 而是每次都运行这个命令:


## 前置条件

前置条件指明了目标是否重新构建的条件: 某个前置文件不存在或者有过更新(时间戳比目标时间戳新).

```makefile
result.txt: source.txt
    cp source.txt result.txt

source.txt:
    echo "this is the source" > source.txt
```

当 `make result.txt` 时, 如果 source.txt 不存在则执行 source.txt 那个规则, 如果已经存在, 就看它是不是比 result.txt 更新.

source.txt 那个规则没有前置条件, 所以只要 source.txt 不存在就执行这个规则.



## 命令

命令表示如何更新 target 文件, 由一行或多行 shell 命令组成.

命令之前必须有一个 tab 键, 也可以使用 `.RECIPEPREFIX` 指定其他键:

```makefile
.RECIPEPREFIX = >
all:
> echo Hello, world
```

上面使用 `>` 代替 tab.


默认情况下, 每行命令都在一个单独的进程中执行, 比如

```makefile
var-lost:
    export foo=bar
    echo "foo=[$$foo]"
```

第二个命令取不到 foo 变量, 因为两行命令在不同的进程中执行.

一个解决方法是写在一行, 用 `;` 分隔:

```makefile
var-kept:
    export foo=bar; echo "foo=[$$foo]"
```

另一个解决方法是在换行符前使用 `\` 转义:

```makefile
var-kept:
    export foo=bar; \
    echo "foo=[$$foo]"
```

第三个解决方法是使用 `.ONESHELL:`:

```makefile
.ONESHELL:
var-kept:
    export foo=bar;
    echo "foo=[$$foo]"
```

## 注释

```makefile
# 这是注释
result.txt: source.txt
    # 这是注释
    cp source.txt result.txt # 这也是注释
```

## 回声

make 会打印每条命令, 包括注释, 然后再执行, 比如下面:

```makefile
test:
    # 这是测试
```

```shell
$ make test
# 这是测试
```

可以在命令前加 `@` 取消回声:

```makefile
test:
    @# 这是测试
```

可以只在注释和 echo 命令前加 `@`, 其他命令显示要执行的命令:

```makefile
test:
    @# 这是测试
    @echo TODO
```


## 通配符

和 bash 一致

- `*`(任意多个字符)
- `?`(任意单个字符)
- `[abc]`(其中某个字符)

## 模式匹配

使用匹配符 `%`, 可以将大量同类型的文件，只用一条规则就完成构建.

比如 `%.o: %.c`, 如果有文件 `f1.c` 和 `f2.c`, 则等同于:

```makefile
f1.o: f1.c
f2.o: f2.c
```


## 变量和赋值符

```makefile
txt = Hello World
test:
    @echo $(txt)
```

调用 shell 变量时需要多加一个 `$`:

```makefile
test:
    @echo $$HOME
	# 也可以使用括号
	@echo $(HOME)
```

### 自动变量

`$@` 表示当前目标, 比如 `make foo` 时 `$@` 表示 `foo`.

```makefile
a.txt b.txt:
	touch $@

# 等同于下面的写法
a.txt:
	touch a.txt

b.txt:
	touch b.txt
```



# Makefile 示例

## 执行多个目标

```makefile
.PHONY: cleanall cleanobj cleandiff

cleanall : cleanobj cleandiff
	rm program

cleanobj :
    rm *.o

cleandiff :
    rm *.diff
```

## 获取当前路径

```makefile
cur_dir := $(shell pwd)

test:
	@echo $(CURDIR)   # 方法1
	@echo $(cur_dirk) # 方法2

```

## 设置环境变量

```makefile
export GOPATH=$(CURDIR)

gopath:
	@echo $(GOPATH)
```


## 执行 golang 程序

```makefile
export GOPATH=$(CURDIR)

.PHONY: test
test:
	go install path/test && bin/test
```





# 参考

- [阮一峰: Make 命令教程](http://www.ruanyifeng.com/blog/2015/02/make.html)
