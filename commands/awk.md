<!-- TOC -->

- [1. 简介](#1-简介)
- [2. 使用方法](#2-使用方法)
- [3. 调用awk](#3-调用awk)
- [4. 入门实例](#4-入门实例)
- [5. awk编程](#5-awk编程)
    - [5.1. awk内置变量](#51-awk内置变量)
    - [5.2. 变量和赋值](#52-变量和赋值)
        - [5.2.1. 文件中的多个变量](#521-文件中的多个变量)
    - [5.3. 运算符](#53-运算符)
        - [5.3.1. 算术运算符](#531-算术运算符)
        - [5.3.2. 赋值运算符](#532-赋值运算符)
        - [5.3.3. 比较运算符](#533-比较运算符)
        - [5.3.4. 逻辑运算符](#534-逻辑运算符)
        - [5.3.5. 串连接运算符](#535-串连接运算符)
    - [5.4. 控制结构](#54-控制结构)
        - [5.4.1. if](#541-if)
        - [5.4.2. while](#542-while)
        - [5.4.3. do while](#543-do-while)
        - [5.4.4. for](#544-for)
    - [5.5. 数组](#55-数组)
    - [5.6. 格式化输出](#56-格式化输出)
        - [5.6.1. 动态宽度和精度](#561-动态宽度和精度)
        - [5.6.2. sprintf](#562-sprintf)
    - [5.7. 正则表达式](#57-正则表达式)
    - [5.8. 函数](#58-函数)
        - [5.8.1. 自定义函数](#581-自定义函数)
        - [5.8.2. 内置数字函数](#582-内置数字函数)
        - [5.8.3. 内置字符串函数](#583-内置字符串函数)
        - [5.8.4. 内置日期时间函数](#584-内置日期时间函数)
        - [5.8.5. 内置位函数](#585-内置位函数)
        - [5.8.6. 内置国际化函数](#586-内置国际化函数)
    - [5.9. 向命令传递参数](#59-向命令传递参数)
    - [5.10. IO](#510-io)
- [6. 在文件中编写](#6-在文件中编写)
    - [6.1. 注释](#61-注释)
    - [6.2. 命令分隔符](#62-命令分隔符)
- [7. 参考](#7-参考)

<!-- /TOC -->

# 1. 简介

awk是一个强大的文本分析工具，相对于grep的查找，sed的编辑，awk在其对数据分析并生成报告时，显得尤为强大。简单来说awk就是把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理。

awk有3个不同版本: awk、nawk和gawk，未作特别说明，一般指gawk，gawk 是 AWK 的 GNU 版本。

awk其名称得自于它的创始人 Alfred Aho 、Peter Weinberger 和 Brian Kernighan 姓氏的首个字母。实际上 AWK 的确拥有自己的语言： AWK 程序设计语言 ， 三位创建者已将它正式定义为“样式扫描和处理语言”。它允许您创建简短的程序，这些程序读取输入文件、为数据排序、处理数据、对输入执行计算以及生成报表，还有无数其他的功能。

# 2. 使用方法

```bash
awk '{pattern + action}' {filenames}
```

尽管操作可能会很复杂，但语法总是这样，其中 pattern 表示 AWK 在数据中查找的内容，而 action 是在找到匹配内容时所执行的一系列命令。花括号（{}）不需要在程序中始终出现，但它们用于根据特定的模式对一系列指令进行分组。 pattern就是要表示的正则表达式，用斜杠括起来。

awk语言的最基本功能是在文件或者字符串中基于指定规则浏览和抽取信息，awk抽取信息后，才能进行其他文本操作。完整的awk脚本通常用来格式化文本文件中的信息。

通常，awk是以文件的一行为处理单位的。awk每接收文件的一行，然后执行相应的命令，来处理文本。

# 3. 调用awk

有三种方式调用awk

1. 命令行方式

    ```bash
    awk [-F  field-separator]  'commands'  input-file(s)
    ```

    其中，commands 是真正awk命令，[-F域分隔符]是可选的。 input-file(s) 是待处理的文件。

    在awk中，文件的每一行中，由域分隔符分开的每一项称为一个域。通常，在不指名-F域分隔符的情况下，默认的域分隔符是空格。

2. shell脚本方式

    将所有的awk命令插入一个文件，并使awk程序可执行，然后awk命令解释器作为脚本的首行，一遍通过键入脚本名称来调用。

    比如创建一个文件 `test.awk`:
    ```bash
    #! /usr/bin/awk -f
    BEGIN {c=0}
    /^$/{c++}
    END {print c}
    ```
    给脚本赋予可执行权限，执行 `./test.awk test.txt`。

3. 将所有的awk命令插入一个单独文件，然后调用：

    ```bash
    awk -f awk-script-file input-file(s)
    ```

    其中，-f选项加载awk-script-file中的awk脚本，input-file(s)跟上面的是一样的。


# 4. 入门实例

假设last -n 5的输出如下

```bash
[root@www ~]# last -n 5 <==仅取出前五行
root     pts/1   192.168.1.100  Tue Feb 10 11:21   still logged in
root     pts/1   192.168.1.100  Tue Feb 10 00:46 - 02:28  (01:41)
root     pts/1   192.168.1.100  Mon Feb  9 11:41 - 18:30  (06:48)
dmtsai   pts/1   192.168.1.100  Mon Feb  9 11:41 - 11:41  (00:00)
root     tty1                   Fri Sep  5 14:09 - 14:10  (00:01)
```

如果只是显示最近登录的5个帐号

```bash
[root@www ~]# last -n 5 | awk  '{print $1}'

root
root
root
dmtsai
root
```

awk工作流程是这样的：读入有'\n'换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，`$0` 则表示所有域, `$1` 表示第一个域, `$n` 表示第n个域。默认域分隔符是"空白键" 或 "[tab]键",所以$1表示登录用户，$3表示登录用户ip,以此类推。

如果只是显示/etc/passwd的账户

```bash
cat /etc/passwd |awk  -F ':'  '{print $1}'  

root
daemon
bin
sys
```

这种是awk+action的示例，每行都会执行action{print $1}。

`-F` 指定域分隔符为':'。

如果只是显示/etc/passwd的账户和账户对应的shell,而账户与shell之间以tab键分割

```bash
cat /etc/passwd |awk  -F ':'  '{print $1"\t"$7}'

root    /bin/bash
daemon  /bin/sh
bin     /bin/sh
sys     /bin/sh
```

如果只是显示/etc/passwd的账户和账户对应的shell,而账户与shell之间以逗号分割,而且在所有行添加列名name,shell,在最后一行添加"blue,/bin/nosh"。

```bash
cat /etc/passwd |awk  -F ':'  'BEGIN {print "name,shell"}  {print $1","$7} END {print "blue,/bin/nosh"}'

name,shell
root,/bin/bash
daemon,/bin/sh
bin,/bin/sh
sys,/bin/sh
....
blue,/bin/nosh
```

awk工作流程是这样的：先执行BEGING，然后读取文件，读入有/n换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，$0则表示所有域,$1表示第一个域,$n表示第n个域,随后开始执行模式所对应的动作action。接着开始读入第二条记录······直到所有的记录都读完，最后执行END操作。

搜索/etc/passwd有root关键字的所有行

```bash
awk -F: '/root/' /etc/passwd

root:x:0:0:root:/root:/bin/bash
```

这种是pattern的使用示例，匹配了pattern(这里是root)的行才会执行action(没有指定action，默认输出每行的内容)。

搜索支持正则，例如找root开头的: `awk -F: '/^root/' /etc/passwd`

搜索/etc/passwd有root关键字的所有行，并显示对应的shell

```bash
awk -F: '/root/{print $7}' /etc/passwd             

/bin/bash
```

这里指定了action{print $7}


# 5. awk编程

## 5.1. awk内置变量

awk有许多内置变量用来设置环境信息，这些变量可以被改变，下面给出了最常用的一些变量。

```bash
ARGC        命令行参数数量
ARGV        命令行参数数组
ARGIND      当前 ARGV 的下标
ENVIRON     环境变量关联数组（ENVIRON["HOME"]）
IGNORECASE  是否忽略大小写
FILENAME    当前处理的文件名
FS          列分隔符，默认是空格
RS          行分隔符，默认是换行符
RT          行分隔符，Gawk 使用
FIELDWIDTHS 用来生成定长文件
NF          当前行的列数
NR          当前处理过的行数
FNR         当前文件的总行数
CONVFMT     默认格式化数字的格式(%.6g)
OFMT        默认数字输出格式(%.6g)
OFS         输出文件列分隔符，默认是空格
ORS         输出文件行分隔符，默认是换行符
RSTART      调用 match 函数后， RSTART 保存匹配的位置
RLENGTH     调用 match 函数后， RLENGTH 保存匹配字符串的长度
ERRNO       保存错误消息
BINMODE     二进制模式
LINT        同参数 --lint
PROCINFO    关联数组，保存当前运行脚本的有关信息，如获取进程号：PROCINFO["pid"]
SUBSEP      分离多个下标，默认（\034）
TEXTDOMAIN  用来本地转换
```

统计/etc/passwd:文件名，每行的行号，每行的列数，对应的完整行内容:

```bash
awk  -F ':'  '{print "filename:" FILENAME ",linenumber:" NR ",columns:" NF ",linecontent:"$0}' /etc/passwd

filename:/etc/passwd,linenumber:1,columns:7,linecontent:root:x:0:0:root:/root:/bin/bash
filename:/etc/passwd,linenumber:2,columns:7,linecontent:daemon:x:1:1:daemon:/usr/sbin:/bin/sh
filename:/etc/passwd,linenumber:3,columns:7,linecontent:bin:x:2:2:bin:/bin:/bin/sh
filename:/etc/passwd,linenumber:4,columns:7,linecontent:sys:x:3:3:sys:/dev:/bin/sh
```

使用printf替代print,可以让代码更加简洁，易读

```bash
 awk  -F ':'  '{printf("filename:%10s,linenumber:%s,columns:%s,linecontent:%s\n",FILENAME,NR,NF,$0)}' /etc/passwd
```


## 5.2. 变量和赋值

除了awk的内置变量，awk还可以自定义变量。

+ 变量类型根据操作来自动转换。比如进行数学运算时，变量会自动被转换成数字，不是数字的会被转换成0.
+ 变量不需要声明。
+ 变量区分大小写。

下面统计/etc/passwd的账户人数

```bash
awk '{count++;print $0;} END{print "user count is ", count}' /etc/passwd

root:x:0:0:root:/root:/bin/bash
......
user count is  40
```

count是自定义变量。之前的action{}里都是只有一个print,其实print只是一个语句，而action{}可以有多个语句，以;号隔开。

 

这里没有初始化count，虽然默认是0，但是妥当的做法还是初始化为0:

```bash
awk 'BEGIN {count=0;print "[start]user count is ", count} {count=count+1;print $0;} END{print "[end]user count is ", count}' /etc/passwd

[start]user count is  0
root:x:0:0:root:/root:/bin/bash
...
[end]user count is  40
```

统计某个文件夹下的文件占用的字节数

```bash
ls -l |awk 'BEGIN {size=0;} {size=size+$5;} END{print "[end]size is ", size}'
[end]size is  8657198
```

如果以M为单位显示:

```bash
ls -l |awk 'BEGIN {size=0;} {size=size+$5;} END{print "[end]size is ", size/1024/1024,"M"}' 
[end]size is  8.25889 M
```

注意，统计不包括文件夹的子目录。

### 5.2.1. 文件中的多个变量

数字变量：

```bash
#! /usr/bin/awk -f

BEGIN {
    # 定义整数
    int1 = 10;  
    int2 = -10;  
    int3 = 10e2; # 10 乘以 10 的 2 次方，e不区分大小写  

    octal = "\377"; # 八进制以 \ 开头  
    hex = "\xFF"; # 十六进制以 \x 开头，a,b,c,d,e,f 不区分大小写

    # 定义浮点数
    float1 = 10.0;  
    float2 = 10.;  
    float3 = 0.25;  
    float4 = .25;  
    float5 = -10.0e2; # -10.0乘以 10 的 2 次方，e不区分大小写
};

{};

END {
    print int1;
    print int2;
    print int3;

    print octal;
    print hex;

    print float1;
    print float2;
    print float3;
    print float4;
    print float5;
};
```

字符串变量：

```bash
#! /usr/bin/awk -f

BEGIN {
    # 定义字符串
    # 注意字符串不能使用单引号定义
    str1 = "Test1\tTest2";

    # 字符串长度
    printf("The length of str1: %s\n", length(str1));

    # 转成小写
    printf("The lower case of str1: %s\n", tolower(str1));

    # 转成大写
    printf("The upper case of str1: %s\n", toupper(str1));

    # 查找子串位置
    printf("The position of Test2 in str1: %s\n", index(str1, "Test2"));

    # 获取子串
    printf("The sub string of str1: %s\n", substr(str1, 7, 4));

    # 查找匹配
    if(match(str1, "[a-zA-Z]+2") != 0) {
        print "find pattern in str1";
    }

    # 替换
    printf("After substituted str1: %s\n", gensub("[a-zA-Z]+2", "***", g, str1));
};

{};
END {};
```



## 5.3. 运算符

### 5.3.1. 算术运算符

```bash
#! /usr/bin/awk -f

BEGIN {
    x=2;
    y=3;
    r=0;

    # 加
    r = x + y;
    printf("x + y = %s\n", r);

    # 减
    r = x - y;
    printf("x - y = %s\n", r);

    # 乘
    r = x * y;
    printf("x * y = %s\n", r);

    # 除
    r = x / y;
    printf("x / y = %s\n", r);

    # 余
    r = x % y;
    printf("x %% y = %s\n", r);

    # 幂，相当于2的3次方
    r = x ^ y;
    printf("x ^ y = %s\n", r);

    # 幂，相当于2的3次方
    r = x ** y;
    printf("x ** y = %s\n", r);
};

{};
END {};
```


### 5.3.2. 赋值运算符

```bash
#! /usr/bin/awk -f

BEGIN {
    # 赋值
    x=2;
    r=3;
    printf("x = %s, r = %s\n", x, r);

    # 自增
    r++;
    ++r;
    printf("r++ = %s\n", r);

    # 自减
    r--;
    --r;
    printf("r-- = %s\n", r);

    # r = r + x
    r+=x;
    printf("r + x = %s\n", r);

    # r = r - x
    r-=x;
    printf("r - x = %s\n", r);

    # r = r * x
    r*=x;
    printf("r * x = %s\n", r);

    # r = r / x
    r/=x;
    printf("r / x = %s\n", r);

    # r = r % x
    r%=x;
    printf("r %% x = %s\n", r);

    # r = r ^ x
    r^=x;
    printf("r ^ x = %s\n", r);

    # r = r ** x
    r**=x;
    printf("r ** x = %s\n", r);
};

{};
END {};
```


### 5.3.3. 比较运算符

```bash
#! /usr/bin/awk -f

BEGIN {
    x=2;
    y=3;

    # 大于
    if(x > y) {
        print "x > y";
    }

    # 大于等于
    if(x >= y) {
        print "x >= y";
    }

    # 小于
    if(x < y) {
        print "x < y";
    }

    # 小于等于
    if(x <= y) {
        print "x <= y";
    }

    # 等于
    if(x == y) {
        print "x == y";
    }

    # 不等于
    if(x != y) {
        print "x != y";
    }

    # 匹配
    if("x"~/x*/) {
        print "x match x*";
    }

    # 不匹配
    if("x"!~/test/) {
        print "x not match test";
    }
};

{};
END {};
```


### 5.3.4. 逻辑运算符

```bash
#! /usr/bin/awk -f

BEGIN {
    x="a";
    y="b";
    z="c";

    # 与
    if(x < y && y < z) {
        print "x < y < z";
    }

    # 或
    if(x < y || y < z) {
        print "x < y || y < z";
    }

    # 非
    if(!(x > y)) {
        print "x <= y";
    }

    # 三元运算符
    r=(y > x) ? y : x;
};

{};
END {};
```

### 5.3.5. 串连接运算符

```bash
 # 空格是串连接运算符
x = "Hello" " World"
print x;
```






## 5.4. 控制结构

+ 循环控制中可以使用 `break`, `continue`。

### 5.4.1. if

统计某个文件夹下的文件占用的字节数,过滤4096大小的文件(一般都是文件夹):

```bash
ls -l |awk 'BEGIN {size=0;print "[start]size is ", size} {if($5!=4096){size=size+$5;}} END{print "[end]size is ", size/1024/1024,"M"}' 

[end]size is  8.22339 M
```

```bash
#! /usr/bin/awk -f

BEGIN {
    x=2;
    y=3;

    #
    if(x < y) {
        print "x < y";
    }

    #
    if(x < y) {
        print "x < y";
    } else {
        print "x >= y";
    }

    #
    if(x < y) {
        print "x < y";
    } else if (x = y) {
        print "x = y";
    } else {
        print "x > y";
    }
};

{};
END {};
```


### 5.4.2. while

```bash
#! /usr/bin/awk -f

BEGIN {
    i=1;
    sum=0;

    while (i <= 10) {
        sum+=i;
        i++;
    }
    printf("sum=%s\n", sum);
};

{};
END {};
```

### 5.4.3. do while

```bash
#! /usr/bin/awk -f

BEGIN {
    i=1;
    sum=0;

    do {
        sum+=i;
        i++;
    } while (i <= 10)
    printf("sum=%s\n", sum);
};

{};
END {};
```


### 5.4.4. for

```bash
#! /usr/bin/awk -f

BEGIN {
    sum=0;

    # 方式1
    for(i=1; i<=10; i++) {
        sum+=i;
    }
    printf("sum=%s\n", sum);

    # 方式2用来迭代数组
    alphabet[0]="a";
    alphabet[1]="b";
    for (key in alphabet) {
        printf("alphabet[%s]=%s\n", key, alphabet[key]);
    }
};

{};
END {};
```




## 5.5. 数组

因为awk中数组的下标可以是数字和字母，数组的下标通常被称为关键字(key)。值和关键字都存储在内部的一张针对key/value应用hash的表格里。由于hash不是顺序存储，因此在显示数组内容时会发现，它们并不是按照你预料的顺序显示出来的。数组和变量一样，都是在使用时自动创建的，awk也同样会自动判断其存储的是数字还是字符串。一般而言，awk中的数组用来从记录中收集信息，可以用于计算总和、统计单词以及跟踪模板被匹配的次数等等。

显示/etc/passwd的账户

```bash
awk -F ':' 'BEGIN {count=0;} {name[count] = $1;count++;}; END{for (i = 0; i < NR; i++) print i, name[i]}' /etc/passwd
root
daemon
bin
sys
sync
games
......
```


```bash
#! /usr/bin/awk -f

BEGIN {
    # 定义索引数组
    names[0]="Shang Bo";
    names[2]="Li Si";
    names[1]="Zhang San";

    # 定义关联数组
    country_code_name_map["CN"]="CHINA";
    country_code_name_map["US"]="UNITED STATES";
    country_code_name_map["JP"]="JAPAN";

    # 通过 split 定义数组
    str1="a,b,c,d";
    split(str1, alphabet, ",");

    # 对数组排序
    asort(names);
    for (key in names) { # 迭代数组
        printf("names[%s]=%s\n", key, names[key]);
    }

    # 抽出并排序关联数组下标
    asorti(country_code_name_map, country_codes);
    for (key in country_codes) {
        printf("country_codes[%s]=%s\n", key, country_codes[key]);
    }

    # 删除数组元素
    delete names[0];
    if (0 in names) { #判断下标是否存在
        print "index 0 exists";
    }

    # 删除数组
    delete alphabet;
    for (key in alphabet) {
        printf("alphabet[%s]=%s\n", key, alphabet[key]);
    }

    # 定义多维数组
    for (i = 1; i <= 9; i++) {
        for (j = 1; j <= 9; j++) {
            table[i, j] = i * j;
        }
    }

    # 迭代多维数组
    for (key in table) {
        printf("table[%s]=%s\n", key, table[key]);
    }

    # 迭代多维数组
    for (key in table) {
        split(key, indexs, SUBSEP); # SUBSEP 是一个内置变量，多维数组下标分隔符
        printf("table[%s, %s]=%s\n", indexs[1], indexs[2], table[indexs[1], indexs[2]]);
    }
};

{};
END {};
```


## 5.6. 格式化输出

```bash
#! /usr/bin/awk -f

BEGIN {
    printf("printf example:%-5.2f\n", 33.698);
};

{};
END {};
```

格式符的组成：

```
%[flags][width][.precision]conversion
```

conversion 转换符如下：

| 转换符 | 描述                                   |
| ------ | -------------------------------------- |
| c      | ASCII 字符 (打印第一个字符)            |
| d      | 十进制整数                             |
| i      | 十进制整数                             |
| e      | 浮点数科学计数法                       |
| E      | 浮点数科学计数法                       |
| f      | 浮点数                                 |
| g      | %e 或 %f, 取决于哪个更短, 删除尾部0    |
| G      | %E 或 %f, 取决于哪个更短, 删除尾部0    |
| u      | 无符号十进制整数                       |
| o      | 无符号八进制整数                       |
| x      | 无符号十六进制整数（a-f for 10 to 15） |
| X      | 无符号十六进制整数（A-F for 10 to 15） |
| %%     | %                                      |
| s      | 字符串                                 |

flags 如下：

| 标志      | 描述               | 举例                   |
| --------- | ------------------ | ---------------------- |
| -         | 左对齐             | `3333.33 `             |
| 空格      | 在正数之前添加空格 | ` 3333.33`，`-3333.33` |
| +         | 打印正负数符号     | `+3333.33`，`-3333.33` |
| 0         | 数字前面补0        | `003333.33`            |
| #(对于%o) | 添加前缀0          | `0515`                 |
| #(对于%x) | 添加前缀0x         | `0x1bc`                |
| #(对于%X) | 添加前缀0X         | `0X1bc`                |
| #(对于%e) | 添加小数点         | `1.000000e+01`         |
| #(对于%E) | 添加小数点         | `1.000000E+01`         |
| #(对于%f) | 添加小数点         | ` 10.000000`           |
| #(对于%g) | 不删除尾部0        | `10.4000`              |
| #(对于%G) | 不删除尾部0        | `10.4000`              |

精度 precision 的意义：

| 转换符            | 精度意义                                      |
| ----------------- | --------------------------------------------- |
| %d,%i,%o,%u,%x,%X | 最少数字位数，如果数字位数少于精度，添加前缀0 |
| %e, %E            | 最少数字位数，如果数字位数少于精度，添加后缀0 |
| %f                | 小数的位数                                    |
| %g, %G            | 最多数字位数                                  |
| %s                | 字符位数                                      |



### 5.6.1. 动态宽度和精度

```bash
# 用星号实现动态宽度和精度
printf("printf example:%-*.*f\n", 5, 2, 33.698);
```


### 5.6.2. sprintf

格式化字符串然后保存到变量中。

```bash
str=sprintf("%5.2f", 33.698);
```



## 5.7. 正则表达式

元字符：

```
\a       响铃
\b       退格
\f       换页
\n       换行
\r       回车
\t       水平制表符
\v       垂直制表符
\ddd     8进制数字
\xhex    16进制数字
\\       反斜线
[abc]    匹配a或b或c
[^abc]   匹配abc之外的任何单个字符
[a-zA-Z] 匹配a到z或A到Z的任何单个字符
\w       组成单词的字符，等价于[a-zA-Z_0-9]
\W       不是组成单词的字符，等价于[^\w]
\y       单词边界
\B       非单词边界
\<       单词的起始位置
\>       单词的结束位置
.        任何字符，包括换行符
^        字符串首
$       字符串行尾
a|b|c    匹配a或b或c
X?       匹配X 0次或1次
X*       匹配X 0次或无数次
X+       匹配X 1次或无数次
X{n}     匹配X n次
X{n,}    匹配X 至少n次
X{n,m}   匹配X 至少n次至多m次. 需要指定 --re-interval
(...)    分组或捕获
```

上面介绍的正则表达式有个缺陷，它只能匹配英语，如果你想匹配其他语言，你可以使用标准的 POSIX 语法，如下。

```bash
[[:alnum:]]             字母和数字  
[[:alpha:]]             字母  
[[:lower:]]             小写字母  
[[:upper:]]             大写字母  
[[:digit:]]             数字  
[[:blank:]]             空格和制表符  
[[:space:]]             空白字符  
[[:graph:]]             非空白字符  
[[:print:]]             类似[[:graph:]]，但是包含空白字符  
[[:punct:]]             标点符号  
[[:cntrl:]]             控制字符  
[[:xdigit:]]            十六进制中容许出现的数字(例如 0-9a-fA-f) 
[. xx .]                将 xx 作为一个整体匹配, xx 可以是任何字母  
[= e =]                 认为等价，在法语中匹配 e, è, 或 é  
```


## 5.8. 函数

### 5.8.1. 自定义函数

```bash
#! /usr/bin/awk -f

BEGIN {};
{};

function max(x, y) {
    return x > y ? x : y;
}

END {
    print max(2,3);
};
```

### 5.8.2. 内置数字函数

```bash
rand()        返回0-1随机数
srand([expr]) 根据种子生成随机数
sqrt(expr)    平方根
exp(expr)     指数
int(expr)     转换成整形
log(expr)     自然对数
sin(expr)     正弦
cos(expr)     余弦
atan2(y, x)   反正切
```

### 5.8.3. 内置字符串函数

```bash
tolower(str)            转成小写
toupper(str)            转成大写
length([s])             返回字符串长度或数组元素个数
index(s, t)             返回字符串 t 在 s中的位置
substr(s, i [, n])      返回子串，i 表示开始位置，n 表示长度
match(s, r [, a])       在字符串 s 中查找正则表达式 r，找到后设置内置变量 RSTART 和 RLENGTH
sub(r, s [, t])         在字符串 t 中查找正则表达式 r，找到后替换成 s，只替换找到的第一个
gsub(r, s [, t])        同sub，贪婪模式，替换所有的
gensub(r, s, h [, t])   在字符串 t 中查找正则表达式 r，找到后替换成 s，如果找到多个，参数 h 表示要替换哪个，g 表示替换所有的
strtonum(str)           将字符串转成数字，如果数字以 0 开头，转成八进制数字，如果数字以 0x 开头，转成十六进制数字。
sprintf(fmt, expr-list) 格式化字符串
split(s, a [, r])       将字符串分隔成数组
asort(s [, d])          对数组 s 排序，将排序后的数组保存在 d 中，s 不变。如果没有指定 d，则修改原数组s
asorti(s [, d])         对数组 s 下标排序
```

### 5.8.4. 内置日期时间函数

```bash
systime()           返回当前时间自 1970-01-01 00:00:00 UTC 以来的秒数
mktime(datespec)    返回 datespec 自 1970-01-01 00:00:00 UTC 以来的秒数， 
                    datespec 的格式为YYYY MM DD HH MM SS[ DST]
                    MM(1-12),DD(1-31),HH(0-23),MM(0-59),SS(0-60)
strftime([format [, timestamp]]) 将日期时间格式化为字符串,format 见 date --help,如:%Y-%m-%d %H:%M:%S
```

### 5.8.5. 内置位函数

```bash
and(v1, v2)         按位与
or(v1, v2)          按位或
compl(val)          按位反
xor(v1, v2)         异或
lshift(val, count)  左移
rshift(val, count)  右移
```

### 5.8.6. 内置国际化函数

```bash
bindtextdomain(directory [, domain])
dcgettext(string [, domain [, category]])
dcngettext(string1 , string2 , number [, domain [, category]])
```


## 5.9. 向命令传递参数

```bash
awk -v myname="ShangBo" 'BEGIN {print myname;};{};' test.txt
```


## 5.10. IO

input:

```bash
getline                  读取下一行到 $0，同时设置 NF, NR, FNR
getline var              读取下一行到 var，同时设置 NR, FNR
getline <file            从文件 file中读取一行到 $0，同时设置 NF
getline var <file        从文件 file中读取一行到 var
command | getline [var]  从管道中读取一行到 $0 或 var
command |& getline [var] 从其他进程中读取一行到 $0 或 var
next                     停止处理当前行，处理下一行，相当于 continue
nextfile                 停止处理当前文件，处理下一文件，相当于 continue
```

output:

```bash
print                          打印 $0
print expr-list                打印 expr-list
print expr-list >file          输出 expr-list到文件
print expr-list >> file        添加 expr-list到文件
print expr-list | command      输出到管道
print expr-list |& command     输出到其他进程
printf fmt, expr-list          格式化打印 expr-list
printf fmt, expr-list >file    格式化打印 expr-list到文件
system(cmd-line)               调用系统命令
fflush([file])                 刷新缓存
close(file [, how])            关闭文件
```

```
/dev/stdin                     标准输入
/dev/stdout                    标准输出
/dev/stderr                    标准错误输出
/dev/fd/n                      文件描述符 n
```

示例1：

假设有下面的文件 test.txt:

```
test1
test2
inline test2.txt
test4
inline test3.txt
done
```

现在，把包含 inline 的行替换成它后面文件的内容：

```bash
#! /usr/bin/awk -f
{
    if($0~/inline/) {
        fileName=substr($0, length("inline ") + 1);
        while ((getline newLine <fileName) > 0) {
            print newLine;
        }
        close(fileName);
    } else {
        print;
    }
}
```







# 6. 在文件中编写

## 6.1. 注释

以 `#` 开头

```bash
#! /usr/bin/awk -f

# 定义变量 empty_row_count
BEGIN {empty_row_count=0}

# 如果是空行，则 empty_row_count+1
/^$/{empty_row_count++} 

# 打印变量 empty_row_count
END {print empty_row_count}
```


## 6.2. 命令分隔符

一行是一条命令，一行中多个命令可以使用分号 `;` 分隔：

```bash
#! /usr/bin/awk -f

# 定义变量 empty_row_count
BEGIN {empty_row_count=0};

# 如果是空行，则 empty_row_count+1
/^$/{empty_row_count++}

# 打印变量 empty_row_count
END {print empty_row_count};
```



# 7. 参考
+ [awk 精萃](http://blog.csdn.net/shangboerds/article/details/49427535)