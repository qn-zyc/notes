<!-- TOC -->

- [查看进程文件描述符](#查看进程文件描述符)
- [TIME_WAIT](#time_wait)
- [查询域名IP注册信息](#查询域名ip注册信息)
- [查看进程的运行时间](#查看进程的运行时间)
- [参考](#参考)

<!-- /TOC -->

# 查看进程文件描述符

* `ulimit -a` 查看系统文件描述符上线.
* 在 `/proc/<进程ID>/fd` 目录下是进程打开的所有文件描述符, 使用 `ll | wc -l` 查看个数.
* `lsof -p <进程ID>` 查看进程占用了哪些文件.
* `netstat -ant | grep -i "80" | wc -l` 查看连接数


# TIME_WAIT

查看各个状态的连接数: `netstat -n | awk '/^tcp/ {++state[$NF]} END {for(key in state) print key,"\t",state[key]}'`

在内核参数: `/etc/sysctl.conf` 中:

* `net.ipv4.tcp_syncookies = 1` 表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
* `net.ipv4.tcp_tw_reuse = 1` 表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
* `net.ipv4.tcp_tw_recycle = 1` 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
* `net.ipv4.tcp_fin_timeout = 30` 修改系統默认的TIMEOUT时间(单位秒)
* `net.ipv4.tcp_max_tw_buckets = 6000`: TIME_WAIT 总数

修改后使用 `/sbin/sysctl -p` 让参数生效.


# 查询域名IP注册信息

```shell
whois 域名|IP
```


# 查看进程的运行时间
* 确定 pid: `pidof <进程>`.
* `ps -p <pid> -o etime`: etime 的格式是 `[[DD-]hh:]mm:ss`.
* `ps -p <pid> -o etimes`: etimes 输出的是已经启动了多少秒.
* 隐藏输出头: `ps -p 6176 -o etime=` 或 `ps -p 6176 -o etimes=`
* 打印更多信息: `ps -p 6176 -o pid,cmd,etime,uid,gid`.


# 参考
* [在 Linux 下如何查看一个进程的运行时间](https://yq.aliyun.com/articles/88158)