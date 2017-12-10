
<!-- TOC -->

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
- [shared_dict](#shared_dict)
    - [获取字典对象](#获取字典对象)
    - [ngx.shared.DICT.set](#ngxshareddictset)
    - [ngx.shared.DICT.safe_set](#ngxshareddictsafe_set)
    - [ngx.shared.DICT.add](#ngxshareddictadd)
    - [ngx.shared.DICT.safe_add](#ngxshareddictsafe_add)
    - [ngx.shared.DICT.get](#ngxshareddictget)
    - [ngx.shared.DICT.get_stale](#ngxshareddictget_stale)
    - [ngx.shared.DICT.replace](#ngxshareddictreplace)
    - [ngx.shared.DICT.delete](#ngxshareddictdelete)
    - [ngx.shared.DICT.incr](#ngxshareddictincr)
    - [ngx.shared.DICT.flush_all](#ngxshareddictflush_all)
    - [ngx.shared.DICT.flush_expired](#ngxshareddictflush_expired)
    - [ngx.shared.DICT.get_keys](#ngxshareddictget_keys)
- [json](#json)
- [负载均衡](#负载均衡)
    - [动态负载均衡](#动态负载均衡)
    - [获取使用的 upstream server 的信息](#获取使用的-upstream-server-的信息)
- [重定向](#重定向)
    - [ngx.redirect](#ngxredirect)
- [原理](#原理)
    - [请求的处理流程](#请求的处理流程)
- [日志](#日志)
    - [将日志打到远程服务](#将日志打到远程服务)
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
```


## 时间
* ngx.today "yyyy-mm-dd" 格式 (2017-08-23)
* ngx.time 时间戳, 秒 (1503487721)
* ngx.now 带毫秒的时间戳 (1503487721.425)
* ngx.update_time 使用系统时间强制更新nginx缓存时间, 不返回数据.
* ngx.localtime "yyyy-mm-dd hh:mm:ss" 格式 (2017-08-23 11:28:41)
* ngx.utctime "yyyy-mm-dd hh:mm:ss" 格式 (2017-08-23 11:28:41)



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
* [](https://moonbingbing.gitbooks.io/openresty-best-practices/openresty/work_with_location.html)

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





# 参考
* [官网](http://openresty.org/cn/)
* [openresty最佳实践](https://moonbingbing.gitbooks.io/openresty-best-practices/content/)
* [ngx_lua模块中的共享内存字典项API](http://blog.csdn.net/weiyuefei/article/details/38487475)
* [lua nginx module](https://github.com/openresty/lua-nginx-module)
* [http client库](https://github.com/pintsized/lua-resty-http)
* [cjson](https://github.com/openresty/lua-cjson/)