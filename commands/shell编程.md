<!-- TOC -->

- [1. 变量](#1-变量)
- [2. 输出](#2-输出)
    - [2.1. echo](#21-echo)
        - [2.1.1. 换行](#211-换行)
        - [不换行](#不换行)
        - [-e 处理特殊字符](#-e-处理特殊字符)
    - [2.2. 重定向](#22-重定向)
- [3. 参数](#3-参数)
- [4. 数组](#4-数组)
- [5. 当前目录](#5-当前目录)
    - [5.1. 获取脚本所在的目录](#51-获取脚本所在的目录)
- [6. case](#6-case)
- [7. 判断](#7-判断)
    - [7.1. 整数比较](#71-整数比较)
    - [7.2. 字符串比较](#72-字符串比较)
- [8. 提取](#8-提取)
- [9. 循环](#9-循环)
    - [9.1. 遍历文件每一行](#91-遍历文件每一行)
    - [遍历序列](#遍历序列)
- [函数](#函数)
    - [使用参数](#使用参数)
    - [返回值](#返回值)

<!-- /TOC -->

# 1. 变量

```bash
# 变量替换
num1=1
echo '$num1' # 不会被替换
echo "$num1" # 会被替换
echo "\$num1" # 输出$num1而不是1
```

# 2. 输出

## 2.1. echo

### 2.1.1. 换行

```bash
echo -e "\n"
```

### 不换行

```bash
$echo -n "123"
$echo "456"
```

最终输出  123456

### -e 处理特殊字符

若字符串中出现以下字符，则特别加以处理，而不会将它当成一般文字输出：

```
\a 发出警告声；
\b 删除前一个字符；
\c 最后不加上换行符号；
\f 换行但光标仍旧停留在原来的位置；
\n 换行且光标移至行首；
\r 光标移至行首，但不换行；
\t 插入tab；
\v 与\f相同；
\\ 插入\字符；
\nnn 插入nnn（八进制）所代表的ASCII字符；
```

`$echo -e "a\bdddd"`

dddd

`$echo -e "a\adddd" //输出同时会发出报警声音`

adddd


`$echo -e "a\ndddd" //自动换行`

a

dddd





## 2.2. 重定向

linux shell下常用输入输出操作符是：

1.  标准输入   (stdin) ：代码为 0 ，使用 < 或 << ； /dev/stdin -> /proc/self/fd/0   0代表：/dev/stdin 
2.  标准输出   (stdout)：代码为 1 ，使用 > 或 >> ； /dev/stdout -> /proc/self/fd/1  1代表：/dev/stdout
3.  标准错误输出(stderr)：代码为 2 ，使用 2> 或 2>> ； /dev/stderr -> /proc/self/fd/2 2代表：/dev/stderr

**注意**: `>` 是覆盖式输出， `>>` 是追加。

- 将标准输出和错误输出分开重定向

    ```bash
    ls abc.txt 1>suc.txt 2>err.txt
    ```

- 忽略输出

    ```bash
    command 2>&-  # &[n]代表是已经存在的文件描述符，&1 代表输出 &2代表错误输出 &-代表关闭与它绑定的描述符
    2>/dev/null   # /dev/null 这个设备，是linux 中黑洞设备，什么信息只要输出给这个设备，都会给吃掉
    ```

- 标准输出和错误输出都重定向到同一设备

    ```bash
    command 2>&1 >/dev/null
    ```





# 3. 参数
* 参数个数和参数列表： `echo "$# params: $@"` 例如: `test.sh a b c`输出`3 a b c`。
* 脚本名称：`$0`, `$1`以后是参数， 没有`$10`, 用 `${10}`
* 当前进程ID： `$$`
* 遍历参数: `for a in $@; do echo $a; done`



# 4. 数组

```bash
# 需要用bash运行脚本, 元素之间用空格分隔
ipList=("A" "B" "C")
for ip in ${ipList[@]}
do
	echo "$ip"
done

for i in 1 2 3
do
	echo $i
done
```

# 5. 当前目录

```bash
dir=`pwd`
echo ${dir}
```



## 5.1. 获取脚本所在的目录

```bash
basepath=$(cd `dirname $0`; pwd)
```

* `dirname $0` 获取脚本文件的父目录。
* `cd` 切换到父目录。
* `pwd` 显示当前目录。

# 6. case

模式:

```bash
case 值 in
模式1)
    command1
    command2
    command3
    ;;
模式2）
    command1
    command2
    command3
    ;;
*)
    command1
    command2
    command3
    ;;
esac
```

case工作方式如上所示。取值后面必须为关键字 in，每一模式必须以右括号结束。取值可以为变量或常数。匹配发现取值符合某一模式后，其间所有命令开始执行直至 ;;。
;; 与其他语言中的 break 类似，意思是跳到整个 case 语句的最后。

取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号 * 捕获该值，再执行后面的命令。

例子:

```bash
echo 'Input a number between 1 to 4'
echo 'Your number is:\c'
read aNum
case $aNum in
    1)  echo 'You select 1'
    ;;
    2)  echo 'You select 2'
    ;;
    3)  echo 'You select 3'
    ;;
    4)  echo 'You select 4'
    ;;
    *)  echo 'You do not select a number between 1 to 4'
    ;;
esac
```

```bash
option="${1}"
case ${option} in
   -f) FILE="${2}"
      echo "File name is $FILE"
      ;;
   -d) DIR="${2}"
      echo "Dir name is $DIR"
      ;;
   *) 
      echo "`basename ${0}`:usage: [-f file] | [-d directory]"
      exit 1 # Command to come out of the program with status 1
      ;;
esac
```

```bash
configPrefix="dev"

case $1 in
	test) configPrefix="test"
	;;

	pre) configPrefix="pre"
	;;

	publish) configPrefix="publish"
	;;

esac

echo "use config file: " ${configPrefix}.config.json " as config.json"
```

注意等号前后没有空格。


# 7. 判断

```bash
#如果文件夹不存在，创建文件夹
if [ ! -d "/myfolder" ]; then
  mkdir /myfolder
fi

# -d 参数判断 $folder 是否存在
folder="/var/www/"
if [ ! -d "$folder"]; then
  mkdir "$folder"
fi

# -f 参数判断 $file 是否存在
file="/var/www/log"
if [ ! -f "$file" ]; then
  touch "$file"
fi

# -x 参数判断 $folder 是否存在并且是否具有可执行权限
if [ ! -x "$folder"]; then
  mkdir "$folder"
fi

# -n 判断一个变量是否有值
if [ ! -n "$var" ]; then
  echo "$var is empty"
  exit 0
fi

# 判断两个变量是否相等
if [ "$var1" = "$var2" ]; then
  echo '$var1 eq $var2'
else
  echo '$var1 not eq $var2'
fi
```

## 7.1. 整数比较

```
-eq 等于,如:if [ "$a" -eq "$b" ]   
-ne 不等于,如:if [ "$a" -ne "$b" ]   
-gt 大于,如:if [ "$a" -gt "$b" ]   
-ge 大于等于,如:if [ "$a" -ge "$b" ]   
-lt 小于,如:if [ "$a" -lt "$b" ]   
-le 小于等于,如:if [ "$a" -le "$b" ]   
<   小于(需要双括号),如:(("$a" < "$b"))   
<=  小于等于(需要双括号),如:(("$a" <= "$b"))   
>   大于(需要双括号),如:(("$a" > "$b"))   
>=  大于等于(需要双括号),如:(("$a" >= "$b")) 
```

## 7.2. 字符串比较

```
= 等于,如:if [ "$a" = "$b" ]   
== 等于,如:if [ "$a" == "$b" ],与=等价
```

**注意**

比较两个字符串是否相等的办法是：

```
if [ "$test"x = "test"x ]; then
```

这里的关键有几点：

1. 使用单个等号
2. 注意到等号两边各有一个空格：这是unix shell的要求
3. 注意到"$test"x最后的x，这是特意安排的，因为当$test为空的时候，上面的表达式就变成了x = testx，显然是不相等的。而如果没有这个x，表达式就会报错：[: =: unary operator expected

**注意** ==的功能在[[]]和[]中的行为是不同的,如下: 

```
[[ $a == z* ]]   # 如果$a以"z"开头(模式匹配)那么将为true   
[[ $a == "z*" ]] # 如果$a等于z*(字符匹配),那么结果为true(加了引号)   
  
[ $a == z* ]     # File globbing 和word splitting将会发生   
[ "$a" == "z*" ] # 如果$a等于z*(字符匹配),那么结果为true 
```

# 8. 提取

<http://blog.csdn.net/ljianhui/article/details/43128465>

- `#`：表示从左边算起第一个
- `%`：表示从右边算起第一个
- `##`：表示从左边算起最后一个
- `%%`：表示从右边算起最后一个

`＊`：表示要删除的内容，对于 `#` 和 `##` 的情况，它位于指定的字符（例子中的'/'和'.'）的左边，表于删除指定字符及其左边的内容；对于 `%` 和 `%%` 的情况，它位于指定的字符（例子中的'/'和'.'）的右边，表示删除指定字符及其右边的内容。这里的 `*` 的位置不能互换，即不能把*号放在#或##的右边，反之亦然.

例如：`${var%%x*}` 表示找出从右边算起最后一个字符x，并删除字符x及其右边的字符

---

提取文件路径中的文件名

```bash
dir=/a/b/c/d.txt
echo ${dir##*/}
d.txt
```

该命令的作用是去掉变量var从左边算起的最后一个`/`字符及其左边的内容，返回从左边算起的最后一个`/`	（不含该字符）的右边的内容

---

提取文件路径中文件的后缀

```bash
dir=/a/b/c/d.txt
echo ${dir##*.}
txt
```

该命令的作用是去掉变量var从左边算起的最后一个'.'字符及其左边的内容，返回从左边算起的最后一个'.'（不含该字符）的右边的内容

---

匹配第一个出现的字符

```bash
dir=/a/b/c/d.tar.gz
echo ${dir#*.}
tar.gz
```

该命令的作用是去掉变量var从左边算起的第一个'.'字符及其左边的内容，返回从左边算起第一个'.'（不含该字符）的右边部分的内容。

---

提取目录

```bash
dir=/a/b/c/d.tar.gz
echo ${dir%/*}
/a/b/c
```

该命令的使用是去掉变量var从右边算起的第一个'/'字符及其右边的内容，返回从右边算起的第一个'/'（不含该字符）的左边的内容



# 9. 循环

## 9.1. 遍历文件每一行

```bash
for line in `cat f.txt`; do echo $line; ....; done
```

## 遍历序列

```bash
for i in {1..10}; do echo $i; done # 生成 1 到 10 的序号
```


# 函数

```shell
function test()
{
    echo 'hello';
}

test
```

* 可以省略 `function`.

## 使用参数

```shell
function test()
{
    echo $0, $1, $2;
}

test 'a' 'b'
```

* `$0` 表示当前脚本的绝对路径, `$1...$n` 表示参数.
* 调用时参数和函数名都是使用空格分隔.
* 先声明, 后调用.

## 返回值

```shell
function test()
{
    return 1;
}

test
echo "1: $?"; # 1

ret=$(test 1)
echo "2: $ret"; # 
echo "3: $?"; # 0
```

* 返回值只能返回一个.
* 只能通过 `$?` 来获取返回值.