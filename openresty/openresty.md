<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [概念](#概念)
	- [epoll](#epoll)
- [安装](#安装)
- [hello world](#hello-world)
	- [启动](#启动)
- [lua ngx api](#lua-ngx-api)
	- [ngx.var.VARIABLE](#ngxvarvariable)
	- [时间](#时间)
	- [查询参数](#查询参数)
	- [请求body](#请求body)
	- [worker](#worker)
	- [timer](#timer)
	- [header](#header)
	- [ctx](#ctx)
- [shared_dict](#shareddict)
	- [获取字典对象](#获取字典对象)
	- [ngx.shared.DICT.set](#ngxshareddictset)
	- [ngx.shared.DICT.safe_set](#ngxshareddictsafeset)
	- [ngx.shared.DICT.add](#ngxshareddictadd)
	- [ngx.shared.DICT.safe_add](#ngxshareddictsafeadd)
	- [ngx.shared.DICT.get](#ngxshareddictget)
	- [ngx.shared.DICT.get_stale](#ngxshareddictgetstale)
	- [ngx.shared.DICT.replace](#ngxshareddictreplace)
	- [ngx.shared.DICT.delete](#ngxshareddictdelete)
	- [ngx.shared.DICT.incr](#ngxshareddictincr)
	- [ngx.shared.DICT.flush_all](#ngxshareddictflushall)
	- [ngx.shared.DICT.flush_expired](#ngxshareddictflushexpired)
	- [ngx.shared.DICT.get_keys](#ngxshareddictgetkeys)
- [json](#json)
- [log](#log)
- [redis](#redis)
	- [scan keys](#scan-keys)
- [编码、加密](#编码加密)
	- [base64](#base64)
	- [hmac_sha1](#hmacsha1)
- [负载均衡](#负载均衡)
	- [动态负载均衡](#动态负载均衡)
	- [获取使用的 upstream server 的信息](#获取使用的-upstream-server-的信息)
- [重定向](#重定向)
	- [ngx.redirect](#ngxredirect)
- [原理](#原理)
	- [请求的处理流程](#请求的处理流程)
- [日志](#日志)
	- [将日志打到远程服务](#将日志打到远程服务)
- [性能](#性能)
	- [性能测试](#性能测试)
		- [注意点](#注意点)
- [正则](#正则)
	- [ngx.re](#ngxre)
		- [ngx.re.match](#ngxrematch)
		- [ngx.re.find](#ngxrefind)
		- [ngx.re.gmatch](#ngxregmatch)
		- [ngx.re.sub](#ngxresub)
		- [ngx.re.gsub](#ngxregsub)
- [参考](#参考)

<!-- /TOC -->


# 概念

## epoll
* 打开的文件描述符不受限制.
* IO就绪后通过回调的方式通知.

# 安装
- https://openresty.org/cn/installation.html

在 Mac 上:

```shell
brew install homebrew/nginx/openresty
```


# hello world

nginx.conf

```
worker_processes  1;        #nginx worker 数量
error_log logs/error.log;   #指定错误日志文件路径
events {
    worker_connections 1024;
}

http {
    server {
        #监听端口，若你的6699端口已经被占用，则需要修改
        listen 6699;
        location / {
            default_type text/html;

            content_by_lua_block {
                ngx.say("HelloWorld")
            }
        }
    }
}
```

提示：如果你安装的是 openresty 1.9.3.1 及以下版本，请使用 `content_by_lua` 命令代替示例中的 `content_by_lua_block`。可使用 `nginx -V` 命令查看版本号。


## 启动

```
/usr/local/openresty/nginx/sbin/nginx -c /usr/local/openresty/nginx/conf/nginx.conf
/usr/local/openresty/nginx/sbin/nginx -p 'pwd' -c /usr/local/openresty/nginx/conf/nginx.conf
```

* `-c` 指定配置文件.
* `-p` 指定工作目录


# lua ngx api
* [Lua Nginx API](http://openresty-reference.readthedocs.io/en/latest/Lua_Nginx_API/#introduction)


## ngx.var.VARIABLE
- [使用 Nginx 内置绑定变量](https://moonbingbing.gitbooks.io/openresty-best-practices/openresty/inline_var.html)

```
location /foo {
     set $my_var 'hi'; # this line is required to create $my_var at config time
     content_by_lua_block {
         ngx.var.my_var = 123;
         ...
     }
 }
```

```lua
ngx.var.scheme
ngx.var.host
ngx.var.request_uri  -- 包含查询参数, 比如 /test?a=1
ngx.var.uri          -- 不包含查询参数, 比如 /test
ngx.var.remote_addr
ngx.var.request_id
```


## 时间
* `ngx.today()`       "yyyy-mm-dd" 格式 (2017-08-23)
* `ngx.time()`        时间戳, 秒 (1503487721)
* `ngx.now()`         带毫秒的时间戳 (1503487721.425)
* `ngx.update_time()` 使用系统时间强制更新nginx缓存时间, 不返回数据.
* `ngx.localtime()`   "yyyy-mm-dd hh:mm:ss" 格式 (2017-08-23 11:28:41)
* `ngx.utctime()`     "yyyy-mm-dd hh:mm:ss" 格式 (2017-08-23 11:28:41)



## 查询参数

- 使用 `ngx.var.arg_xxx` 的形式, 比如获取参数 a 的值: `ngx.var.arg_a`, 返回的是 string 类型.
- 使用 `ngx.req.get_uri_args()["param_name"]` 的形式, 比如获取参数 a 的值: `ngx.req.get_uri_args()["a"]`, 如果 a 只有一个值, 那么返回 string 类型, 如果 a 有多个值, 返回一个 table.




## 请求body

单个接口读取:

```lua
ngx.req.read_body() -- nginx 默认不读取body, 需要先使用这个读取
local data = ngx.req.get_body_data()
```

所有接口都读取:

在 nginx.conf 中的 server 下配置: `lua_need_request_body on;`

请求参数:

```lua
ngx.req.read_body()
local post_args = ngx.req.get_post_args()
```


## worker

```lua
worker_id = ngx.worker.id() -- 范围是 [0, n)
ngx.worker.exiting() -- 当前 worker 是否正在关闭(如reload、shutdown期间)
```


## timer

```lua
ok, err = ngx.timer.at(delay, callback, user_arg1, user_arg2, ...)
```
* delay: 单位秒, 可以使用小数指定毫秒(0.001表示1毫秒), 0表示立即执行.
* 参数是: `premature, user_arg1, user_arg2...`, premature 为 true 表示未到达指定时间, 比如进程 shutdown 了, 这是就不能再调用 `timer.at` 了(delay指定为0除外)


## header

设置 header:

```lua
ngx.req.set_header("name", "value")  -- 设置请求 header
ngx.header.name = "value"            -- 设置响应 header
```

读取 header:

```lua
ngx.req.get_headers()["name"]
```


## ctx

`ngx.ctx` 是一个 table, 可以在各个阶段之间共享数据.

```lua
ngx.ctx.name = "value
```




# shared_dict

在配置中的 http 下定义shared_dict, 语法: `lua_shared_dict <name> <size>`.

共享内存对象对所有 worker 进程都是可见的. reload 时字典会重新获取内容, 退出时内容丢失.

示例:

```nginx
http {
    lua_shared_dict dogs 10m;
    server {
        location /set {
            content_by_lua '
                local dogs = ngx.shared.dogs
                dogs:set("Jim", 8)
                ngx.say("STORED")
            ';
        }
        location /get {
            content_by_lua '
                local dogs = ngx.shared.dogs
                ngx.say(dogs:get("Jim"))
            ';
        }
    }
}
```

## 获取字典对象

```
dict = ngx.shared.dogs
dict = ngx.shared["dogs"]
```


## ngx.shared.DICT.set

语法: `success, err, forcible = ngx.shared.DICT:set(key, value, exptime?, flags?)`

* 即使key已存在, 也会set(覆盖原值).
* `forcible`: true表明删除了其他key(LRU算法), false表明没有删除其他key.
* `exptime`: 过期时间, 单位秒, 默认值0(永不过期).
* `flags`: 用户标记值, 在 get 时会返回.

```
local cats = ngx.shared.cats
local succ, err, forcible = cats.set(cats, "Marry", "it is a nice cat!")
```

或者

```
local cats = ngx.shared.cats
local succ, err, forcible = cats:set("Marry", "it is a nice cat!")
```


## ngx.shared.DICT.safe_set

语法: `ok, err = ngx.shared.DICT:safe_set(key, value, exptime?, flags?)`

与 set 的区别: 当设置的内存大小用完时, 不会删除其他key, 会返回 nil 和 err(no memory).


## ngx.shared.DICT.add

语法: `success, err, forcible = ngx.shared.DICT:add(key, value, exptime?, flags?)`

与 set 类似, 但不会插入重复的key, 如果 key 已存在, 返回 nil 和 err(exists).

## ngx.shared.DICT.safe_add

语法: `ok, err = ngx.shared.DICT:safe_add(key, value, exptime?, flags?)`

与 safe_set 类似, key 已存在时不插入, 返回 nil 和 err(exists).


## ngx.shared.DICT.get

语法: `value, flags = ngx.shared.DICT:get(key)`

若 key 不存在或已过期, 返回 nil. flags 是 set 时传的值, 默认0.

```
local cats = ngx.shared.cats
local value, flags = cats.get(cats, "Marry")
-- 或者
local cats = ngx.shared.cats
local value, flags = cats:get("Marry")
```


## ngx.shared.DICT.get_stale

语法: `value, flags, stale = ngx.shared.DICT:get_stale(key)`

过期的 key 也会返回, stale 为 true 表示key已过期.


## ngx.shared.DICT.replace

语法: `success, err, forcible = ngx.shared.DICT:replace(key, value, exptime?, flags?)`

只对已存在的 key 操作, 若 key 不存在, 返回 nil 和 err(not found).


## ngx.shared.DICT.delete

语法: `ngx.shared.DICT:delete(key)`

等价于: `ngx.shared.DICT:set(key, nil)`


## ngx.shared.DICT.incr

语法: `newval, err, forcible? = ngx.shared.DICT:incr(key, value, init?)`

value 必须是 number 类型, 可正可负, 若不是 number 类型, 会返回 nil 和 err(not a number).

如果 key 不存在或者已过期:
- init 如果没有指定或者设置为 nil, incr 将返回 `nil, err="not found"`.
- 如果 init 指定了 number 类型的值, incr 会创建 key, value 使用 `init+value`.

forcible 在没有指定 init 时总是返回 nil, 如果 incr 移除了其他未过期的项, 则 forcible 为 true, 如果没有移除其他未过期项, forcible 为 false.


## ngx.shared.DICT.flush_all

语法: `ngx.shared.DICT:flush_all()`

将每个字段标记为过期, 不会真正释放占用的内存.


## ngx.shared.DICT.flush_expired

语法: `flushed = ngx.shared.DICT:flush_expired(max_count?)`

清除过期字段, 与 flush_all 不同的是会释放掉占用的内存.

如果 max_count 没设置或设置为 0, 清除所有过期的字段. flushed 为清除的字段数目.


## ngx.shared.DICT.get_keys

语法: `keys = ngx.shared.DICT:get_keys(max_count?)`

max_count 指定返回的 key 的个数, 默认 1024, 没设置或设置为0, 表示不限定个数.

当返回的 key 过多时, 可能导致字典被锁定, 阻塞其他 worker.



# json

```lua
local cjson = require "cjson"
local ok, json_obj = pcall(cjson.decode, json_str)
local str = cjson.encode(obj)
```



# log

```lua
ngx.log(ngx.ERR, "message")
```

日志级别:
- ngx.STDERR     -- 0, 标准输出
- ngx.EMERG      -- 1, 紧急报错
- ngx.ALERT      -- 2, 报警
- ngx.CRIT       -- 3, 严重，系统故障，触发运维告警系统
- ngx.ERR        -- 4, 错误，业务不可恢复性错误
- ngx.WARN       -- 5, 告警，业务中可忽略错误
- ngx.NOTICE     -- 6, 提醒，业务比较重要信息
- ngx.INFO       -- 7, 信息，业务琐碎日志信息，包含不同情况判断等
- ngx.DEBUG      -- 8, 调试



# redis
- [lua-resty-redis](https://github.com/openresty/lua-resty-redis)


## scan keys

```lua
local index = 0
local query_count = 0
repeat
    local results, err = red:scan(index)
    if not results then
        log(ERR, "Failed to query, index:", index, ", err:", err)
        return
    end
    index = tonumber(results[1])
    local keys = results[2]
    if keys and #keys > 0 then
        local values, err = red:mget(unpack(keys))
        if not values then
            log(ERR, "Failed to query values, keys:", table.concat(keys, ','), ", err:", err)
            return
        end
        for i, key in ipairs(keys) do
            -- TODO: do something with key and values[i]
        end
    end
    query_count = query_count + #keys
until index == nil or index == 0
```


# 编码、加密

## base64

```lua
ngx.encode_base64(msg)
ngx.decode_base64(digest)
```



## hmac_sha1

语法: `digest = ngx.hmac_sha1(secret_key, str)`

该方法主要用于计算输入字符串str的HMAC-SHA1的摘要，并根据secret_key对结果进行转换，计算后得到的结果是二进制格式的，可以通过ngx.encode_base64转换成非二进制格式的字符串，例如：

```lua
local key = "thisisverysecretstuff"
local src = "some string we want to sign"
local digest = ngx.hmac_sha1(key, src)
ngx.say(ngx.encode_base64(digest))
```



# 负载均衡

## 动态负载均衡
* [ngx.balancer](https://github.com/openresty/lua-resty-core/blob/master/lib/ngx/balancer.md)

```lua
http {
    upstream backend {
        server 0.0.0.1;   # just an invalid address as a place holder

        balancer_by_lua_block {
            local balancer = require "ngx.balancer"

            -- well, usually we calculate the peer's host and port
            -- according to some balancing policies instead of using
            -- hard-coded values like below
            local host = "127.0.0.2"
            local port = 8080

            local ok, err = balancer.set_current_peer(host, port)
            if not ok then
                ngx.log(ngx.ERR, "failed to set the current peer: ", err)
                return ngx.exit(500)
            end
        }

        keepalive 10;  # connection pool
    }

    server {
        # this is the real entry point
        listen 80;

        location / {
            # make use of the upstream named "backend" defined above:
            proxy_pass http://backend/fake;
        }
    }

    server {
        # this server is just for mocking up a backend peer here...
        listen 127.0.0.2:8080;

        location = /fake {
            echo "this is the fake backend peer...";
        }
    }
}
```

- 这种方式下 request 的 Host 头可能会是 upstream 的名称, 可以使用 `proxy_set_header Host $proxy_host;` 重新设置, [参考](http://liuluo129.iteye.com/blog/1943311)




## 获取使用的 upstream server 的信息

通过 upstream 访问后端服务时, 在 log 阶段通过 `ngx.var.upstream_addr` 和 `ngx.var.upstream_status` 访问使用的是哪个服务以及状态.

参考:
- [1](https://www.zhihu.com/question/56193805)
- [2](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#var_upstream_status)




# 重定向
* [work_with_location](https://moonbingbing.gitbooks.io/openresty-best-practices/openresty/work_with_location.html)

## ngx.redirect

语法: `ngx.redirect(uri, status?)`

* status 可以是301, 302(默认), 307

示例:

```lua
return ngx.redirect("/foo")
return ngx.redirect("/foo", ngx.HTTP_MOVED_TEMPORARILY)
return ngx.redirect("http://www.google.com")
return ngx.redirect("/foo", 301)
return ngx.redirect('/foo?a=3&b=4')
```




# 原理

## 请求的处理流程

![](pic/openresty_phases.png)

示例代码:

```
location /mixed {
    set_by_lua_block $a {
        ngx.log(ngx.ERR, "set_by_lua*")
    }
    rewrite_by_lua_block {
        ngx.log(ngx.ERR, "rewrite_by_lua*")
    }
    access_by_lua_block {
        ngx.log(ngx.ERR, "access_by_lua*")
    }
    content_by_lua_block {
        ngx.log(ngx.ERR, "content_by_lua*")
    }
    header_filter_by_lua_block {
        ngx.log(ngx.ERR, "header_filter_by_lua*")
    }
    body_filter_by_lua_block {
        ngx.log(ngx.ERR, "body_filter_by_lua*")
    }
    log_by_lua_block {
        ngx.log(ngx.ERR, "log_by_lua*")
    }
}
```

* `set_by_lua*`: 流程分支处理判断变量初始化
* `rewrite_by_lua*`: 转发、重定向、缓存等功能(例如特定请求代理到外网)
* `access_by_lua*`: IP 准入、接口权限等情况集中处理(例如配合 iptable 完成简单防火墙)
* `content_by_lua*`: 内容生成
* `header_filter_by_lua*`: 响应头部过滤处理(例如添加头部信息)
* `body_filter_by_lua*`: 响应体过滤处理(例如完成应答内容统一成大写)
* `log_by_lua*`: 会话完成后本地异步完成日志记录(日志可以记录在本地，还可以同步到其他机器)


- log 阶段不能调用 redis.



# 日志

## 将日志打到远程服务

- [lua-resty-logger-socket](https://github.com/cloudflare/lua-resty-logger-socket)



# 性能

## 性能测试


### 注意点

以下摘自 https://groups.google.com/forum/#!topic/openresty/gXxDThKEGss:
1. 当前 shell 的 ulimit -n 和 nginx 的 worker_connections 配置都应调整得足够大。
2. 同时压测时应启用 ab 的 -k 选项以及 lua-resty-redis 的连接池，否则很容易把系统的临时端口用尽。如果你坚持不使用
ab 的 -k 选项以启用 http 1.0 keepalive，则最好配置  lua_http10_buffering off 指令以提高
http 1.0 短连接访问下的性能。
3. 你的系统检测出多少个 CPU 核，就启用多少个 nginx worker 进程，同时恰当配置 Nginx 的
worker_cpu_affinity 指令设置 CPU 亲缘。建议禁用 access_log 或者至少设置较大的 access log
buffer，以降低写日志开销。
4. openresty 一侧应启用 LuaJIT 2.0.
5. 大并发测试时应调大 nginx 一侧和 redis 一侧的 backlog 设置。
6. 仔细 ab 客户端或者 redis 服务器本身成为性能瓶颈 ;)



# 正则

## ngx.re

### ngx.re.match

语法：`captures, err = ngx.re.match(subject, regex, options?, ctx?, res_table?)`

options:
```
a             锚定模式，只从头开始匹配.
d             DFA模式，或者称最长字符串匹配语义，需要PCRE 6.0+支持.
D             允许重复的命名的子模式，该选项需要PCRE 8.12+支持，例如
                local m = ngx.re.match("hello, world",
                                       "(?<named>\w+), (?<named>\w+)",
                                       "D")
                -- m["named"] == {"hello", "world"}
i             大小写不敏感模式.
j             启用PCRE JIT编译, 需要PCRE 8.21+ 支持，并且必须在编译时加上选项--enable-jit，
                为了达到最佳性能，该选项总是应该和'o'选项搭配使用.		  
J             启用PCRE Javascript的兼容模式，需要PCRE 8.12+ 支持.
m             多行模式.
o             一次编译模式，启用worker-process级别的编译正则表达式的缓存.
s             单行模式.
u             UTF-8模式. 该选项需要在编译PCRE库时加上--enable-utf8 选项.
U             与"u" 选项类似，但是该项选禁止PCRE对subject字符串UTF-8有效性的检查.
x             扩展模式
```

只有第一次匹配的结果被返回，如果没有匹配，则返回nil；或者匹配过程中出现错误时，也会返回nil，此时错误信息会被保存在err中。

当匹配的字符串找到时，一个Lua table captures会被返回，captures[0]中保存的就是匹配到的字串，captures[1]保存的是用括号括起来的第一个子模式的结果，captures[2]保存的是第二个子模式的结果，依次类似。

```lua
-- 1
local m, err = ngx.re.match("hello, 1234", "[0-9]+")
if m then
    -- m[0] == "1234"
else
    if err then
        ngx.log(ngx.ERR, "error: ", err)
        return
    end
    ngx.say("match not found")
end

-- 2
local m, err = ngx.re.match("hello, 1234", "([0-9])[0-9]+")
-- m[0] == "1234"
-- m[1] == "1"

-- 命名
local m, err = ngx.re.match("hello, 1234", "([0-9])(?<remaining>[0-9]+)")
-- m[0] == "1234"
-- m[1] == "1"
-- m[2] == "234"
-- m["remaining"] == "234"
local m, err = ngx.re.match("hello, world", "(world)|(hello)|(?<named>howdy)")
-- m[0] == "hello"
-- m[1] == nil
-- m[2] == "hello"
-- m[3] == nil
-- m["named"] == nil

-- options
local m, err = ngx.re.match("hello, world", "HEL LO", "ix")
-- m[0] == "hello"

local m, err = ngx.re.match("hello, 美好生活", "HELLO, (.{2})", "iu")
-- m[0] == "hello, 美好"
-- m[1] == "美好"
```

第四个可选参数ctx可以传入一个Lua Table，传入的Lua Table可以是一个空表，也可以是包含pos字段的Lua Table。如果传入的是一个空的Lua Table，那么，ngx.re.match将会从subject字符串的起始位置开始匹配查找，查找到匹配串后，修改pos的值为匹配字符串的**下一个位置的值**，并将pos的值保存到ctx中，如果匹配失败，那么pos的值保持不变；如果传入的是一个非空的Lua Table，即指定了pos的初值，那么ngx.re.match将会从指定的pos的位置开始进行匹配，如果匹配成功了，修改pos的值为匹配字符串的下一个位置的值，并将pos的值保存到ctx中，如果匹配失败，那么pos的值保持不变。

```lua
local ctx = {}
local m, err = ngx.re.match("1234, hello", "[0-9]+", "", ctx)
-- m[0] = "1234"
-- ctx.pos == 5

local ctx = { pos = 2 }
local m, err = ngx.re.match("1234, hello", "[0-9]+", "", ctx)
-- m[0] = "34"
-- ctx.pos == 5
```

注意：如果需要传入ctx参数，但并不需要第三个可选参数options时，第三个参数也不能简单去掉，这时需要传入一个空的字符串作为第三个参数的值。


### ngx.re.find

语法：`from, to, err = ngx.re.find(subject, regex, options?, ctx?, nth?)`

该方法与ngx.re.match方法基本类似，不同的地方在于ngx.re.find返回的是匹配的字串的起始位置索引和结束位置索引，如果没有匹配成功，那么将会返回两个nil，如果匹配出错，还会返回错误信息到err中。

```lua
local s = "hello, 1234"
local from, to, err = ngx.re.find(s, "([0-9]+)", "jo")
if from then
    ngx.say("from: ", from)
    ngx.say("to: ", to)
    ngx.say("matched: ", string.sub(s, from, to))
else
    if err then
        ngx.say("error: ", err)
        return
    end
    ngx.say("not matched!")
end
```

该方法相比ngx.re.match，不会创建新的Lua字符串，也不会创建新的Lua Table，因此，该方法比ngx.re.match更加高效，因此，在可以使用ngx.re.find的地方应该尽量使用。

第五个参数可以指定返回第几个子模式串的起始位置和结束位置的索引值，默认值是0，此时将会返回匹配的整个字串；如果nth等于1，那么将返回第一个子模式串的始末位置的索引值；如果nth等于2，那么将返回第二个子模式串的始末位置的索引值，依次类推。如果nth指定的子模式没有匹配成功，那么将会返回两个nil。

```lua
local str = "hello, 1234"
local from, to = ngx.re.find(str, "([0-9])([0-9]+)", "jo", nil, 2)
if from then
    ngx.say("matched 2nd submatch: ", string.sub(str, from, to))  -- yields "234"
end
```


### ngx.re.gmatch

语法：`iterator, err = ngx.re.gmatch(subject, regex, options?)`

与ngx.re.match相似，区别在于该方法返回的是一个Lua的迭代器，这样就可以通过迭代器遍历所有匹配的结果。如果匹配失败，将会返回nil，如果匹配出现错误，那么还会返回错误信息到err中。

```lua
local it, err = ngx.re.gmatch("hello, world!", "([a-z]+)", "i")
if not it then
    ngx.log(ngx.ERR, "error: ", err)
    return
end

while true do
    local m, err = it()
    if err then
        ngx.log(ngx.ERR, "error: ", err)
        return
    end

    if not m then
        -- no match found (any more)
        break
    end

    -- found a match
    ngx.say(m[0])
    ngx.say(m[1])
end
```

ngx.re.gmatch返回的迭代器只能在一个请求所在的环境中使用，就是说，我们不能把返回的迭代器赋值给持久存在的命名空间（比如一个Lua Packet）中的某一个变量。



### ngx.re.sub

语法：`newstr, n, err = ngx.re.sub(subject, regex, replace, options?)`

该方法主要实现匹配字符串的替换，会用replace替换匹配的字串，replace可以是纯字符串，也可以是使用$0, $1等子模式串的形式，ngx.re.sub返回进行替换后的完整的字符串，同时返回替换的总个数；options选项，与ngx.re.match中的options选项是一样的。

```lua
local newstr, n, err = ngx.re.sub("hello, 1234", "([0-9])[0-9]", "[$0][$1]")
if newstr then
    -- newstr == "hello, [12][1]34"
    -- n == 1
else
    ngx.log(ngx.ERR, "error: ", err)
    return
end
```

在上面例子中，$0表示整个匹配的子串，$1表示第一个子模式匹配的字串，以此类推。

可以用大括号{}将相应的0，1，2...括起来，以区分一般的数字：

```lua
local newstr, n, err = ngx.re.sub("hello, 1234", "[0-9]", "${0}00")
-- newstr == "hello, 100234"
-- n == 1
```

如果想在replace字符串中显示$符号，可以用$进行转义（不要用反斜杠`\$`对美元符号进行转义，这种方法不会得到期望的结果）：

```lua
local newstr, n, err = ngx.re.sub("hello, 1234", "[0-9]", "$$")
-- newstr == "hello, $234"
-- n == 1
```

如果replace是一个函数，那么函数的参数是一个"match table"， 而这个"match table"与ngx.re.match中的返回值captures是一样的，replace这个函数根据"match table"产生用于替换的字符串。

```lua
local func = function (m)
    return "[" .. m[0] .. "][" .. m[1] .. "]"
end
local newstr, n, err = ngx.re.sub("hello, 1234", "( [0-9] ) [0-9]", func, "x")
-- newstr == "hello, [12][1]34"
-- n == 1
```

注意：通过函数形式返回的替换字符串中的美元符号$不再是特殊字符，而只是被看作一个普通字符。

### ngx.re.gsub

语法：`newstr, n, err = ngx.re.gsub(subject, regex, replace, options?)`

该方法与ngx.re.sub是类似的，但是该方法进行的是全局替换。

```lua
local newstr, n, err = ngx.re.gsub("hello, world", "([a-z])[a-z]+", "[$0,$1]", "i")
if newstr then
    -- newstr == "[hello,h], [world,w]"
    -- n == 2
else
    ngx.log(ngx.ERR, "error: ", err)
    return
end


local func = function (m)
    return "[" .. m[0] .. "," .. m[1] .. "]"
end
local newstr, n, err = ngx.re.gsub("hello, world", "([a-z])[a-z]+", func, "i")
-- newstr == "[hello,h], [world,w]"
-- n == 2
```





# 参考
* [官网](http://openresty.org/cn/)
* [openresty最佳实践](https://moonbingbing.gitbooks.io/openresty-best-practices/content/)
* [ngx_lua模块中的共享内存字典项API](http://blog.csdn.net/weiyuefei/article/details/38487475)
* [lua nginx module](https://github.com/openresty/lua-nginx-module)
* [http client库](https://github.com/pintsized/lua-resty-http)
* [cjson](https://github.com/openresty/lua-cjson/)
* [ngx_lua模块中正则表达式相关的api](http://blog.csdn.net/weiyuefei/article/details/38439017)
* [正则表达式](https://moonbingbing.gitbooks.io/openresty-best-practices/lua/re.html)
* [加密编码相关](https://github.com/openresty/lua-resty-string)
* [lua-resty-core](https://github.com/openresty/lua-resty-core)
* [ngx_lua模块中正则表达式相关的api](https://blog.csdn.net/weiyuefei/article/details/38439017)
