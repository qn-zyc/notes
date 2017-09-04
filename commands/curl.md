<!-- TOC -->

- [curl](#curl)
    - [获取页面](#获取页面)
    - [POST](#post)
    - [指定方法](#指定方法)
    - [将输出写入到文件](#将输出写入到文件)
    - [上传文件](#上传文件)
    - [https](#https)
        - [忽略校验](#忽略校验)
        - [resolve](#resolve)
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


## https

### 忽略校验

```shell
curl --insecure https://localhost:443
```

### resolve
访问一个ip但是要带上sni, 可以使用resolve达到类似hosts文件的效果.

```shell
curl --resolve www.baidu.com:443:127.0.0.1 https://www.baidu.com -v
```



# 参考
* [Linux curl命令详解](http://aiezu.com/article/linux_curl_command.html)