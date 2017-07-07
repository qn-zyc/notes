<!-- TOC -->

- [1. supervisor](#1-supervisor)
    - [1.1. 安装](#11-安装)
    - [1.2. supervisorctl](#12-supervisorctl)
    - [1.3. 配置](#13-配置)
        - [1.3.1. 日志](#131-日志)

<!-- /TOC -->



# 1. supervisor

## 1.1. 安装

- 官网 http://supervisord.org/

## 1.2. supervisorctl

- `supervisorctl status` 查看状态
- `supervisorctl help` 查看帮助
- `supervisorctl stop programxxx`，停止某一个进程(programxxx)，programxxx为[program:chatdemon]里配置的值
- `supervisorctl start programxxx`，启动某个进程
- `supervisorctl restart programxxx`，重启某个进程
- `supervisorctl stop groupworker` ，重启所有属于名为groupworker这个分组的进程(start,restart同理)
- `supervisorctl stop all`，停止全部进程，注：start、restart、stop都不会载入最新的配置文件。
- `supervisorctl reload`，载入最新的配置文件，停止原有进程并按新的配置启动、管理所有进程。
- `supervisorctl update`，根据最新的配置文件，启动新配置或有改动的进程，配置没有改动的进程不会受影响而重启。

注意：显示用stop停止掉的进程，用reload或者update都不会自动重启。

## 1.3. 配置

示例：

```
[program:redis]
command=/usr/bin/redis-server /usr/local/redis/redis.conf
autorstart=true
autorestart=true
stdout_logfile=/tmp/supervisor.log
```



### 1.3.1. 日志

```
;redirect_stderr=true          ; 重定向程序的标准错误到标准输出 (default false)
;stdout_logfile=/a/path        ; 标准输出的日志路径, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; 日志文件最大值，否则循环写入 (default 50MB)
;stdout_logfile_backups=10     ; 标准输出日志备份数目 (default 10)
;stdout_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; 标准错误输出日志路径, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; 日志文件最大值，否则循环写入 (default 50MB)
;stderr_logfile_backups=10     ; 标准错误日志备份数目 (default 10)
;stderr_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
```

