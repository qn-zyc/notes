<!-- TOC -->

- [urllib 与 urllib2 的比较](#urllib-与-urllib2-的比较)
- [urllib2](#urllib2)
    - [urlopen()](#urlopen)
        - [以GET方式拉取网页](#以get方式拉取网页)
        - [添加超时](#添加超时)
        - [POST](#post)
        - [设置header](#设置header)
- [httplib](#httplib)
    - [访问https网页](#访问https网页)
- [urlparse](#urlparse)
- [参考](#参考)

<!-- /TOC -->


# urllib 与 urllib2 的比较
* urllib2 可以接受 Request 对象, 设置 header 等, 但 urllib 只接受 URL.
* urllib 提供 urlencode 方法, urllib2 不提供.
* urllib2 是对 urllib 的补充.


# urllib2

## urlopen()
```python
urllib2.urlopen(url[, data[, timeout[, cafile[, capath[, cadefault[, context]]]]])
```
* url 可以是字符串或者 Request 对象.
* data 可以是字符串或者 None, 当 data 提供时, 请求会变为 POST 请求. data 是 `application/x-www-form-urlencoded` 格式, 可以使用 `urllib.urlencode()` 转换其他类型为字符串. urllib2 包会发送 `Connection:close` header.
* timeout 单位是秒.
* context 必须是 `ssl.SSLContext` 的示例.


### 以GET方式拉取网页

```python
import urllib2

response = urllib2.urlopen("http://cn.bing.com")
page = response.read()
print(page)
```

### 添加超时

```python
response = urllib2.urlopen("http://cn.bing.com", timeout=5)  # 5秒后超时
```



### POST

```python
data = {'name': 'Bob'}
data = urllib.urlencode(data)
req = urllib2.Request(url, data)
resp = urllib2.urlopen(req).read()
```

### 设置header

```python
# 调用 map.setdefault() 方法设置, 只在 key 不存在时才会设置.
req = urllib2.Request(url, data)
req.headers.setdefault('Content-Type', 'text/xml; charset=utf-8')

# 或者使用字典作为 header
headers = {'User-Agent': '...'}
req = urllib2.Request(url, data, headers)

# 使用 req.add_header() 设置
req = urllib2.Request(url, data)
req.add_header('Referer', '...')
```



# httplib

## 访问https网页

```python
import httplib

conn = httplib.HTTPSConnection("www.baidu.com")
conn.request("GET", "/")
resp = conn.getresponse()
print resp.status, resp.reason, resp.read()
```


# urlparse

解析url:

```py
>>> from urlparse import urlparse
>>> o = urlparse('http://www.cwi.nl:80/%7Eguido/Python.html')
>>> o   
ParseResult(scheme='http', netloc='www.cwi.nl:80', path='/%7Eguido/Python.html',
            params='', query='', fragment='')
```


# 参考
* [urllib与urllib2的学习总结(python2.7.X)](http://www.cnblogs.com/wly923/archive/2013/05/07/3057122.html)