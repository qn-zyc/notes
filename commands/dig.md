<!-- TOC -->

- [1. 概述](#1-概述)
- [2. 语法](#2-语法)
    - [2.1. 查询选项](#21-查询选项)
- [3. 常用格式](#3-常用格式)
- [4. 从文件中读取域名](#4-从文件中读取域名)
- [5. 查询IP到域名的映射](#5-查询ip到域名的映射)
- [6. 追踪dig全过程](#6-追踪dig全过程)
- [7. 参考](#7-参考)

<!-- /TOC -->

# 1. 概述

* 查询DNS域名服务器的工具。

# 2. 语法

```bash
dig [@server] [-b address] [-c class] [-f filename] [-k filename] [ -n ][-p port#] [-t type] [-x addr] [-y name:key] [name] [type] [class] [queryopt...]
```

* `-b address`: 设置所要询问地址的源 IP 地址。这必须是主机网络接口上的某一合法的地址。
* `-c class`: 缺省查询类（IN for internet）由选项 -c 重设。class 可以是任何合法类，比如查询 Hesiod 记录的 HS 类或查询 CHAOSNET 记录的 CH 类
* `-f filename`: 使 dig 在批处理模式下运行，通过从文件 filename 读取一系列搜索请求加以处理。文件包含许多查询；每行一个。文件中的每一项都应该以和使用命令行接口对 dig 的查询相同的方法来组织。 
* `-h`: 当使用选项 -h 时，显示一个简短的命令行参数和选项摘要。 
* `-k filename`: 要签署由 dig 发送的 DNS 查询以及对它们使用事务签名（TSIG）的响应，用选项 -k 指定 TSIG 密钥文件。
* `-n`: 缺省情况下，使用 IP6.ARPA 域和 RFC2874 定义的二进制标号搜索 IPv6 地址。为了使用更早的、使用 IP6.INT 域和 nibble 标签的 RFC1886 方法，指定选项 -n（nibble）。 
* `-p port`: 如果需要查询一个非标准的端口号，则使用选项 -p。 port 是 dig 将发送其查询的端口号，而不是标准的 DNS 端口号 53。该选项可用于测试已在非标准端口号上配置成侦听查询的域名服务器。 
`-t type`: 设置查询类型为 type。可以是 BIND9 支持的任意有效查询类型。缺省查询类型是 A，除非提供 -x 选项来指示一个逆向查询。通过指定 AXFR 的 type 可以请求一个区域传输。当需要增量区域传输（IXFR）时，type 设置为 ixfr=N。增量区域传输将包含自从区域的 SOA 记录中的序列号改为 N 之后对区域所做的更改。 
`-x addr`: 逆向查询（将地址映射到名称）可以通过 -x 选项加以简化。addr 是一个以小数点为界的 IPv4 地址或冒号为界的 IPv6 地址。当使用这个选项时，无需提供 name、class 和 type 参数。dig 自动运行类似 11.12.13.10.in-addr.arpa 的域名查询，并分别设置查询类型和类为 PTR 和 IN。 
`-y name:key`: 您可以通过命令行上的 -y 选项指定 TSIG 密钥；name 是 TSIG 密码的名称，key 是实际的密码。密码是 64 位加密字符串，通常由 dnssec-keygen（8）生成。当在多用户系统上使用选项 -y 时应该谨慎，因为密码在 ps（1）的输出或 shell 的历史文件中可能是可见的。当同时使用 dig 和 TSCG 认证时，被查询的名称服务器需要知道密码和解码规则。在 BIND 中，通过提供正确的密码和 named.conf 中的服务器声明实现。 


## 2.1. 查询选项
dig 提供查询选项号，它影响搜索方式和结果显示。一些在查询请求报头设置或复位标志位，一部分决定显示哪些回复信息，其他的确定超时和重试战略。每个查询选项被带前缀（+）的关键字标识。一些关键字设置或复位一个选项。通常前缀是求反关键字含义的字符串 no。其他关键字分配各选项的值，比如超时时间间隔。它们的格式形如 +keyword=value。查询选项是：

- `+[no]tcp`: 查询域名服务器时使用 [不使用] TCP。缺省行为是使用 UDP，除非是 AXFR 或 IXFR 请求，才使用 TCP 连接。 
- `+[no]vc`: 查询名称服务器时使用 [不使用] TCP。+[no]tcp 的备用语法提供了向下兼容。vc 代表虚电路。 
- `+[no]ignore`: 忽略 UDP 响应的中断，而不是用 TCP 重试。缺省情况运行 TCP 重试。 
- `+domain=somename`: 设定包含单个域 somename 的搜索列表，好像被 /etc/resolv.conf 中的域伪指令指定，并且启用搜索列表处理，好像给定了 +search 选项。 
- `+[no]search`: 使用 [不使用] 搜索列表或 resolv.conf 中的域伪指令（如果有的话）定义的搜索列表。缺省情况不使用搜索列表。 
- `+[no]defname`: 不建议看作 +[no]search 的同义词。 
- `+[no]aaonly`: 该选项不做任何事。它用来提供对设置成未实现解析器标志的 dig 的旧版本的兼容性。 
- `+[no]adflag`: 在查询中设置 [不设置] AD（真实数据）位。目前 AD 位只在响应中有标准含义，而查询中没有，但是出于完整性考虑在查询中这种性能可以设置。 
- `+[no]cdflag`: 在查询中设置 [不设置] CD（检查禁用）位。它请求服务器不运行响应信息的 DNSSEC 合法性。 
- `+[no]recursive`: 切换查询中的 RD（要求递归）位设置。在缺省情况下设置该位，也就是说 dig 正常情形下发送递归查询。当使用查询选项 +nssearch 或 +trace 时，递归自动禁用。 
- `+[no]nssearch`: 这个选项被设置时，dig 试图寻找包含待搜名称的网段的权威域名服务器，并显示网段中每台域名服务器的 SOA 记录。 
- `+[no]trace`: 切换为待查询名称从根名称服务器开始的代理路径跟踪。缺省情况不使用跟踪。一旦启用跟踪，dig 使用迭代查询解析待查询名称。它将按照从根服务器的参照，显示来自每台使用解析查询的服务器的应答。 
- `+[no]cmd`: 设定在输出中显示指出 dig 版本及其所用的查询选项的初始注释。缺省情况下显示注释。 
- `+[no]short`: 提供简要答复。缺省值是以冗长格式显示答复信息。 
- `+[no]identify`: 当启用 +short 选项时，显示 [或不显示] 提供应答的 IP 地址和端口号。如果请求简短格式应答，缺省情况不显示提供应答的服务器的源地址和端口号。 
- `+[no]comments`: 切换输出中的注释行显示。缺省值是显示注释。 
- `+[no]stats`: 该查询选项设定显示统计信息：查询进行时，应答的大小等等。缺省显示查询统计信息。 
- `+[no]qr`: 显示 [不显示] 发送的查询请求。缺省不显示。 
- `+[no]question`: 当返回应答时，显示 [不显示] 查询请求的问题部分。缺省作为注释显示问题部分。 
- `+[no]answer`: 显示 [不显示] 应答的回答部分。缺省显示。 
- `+[no]authority`: 显示 [不显示] 应答的权限部分。缺省显示。 
- `+[no]additional`: 显示 [不显示] 应答的附加部分。缺省显示。 
- `+[no]all`: 设置或清除所有显示标志。 
- `+time=T`: 为查询设置超时时间为 T 秒。缺省是 5 秒。如果将 T 设置为小于 1 的数，则以 1 秒作为查询超时时间。 
- `+tries=A`: 设置向服务器发送 UDP 查询请求的重试次数为 A，代替缺省的 3 次。如果把 A 小于或等于 0，则采用 1 为重试次数。 
- `+ndots=D`: 出于完全考虑，设置必须出现在名称 D 的点数。缺省值是使用在 /etc/resolv.conf 中的 ndots 语句定义的，或者是 1，如果没有 ndots 语句的话。带更少点数的名称被解释为相对名称，并通过搜索列表中的域或文件 /etc/resolv.conf 中的域伪指令进行搜索。 
- `+bufsize=B`: 设置使用 EDNS0 的 UDP 消息缓冲区大小为 B 字节。缓冲区的最大值和最小值分别为 65535 和 0。超出这个范围的值自动舍入到最近的有效值。 
- `+[no]multiline`: 以详细的多行格式显示类似 SOA 的记录，并附带可读注释。缺省值是每单个行上显示一条记录，以便于计算机解析 dig 的输出。 





# 3. 常用格式

```bash
dig @dnsserver name querytype
# 示例
dig @8.8.8.8 www.baidu.com A
```

* 如果 dnsserver 是一个域名，dig 先通过默认的上连DNS服务器查询该域名的ip，然后以这个 dnsserver 为上连DNS服务器来查询name.
* 如果没有 dnsserver，dig 会依次使用 `/etc/resolv.conf` 里的地址作为上连DNS服务器。
* name 就是需要解析的域名。


# 4. 从文件中读取域名

```bash
dig -f querylist.txt -c IN -t A
```

* 文件中一个域名一行。


# 5. 查询IP到域名的映射

```bash
dig -x 127.0.0.1
```

# 6. 追踪dig全过程

```bash
dig +trace www.baidu.com
```



# 7. 参考
* [《dig挖出DNS的秘密》-linux命令五分钟系列之三十四](http://roclinux.cn/?p=2449)
* [dig命令详解](http://blog.csdn.net/adparking/article/details/5819725)