<!-- TOC -->

- [简介](#简介)
    - [Lua与LuaJIT](#lua与luajit)
    - [hello world](#hello-world)
    - [交互模式](#交互模式)
- [数据结构](#数据结构)
    - [boolean](#boolean)
    - [number](#number)
    - [string](#string)
        - [字符串连接](#字符串连接)
        - [转换](#转换)
        - [提取](#提取)
            - [match](#match)
            - [gmatch](#gmatch)
        - [查找](#查找)
            - [find](#find)
            - [gsub](#gsub)
        - [长度](#长度)
        - [分割](#分割)
    - [table](#table)
        - [数组](#数组)
        - [排序 table.sort](#排序-tablesort)
        - [打印](#打印)
        - [table.concat](#tableconcat)
        - [table.insert](#tableinsert)
    - [function](#function)
        - [参数](#参数)
            - [变长参数](#变长参数)
            - [具名参数](#具名参数)
        - [返回值](#返回值)
        - [动态函数调用](#动态函数调用)
        - [忽略括号](#忽略括号)
        - [递归的局部变量](#递归的局部变量)
- [表达式](#表达式)
    - [赋值](#赋值)
- [控制结构](#控制结构)
    - [if else](#if-else)
    - [while](#while)
    - [repeat](#repeat)
    - [for](#for)
    - [break和return](#break和return)
    - [显示界定一个块](#显示界定一个块)
    - [迭代器](#迭代器)
        - [泛型for](#泛型for)
- [模块](#模块)
    - [require](#require)
    - [模块的加载](#模块的加载)
    - [模块的编写](#模块的编写)
- [面向对象编程](#面向对象编程)
    - [类](#类)
    - [继承](#继承)
- [系统函数](#系统函数)
    - [unpack](#unpack)
    - [异常](#异常)
        - [assert](#assert)
- [协同程序 coroutine](#协同程序-coroutine)
    - [yield](#yield)
- [元表metatable与元方法metamethod](#元表metatable与元方法metamethod)
    - [获取和设置metatable](#获取和设置metatable)
    - [相关方法](#相关方法)
    - [示例](#示例)
        - [保护metatable, 使其他人对值得metatable没有读取和设置的权限](#保护metatable-使其他人对值得metatable没有读取和设置的权限)
        - [table 的 `__index` 方法](#table-的-__index-方法)
- [环境](#环境)
- [代码片段](#代码片段)
    - [ip是否匹配网段ip](#ip是否匹配网段ip)
- [参考](#参考)

<!-- /TOC -->

# 简介
* lua由标准C编写.

## Lua与LuaJIT
* LuaJIT利用即时编译（Just-in Time）技术把 Lua 代码编译成本地机器码后交由 CPU 直接执行.
* LuaJIT 是采用 C 和汇编语言编写的 Lua 解释器与即时编译器.

## hello world

```shell
# cat hello.lua
print("hello world")
# luajit hello.lua
hello world
```

## 交互模式
* 退出: `end-of-file`(unix中为Ctrl+d), 或者 `os.exit()`


# 数据结构

```lua
print(type("hello world")) -->output:string
print(type(print))         -->output:function
print(type(true))          -->output:boolean
print(type(360.0))         -->output:number
print(type(nil))           -->output:nil
```

## boolean
* nil 和 false 为“假”，其它所有值均为“真”。比如 0 和空字符串就是“真”.


## number
* 双精度浮点数.
    ```lua
    local order = 3.99
    local score = 98.01
    print(math.floor(order))   -->output:3
    print(math.ceil(score))    -->output:99
    ```
* LuaJIT 支持所谓的“dual-number”（双数）模式，即 LuaJIT 会根据上下文用整型来存储整数，而用双精度浮点数来存放浮点数.
* LuaJIT 还支持“长长整型”的大整数
    ```lua
    print(9223372036854775807LL - 1)  -->output:9223372036854775806LL
    ```


## string

3种方式:
1. `"hello"`
2. `'hello'`
3. `[[abc]]` `[==[abc\n]==]` 其中的内容不做任何处理.

```lua
local str3 = [["add\name",'hello']]
local str4 = [=[string have a [[]].]=]
print(str3)    -->output:"add\name",'hello'
print(str4)    -->output:string have a [[]].
```


### 字符串连接
* 操作符 `..`
* 如果其任意一个操作数是数字的话，Lua 会将这个数字转换成字符串.
* 连接操作符只会创建一个新字符串，而不会改变原操作数.

```lua
print("Hello " .. "World")    -->打印 Hello World
print(0 .. 1)                 -->打印 01

str1 = string.format("%s-%s","hello","world")
print(str1)              -->打印 hello-world

str2 = string.format("%d-%s-%.2f",123,"world",1.21)
print(str2)              -->打印 123-world-1.21
```


### 转换
* 字符串转成数字: `tonumber(str)`, 返回数字或者 nil.


### 提取

#### match

```lua
pair = "name = Anna"
key, value = string.match(pair,"(%a+)%s*=%s*(%a+)")
print(key, value)  --输出name anna

date = "Today is 2012-01-02"
y, m, d = string.match(date, "(%d+)\-(%d+)\-(%d+)")
print(y, m, d)      --输出2012    01      02

-- 匹配ip网段
ip = "172.1.1.2/16"
string.match(ip, "(%d+)%.(%d+)%.(%d+)%.(%d+)/(%d+)")
```

#### gmatch
* gmatch 返回一个迭代器.


```lua
-- 提取每个单词
s = "hello world from Lua"
for w in string.gmatch(s, "%a+") do
    print(w)
end

-- 提取键值对
s = "Lua=5.2, Java=8.0"
for k, v in string.gmatch(s, "(%w+)=(%w+)") do
    print(k, v)
end
```



### 查找

#### find

```lua
local a, b = string.find("hello", "lo") -- 4 5
local a, b = string.find("hello", "li") -- nil nil
local a, b = string.find("hello", "her") -- nil nil
```

如果查找的到, 返回开始位置和结束位置. 否则返回 nil.

```
.  任意字符
%s 空白符
%p 标点
%c 控制字符
%d 数字
%x 十六进制数
%z 代表0的字符
%a 字母
%l 小写字母
%u 大写字母
%w 字母数字
```

字符类的大写形式代表相应集合的补集， 比如 `%A` 表示除了字母以外的字符集.

另外，* + - 三个，作为通配符分别表示：

* `*`： 匹配前面指定的 0 或多个同类字符， 尽可能匹配更长的符合条件的字串
* `+`： 匹配前面指定的 1 或多个同类字符， 尽可能匹配更长的符合条件的字串
* `-`： 匹配前面指定的 0 或多个同类字符， 尽可能匹配更短的符合条件的字串


#### gsub

```lua
local s = string.gsub("Lua is good", "good", "bad") -- Lua is bad
```

在参1中查找参2, 替换成参3, 参2可以是模式串. 返回替换后的字符串.


### 长度

```lua
print(#"hello")
```


### 分割

```lua
local s = "127.0.0.1/16,128.0.0.2/18"
local start = 1
while start < #s do
    local i = string.find(s, ",", start)
    if i then
        print(string.sub(s, start, i - 1))
        start = i + 1
    else
        print(string.sub(s, start))
        start = #s
    end
end
```




## table
* key 可以是除 nil 之外的任意类型.
* 最后一个元素后可以加 `,` 也可以不加.
* 还可以使用分号`;`代替逗号`,`.

```lua
local corp = {
    web = "www.google.com",   --索引为字符串，key = "web",
                              --            value = "www.google.com"
    telephone = "12345678",   --索引为字符串
    staff = {"Jack", "Scott", "Gary"}, --索引为字符串，值也是一个表
    100876,              --相当于 [1] = 100876，此时索引为数字
                         --      key = 1, value = 100876
    100191,              --相当于 [2] = 100191，此时索引为数字
    [10] = 360,          --直接把数字索引给出
    ["city"] = "Beijing" --索引为字符串
}

print(corp.web)               -->output:www.google.com
print(corp["telephone"])      -->output:12345678
print(corp[2])                -->output:100191
print(corp["city"])           -->output:"Beijing"
print(corp.staff[1])          -->output:Jack
print(corp[10])               -->output:360
```


### 数组
* 下标从1开始.

```lua
local a = {}
a[1] = "a"
a[2] = "b"

-- 打印所有行
for i = 1, #a do
    print(a[i])
end

print(a[#a]) -- 最后一个元素
a[#a] = nil -- 删除最后一个元素
a[#a + 1] = "c" -- 添加到末尾

-- 直接构造数组
local a = {"a", "b", "c"}
for i, v in pairs(a) do
    print(i, v)
end
```

* 当数组中间有值为nil的空隙时, `#` 获取的长度是将第一个遇到的nil作为结尾, 此时可以使用 `math.maxn()` 来获取最大正索引值.



### 排序 table.sort

```lua
local network = {
    {name = "grauna", ip = "210.26.30.34"},
    {name = "arraial", ip = "210.26.30.23"},
    {name = "lua", ip = "210.26.30.12"},
    {name = "derain", ip = "210.26.30.20"}
}

table.sort(
    network,
    function(a, b)
        return a.name > b.name
    end
)
```


### 打印

```lua
local function s_table(tbl, index)
    if not tbl then
        return ""
    end
    local indent = ""
    for i = 1, index do
        indent = indent .. "  "
    end
    local ret_val = "{\r\n"
    for k, v in pairs(tbl) do
        if "number" == type(k) then
            ret_val = ret_val .. "  " .. indent .. "[" .. k .. "] = "
        elseif "string" == type(k) then
            ret_val = ret_val .. "  " .. indent .. '["' .. k .. '"] = '
        end
        if "number" == type(v) then
            ret_val = ret_val .. v .. ",\r\n"
        elseif "string" == type(v) then
            ret_val = ret_val .. '"' .. v .. '"' .. ",\r\n"
        elseif "table" == type(v) then
            ret_val = ret_val .. s_table(v, index + 1) .. ",\r\n"
        else
            ret_val = ret_val .. tostring(v) .. ",\r\n"
        end
    end
    ret_val = ret_val .. indent .. "}"
    return ret_val
end

-- 将table序列化成字符串
local function serialize_table(tbl)
    if type(tbl) == "string" then
        return tbl
    end
    return "\r\n" .. s_table(tbl, 0)
end

-- 注册到table库中
table.print = function(tbl)
    print(serialize_table(tbl))
end
```


### table.concat

```lua
table.concat (table [, sep [, i [, j ] ] ])
```

对于元素是 string 或者 number 类型的表 table，返回 table[i]..sep..table[i+1] ··· sep..table[j] 连接成的字符串。填充字符串 sep 默认为空白字符串。起始索引位置 i 默认为 1，结束索引位置 j 默认是 table 的长度。如果 i 大于 j，返回一个空字符串.

```lua
local a = {1, 3, 5, "hello" }
print(table.concat(a))              -- output: 135hello
print(table.concat(a, "|"))         -- output: 1|3|5|hello
print(table.concat(a, " ", 4, 2))   -- output:
print(table.concat(a, " ", 2, 4))   -- output: 3 5 hello
```

对于 `table.concat(a, "\n") .. "\n"`, 当 a 很大时, 需要复制整个字符串然后和最后的换行组合, 开销很大, 可以在 a 的最后插入一个 "\n", 这样就不需要最后的字符串连接操作了.


### table.insert

```lua
table.insert (table, [pos ,] value)
```

在（数组型）表 table 的 pos 索引位置插入 value，其它元素向后移动到空的地方。pos 的默认值是表的长度加一，即默认是插在表的最后。

```lua
local a = {1, 8}             --a[1] = 1,a[2] = 8
table.insert(a, 1, 3)   --在表索引为1处插入3
print(a[1], a[2], a[3])
table.insert(a, 10)    --在表的最后插入10
print(a[1], a[2], a[3], a[4])
```






## function

```lua
local function foo()
    print("in the function")
    --dosomething()
    local x = 10
    local y = 20
    return x + y
end

local a = foo    --把函数赋给变量

print(a())

--output:
in the function
30
```

上面等价于:

```lua
local foo = function()
end
```

匿名函数:

```lua
foo = function ()
end
```

将函数赋值给表的某个字段:

```lua
function foo.bar(a, b, c)
    -- body ...
end
-- 等价于
foo.bar = function (a, b, c)
    print(a, b, c)
end
```


### 参数
* 值传递, 除了 table 是按引用传递.
* 如果形参多于实参, 多余的形参赋值为 nil, 如果形参少于实参, 多余的实参被忽略.

#### 变长参数

```lua
local function func( ... )                -- 形参为 ... ,表示函数采用变长参数

   local temp = {...}                     -- 访问的时候也要使用 ..., 构造成一个数组
   local ans = table.concat(temp, " ")    -- 使用 table.concat 库函数对数
                                          -- 组内容使用 " " 拼接成字符串。
   print(ans)
end

func(1, 2)        -- 传递了两个参数
func(1, 2, 3, 4)  -- 传递了四个参数

-->output
1 2
1 2 3 4
```

**值得一提的是，LuaJIT 2 尚不能 JIT 编译这种变长参数的用法，只能解释执行。所以对性能敏感的代码，应当避免使用此种形式。**

变长参数赋值给局部变量: `local a, b = ...`.

直接返回变长参数:

```lua
function foo(...)
    print("args:", ...)
    return ...
end

local a, b = foo("a", "b")
print(string.format("a = %s, b = %s", a, b))
```

遍历带有nil的可变参数:

```lua
function foo(...)
    for i = 1, select("#", ...) do -- 第一个参数是#时将返回可变参数的个数, 包括nil
        local arg = select(i, ...) -- 第一个参数是数字时将返回第i个参数
        print(string.format("arg[%d] = %s", i, arg))
    end
end

foo("a", "b", nil)
```



#### 具名参数

```lua
function rename( arg )
    return os.rename(arg.old, arg.new)
end

rename{old="a.lua", new="b.lua"} -- 利用只有一个参数时可以省略括号的特性
-- 等价于下面的语句
rename({old="a.lua", new="b.lua"})
```


### 返回值

可以返回多个值:

```lua
local function swap(a, b)   -- 定义函数 swap，实现两个变量交换值
   return b, a              -- 按相反顺序返回变量的值
end

local x = 1
local y = 20
x, y = swap(x, y)           -- 调用 swap 函数
print(x, y)                 --> output   20     1
```

当函数返回值的个数和接收返回值的变量的个数不一致时，Lua 也会自动调整参数个数。

调整规则： 若返回值个数大于接收变量的个数，多余的返回值会被忽略掉； 若返回值个数小于参数个数，从左向右，没有被返回值初始化的变量会被初始化为 nil。

当一个函数有一个以上返回值，且函数调用不是一个列表表达式的最后一个元素，那么函数调用只会产生一个返回值, 也就是第一个返回值。

```lua
local function init()       -- init 函数 返回两个值 1 和 "lua"
    return 1, "lua"
end

local x, y, z = init(), 2   -- init 函数的位置不在最后，此时只返回 1
print(x, y, z)              -->output  1  2  nil

local a, b, c = 2, init()   -- init 函数的位置在最后，此时返回 1 和 "lua"
print(a, b, c)              -->output  2  1  lua
```

函数调用的实参列表也是一个列表表达式。考虑下面的例子：

```lua
local function init()
    return 1, "lua"
end

print(init(), 2)   -->output  1  2
print(2, init())   -->output  2  1  lua
```

如果你确保只取函数返回值的第一个值，可以使用括号运算符，例如

```lua
local function init()
    return 1, "lua"
end

print((init()), 2)   -->output  1  2
print(2, (init()))   -->output  2  1
```


**值得一提的是，如果实参列表中某个函数会返回多个值，同时调用者又没有显式地使用括号运算符来筛选和过滤，则这样的表达式是不能被 LuaJIT 2 所 JIT 编译的，而只能被解释执行。**


### 动态函数调用

```lua
local function run(x, y)
    print('run', x, y)
end

local function attack(targetId)
    print('targetId', targetId)
end

local function do_action(method, ...)
    local args = {...} or {}
    method(unpack(args, 1, table.maxn(args)))
end

do_action(run, 1, 2)         -- output: run 1 2
do_action(attack, 1111)      -- output: targetId    1111
```

### 忽略括号
如果只有一个参数, 且这个参数是一个字面字符串或table构造式, 那么括号可以忽略.

```lua
print "hello"
dofile 'a.lua'
f {x = 1, y = 2}
type {}
```


### 递归的局部变量

在递归中引用自身是**错误**的:

```lua
local fact = function(n)
    if n == 0 then return 1
    else return n * fact(n-1)
    end
end
```

在编译到 `fact(n-1)` 时, fact 还未定义完, 所以这里调用的是全局的 fact 函数. 解决方法是先定义变量再赋值:

```lua
local fact

fact = function(n)
    ...
end
```

或者

```lua
local fact

function fact(n) -- 不能写 local function fact, 否则会创建新的局部变量
    ...
end
```

语法糖 `local function fact(n)` 就是先定义的 fact 变量再赋值的.





# 表达式
* 算术运算符: `+, -, *, /, ^(指数), %(取模)`
* 关系运算符: `<, >, <=, >=, ==, ~=(不等于)`
* 逻辑运算符: `and, or, not`
* 对于 table, userdate 和函数, lua 是做引用比较.
* 字符串会被内化, 相同字符串只保存一份, 所以可以直接比较.
* `a and b` 如果 a 为 nil，则返回 a，否则返回 b. (尽量返回nil)
* `a or b` 如果 a 为 nil，则返回 b，否则返回 a。 (尽量不返回nil)
* 所有逻辑操作符将 false 和 nil 视作假，其他任何值视作真，对于 and 和 or，“短路求值”，对于 not，永远只返回 true 或者 false。
* 取模操作: `a % b == a - floor(a/b) * b`.

整数相除结果可能是浮点数:

```lua
print(5 / 10) -- 0.5 注意这里是浮点数
print(10 / 5) -- 2
```

其他一些实例:

```lua
print(3.14%1) -- 小数部分 0.14
print(3.14 - 3.14%1) -- 整数部分 3
print(3.1415 - 3.1415%0.01) -- 精确到小数点后两位 3.14
```

## 赋值
* 多重赋值: `a, b = 1, 2`, 先计算右边的, 后赋值. 互换变量:`x, y = y, x`.
* 多重赋值两个个数可以不对等: `a, b = 1, 2, 3`, 多的丢弃, 少的赋值为nil.



# 控制结构

## if else

```lua
score = 90
if score == 100 then
    print("Very good!Your score is 100")
elseif score >= 60 then
    print("Congratulations, you have passed it,your score greater or equal to 60")
--此处可以添加多个elseif
else
    print("Sorry, you do not pass the exam! ")
end
```

```lua
x = x or y -- 等价于 if not x then x = y end
max = (x > y) and x or y -- 选出最大值, 等价于 x > y ? x:y, 前提是 x 不为假, 如果 x 为假, 结果会是 y.
```


## while
* 没有 continue, 但是有 break.

```lua
while 表达式 do
--body
end
```

## repeat
* 知道 until 的条件为 true 时才跳出循环.

```lua
x = 10
repeat
    print(x)
until false
```

在代码块中声明的局部变量在util中仍然可以访问:

```lua
a = 1
repeat
    local b = a%5
    a = a+1
until b == 0 -- 仍然可以访问b

print(a)
```


## for

数字型 for 的语法如下：

```lua
for var = begin, finish, step do
    --body
end

-- 示例
for i = 1, 5 do
  print(i) -- 输出 1 到 5
end

-- 倒序
for i = 10, 1, -1 do
  print(i)
end

-- finish 没有上限
for i = 1, math.huge do
    if (0.3*i^3 - 20*i^2 - 500 >=0) then
      print(i)
      break
    end
end
```

1. var 从 begin 变化到 finish，每次变化都以 step 作为步长递增 var
2. begin、finish、step 三个表达式只会在循环开始时执行一次
3. 第三个表达式 step 是可选的，默认为 1
4. 控制变量 var 的作用域仅在 for 循环内，需要在外面控制，则需将值赋给一个新的变量
5. 循环过程中不要改变控制变量的值，那样会带来不可预知的影响


for 泛型:

* 泛型 for 循环通过一个迭代器（iterator）函数来遍历所有值
* 决不应该对循环变量作任何赋值.

遍历数组:

```lua
-- 打印数组a的所有值
local a = {"a", "b", "c", "d"}
for i, v in ipairs(a) do
  print("index:", i, " value:", v)
end

-- output:
index:  1  value: a
index:  2  value: b
index:  3  value: c
index:  4  value: d
```

遍历 table:

```lua
-- 打印table t中所有的key
for k in pairs(t) do
    print(k)
end
```

迭代器:
* `io.lines`: 迭代文件中每行.
* `pairs`: 迭代 table.
* `ipairs`: 迭代数组.
* `string.gmatch`: 迭代字符串中单词.


## break和return

调试:

```lua
local function foo()
    print("before")
    do return end
    print("after")  -- 这一行语句永远不会执行到
end
```

## 显示界定一个块

```lua
do
    ...
end
```

相当于 `{}`.


## 迭代器

```lua
local function values(t) -- values 就是一个工厂
    local i = 0
    return function()
        i = i + 1
        return t[i]
    end
end

local t = {10, 20, 30}

-- 在 while 中使用迭代器
local iter = values(t)
while true do
    local v = iter()
    if v == nil then
        break
    end
    print(v)
end

-- 在 for 中使用迭代器
for v in values(t) do -- 内部保存了iter函数, 遇到nil时停止
    print(v)
end
```


### 泛型for

语法:

```lua
for <val-list> in <exp-list> do
    <body>
end
```

* exp-list 可以返回3个值(多余3个的被忽略), 迭代器函数, 恒定状态, 控制变量的初值.(恒定状态比如一个数组,控制变量是下标)
* 每次循环, 将恒定状态和控制变量传给迭代器函数.
* 可以实现无状态的迭代器, 相比如上面的迭代器不需要记住local变量.

伪代码:

```lua
do
    local _f, _s, _var = <exp-list>
    while true do
        local var_1, ..., var_n = _f(_s, _var)
        _var = var_1
        if _var == nil then
            break
        end
        <body>
    end
end
```

实现ipairs:

```lua
local function iter(a, i)
    i = i + 1
    local v = a[i]
    if v then
        return i, v
    end
end

local function my_ipairs(a)
    return iter, a, 0
end

a = {10, 20, 30}
for i, v in my_ipairs(a) do
    print(i, ": ", v)
end
```






# 模块
* 通过 `require("模块名即文件名")` 的方式导入模块.
* 模块的最后要 return 一个 table.
* 已加载的模块可以再次被 require.

## require

```lua
require("mod") -- luajit 中失败
mod.func()

local m = require("mod")
m.func()
```

## 模块的加载
* Lua 文件的搜索路径存放在 `package.path` 变量中(环境变量 `LUA_PATH`). c 程序库的搜索路径在 `package.cpath` 变量中(环境变量 `LUA_CPATH`).
* 路径值以分号分隔, 比如 `?;?.lua;/usr/local/lua/?/?.lua`, 当要搜索的路径是 sql 时, 查找的顺序是 `sql;sql.lua;/usr/local/lua/sql/sql.lua`.
* 路径中的 `;;` 会替换成默认路径.
* 子模块 `a.b` 在搜索文件时会将 `.` 换成系统的目录分隔符.


## 模块的编写

```lua
local _M = {_VERSION = "0.10"}

function _M.hello() -- 函数
    print("hello")
end

_M.i = 123 -- 变量

local function world()
    _M.hello() -- 内部调用模块的属性
end

return _M
```


# 面向对象编程

```lua
local _M = {_VERSION = "0.10"}

function _M.hello(self, name)
    print("version: ", self._VERSION, " hello ", name)
end

function _M.haha(m, name) -- 不一定非得使用 self.
    print(string.format("[%s] haha %s", m._VERSION, name))
end

function _M:hello2(name)
    print(string.format("[%s] hello %s", self._VERSION, name))
end

_M:hello("chen")
_M:haha("name")
_M:hello2("zhang")
```

* 调用时 `:` 相当于 `_M.hello(_M, "chen")` 的缩写.
* 使用 self 而不是 `_M` 是为了避免改名.
* 冒号的作用是在定义中添加一个额外的隐藏参数, 在调用中添加一个额外的实参.

## 类
* Lua 中没有类的概念, 但可以模拟类.

```lua
Account = {
    balance = 0
}

function Account:new( o )
    o = o or {} -- 没有提供的话就创建一个
    setmetatable(o, self) -- 以自身为元表
    self.__index = self -- 复写 __index, 这样 o 才具有 Account 的方法
    return o
end

function Account:deposit( v )
    self.balance = self.balance + v
end

a = Account:new({balance = 100})
a:deposit(20)
print(a.balance) -- 120
```

## 继承

```lua
SpecialAccount = Account:new({limit = 1000}) -- 定义一个新类

function SpecialAccount:deposit(v) -- 复写方法
    if v > self.limit then
        error("exceed limit:" .. self.limit)
    end
    self.balance = self.balance + v
    print("now, balance is ", self.balance)
end

sa = SpecialAccount:new() -- 继承自Account的方法
sa:deposit(20)
sa:deposit(2000)
```

* `SpecialAccount:new()` 中 self 变成了 SpecialAccount, sa 的元表是 SpecialAccount, SpecialAccount 的 `__index` 是 SpecialAccount.
* s 继承自 SpecialAccount, 而 SpecialAccount 继承自 Account.



# 系统函数

## unpack
接收一个数组参数, 并从1开始返回数组的所有元素.

```lua
unpack({4, 6, 2, 1})

a = {"hello", "ll"}
print(string.find(unpack(a))) -- 等价于下面的直接调用
print(string.find("hello", "ll"))
```


## 异常

* 使用 `error()` 抛出异常.
* 使用 `pcall(func[, arg1, ...])` 捕获异常, 第一个返回值表示是否有异常, 其他返回值为函数返回值.
* pcall => protected call, 受保护的调用.

```lua
local f = function(panic)
    if panic then
        error("exception")
    end
    return "ok"
end

print(pcall(f, false)) -- true ok
print(pcall(f, true)) -- false hello.lua:3: exception
print(f(true)) -- 将会打印堆栈信息
```

error() 中也可以返回除字符串之外的其他类型, 这个类型同样也会返回给pcall:

```lua
local function f()
    error({code = 1})
end

local ok, err = pcall(f)
if not ok then
    print(err.code) -- 1
end
```


### assert

```lua
assert(f(), "message")
```

* assert 做的工作相当于: `if not <condition> then error(message) end`.
* 当 f 返回 nil 或 false 时抛出一个错误, 只检查返回的第一个参数.
* 如果 f() 返回了 `nil, "msg"`, 并且 assert 没有第二个参数, 那么就将 "msg" 当做错误信息.



# 协同程序 coroutine

```lua
local function f()
    print("hello")
end
-- 创建
local co = coroutine.create(f)

-- 检查状态
print(coroutine.status(co)) -- suspended

-- 启动或再次启动
coroutine.resume(co)

print(coroutine.status(co)) -- dead
```

* 4个状态: suspended(挂起), running(运行), dead(死亡), normal(正常)


## yield
* yield() 挂起当前coroutine, 将执行权交给 resume() 的调用.
* resume() 的参数如果是一个已经 dead 的 coroutine 时, 将返回 false 和 错误信息, 正常情况下返回 true.
* 当 A 唤醒 B 时, A 处于 "正常" 状态.

```lua
local function f()
    for i = 1, 10 do
        print("co: ", i)
        coroutine.yield()
    end
end
-- 创建
local co = coroutine.create(f)

for i = 1, 12 do
    print(coroutine.resume(co))
    print("main: ", i)
end
```


使用 resume-yield 机制交换数据:

```lua
local function add(a, b)
    for i = 1, 3 do
        print(string.format("a = %s, b = %s", a, b))
        a, b = coroutine.yield(a + b) -- yield 的参数会返回给 resume
    end
    return 0
end
-- 创建
local co = coroutine.create(add)

local a, b = 1, 1
while true do
    local ok, res = coroutine.resume(co, a, b)
    if not ok then
        break
    end
    print(string.format("%s + %s = %s", a, b, res)) -- 最后一次返回的是 add 最后的 return
    a = b
    b = res
end
```




# 元表metatable与元方法metamethod
* 修改值的行为, 比如 `a + b`.
* 每个值都有一个元表. table 和 userdata 有各自独立的元表, 其他类型的值共享类型的单一元表.
* table 可以成为自己的元表, 描述其特有的行为.
* table 创建时不会创建元表
* 只能设置 table 的元表, 其他类型需要通过c代码完成.
* 查找元表: 先找第一个值的, 没有相应方法的话再找第二个值的, 都没有的话报错.

## 获取和设置metatable

```lua
t = {}
print(getmetatable(t)) -- nil

t1 = {}
setmetatable(t, t1)
print(getmetatable(t) == t1) -- true
```

## 相关方法
算术类的元方法:
* `__add`: + 加
* `__sub`: - 减
* `__mul`: * 乘
* `__div`: / 除
* `__unm`: 相反数
* `__mod`: 取模
* `__pow`: 乘幂

关系类的元方法:
* `__eq`: 等于, `~=` 会转化为 `not (a == b)`
* `__lt`: 小于, 大于会转化为小于
* `__le`: 小于等于, 大于等于会转化为小于等于
* 关系类的元方法不能用于混合类型, 否则会引发错误. 等于比较用于不会引发错误(如果两个对象拥有不同的元方法, 等于操作不会调用任何元方法, 直接返回false)

table的元方法:
* `__index`: 查询 key 时如果key不存在, 调用该方法. 参数为 table, key
* `__rawget`: 查询 key 时不管 key 是否存在都不调用 `__index`.
* `__newindex`: 更新 table 时如果 key 不存在, 调用该方法. 参数为 table, key, value
* `__rawset`: 更新 table 时不管 key 是否存在都不调用 `__newindex`.


其他:
* `__tostring`: 参1: 当前对象.


## 示例

```lua
local mt = {
    __add = function(a, b)
        return 250
    end,
    __tostring = function()
        return "hello"
    end
}

function newSet()
    local set = {}
    setmetatable(set, mt)
    return set
end

local set1 = newSet()
local set2 = newSet()
print(set1 + set2) -- 250
print(set1)
```

### 保护metatable, 使其他人对值得metatable没有读取和设置的权限

```lua
local mt = {
    __metatable = "not your business" -- 设置这个值用于保护metatable
}

function newSet()
    local set = {}
    setmetatable(set, mt)
    return set
end

local set = newSet()
print(getmetatable(set)) -- not your business

setmetatable(set, {}) -- error: cannot change a protected metatable
```


### table 的 `__index` 方法

```lua
Window = {}
-- 原型
Window.prototype = {x = 0, y = 0, w = 100, h = 100}
-- 元表
Window.mt = {
    __index = function(table, key)
        return Window.prototype[key]
    end,
    __tostring = function(w)
        return string.format("x: %s, y: %s, w: %s, h: %s", w.x, w.y, w.w, w.h)
    end
}

function Window.new(o)
    setmetatable(o, Window.mt)
    return o
end

w = Window.new {x = 10, y = 20}
print(w) -- x: 10, y: 20, w: 100, h: 100
```

* `__index` 还可以是一个 table, 比如上例可以直接写成: `__index = Window.prototype`.
* 当不想访问 `__index` 时可以使用 `rawget(table, key)` 的方式来访问而不触发 `__index`.




# 环境
* Lua 中所有的全局变量都放在 `_G` table 中, key 是变量名, value 是一个 table.






# 代码片段

## ip是否匹配网段ip

```lua
local binary_cache = {}
do
    for i = 1, 8 do
        binary_cache[i] = 2 ^ (8 - i)
    end
end

-- print(table.concat(binary_cache, ", "))

local function num_to_bin(n)
    local bin = {0, 0, 0, 0, 0, 0, 0, 0}
    for i = 1, 8 do
        -- print(i, ": ", n, ", ", binary_cache[i])
        if n >= binary_cache[i] then
            bin[i] = 1
            n = n - binary_cache[i]
        end
    end
    return table.concat(bin)
end

local function compare_binary(s1, s2, mask)
    local n1, n2 = tonumber(s1), tonumber(s2)
    local b1, b2 = num_to_bin(n1), num_to_bin(n2)
    print(string.format("b1: %s, b2: %s, mask: %s", b1, b2, mask))
    return string.sub(b1, 1, mask) == string.sub(b2, 1, mask)
end

-- ip_tpl: 172.3.4.5/18
-- ip:     172.3.5.6
-- 判断 ip 是否符合 ip_tpl 指定的网段
local function ip_match(ip_tpl, ip)
    local tpl, mask = {}, nil
    tpl[1], tpl[2], tpl[3], tpl[4], mask = string.match(ip_tpl, "(%d+)%.(%d+)%.(%d+)%.(%d+)/(%d+)")
    if not mask then
        print("not ip_tpl")
        return
    end
    mask = tonumber(mask)
    -- print(table.concat(tpl, ", "), mask)

    local parts = {}
    parts[1], parts[2], parts[3], parts[4] = string.match(ip, "(%d+)%.(%d+)%.(%d+)%.(%d+)")
    if not parts[4] then
        print("not ip")
        return
    end
    -- print(table.concat(parts, ", "))

    local seg = 1
    while mask > 0 and seg <= 4 do
        -- print("seg: ", parts[seg], ", ", tpl[seg])
        -- 大于等于8位直接比较ip中的字符串
        if mask >= 8 then
            if parts[seg] ~= tpl[seg] then
                return false
            end
            mask = mask - 8
            seg = seg + 1
        else -- 少于8位需要转换成二进制
            return compare_binary(parts[seg], tpl[seg], mask)
        end
    end

    return true
end

print("match: ", ip_match("172.3.4.5/18", "172.3.5.6"))
print("match: ", ip_match("172.3.4.5/18", "173.3.4.5"))
print("match: ", ip_match("172.3.4.5/24", "172.3.4.255"))
```




# 参考
* [Lua 官网](http://www.lua.org)
* [LuaJIT 官网](http://luajit.org)
