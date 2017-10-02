<!-- TOC -->

- [机制](#机制)
- [启动关闭](#启动关闭)
- [变量](#变量)
- [配置](#配置)
    - [模板](#模板)
    - [location](#location)
    - [日志](#日志)
    - [server_name](#server_name)
    - [client_max_body_size](#client_max_body_size)
- [模块](#模块)
    - [upstream](#upstream)
        - [轮询(默认)](#轮询默认)
        - [weight权重](#weight权重)
        - [ip_hash](#ip_hash)
        - [fair](#fair)
        - [url_hash](#url_hash)
        - [least_conn最少连接](#least_conn最少连接)

<!-- /TOC -->

# 机制
* 一个Master, 多个Worker.
* 每个 Worker 只有一个主线程, 异步非阻塞方式. 没有上下文切换, 阻塞调用时就去处理其他事情.
* worker 个数推荐为 cpu 核数. cpu亲缘性, 将一个进程绑定在一个核上, 避免 cache 失效.


# 启动关闭

启动:

```
/usr/local/openresty/nginx/sbin/nginx -c /usr/local/openresty/nginx/conf/nginx.conf
/usr/local/openresty/nginx/sbin/nginx -p 'pwd' -c /usr/local/openresty/nginx/conf/nginx.conf
```

* `-c` 指定配置文件.
* `-p` 指定工作目录

* 从容停止Nginx: `kill -QUIT master进程号`
* 快速停止Nginx: `kill -TERM master进程号`
* 强制停止Nginx: `pkill -9 nginx`
* 平滑重启: `kill -HUP master进程号或进程号文件路径` 或者: `/usr/nginx/sbin/nginx -s reload`
* 判断配置文件是否正确: `nginx -t -c /usr/nginx/conf/nginx.conf` 或者 `/usr/nginx/sbin/nginx -t`



# 变量
* 变量只有一种类型: 字符串.
* Nginx 变量的创建只能发生在 Nginx 配置加载的时候，或者说 Nginx 启动的时候；而赋值操作则只会发生在请求实际处理的时候。这意味着不创建而直接使用变量会导致启动失败，同时也意味着我们无法在请求处理时动态地创建新的 Nginx 变量.
* 变量的作用范围是整个配置, 甚至可以跨越不同虚拟主机的 server 配置块. 但是每个请求都会有自己的变量副本, 互相不干扰.

```nginx
set $a "hello world";
set $a hello;
set $b "$a, $a"; // 拼接
echo "${first}world"; // 避免歧义
```

# 配置

## 模板

```
# 使用的用户和组 
user  www www;

# 指定工作进程数  
worker_processes  1;

# 可以使用 [ debug | info | notice | warn | error | crit ]  参数  
#error_log  logs/error.log;  
error_log  logs/error.log  notice;

# 指定 pid 存放的路径  
#pid logs/nginx.pid; 

#-----------------------------------事件模块   
events {  
    #每个worker的最大连接数  
    worker_connections  1024;  
}

#-----------------------------------HTTP 模块   
  
http {  
    #包含一个文件描述了：不同文件后缀对应的MIME，见案例分析  
    include       mime.types;  

    #制定默认MIME类型为二进制字节流  
    default_type  application/octet-stream;  
  
    #指令 access_log 指派路径、格式和缓存大小。  
    #access_log  off;  
  
    #开启调用Linux的sendfile()，提供文件传输效率  
    sendfile        on;  
  
    #是否允许使用socket的TCP_NOPUSH或TCP_CORK选项  
    #tcp_nopush     on;  
  
    #指定客户端连接保持活动的超时时间，在这个时间之后，服务器会关掉连接。  
    keepalive_timeout  65;  
  
    #设置gzip，压缩文件  
    #gzip  on;  
  
    #为后端服务器提供简单的负载均衡  
    upstream apaches {  
        server 127.0.0.1:8001;  
        server 127.0.0.1:8002;  
    }

    # 使用lua初始化
    init_by_lua_file '/usr/local/openresty/nginx/lua/init_phase/init.lua';
    init_worker_by_lua_file '/usr/local/openresty/nginx/lua/init_worker_phase/init_worker_by_lua.lua';

    # 包含其他配置文件
    include /usr/local/openresty/nginx/conf/other.conf;

    # 共享字典
    lua_shared_dict dict_name 10m;
  
    #配置一台虚拟机  
    server {  
        listen       8012;  
        server_name  localhost;  

        client_max_body_size 10m;

        location / {  
            proxy_pass http://apaches;  
        }  
    }  
}  
```


## location

匹配规则: `location [=|~|~*|^~] /uri/ { … }`

|        模式         |                                    含义                                     |
| ------------------- | --------------------------------------------------------------------------- |
| location = /uri     | = 表示精确匹配，只有完全匹配上才能生效                                      |
| location ^~ /uri    | ^~ 开头对URL路径进行前缀匹配，并且在正则之前。                              |
| location ~ pattern  | 开头表示区分大小写的正则匹配                                                |
| location ~* pattern | 开头表示不区分大小写的正则匹配                                              |
| location /uri       | 不带任何修饰符，也表示前缀匹配，但是在正则匹配之后                          |
| location /          | 通用匹配，任何未匹配到其它location的请求都会匹配到，相当于switch中的default |

* 前缀匹配时，Nginx 不对 url 做编码，因此请求为 `/static/20%/aa`，可以被规则 `^~ /static/ /aa` 匹配到（注意是空格）


多个 location 配置的情况下匹配顺序为:
1. 首先精确匹配 `=`
2. 其次前缀匹配 `^~`
3. 其次是按文件中顺序的正则匹配
4. 然后匹配不带任何修饰的前缀匹配。
5. 最后是交给 `/` 通用匹配
6. 当有匹配成功时候，停止匹配，按当前匹配规则处理请求

注意：前缀匹配，如果有包含关系时，按最大匹配原则进行匹配。比如在前缀匹配：`location /dir01` 与 `location /dir01/dir02`，如有请求 `http://localhost/dir01/dir02/file` 将最终匹配到 `location /dir01/dir02`

示例:
* 文件后缀: `location ~ \.(gif|jpg|png|js|css)$`
* 基本配置:
    ```
    # 直接匹配网站根，通过域名访问网站首页比较频繁，使用这个会加速处理，官网如是说。
    # 这里是直接转发给后端应用服务器了，也可以是一个静态首页
    # 第一个必选规则
    location = / {
        proxy_pass http://tomcat:8080/index
    }

    # 第二个必选规则是处理静态文件请求，这是 nginx 作为 http 服务器的强项
    # 有两种配置模式，目录匹配或后缀匹配，任选其一或搭配使用
    location ^~ /static/ {
        root /webroot/static/;
    }
    location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
        root /webroot/res/;
    }

    # 第三个规则就是通用规则，用来转发动态请求到后端应用服务器
    # 非静态文件请求就默认是动态请求，自己根据实际把握
    # 毕竟目前的一些框架的流行，带.php、.jsp后缀的情况很少了
    location / {
        proxy_pass http://tomcat:8080/
    }
    ```


## 日志

语法: `error_log /path/file level;`

* `/path/file`
    1. 参数可以是一个具体的文件，例如，默认情况下是logs/error.log文件，最好将它放到一个磁盘空间足够大的位置；
    2. /path/file也可以是/dev/null，这样就不会输出任何日志了，这也是关闭error日志的唯一手段；
    3. /path/file也可以是stderr，这样日志会输出到标准错误文件中。
* `level`: debug, info, notice, warn, error, crit, alert, emerg

```
access_log  logs/chentest.access.log;
error_log   logs/chentest.error.log info;
```


## server

指定一个默认的server, 只能指定一个：

```nginx
server {
    listen 80 default;
}
```


### server_name

匹配顺序:

```
# 精确
server {
    listen       80;
    server_name  example.org  www.example.org;
    ...
}

# 星号在前
server {
    listen       80;
    server_name  *.example.org;
    ...
}

# 星号在后
server {
    listen       80;
    server_name  mail.*;
    ...
}

# 正则
server {
    listen       80;
    server_name  ~^(?<user>.+)\.example\.net$;
    ...
}
```

* `server_name _;` 表示默认匹配, 其他匹配不到就使用这个server.
* `server_name ~.*;` 表示匹配所有域名.


## client_max_body_size
* [doc](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size)

* Syntax:  `client_max_body_size size`;
* Default: `client_max_body_size 1m`;
* Context: http, server, location
* 检查 `Content-Length` header, 超出限制返回 413.
* 设置为0不检查大小了.



# 模块

* **event module**: 搭建了独立于操作系统的事件处理机制的框架，及提供了各具体事件的处理。包括 `ngx_events_module`，`ngx_event_core_module` 和`ngx_epoll_module`等。nginx具体使用何种事件处理模块，这依赖于具体的操作系统和编译选项。
* **phase handler**: 此类型的模块也被直接称为handler模块。主要负责处理客户端请求并产生待响应内容，比如 `ngx_http_static_module` 模块，负责客户端的静态页面请求处理并将对应的磁盘文件准备为响应内容输出。
* **output filter**: 也称为filter模块，主要是负责对输出的内容进行处理，可以对输出进行修改。例如，可以实现对输出的所有html页面增加预定义的footbar一类的工作，或者对输出的图片的URL进行替换之类的工作。
* **upstream**: upstream模块实现反向代理的功能，将真正的请求转发到后端服务器上，并从后端服务器上读取响应，发回客户端。upstream模块是一种特殊的handler，只不过响应内容不是真正由自己产生的，而是从后端服务器上读取的。
* **load-balancer**: 负载均衡模块，实现特定的算法，在众多的后端服务器中，选择一个服务器出来作为某个请求的转发服务器。

## upstream

* upstream 定义在 server 之外.

### 轮询(默认)

配置(默认是轮询):

```
upstream linuxidc { 
    server 10.0.6.108:7080; 
    server 10.0.0.85:8980; 
}
```

在 proxy_pass 中使用:

```
location / { 
    root  html; 
    index  index.html index.htm; 
    proxy_pass http://linuxidc; 
}
```

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端某台服务器宕机，故障系统被自动剔除，使用户访问不受影响。Weight 指定轮询权值，Weight 值越大，分配到的访问机率越高，主要用于后端每个服务器性能不均的情况下.


### weight权重

```
upstream linuxidc{ 
      server 10.0.0.77 weight=5; 
      server 10.0.0.88 weight=10; 
}
```

### ip_hash

```
upstream favresin{ 
      ip_hash; 
      server 10.0.0.10:8080; 
      server 10.0.0.11:8080; 
}
```

* 每个请求按访问 IP 的 hash 结果分配，这样来自同一个 IP 的访客固定访问一个后端服务器，有效解决了动态网页存在的 session 共享问题.
* 当负载调度算法为 ip_hash 时，后端服务器在负载均衡调度中的状态不能是 backup。

### fair

此种算法可以依据页面大小和加载时间长短智能地进行负载均衡，也就是根据后端服务器的响应时间来分配请求，响应时间短的优先分配。Nginx 本身是不支持 fair 的，如果需要使用这种调度算法，必须下载 Nginx 的 upstream_fair 模块.

```
upstream favresin{      
      server 10.0.0.10:8080; 
      server 10.0.0.11:8080; 
      fair; 
}
```

### url_hash

此方法按访问 url 的 hash 结果来分配请求，使每个 url 定向到同一个后端服务器，可以进一步提高后端缓存服务器的效率。Nginx 本身是不支持 url_hash 的，如果需要使用这种调度算法，必须安装 Nginx 的 hash 软件包。

```
upstream resinserver{ 
      server 10.0.0.10:7777; 
      server 10.0.0.11:8888; 
      hash $request_uri; 
      hash_method crc32; 
}
```

* server 中不能写 weight 等其他参数.
* hash_method 指定使用的 hash 函数.


### least_conn最少连接

最少连接负载均衡算法，简单来说就是每次选择的后端都是当前最少连接的一个 server(这个最少连接不是共享的，是每个 worker 都有自己的一个数组进行记录后端 server 的连接数)。

```
upstream myapp1 {
    least_conn;
    server srv1.example.com;
    server srv2.example.com;
    server srv3.example.com;
}
 ```


### 为每台设备设置状态值

* down 表示单前的server暂时不参与负载.
* weight 默认为1.weight越大，负载的权重就越大。
* `max_fails` ：允许请求失败的次数默认为1.当超过最大次数时，返回 `proxy_next_upstream` 模块定义的错误.
* `fail_timeout` : max_fails次失败后，暂停的时间。
* backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

```
upstream bakend{ #定义负载均衡设备的Ip及设备状态 
      ip_hash; 
      server 10.0.0.11:9090 down; 
      server 10.0.0.11:8080 weight=2; 
      server 10.0.0.11:6060; 
      server 10.0.0.11:7070 backup; 
}
```





# 参考
* [nginx教程](https://openresty.org/download/agentzh-nginx-tutorials-zhcn.html)
* [Nginx开发从入门到精通](http://tengine.taobao.org/book/index.html)
* [Nginx配置upstream实现负载均衡](http://www.linuxidc.com/Linux/2015-03/115207.htm)