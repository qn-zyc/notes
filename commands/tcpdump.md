<!-- TOC -->

- [简介](#简介)
    - [格式](#格式)
- [实例](#实例)
    - [默认启动](#默认启动)
    - [监视指定网络接口的数据包](#监视指定网络接口的数据包)
    - [监视指定主机的数据包](#监视指定主机的数据包)
    - [监视指定主机和端口的数据包](#监视指定主机和端口的数据包)
    - [监视指定网络的数据包](#监视指定网络的数据包)
    - [监视指定协议的数据包](#监视指定协议的数据包)
- [配合wireshark](#配合wireshark)
- [抓取HTTP包](#抓取http包)
- [参考](#参考)

<!-- /TOC -->

# 简介

tcpdump 根据使用者的定义对网络上的数据包进行截获的包分析工具。 tcpdump可以将网络中传送的数据包的“头”完全截获下来提供分析。它支持针对网络层、协议、主机、网络或端口的过滤，并提供and、or、not等逻辑语句来帮助你去掉无用的信息.


## 格式

```
tcpdump [ -AdDeflLnNOpqRStuUvxX ] [ -c count ]
           [ -C file_size ] [ -F file ]
           [ -i interface ] [ -m module ] [ -M secret ]
           [ -r file ] [ -s snaplen ] [ -T type ] [ -w file ]
           [ -W filecount ]
           [ -E spi@ipaddr algo:secret,...  ]
           [ -y datalinktype ] [ -Z user ]
           [ expression ]
```

* `-A`: 以ASCII码方式显示每一个数据包(不会显示数据包中链路层头部信息). 在抓取包含网页数据的数据包时, 可方便查看数据(nt: 即Handy for capturing web pages).
* `-c count`: tcpdump将在接受到count个数据包后退出.
* `-C file-size`: (nt: 此选项用于配合-w file 选项使用) 该选项使得tcpdump 在把原始数据包直接保存到文件中之前, 检查此文件大小是否超过file-size. 如果超过了, 将关闭此文件,另创一个文件继续用于原始数据包的记录. 新创建的文件名与-w 选项指定的文件名一致, 但文件名后多了一个数字.该数字会从1开始随着新创建文件的增多而增加. file-size的单位是百万字节(nt: 这里指1,000,000个字节,并非1,048,576个字节, 后者是以1024字节为1k, 1024k字节为1M计算所得, 即1M=1024 ＊ 1024 ＝ 1,048,576)
* `-d`: 以容易阅读的形式,在标准输出上打印出编排过的包匹配码, 随后tcpdump停止.(nt | rt: human readable, 容易阅读的,通常是指以ascii码来打印一些信息. compiled, 编排过的. packet-matching code, 包匹配码,含义未知, 需补充)
* `-dd`: 以C语言的形式打印出包匹配码.
* `-ddd`: 以十进制数的形式打印出包匹配码(会在包匹配码之前有一个附加的'count'前缀).
* `-F file`: 使用file 文件作为过滤条件表达式的输入, 此时命令行上的输入将被忽略.
* `-i interface`:

    指定tcpdump 需要监听的接口.  如果没有指定, tcpdump 会从系统接口列表中搜寻编号最小的已配置好的接口(不包括 loopback 接口).一但找到第一个符合条件的接口, 搜寻马上结束.

    在采用2.2版本或之后版本内核的Linux 操作系统上, 'any' 这个虚拟网络接口可被用来接收所有网络接口上的数据包(nt: 这会包括目的是该网络接口的, 也包括目的不是该网络接口的). 需要注意的是如果真实网络接口不能工作在'混杂'模式(promiscuous)下,则无法在'any'这个虚拟的网络接口上抓取其数据包.

    如果 -D 标志被指定, tcpdump会打印系统中的接口编号，而该编号就可用于此处的interface 参数.

* `-t`: 在每行输出中不打印时间戳
* `-v`: 当分析和打印的时候, 产生详细的输出. 比如, 包的生存时间, 标识, 总长度以及IP包的一些选项. 这也会打开一些附加的包完整性检测, 比如对IP或ICMP包头部的校验和.
* `-vv`: 产生比-v更详细的输出. 比如, NFS回应包中的附加域将会被打印, SMB数据包也会被完全解码.
* `-vvv`: 产生比-vv更详细的输出. 比如, telent 时所使用的SB, SE 选项将会被打印, 如果telnet同时使用的是图形界面, 其相应的图形选项将会以16进制的方式打印出来(nt: telnet 的SB,SE选项含义未知, 另需补充).
* `-w`: 把包数据直接写入文件而不进行分析和打印输出. 这些包数据可在随后通过 `-r` 选项来重新读入并进行分析和打印.




# 实例

## 默认启动

```shell
tcpdump
```

直接启动tcpdump将监视第一个网络接口上所有流过的数据包.

## 监视指定网络接口的数据包

```shell
tcpdump -i eth1
```

网络接口可以使用 `ifconfig` 查看.


## 监视指定主机的数据包

打印所有进入或离开sundown的数据包: `tcpdump host sundown`.

也可以指定ip,例如截获所有 210.27.48.1 的主机收到的和发出的所有的数据包: `tcpdump host 210.27.48.1`.

打印 helios 与 hot 或者与 ace 之间通信的数据包: `tcpdump host helios and \( hot or ace \)`.

截获主机 210.27.48.1 和主机 210.27.48.2 或 210.27.48.3 的通信: `tcpdump host 210.27.48.1 and \ (210.27.48.2 or 210.27.48.3 \)`.

打印ace与任何其他主机之间通信的IP 数据包, 但不包括与helios之间的数据包: `tcpdump ip host ace and not helios`

如果想要获取主机210.27.48.1除了和主机210.27.48.2之外所有主机通信的ip包，使用命令: `tcpdump ip host 210.27.48.1 and ! 210.27.48.2`

截获主机hostname发送的所有数据: `tcpdump -i eth0 src host hostname`

捕获目标主机发出的数据包: `tcpdump -i eth0 src 目标ip`

监视所有送到主机hostname的数据包: `tcpdump -i eth0 dst host hostname`

捕捉发送到百度的包: `tcpdump -i eth0 dst www.baidu.com`

捕获与目标ip之间的数据往来: `tcpdump ip host 目标ip`

监控本机端口: `tcpdump -i lo port 9000`



## 监视指定主机和端口的数据包

如果想要获取主机 210.27.48.1 接收或发出的telnet包，使用: `tcpdump tcp port 23 and host 210.27.48.1`

对本机的 udp 123 端口进行监视 123 为ntp的服务端口: `tcpdump udp port 123`


## 监视指定网络的数据包

打印本地主机与Berkeley网络上的主机之间的所有通信数据包(nt: ucb-ether, 此处可理解为'Berkeley网络'的网络地址,此表达式最原始的含义可表达为: 打印网络地址为ucb-ether的所有数据包): `tcpdump net ucb-ether`

打印所有通过网关snup的ftp数据包(注意, 表达式被单引号括起来了, 这可以防止shell对其中的括号进行错误解析): `tcpdump 'gateway snup and (port ftp or ftp-data)'`

打印所有源地址或目标地址是本地主机的IP数据包(如果本地网络通过网关连到了另一网络, 则另一网络并不能算作本地网络.(nt: 此句翻译曲折,需补充).localnet 实际使用时要真正替换成本地网络的名字): `tcpdump ip and not net localnet`


## 监视指定协议的数据包

打印TCP会话中的的开始和结束数据包, 并且数据包的源或目的不是本地网络上的主机.(nt: localnet, 实际使用时要真正替换成本地网络的名字))

`tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0 and not src and dst net localnet'`

打印所有源或目的端口是80, 网络层协议为IPv4, 并且含有数据,而不是SYN,FIN以及ACK-only等不含数据的数据包.(ipv6的版本的表达式可做练习)

`tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'`

(nt: 可理解为, ip[2:2]表示整个ip数据包的长度, (ip[0]&0xf)<<2)表示ip数据包包头的长度(ip[0]&0xf代表包中的IHL域, 而此域的单位为32bit, 要换算成字节数需要乘以4,　即左移2.　(tcp[12]&0xf0)>>4 表示tcp头的长度, 此域的单位也是32bit,　换算成比特数为 ((tcp[12]&0xf0) >> 4)　<<　２, 即 ((tcp[12]&0xf0)>>2).　((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0　表示: 整个ip数据包的长度减去ip头的长度,再减去tcp头的长度不为0, 这就意味着, ip数据包中确实是有数据.对于ipv6版本只需考虑ipv6头中的'Payload Length' 与 'tcp头的长度'的差值, 并且其中表达方式'ip[]'需换成'ip6[]'.)

打印长度超过576字节, 并且网关地址是snup的IP数据包

`tcpdump 'gateway snup and ip[2:2] > 576'`

打印所有IP层广播或多播的数据包， 但不是物理以太网层的广播或多播数据报

`tcpdump 'ether[0] & 1 = 0 and ip[16] >= 224'`

打印除'echo request'或者'echo reply'类型以外的ICMP数据包( 比如,需要打印所有非ping 程序产生的数据包时可用到此表达式 (nt: 'echo reuqest' 与 'echo reply' 这两种类型的ICMP数据包通常由ping程序产生))

`tcpdump 'icmp[icmptype] != icmp-echo and icmp[icmptype] != icmp-echoreply'`



# 配合wireshark

```bash
tcpdump tcp -i eth1 -t -s 0 -c 100 and dst port ! 22 and src net 192.168.1.0/24 -w ./target.cap
```

* `tcp`: ip icmp arp rarp 和 tcp、udp、icmp这些选项等都要放到第一个参数的位置，用来过滤数据报的类型
* `-i eth1`: 只抓经过接口eth1的包
* `-t`: 不显示时间戳
* `-s 0`: 抓取数据包时默认抓取长度为68字节。加上-S 0 后可以抓到完整的数据包
* `-c 100`: 只抓取100个数据包
* `dst port ! 22`: 不抓取目标端口是22的数据包
* `src net 192.168.1.0/24`: 数据包的源网络地址为192.168.1.0/24
* `-w ./target.cap`: 保存成cap文件，方便用ethereal(即wireshark)分析

# 抓取HTTP包

```bash
tcpdump  -XvvennSs 0 -i eth0 tcp[20:2]=0x4745 or tcp[20:2]=0x4854
```

0x4745 为"GET"前两个字母"GE",0x4854 为"HTTP"前两个字母"HT"。









# 参考

* [Linux tcpdump命令详解](http://www.cnblogs.com/ggjucheng/archive/2012/01/14/2322659.html)