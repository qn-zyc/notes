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




# 参考
- [在 Linux 下如何查看一个进程的运行时间](https://yq.aliyun.com/articles/88158)
- [Linux TOP命令 按内存占用排序和按CPU占用排序](http://www.linuxidc.com/Linux/2011-03/33582.htm)
- [Linux命令:修改文件权限命令chmod、chgrp、chown详解](https://yusi123.com/3097.html)
