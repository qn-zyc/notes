<!-- TOC -->

- [查看进程文件描述符](#查看进程文件描述符)
- [TIME_WAIT](#time_wait)
- [查询域名IP注册信息](#查询域名ip注册信息)
- [查看进程的运行时间](#查看进程的运行时间)
- [查看系统信息](#查看系统信息)
    - [用户相关](#用户相关)
- [md5](#md5)
- [top](#top)
    - [top 命令的显示结果](#top-命令的显示结果)
- [权限](#权限)
    - [chmod](#chmod)
    - [chgrp](#chgrp)
    - [chown](#chown)
    - [nc 测试 udp 端口](#nc-测试-udp-端口)
- [磁盘](#磁盘)
    - [du](#du)
        - [介绍](#介绍)
        - [示例](#示例)
    - [df](#df)
        - [Examples](#examples)
    - [iostat](#iostat)
    - [lsscsi](#lsscsi)
- [网络](#网络)
    - [nload](#nload)
- [常用命令](#常用命令)
    - [ls](#ls)
    - [grep](#grep)
    - [sort](#sort)
    - [uniq](#uniq)
    - [zcat](#zcat)
    - [tail](#tail)
    - [head](#head)
- [参考](#参考)

<!-- /TOC -->

# 查看进程文件描述符

* `lsof -p <进程ID>` 查看进程占用了哪些文件.
* `netstat -ant | grep -i "80" | wc -l` 查看连接数
* `lsof -n | awk '{print $2}' | sort | uniq -c | sort -nr | more` 查看各个进程占用的句柄数, 第一列是句柄数, 第二列是进程号
* `cat /proc/sys/fs/file-max` 查看系统最大打开文件描述符.
* `echo 10000 > /proc/sys/fs/file-max` 临时修改系统最大文件描述符.
* 在 `/etc/sysctl.conf` 中 **永久** 设置最大文件描述符: `fs.file-max = 10000`.
* `ulimit -n` (soft limit)查看进程最大文件描述符(`ulimit -Hn` 查看 hard limit). `cat /proc/<pid>/limits` 查看具体进程的限制.
* (临时)`ulimit -n 1000` 设置 soft limit 和 hard limit, `ulimit -Sn 1000` 设置 soft limit, `ulimit -Hn 1000` 设置 hard limit.
* (永久)在 `/etc/security/limits.conf` 中添加下面两行(需要注意设置 nofile 的 hard limit 不能大于 `/proc/sys/fs/nr_open`, 如果大于注销后不能正常登陆, 可以修改 `nr_open` 的值: `echo 20000 > /proc/sys/fs/nr_open`)
    ```
    user_name   soft    nofile  180000
    user_name   hard    nofile  200000
    ```
* `ll /proc/<pid>/fd | wc -l` 查看进程打开的文件描述符数.



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

## 用户相关

- `who` 或 `finger`: 查看当前登录用户.
- `finger user`: 查看 user 用户的信息.
- `groups user`: 查看 user 用户所属组.
- `usermod -a -G testgroup user`: 相关用户所属组.


# md5

```shell
md5 文件名
```


# top

执行 top 命令后, 再输入 `P` 可按 CPU 占用降序排序, 输入 `M` 可按内存占用降序排序.

## top 命令的显示结果

```
top - 01:06:48 up 1:22, 1 user, load average: 0.06, 0.60, 0.48
Tasks: 29 total, 1 running, 28 sleeping, 0 stopped, 0 zombie
Cpu(s): 0.3% us, 1.0% sy, 0.0% ni, 98.7% id, 0.0% wa, 0.0% hi, 0.0% si
Mem: 191272k total, 173656k used, 17616k free, 22052k buffers
Swap: 192772k total, 0k used, 192772k free, 123988k cached

PID USER PR NI VIRT RES SHR S %CPU %MEM TIME+ COMMAND
1379 root 16 0 7976 2456 1980 S 0.7 1.3 0:11.03 sshd
14704 root 16 0 2128 980 796 R 0.7 0.5 0:02.72 top
1 root 16 0 1992 632 544 S 0.0 0.3 0:00.90 init
2 root 34 19 0 0 0 S 0.0 0.0 0:00.00 ksoftirqd/0
3 root RT 0 0 0 0 S 0.0 0.0 0:00.00 watchdog
```

- `01:06:48 up 1:22`: 系统当前时间, 已经运行的时间(单位 时:分).
- `1 user`: 当前登录用户数.
- `load average: 0.06, 0.60, 0.48`: 系统负载, 分别是当前时刻之前1分钟,5分钟,15分钟的平均值.
- `Tasks: 29 total, 1 running, 28 sleeping, 0 stopped, 0 zombie`: 进程总数, 正在运行的进程数, 休眠进程数, 停止进程数, 僵尸进程数.
- `Cpu(s): 0.3% us, 1.0% sy, 0.0% ni, 98.7% id, 0.0% wa, 0.0% hi, 0.0% si`: us 用户空间, sy 内核空间, ni 用户进程空间内改变过优先级的进程占用CPU百分比, id 空闲CPU百分比, wa 等待输入输出的CPU时间百分比, hi TODO:, si TODO:
- `Mem: 191272k total, 173656k used, 17616k free, 22052k buffers`: total 物理内存总量, used 使用的物理内存总量, free 空闲内存, buffers 用作内核缓存的内存.
- `Swap: 192772k total, 0k used, 192772k free, 123988k cached`: total 交换区总量, used 使用的交换区总量, free 空闲的交换区总量, cached 缓冲的交换区总量.

统计信息的各列含义:

| 序号 |  列名   |                                  含义                                   |
| ---- | ------- | ----------------------------------------------------------------------- |
| a    | PID     | 进程id                                                                  |
| b    | PPID    | 父进程id                                                                |
| c    | RUSER   | Real user name                                                          |
| d    | UID     | 进程所有者的用户id                                                      |
| e    | USER    | 进程所有者的用户名                                                      |
| f    | GROUP   | 进程所有者的组名                                                        |
| g    | TTY     | 启动进程的终端名。不是从终端启动的进程则显示为 ?                        |
| h    | PR      | 优先级                                                                  |
| i    | NI      | nice值。负值表示高优先级，正值表示低优先级                              |
| j    | P       | 最后使用的CPU，仅在多CPU环境下有意义                                    |
| k    | %CPU    | 上次更新到现在的CPU时间占用百分比                                       |
| l    | TIME    | 进程使用的CPU时间总计，单位秒                                           |
| m    | TIME+   | 进程使用的CPU时间总计，单位1/100秒                                      |
| n    | %MEM    | 进程使用的物理内存百分比                                                |
| o    | VIRT    | 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES                           |
| p    | SWAP    | 进程使用的虚拟内存中，被换出的大小，单位kb。                            |
| q    | RES     | 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA               |
| r    | CODE    | 可执行代码占用的物理内存大小，单位kb                                    |
| s    | DATA    | 可执行代码以外的部分(数据段+栈)占用的物理内存大小，单位kb               |
| t    | SHR     | 共享内存大小，单位kb                                                    |
| u    | nFLT    | 页面错误次数                                                            |
| v    | nDRT    | 最后一次写入到现在，被修改过的页面数。                                  |
| w    | S       | 进程状态。D=不可中断的睡眠状态, R=运行, S=睡眠, T=跟踪/停止, Z=僵尸进程 |
| x    | COMMAND | 命令名/命令行                                                           |
| y    | WCHAN   | 若该进程在睡眠，则显示睡眠中的系统函数名                                |
| z    | Flags   | 任务标志，参考 sched.h                                                  |

默认情况下仅显示比较重要的 PID、USER、PR、NI、VIRT、RES、SHR、S、%CPU、%MEM、TIME+、COMMAND 列。可以通过下面的快捷键来更改显示内容。

- 通过 `f` 键可以选择显示的内容。按 `f` 键之后会显示列的列表，按 `a-z` 即可显示或隐藏对应的列，最后按回车键确定。
- 按 `o` 键可以改变列的显示顺序。按小写的 `a-z` 可以将相应的列向右移动，而大写的 `A-Z` 可以将相应的列向左移动。最后按回车键确定。
- 按大写的 `F` 或 `O` 键，然后按 `a-z` 可以将进程按照相应的列进行排序。而大写的 `R` 键可以将当前的排序倒转。



# 权限

- `rwx` 读, 写, 执行权限.
- `rwxr-xr--` 分别表示文件属主, 同组, 其他人的权限.

## chmod

改变文件或目录的权限. 格式 `chmod ［who］ ［+ | – | =］ ［mode］ 文件名`

- who 可以有以下选项:
    - `u`: 文件所有者.
    - `g`: 同组用户.
    - `o`: 其他用户.
    - `a`: 所有用户, 缺省值.
- 操作符合可以是:
    - `+`: 添加权限.
    - `-`: 取消权限.
    - `=`: 赋予给定权限并取消其他权限(相当于重置权限).
- mode 可以是以下值:
    - `r`: 可读.
    - `w`: 可写.
    - `x`: 可执行.
    - `u`: 与文件属主拥有一样的权限.
    - `g`: 与文件属主的同组用户拥有一样的权限.
    - `o`: 与其他用户拥有一样的权限.
    - `c`: 若该档案权限确实已经更改，才显示其更改动作
    - `f`: 若该档案权限无法被更改也不要显示错误讯息
    - `v`: 显示权限变更的详细资料
    - `R`: 对目前目录下的所有档案与子目录进行相同的权限变更(即以递回的方式逐个变更)
    - ...
- 文件名可以有多个, 使用空格分隔, 支持通配符.

示例:
- `chmod 777 a.file`: 修改文件的权限.
- `chmod -R 777 a.dir`: 修改目录及目录下子目录权限.
- `chmod g+r，o+r a.file`: 使同组用户和其他用户具有读权限.
- `chmod ug+w，o-x a.file`: 使属主和同组用户具有写权限, 其他用户取消执行权限.
- `chmod go-rw a.file`: 删除同组用户和其他用户的读写权限.


## chgrp

改变文件或目录所属的组: `chgrp [选项] group filename`

- 选项:
    - `-c` 或 `–changes`: 效果类似 `-v` 参数，但仅回报更改的部分。
    - `-f` 或 `–quiet` 或 `–silent`: 不显示错误信息。
    - `-h` 或 `–no-dereference`: 只对符号连接的文件作修改，而不更动其他任何相关文件。
    - `-R` 或 `–recursive`: 递归处理，将指定目录下的所有文件及子目录一并处理。
    - `-v` 或 `–verbose`: 显示指令执行过程。
    - `-help`: 在线帮助。
    - `-reference=<参考文件或目录>`: 把指定文件或目录的所属群组全部设成和参考文件或目录的所属群组相同。
    - `-version`: 显示版本信息。
- group 可以是用户组 ID 或者 `/etc/group` 中的名称.
- filename 支持多个文件, 以空格分隔, 支持通配符.


## chown

更改文件的属主和用户组: `chown [选项] owner[:group] 文件` 或 `chown [选项] :group 文件`

- 选项:
    - `c`: 若该档案拥有者确实已经更改，才显示其更改动作
    - `f`: 若该档案拥有者无法被更改也不要显示错误讯息
    - `h`: 只对于连结(link)进行变更，而非该 link 真正指向的档案
    - `v`: 显示拥有者变更的详细资料
    - `R`: 对目前目录下的所有档案与子目录进行相同的拥有者变更(即以递回的方式逐个变更)
    - `help`: 显示辅助说明
    - `version`: 显示版本

示例:
- `chown user1 a.dir`: 将用户的所有者改为 user1.
- `chown -R user1.users /data`: 递归地将 data 目录及其子目录的属主改为 user1, 用户组改为 users.



## nc 测试 udp 端口

nc命令用法：

usage: `nc [-46DdhklnrStUuvzC] [-i interval] [-p source_port] [-s source_ip_address] [-T ToS] [-w timeout] [-X proxy_version] [-x proxy_address[:port]] [hostname] [port[s]]`

```
# nc -vuz 1.1.1.1 123
Connection to 1.1.1.1 123 port [udp/ntp] succeeded!
```


# 磁盘

## du

### 介绍

du 命令是对文件和目录磁盘使用空间的查看。

命令参数:

```
-a或-all  显示目录中个别文件的大小。   
-b或-bytes  显示目录或文件大小时，以byte为单位。   
-c或--total  除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。
-k或--kilobytes  以KB(1024bytes)为单位输出。
-m或--megabytes  以MB为单位输出。   
-s或--summarize  仅显示总计，只列出最后加总的值。
-h或--human-readable  以K，M，G为单位，提高信息的可读性。
-x或--one-file-xystem  以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过。
-L<符号链接>或--dereference<符号链接> 显示选项中所指定符号链接的源文件大小。   
-S或--separate-dirs   显示个别目录的大小时，并不含其子目录的大小。
-X<文件>或--exclude-from=<文件>  在<文件>指定目录或文件。   
--exclude=<目录或文件>         略过指定的目录或文件。    
-D或--dereference-args   显示指定符号链接的源文件大小。   
-H或--si  与-h参数相同，但是K，M，G是以1000为换算单位。   
-l或--count-links   重复计算硬件链接的文件。
```


### 示例

- 指定目录磁盘占用top10: `du -a ./dir | sort -n -r | head -n 10`
- 显示指定文件所占空间: `du test.log`
- 显示指定目录所占空间: `du dir`
- 显示多个文件所占空间: `du t1.log t2.log`
- 显示总和大小: `du -s`, `du -s dir`
- 以方便阅读的格式显示: `du -h dir`
- 输出当前目录下各子目录所占空间: `du -h --max-depth=1`


## df

df 命令是linux系统以磁盘分区为单位查看文件系统, 可以加上参数查看磁盘剩余空间信息.

### Examples

- 查看磁盘剩余空间: `df -h`, `df -hl`


## iostat

iostat是I/O statistics（输入/输出统计）的缩写。iostat工具将对系统的磁盘操作活动进行监视。它的特点是汇报磁盘活动统计情况，同时也会汇报出CPU使用情况。

命令格式：`iostat[参数][时间][次数]`

参数：

```
-C 显示CPU使用情况
-d 显示磁盘使用情况
-k 以 KB 为单位显示
-m 以 M 为单位显示
-N 显示磁盘阵列(LVM) 信息
-n 显示NFS 使用情况
-p[磁盘] 显示磁盘和分区的情况
-t 显示终端和CPU的信息
-x 显示详细信息
-V 显示版本信息
```

输出：

```
[root@CT1186 ~]# iostat
Linux 2.6.18-128.el5 (CT1186)   2012年12月28日

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           8.30    0.02    5.07    0.17    0.00   86.44

Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
sda              22.73        43.70       487.42  674035705 7517941952
sda1              0.00         0.00         0.00       2658        536
sda2              0.11         3.74         3.51   57721595   54202216
sda3              0.98         0.61        17.51    9454172  270023368
sda4              0.00         0.00         0.00          6          0
sda5              6.95         0.12       108.73    1924834 1677123536
sda6              2.20         0.18        31.22    2837260  481488056
sda7             12.48        39.04       326.45  602094508 5035104240
```

cpu属性值说明：

```
%user：CPU处在用户模式下的时间百分比。
%nice：CPU处在带NICE值的用户模式下的时间百分比。
%system：CPU处在系统模式下的时间百分比。
%iowait：CPU等待输入输出完成时间的百分比。
%steal：管理程序维护另一个虚拟处理器时，虚拟CPU的无意识等待时间百分比。
%idle：CPU空闲时间百分比。
```

备注：如果%iowait的值过高，表示硬盘存在I/O瓶颈，%idle值高，表示CPU较空闲，如果%idle值高但系统响应慢时，有可能是CPU等待分配内存，此时应加大内存容量。%idle值如果持续低于10，那么系统的CPU处理能力相对较低，表明系统中最需要解决的资源是CPU。

disk属性值说明：

```
rrqm/s:  每秒进行 merge 的读操作数目。即 rmerge/s
wrqm/s:  每秒进行 merge 的写操作数目。即 wmerge/s
r/s:  每秒完成的读 I/O 设备次数。即 rio/s
w/s:  每秒完成的写 I/O 设备次数。即 wio/s
rsec/s:  每秒读扇区数。即 rsect/s
wsec/s:  每秒写扇区数。即 wsect/s
rkB/s:  每秒读K字节数。是 rsect/s 的一半，因为每扇区大小为512字节。
wkB/s:  每秒写K字节数。是 wsect/s 的一半。
avgrq-sz:  平均每次设备I/O操作的数据大小 (扇区)。
avgqu-sz:  平均I/O队列长度。
await:  平均每次设备I/O操作的等待时间 (毫秒)。
svctm: 平均每次设备I/O操作的服务时间 (毫秒)。
%util:  一秒中有百分之多少的时间用于 I/O 操作，即被io消耗的cpu百分比
```

备注：如果 %util 接近 100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘可能存在瓶颈。如果 svctm 比较接近 await，说明 I/O 几乎没有等待时间；如果 await 远大于 svctm，说明I/O 队列太长，io响应太慢，则需要进行必要优化。如果avgqu-sz比较大，也表示有当量io在等待。

吞吐量信息：

```
tps：该设备每秒的传输次数（Indicate the number of transfers per second that were issued to the device.）。“一次传输”意思是“一次I/O请求”。多个逻辑请求可能会被合并为“一次I/O请求”。“一次传输”请求的大小是未知的。
kB_read/s：每秒从设备（drive expressed）读取的数据量；
kB_wrtn/s：每秒向设备（drive expressed）写入的数据量；
kB_read：读取的总数据量；kB_wrtn：写入的总数量数据量；
这些单位都为Kilobytes。
```



示例：

```bash
iostat # 单独执行iostat，显示的结果为从系统开机到当前执行时刻的统计信息。
iostat -x # 显示详细信息
iostat -x 1 # 每秒显示一次详细信息
iostat 2 3 # 每2秒刷新一次，显示3次
iostat -d sda1 # 显示指定磁盘信息
iostat -t # 显示 tty 和 cpu 信息
iostat -m # 以M为单位显示
iostat -d -k # 查看 tps 和吞吐量信息
iostat -c # 查看 cpu 状态
```

查看设备使用率 %util, 响应时间 await:

```
iostat -d -x -k 1 1

Device:         rrqm/s   wrqm/s   r/s   w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await  svctm  %util
sda               0.44    38.59  0.40 22.32    21.85   243.71    23.37     0.04    1.78   4.20   9.54
sda1              0.00     0.00  0.00  0.00     0.00     0.00    18.90     0.00    8.26   6.46   0.00
sda2              0.36     0.43  0.11  0.01     1.87     1.76    63.57     0.01   63.75   1.94   0.02
sda3              0.00     1.24  0.04  0.95     0.31     8.75    18.42     0.04   39.77   8.73   0.86
sda4              0.00     0.00  0.00  0.00     0.00     0.00     2.00     0.00   19.67  19.67   0.00
sda5              0.00     6.65  0.00  6.94     0.06    54.37    15.67     0.26   36.81   4.48   3.11
sda6              0.00     1.71  0.01  2.19     0.09    15.61    14.29     0.03   12.40   5.84   1.28
sda7              0.08    28.56  0.25 12.24    19.52   163.22    29.28     0.27   21.46   5.00   6.25 

rrqm/s：  每秒进行 merge 的读操作数目.即 delta(rmerge)/s
wrqm/s： 每秒进行 merge 的写操作数目.即 delta(wmerge)/s
r/s：  每秒完成的读 I/O 设备次数.即 delta(rio)/s
w/s：  每秒完成的写 I/O 设备次数.即 delta(wio)/s
rsec/s：  每秒读扇区数.即 delta(rsect)/s
wsec/s： 每秒写扇区数.即 delta(wsect)/s
rkB/s：  每秒读K字节数.是 rsect/s 的一半,因为每扇区大小为512字节.(需要计算)
wkB/s：  每秒写K字节数.是 wsect/s 的一半.(需要计算)
avgrq-sz：平均每次设备I/O操作的数据大小 (扇区).delta(rsect+wsect)/delta(rio+wio)
avgqu-sz：平均I/O队列长度.即 delta(aveq)/s/1000 (因为aveq的单位为毫秒).
await：  平均每次设备I/O操作的等待时间 (毫秒).即 delta(ruse+wuse)/delta(rio+wio)
svctm： 平均每次设备I/O操作的服务时间 (毫秒).即 delta(use)/delta(rio+wio)
%util： 一秒中有百分之多少的时间用于 I/O 操作,或者说一秒中有多少时间 I/O 队列是非空的，即 delta(use)/s/1000 (因为use的单位为毫秒)
如果 %util 接近 100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘可能存在瓶颈。
idle小于70% IO压力就较大了，一般读取速度有较多的wait。
同时可以结合vmstat 查看查看b参数(等待资源的进程数)和wa参数(IO等待所占用的CPU时间的百分比，高过30%时IO压力高)。
另外 await 的参数也要多和 svctm 来参考。差的过高就一定有 IO 的问题。
avgqu-sz 也是个做 IO 调优时需要注意的地方，这个就是直接每次操作的数据的大小，如果次数多，但数据拿的小的话，其实 IO 也会很小。如果数据拿的大，才IO 的数据会高。也可以通过 avgqu-sz × ( r/s or w/s ) = rsec/s or wsec/s。也就是讲，读定速度是这个来决定的。
svctm 一般要小于 await (因为同时等待的请求的等待时间被重复计算了)，svctm 的大小一般和磁盘性能有关，CPU/内存的负荷也会对其有影响，请求过多也会间接导致 svctm 的增加。await 的大小一般取决于服务时间(svctm) 以及 I/O 队列的长度和 I/O 请求的发出模式。如果 svctm 比较接近 await，说明 I/O 几乎没有等待时间；如果 await 远大于 svctm，说明 I/O 队列太长，应用得到的响应时间变慢，如果响应时间超过了用户可以容许的范围，这时可以考虑更换更快的磁盘，调整内核 elevator 算法，优化应用，或者升级 CPU。
队列长度(avgqu-sz)也可作为衡量系统 I/O 负荷的指标，但由于 avgqu-sz 是按照单位时间的平均值，所以不能反映瞬间的 I/O 洪水。

       形象的比喻：
       r/s+w/s 类似于交款人的总数
      平均队列长度(avgqu-sz)类似于单位时间里平均排队人的个数
      平均服务时间(svctm)类似于收银员的收款速度
      平均等待时间(await)类似于平均每人的等待时间
      平均I/O数据(avgrq-sz)类似于平均每人所买的东西多少
       I/O 操作率 (%util)类似于收款台前有人排队的时间比例

       设备IO操作:总IO(io)/s = r/s(读) +w/s(写) =1.46 + 25.28=26.74
      平均每次设备I/O操作只需要0.36毫秒完成,现在却需要10.57毫秒完成，因为发出的	请求太多(每秒26.74个)，假如请求时同时发出的，可以这样计算平均等待时间:
      平均等待时间=单个I/O服务器时间*(1+2+...+请求总数-1)/请求总数 
       每秒发出的I/0请求很多,但是平均队列就4,表示这些请求比较均匀,大部分处理还是比较及时。
```


## lsscsi

可以看到Raid卡信息和所有虚拟磁盘以及光驱的信息，如果没有硬件SCSI控制器，那就不会返回信息：

```
[root@localhost /]# lsscsi
[0:2:0:0]    disk    DELL     PERC H730P Mini  4.27  /dev/sda
[0:2:1:0]    disk    DELL     PERC H730P Mini  4.27  /dev/sdb
[0:2:2:0]    disk    DELL     PERC H730P Mini  4.27  /dev/sdc
[10:0:0:0]   cd/dvd  PLDS     DVD+-RW DS-8ABSH LD51  /dev/sr0
```


# 网络

## nload

查看流量。



# 常用命令

## ls

- `ls -a`: 显示所有的文件和目录，包括隐藏文件。
- `ls -F`: 在目录后添加 `/`，在可执行文件后添加 `*`，在链接后添加 `@`，在 socket 后添加 `=` ... ...
- `ls -l`: 每个项目显示更详细的信息。
- `ls -R`: 递归显示子目录的文件。

ls 也支持正则，比如 `ls my_scr?pt`。
- `?`    : 代表一个字符。
- `*`    : 代表一个或多个字符。
- `[xyz]`: 其中一个字符。
- `[a-z]`: 字符范围。
- `[!a]` : 除了 a 之外的其他字符。


## grep

grep（global search regular expression(RE) and print out the line，全面搜索正则表达式并把行打印出来）是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。

选项:

```bash
-a 不要忽略二进制数据。
-A<显示列数> 除了显示符合范本样式的那一行之外，并显示该行之后的内容。
-B 在显示符合范本样式的那一行之外，并显示该行之前的内容。
-b 打印出匹配文本的偏移字节数.
-c 计算符合范本样式的列数。
-C<显示列数>或-<显示列数>  除了显示符合范本样式的那一列之外，并显示该列之前后的内容。
-d<进行动作> 当指定要查找的是目录而非文件时，必须使用这项参数，否则grep命令将回报信息并停止动作。
-e<范本样式> 指定字符串作为查找文件内容的范本样式。
-E 将范本样式为延伸的普通表示法来使用，意味着使用能使用扩展正则表达式。
-f<范本文件> 指定范本文件，其内容有一个或多个范本样式，让grep查找符合范本条件的文件内容，格式为每一列的范本样式。
-F 将范本样式视为固定字符串的列表。
-G 将范本样式视为普通的表示法来使用。
-h 在显示符合范本样式的那一列之前，不标示该列所属的文件名称。
-H 在显示符合范本样式的那一列之前，标示该列的文件名称。
-i 忽略字符大小写的差别。
-l 列出文件内容符合指定的范本样式的文件名称。
-L 列出文件内容不符合指定的范本样式的文件名称。
-n 在显示符合范本样式的那一列之前，标示出该列的编号。
-q 不显示任何信息。
-R/-r 此参数的效果和指定“-d recurse”参数相同。
-s 不显示错误信息。
-v 反转查找。
-w 只显示全字符合的列。
-x 只显示全列符合的列。
-y 此参数效果跟“-i”相同。
-o 只输出文件中匹配到的部分。
```

常见用法:

```bash
# 找出filename中带有keyword的行，输出中除显示该行外，还显示之后的一行(After 1)
grep -A1 keyword filename
# 找出filename中带有keyword的行，输出中除显示该行外，还显示之前的一行(Before 1)
grep -B1 keyword filename
# 找出filename中带有keyword的行，输出中除显示该行外，还显示之前的一行(After 1)和显示之后的一行(After 1）
grep -1 keyword filename

# 在文件中搜索一个单词:
grep match_pattern file_name
grep "match_pattern" file_name

# 在多个文件中查找：
grep "match_pattern" file_1 file_2 file_3 ...

# 输出除之外的所有行 -v 选项：
grep -v "match_pattern" file_name

# 标记匹配颜色 --color=auto 选项：
grep "match_pattern" file_name --color=auto

# 使用正则表达式 -E 选项：
grep -E "[1-9]+"
# 或
egrep "[1-9]+"

# 只输出文件中匹配到的部分 -o 选项：
echo this is a test line. | grep -o -E "[a-z]+\."
line.
echo this is a test line. | egrep -o "[a-z]+\."
line.

# 统计文件或者文本中包含匹配字符串的行数 -c 选项：
grep -c "text" file_name

# 输出包含匹配字符串的行数 -n 选项：
grep "text" -n file_name
# 多个文件
grep "text" -n file_1 file_2

# 打印样式匹配所位于的字符或字节偏移：
echo gun is not unix | grep -b -o "not"
7:not #一行中字符串的字符便宜是从该行的第一个字符开始计算，起始值为0。选项 -b -o 一般总是配合使用。

# 搜索多个文件并查找匹配文本在哪些文件中：
grep -l "text" file1 file2 file3...

# 在多级目录中对文本进行递归搜索
grep 'txt' . -r -n  # . 表示当前目录
```


## sort


## uniq


## zcat

zcat命令用于不真正解压缩文件，就能显示压缩包中文件的内容的场合。

语法: `zcat(选项)(参数)`

选项:

```
-S：指定gzip格式的压缩包的后缀。当后缀不是标准压缩包后缀时使用此选项；
-c：将文件内容写到标注输出；
-d：执行解压缩操作；
-l：显示压缩包中文件的列表；
-L：显示软件许可信息；
-q：禁用警告信息；
-r：在目录上执行递归操作；
-t：测试压缩文件的完整性；
-V：显示指令的版本信息；
-l：更快的压缩速度；
-9：更高的压缩比。
```

参数: 指定要显示其中文件内容的压缩包。


## tail

```bash
tail -n 1000 # 显示最后1000行
tail -n +1000 # 从第1000行开始显示
```

## head

```bash
head -n 1000 # 显示前面1000行
```


# 参考
- [在 Linux 下如何查看一个进程的运行时间](https://yq.aliyun.com/articles/88158)
- [Linux TOP命令 按内存占用排序和按CPU占用排序](http://www.linuxidc.com/Linux/2011-03/33582.htm)
- [Linux命令:修改文件权限命令chmod、chgrp、chown详解](https://yusi123.com/3097.html)
- [每天一个linux命令（47）：iostat命令](http://www.cnblogs.com/peida/archive/2012/12/28/2837345.html)
- [Linux下lshw,lsscsi,lscpu,lsusb,lsblk硬件查看命令](http://www.heminjie.com/system/linux/6253.html)
