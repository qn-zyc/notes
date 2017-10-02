<!-- TOC -->

- [查看进程文件描述符](#查看进程文件描述符)
- [TIME_WAIT](#time_wait)
- [查询域名IP注册信息](#查询域名ip注册信息)
- [查看进程的运行时间](#查看进程的运行时间)
- [查看系统信息](#查看系统信息)
- [md5](#md5)
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


# 查看系统信息

* uname -a # 查看内核/操作系统/CPU信息 
* head -n 1 /etc/issue # 查看操作系统版本 
* cat /proc/cpuinfo # 查看CPU信息 
* hostname # 查看计算机名 
* lspci -tv # 列出所有PCI设备 
* lsusb -tv # 列出所有USB设备 
* lsmod # 列出加载的内核模块 
* env # 查看环境变量资源 
* free -m # 查看内存使用量和交换区使用量 
* df -h # 查看各分区使用情况 
* du -sh <目录名> # 查看指定目录的大小 
* grep MemTotal /proc/meminfo # 查看内存总量 
* grep MemFree /proc/meminfo # 查看空闲内存量 
* uptime # 查看系统运行时间、用户数、负载 
* cat /proc/loadavg # 查看系统负载磁盘和分区 
* mount | column -t # 查看挂接的分区状态 
* fdisk -l # 查看所有分区 
* swapon -s # 查看所有交换分区 
* hdparm -i /dev/hda # 查看磁盘参数(仅适用于IDE设备) 
* dmesg | grep IDE # 查看启动时IDE设备检测状况网络 
* ifconfig # 查看所有网络接口的属性 
* iptables -L # 查看防火墙设置 
* route -n # 查看路由表 
* netstat -lntp # 查看所有监听端口 
* netstat -antp # 查看所有已经建立的连接 
* netstat -s # 查看网络统计信息进程 
* ps -ef # 查看所有进程 
* top # 实时显示进程状态用户 
* w # 查看活动用户 
* id <用户名> # 查看指定用户信息 
* last # 查看用户登录日志 
* cut -d: -f1 /etc/passwd # 查看系统所有用户 
* cut -d: -f1 /etc/group # 查看系统所有组 
* crontab -l # 查看当前用户的计划任务服务 
* chkconfig –list # 列出所有系统服务 
* chkconfig –list | grep on # 列出所有启动的系统服务程序 
* rpm -qa # 查看所有安装的软件包

# md5

```shell
md5 文件名
```

# 参考
* [在 Linux 下如何查看一个进程的运行时间](https://yq.aliyun.com/articles/88158)