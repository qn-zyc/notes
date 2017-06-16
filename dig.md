<!-- TOC -->

- [1. 常用格式](#1-常用格式)
- [2. 从文件中读取域名](#2-从文件中读取域名)
- [3. 查询IP到域名的映射](#3-查询ip到域名的映射)
- [4. 追踪dig全过程](#4-追踪dig全过程)
- [5. 参考](#5-参考)

<!-- /TOC -->

# 1. 常用格式

```bash
dig @dnsserver name querytype
# 示例
dig @8.8.8.8 www.baidu.com A
```

* 如果 dnsserver 是一个域名，dig 先通过默认的上连DNS服务器查询该域名的ip，然后以这个 dnsserver 为上连DNS服务器来查询name.
* 如果没有 dnsserver，dig 会依次使用 `/etc/resolv.conf` 里的地址作为上连DNS服务器。
* name 就是需要解析的域名。


# 2. 从文件中读取域名

```bash
dig -f querylist.txt -c IN -t A
```

* 文件中一个域名一行。


# 3. 查询IP到域名的映射

```bash
dig -x 127.0.0.1
```

# 4. 追踪dig全过程

```bash
dig +trace www.baidu.com
```



# 5. 参考
* [《dig挖出DNS的秘密》-linux命令五分钟系列之三十四](http://roclinux.cn/?p=2449)