# urllib2

## 以GET方式拉取网页

```python
import urllib2

response = urllib2.urlopen("http://cn.bing.com")
page = response.read()
print(page)
```

## 添加超时

```python
response = urllib2.urlopen("http://cn.bing.com", timeout=5)  # 5秒后超时
```



## POST

```python
data = urllib.urlencode(data)
req = urllib2.Request(url, data)
resp = urllib2.urlopen(req).read()
```

### 设置header

```python
req = urllib2.Request(url, data)
req.headers.setdefault('Content-Type', 'text/xml; charset=utf-8')
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

