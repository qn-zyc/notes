<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [curl](#curl)
	- [获取页面](#获取页面)
	- [POST](#post)
	- [指定方法](#指定方法)
	- [将输出写入到文件](#将输出写入到文件)
	- [上传文件](#上传文件)
	- [代理](#代理)
	- [https](#https)
		- [忽略校验](#忽略校验)
		- [resolve](#resolve)
	- [参数列表](#参数列表)
- [参考](#参考)

<!-- /TOC -->



# curl

## 获取页面

```bash
curl http://curl.haxx.se
```

这是最简单的使用方法。用这个命令获得了http://curl.haxx.se指向的页面，同样，如果这里的URL指向的是一个文件或者一幅图都可以直接下载到本地。如果下载的是HTML文档，那么缺省的将不显示文件头部，即HTML文档的header。要全部显示，请加参数 -i，要只显示头部，用参数 -I。任何时候，可以使用 -v 命令看curl是怎样工作的，它向服务器发送的所有命令都会显示出来。为了断点续传，可以使用-r参数来指定传输范围。


## POST

```bash
curl -d "birthyear=1905&press=OK" www.hotmail.com/when/junk.cgi
curl --data "param1=value1&param2=value" https://api.github.com
```

也可以指定一个文件，将该文件中的内容当作数据传递给服务器端

```bash
curl --data @filename https://github.api.com/authorizations
```

默认情况下，通过POST方式传递过去的数据中若有特殊字符，首先需要将特殊字符转义在传递给服务器端，如value值中包含有空格，则需要先将空格转换成%20，如

```bash
curl -d "value%201" http://hostname.com
```

在新版本的CURL中，提供了新的选项 --data-urlencode, 通过该选项提供的参数会自动转义特殊字符。

```bash
curl --data-urlencode "value 1" http://hostname.com
```

**data-urlencode也会把&编码**

提交 JSON 字符串:

```bash
curl -x localhost:80 -X POST -H "Content-Type: applecation/json" 'http://example.com/path' --data-binary '{"json":true}'
```


## 指定方法

除了使用GET和POST协议外，还可以通过 -X 选项指定其它协议，如：

```bash
curl -I -X DELETE https://api.github.cim
```

## 将输出写入到文件

`-o/--output` 把输出写到该文件中
`-O/--remote-name` 把输出写到该文件中，保留远程文件的文件名

将文件下载到本地并命名为mygettext.html

```bash
curl -o mygettext.html http://www.gnu.org/software/gettext/manual/gettext.html
```

将文件保存到本地并命名为gettext.html

```bash
curl -O http://www.gnu.org/software/gettext/manual/gettext.html
```

同样可以使用转向字符">"对输出进行转向输出

## 上传文件

```bash
curl -F "user=username" -F "pwd=userpassword" -F "csvFile=@test1.csv" http://localhost:9030/sms/marketing/upload/csv
```


## 代理

```shell
curl -x 127.0.0.1:8080 http://a.b.com
```


## https

### 忽略校验

```shell
curl --insecure https://localhost:443
curl -k https://localhost:443
```

### resolve
访问一个ip但是要带上sni, 可以使用resolve达到类似hosts文件的效果.

```shell
curl --resolve www.baidu.com:443:127.0.0.1 https://www.baidu.com -v
```


## 参数列表

|                 parameter                 |                        description                        | example |
| ----------------------------------------- | --------------------------------------------------------- | ------- |
| `-a/--append`                             | 上传文件时，附加到目标文件                                |         |
| `-A/--user-agent <string>`                | 设置用户代理发送给服务器                                  |         |
| `--anyauth`                               | 可以使用“任何”身份验证方法                                |         |
| `-b/--cookie <name=string/file>`          | cookie字符串或文件读取位置                                |         |
| `--basic`                                 | 使用HTTP基本验证                                          |         |
| `-B/--use-ascii`                          | 使用ASCII文本传输                                         |         |
| `-c/--cookie-jar <file>`                  | 操作结束后把cookie写入到这个文件中                        |         |
| `-C/--continue-at <offset>`               | 断点续转                                                  |         |
| `-d/--data <data>`                        | HTTP POST方式传送数据                                     |         |
| `--data-ascii <data>`                     | 以ascii的方式post数据                                     |         |
| `--data-binary <data>`                    | 以二进制的方式post数据                                    |         |
| `--negotiate`                             | 使用HTTP身份验证                                          |         |
| `--digest`                                | 使用数字身份验证                                          |         |
| `--disable-eprt`                          | 禁止使用EPRT或LPRT                                        |         |
| `--disable-epsv`                          | 禁止使用EPSV                                              |         |
| `-D/--dump-header <file>`                 | 把header信息写入到该文件中                                |         |
| `--egd-file <file>`                       | 为随机数据(SSL)设置EGD socket路径                         |         |
| `--tcp-nodelay`                           | 使用TCP_NODELAY选项                                       |         |
| `-e/--referer`                            | 来源网址                                                  |         |
| `-E/--cert <cert[:passwd]>`               | 客户端证书文件和密码 (SSL)                                |         |
| `--cert-type <type>`                      | 证书文件类型 (DER/PEM/ENG) (SSL)                          |         |
| `--key <key>`                             | 私钥文件名 (SSL)                                          |         |
| `--key-type <type>`                       | 私钥文件类型 (DER/PEM/ENG) (SSL)                          |         |
| `--pass  <pass>`                          | 私钥密码 (SSL)                                            |         |
| `--engine <eng>`                          | 加密引擎使用 (SSL). "--engine list" for list              |         |
| `--cacert <file>`                         | CA证书 (SSL)                                              |         |
| `--capath <directory>`                    | CA目   (made using c_rehash) to verify peer against (SSL) |         |
| `--ciphers <list>`                        | SSL密码                                                   |         |
| `--compressed`                            | 要求返回是压缩的形势 (using deflate or gzip)              |         |
| `--connect-timeout <seconds>`             | 设置最大请求时间                                          |         |
| `--create-dirs`                           | 建立本地目录的目录层次结构                                |         |
| `--crlf`                                  | 上传是把LF转变成CRLF                                      |         |
| ` -f/--fail`                              | 连接失败时不显示http错误                                  |         |
| `--ftp-create-dirs`                       | 如果远程目录不存在，创建远程目录                          |         |
| `--ftp-method [multicwd/nocwd/singlecwd]` | 控制CWD的使用                                             |         |
| `--ftp-pasv`                              | 使用 PASV/EPSV 代替端口                                   |         |
| `--ftp-skip-pasv-ip`                      | 使用PASV的时候,忽略该IP地址                               |         |
| `--ftp-ssl`                               | 尝试用 SSL/TLS 来进行ftp数据传输                          |         |
| `--ftp-ssl-reqd`                          | 要求用 SSL/TLS 来进行ftp数据传输                          |         |
| `-F/--form <name=content>`                | 模拟http表单提交数据                                      |         |
| `-form-string <name=string>`              | 模拟http表单提交数据                                      |         |
| `-g/--globoff`                            | 禁用网址序列和范围使用{}和[]                              |         |
| `-G/--get`                                | 以get的方式来发送数据                                     |         |
| `-h/--help`                               | 帮助                                                      |         |
| `-H/--header <line>`                      | 自定义头信息传递给服务器                                  |         |
| `--ignore-content-length`                 | 忽略的HTTP头信息的长度                                    |         |
| `-i/--include`                            | 输出时包括protocol头信息                                  |         |
| `-I/--head`                               | 只显示文档信息                                            |         |
| `-j/--junk-session-cookies`               | 读取文件时忽略session cookie                              |         |
| `--interface <interface>`                 | 使用指定网络接口/地址                                     |         |
| `--krb4 <level>`                          | 使用指定安全级别的krb4                                    |         |
| `-k/--insecure`                           | 允许不使用证书到SSL站点                                   |         |
| `-K/--config`                             | 指定的配置文件读取                                        |         |
| `-l/--list-only`                          | 列出ftp目录下的文件名称                                   |         |
| `--limit-rate <rate>`                     | 设置传输速度                                              |         |
| `--local-port<NUM>`                       | 强制使用本地端口号                                        |         |
| `-m/--max-time <seconds>`                 | 设置最大传输时间                                          |         |
| `--max-redirs <num>`                      | 设置最大读取的目录数                                      |         |
| `--max-filesize <bytes>`                  | 设置最大下载的文件总量                                    |         |
| `-M/--manual`                             | 显示全手动                                                |         |
| `-n/--netrc`                              | 从netrc文件中读取用户名和密码                             |         |
| `--netrc-optional`                        | 使用 .netrc 或者 URL来覆盖-n                              |         |
| `--ntlm`                                  | 使用 HTTP NTLM 身份验证                                   |         |
| `-N/--no-buffer`                          | 禁用缓冲输出                                              |         |
| `-p/--proxytunnel`                        | 使用HTTP代理                                              |         |
| `--proxy-anyauth`                         | 选择任一代理身份验证方法                                  |         |
| `--proxy-basic`                           | 在代理上使用基本身份验证                                  |         |
| `--proxy-digest`                          | 在代理上使用数字身份验证                                  |         |
| `--proxy-ntlm`                            | 在代理上使用ntlm身份验证                                  |         |
| `-P/--ftp-port <address>`                 | 使用端口地址，而不是使用PASV                              |         |
| `-Q/--quote <cmd>`                        | 文件传输前，发送命令到服务器                              |         |
| `--range-file`                            | 读取（SSL）的随机文件                                     |         |
| `-R/--remote-time`                        | 在本地生成文件时，保留远程文件时间                        |         |
| `--retry <num>`                           | 传输出现问题时，重试的次数                                |         |
| `--retry-delay <seconds>`                 | 传输出现问题时，设置重试间隔时间                          |         |
| `--retry-max-time <seconds>`              | 传输出现问题时，设置最大重试时间                          |         |
| `-S/--show-error`                         | 显示错误                                                  |         |
| `--socks4 <host[:port]>`                  | 用socks4代理给定主机和端口                                |         |
| `--socks5 <host[:port]>`                  | 用socks5代理给定主机和端口                                |         |
| `-t/--telnet-option <OPT=val>`            | Telnet选项设置                                            |         |
| `--trace <file>`                          | 对指定文件进行debug                                       |         |
| `--trace-ascii <file>`                    | Like --跟踪但没有hex输出                                  |         |
| `--trace-time`                            | 跟踪/详细输出时，添加时间戳                               |         |
| `--url <URL>`                             | Spet URL to work with                                     |         |
| `-U/--proxy-user <user[:password]>`       | 设置代理用户名和密码                                      |         |
| `-V/--version`                            | 显示版本信息                                              |         |
| `-X/--request <command>`                  | 指定什么命令                                              |         |
| `-y/--speed-time`                         | 放弃限速所要的时间。默认为30                              |         |
| `-Y/--speed-limit`                        | 停止传输速度的限制，速度时间'秒                           |         |
| `-z/--time-cond`                          | 传送时间设置                                              |         |
| `-0/--http1.0`                            | 使用HTTP 1.0                                              |         |
| `-1/--tlsv1`                              | 使用TLSv1（SSL）                                          |         |
| `-2/--sslv2`                              | 使用SSLv2的（SSL）                                        |         |
| `-3/--sslv3`                              | 使用的SSLv3（SSL）                                        |         |
| `--3p-quote`                              | like -Q for the source URL for 3rd party transfer         |         |
| `--3p-url`                                | 使用url，进行第三方传送                                   |         |
| `--3p-user`                               | 使用用户名和密码，进行第三方传送                          |         |
| `-4/--ipv4`                               | 使用IP4                                                   |         |
| `-6/--ipv6`                               | 使用IP6                                                   |         |


# 参考
* [Linux curl命令详解](http://aiezu.com/article/linux_curl_command.html)
* [curl命令详解](http://www.cnblogs.com/duhuo/p/5695256.html)
