<!-- TOC -->

- [Web基础](#web基础)
    - [搭建一个简单的Web服务器](#搭建一个简单的web服务器)
    - [自定义路由](#自定义路由)
    - [静态文件](#静态文件)
    - [执行流程](#执行流程)
    - [重定向](#重定向)
    - [not found](#not-found)
    - [代理服务](#代理服务)
- [表单](#表单)
    - [处理表单的输入](#处理表单的输入)
    - [验证表单](#验证表单)
        - [必填字段](#必填字段)
        - [数字](#数字)
        - [中文](#中文)
        - [英文](#英文)
        - [电子邮件](#电子邮件)
        - [手机号码](#手机号码)
- [模板](#模板)
    - [获取模板](#获取模板)
        - [gtpl模板文件](#gtpl模板文件)
    - [转义](#转义)
- [文件上传](#文件上传)
    - [表单上传文件](#表单上传文件)
    - [客户端上传文件](#客户端上传文件)
- [文件下载](#文件下载)
- [Session和Cookie](#session和cookie)
    - [设置和读取Cookie](#设置和读取cookie)
    - [Session](#session)

<!-- /TOC -->

# Web基础

## 搭建一个简单的Web服务器

```go
import (
	"fmt"
	"log"
	"net/http"
	"strings"
)

func Main() {
	http.HandleFunc("/", sayHello)           // 设置访问的路由
	err := http.ListenAndServe(":9090", nil) //设置监听的端口
	if err != nil {
		log.Fatal("listenAndServe Err:", err)
	}
}

func sayHello(w http.ResponseWriter, r *http.Request) {
	r.ParseForm() //解析参数，默认是不会解析的
	fmt.Println(r.Form)
	fmt.Println("url_long", r.Form["url_long"])

	fmt.Println("path:", r.URL.Path)
	fmt.Println("scheme:", r.URL.Scheme)

	for k, v := range r.Form {
		s := "key:" + k + ", "
		s += "value:" + strings.Join(v, ",")
		fmt.Println(s)
	}

	fmt.Fprintf(w, "hello")
}
```

## 自定义路由

```go
import (
	"fmt"
	"net/http"
)

type MyMux struct {
}

func (p *MyMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	if r.URL.Path == "/" {
		sayHello(w, r)
		return
	}
	http.NotFound(w, r)
	return
}

func sayHello(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "hello")
}

func Main() {
	mux := &MyMux{}
	http.ListenAndServe(":9090", mux)
}
```

## 静态文件

注册：

```go
http.Handle("/tpl/css/", http.FileServer(http.Dir("./")))
```

请求时处理：

```go
http.ServeFile()
```

Or:

```go
fileHandler := http.FileServer(http.Dir("static/websocket"))
http.Handle("/wsfile/", http.StripPrefix("/wsfile/", fileHandler))
```

访问时使用 `http://localhost:9090/wsfile/ws.html`


Or:

```go
func StaticServer(addr string, paths map[string]string) {
	servers := make(map[string]http.Handler, len(paths))
	for pattern, path := range paths {
		servers[pattern] = http.FileServer(http.Dir(path))
	}

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		urlPath := r.URL.Path
		for pattern, handler := range servers {
			if strings.HasPrefix(urlPath, pattern) {
				http.StripPrefix(pattern, handler).ServeHTTP(w, r)
				return
			}
		}
		http.NotFound(w, r)
	})
	log.Fatal(http.ListenAndServe(addr, nil))
}
```



## 执行流程

通过对http包的分析之后，现在让我们来梳理一下整个的代码执行过程。

首先调用Http.HandleFunc

按顺序做了几件事：

1. 调用了DefaultServeMux的HandleFunc
2. 调用了DefaultServeMux的Handle
3. 往DefaultServeMux的map[string]muxEntry中增加对应的handler和路由规则

其次调用http.ListenAndServe(":9090", nil)

按顺序做了几件事情：

1. 实例化Server
2. 调用Server的ListenAndServe()
3. 调用net.Listen("tcp", addr)监听端口
4. 启动一个for循环，在循环体中Accept请求
5. 对每个请求实例化一个Conn，并且开启一个goroutine为这个请求进行服务go c.serve()
6. 读取每个请求的内容w, err := c.readRequest()
7. 判断handler是否为空，如果没有设置handler（这个例子就没有设置handler），handler就设置为DefaultServeMux
8. 调用handler的ServeHttp
9. 在这个例子中，下面就进入到DefaultServeMux.ServeHttp
10. 根据request选择handler，并且进入到这个handler的ServeHTTP
  mux.handler(r).ServeHTTP(w, r)
11. 选择handler：
- A 判断是否有路由能满足这个request（循环遍历ServerMux的muxEntry）
- B 如果有路由满足，调用这个路由handler的ServeHttp
   - C 如果没有路由满足，调用NotFoundHandler的ServeHttp

## 重定向

	http.Redirect(w, r, "/", 302)

302 可换成 `http.StatusFound`

## not found

	http.NotFount(w, r)

## 代理服务

```go
import (
	"fmt"
	"log"
	"net/http"
	"net/http/httputil"
	"strings"
	"time"
)

func main() {
	server := http.Server{
		Addr:           ":8080",
		Handler:        NewReverseProxyHandler(),
		ReadTimeout:    5 * time.Second,
		WriteTimeout:   120 * time.Second,
		MaxHeaderBytes: 1 << 20,
	}
	log.Fatal(server.ListenAndServe())
}

type ReverseProxyHandler struct {
	innerProxy *httputil.ReverseProxy
}

func NewReverseProxyHandler() *ReverseProxyHandler {
	innerProxy := &httputil.ReverseProxy{
		Director: redirectRequest,
	}
	return &ReverseProxyHandler{
		innerProxy: innerProxy,
	}
}

func (handler *ReverseProxyHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	path := r.URL.Path
	if !strings.HasPrefix(path, "/index") {
		w.WriteHeader(http.StatusBadRequest)
		w.Write([]byte("bad request"))
		return
	}
	fmt.Println(path)
	handler.innerProxy.ServeHTTP(w, r)
}

func redirectRequest(r *http.Request) {
	u := r.URL
	u.Host = "127.0.0.1:9099"
	u.Scheme = "http"
}
```

# 表单

## 处理表单的输入

在src/web/form下新建一个login.gtpl

```html
<html>
<head>
<title></title>
</head>
<body>
<form action="/login" method="post">
    用户名:<input type="text" name="username">
    密码:<input type="password" name="password">
    <input type="submit" value="登陆">
</form>
</body>
</html>
```

编写处理逻辑

```go
package form

import (
	"fmt"
	"html/template"
	"log"
	"net/http"
)

func login(w http.ResponseWriter, r *http.Request) {
	fmt.Println("method:", r.Method)
	if r.Method == "GET" {
		t, err := template.ParseFiles("form/login.gtpl")
		if err != nil {
			fmt.Println("parseFiles:", err)
			log.Fatal("parseFiles:", err)
			return
		}
		fmt.Println(t.Name())
		t.Execute(w, nil)
	} else {
		r.ParseForm()
		fmt.Println("username:", r.Form["username"])
		fmt.Println("password", r.Form["password"])
	}
}

func Main() {
	http.HandleFunc("/login", login)
	err := http.ListenAndServe(":9090", nil)
	if err != nil {
		log.Fatal("listenAndServe:", err)
	}
}
```

在浏览器访问http://localhost:9090/login


## 验证表单

### 必填字段

`r.Form[“name”]`获取的是数组

`r.Form.Get(“name”)`如果不存在则返回空字符串，但只能返回第一个name的值

### 数字

```go
	getint, err := strconv.Atoi(r.Form.Get("age"))
	if err != nil {
		//数字转化出错了，那么可能就不是数字
	}
```

或者使用正则表达式：

```go
if m, _ := regexp.MatchString("^[0-9]+$", r.Form.Get("age")); !m {
    return false
}
```

### 中文

```go
if m, _ := regexp.MatchString("^\\p{Han}+$", r.Form.Get("realname")); !m {
    return false
}
```

### 英文

```go
if m, _ := regexp.MatchString("^[a-zA-Z]+$", r.Form.Get("engname")); !m {
    return false
}
```

### 电子邮件

```go
if m, _ := regexp.MatchString(`^([\w\.\_]{2,10})@(\w{1,}).([a-z]{2,4})$`, r.Form.Get("email")); !m {
    fmt.Println("no")
}else{
    fmt.Println("yes")
}
```

### 手机号码

```go
if m, _ := regexp.MatchString(`^(1[3|4|5|8][0-9]\d{4,8})$`, r.Form.Get("mobile")); !m {
    return false
}
```

# 模板

## 获取模板

### gtpl模板文件

		t, err := template.ParseFiles("form/login.gtpl") // 相对于运行的exe程序
		if err != nil {
			log.Fatal("parseFiles:", err)
			return
		}
		t.Execute(w, nil)



## 转义

    func HTMLEscape(w io.Writer, b []byte) //把b进行转义之后写到w
    func HTMLEscapeString(s string) string //转义s之后返回结果字符串
    func HTMLEscaper(args ...interface{}) string //支持多个参数一起转义，返回结果字符串

示例：

```go
import (
	"fmt"
	"html/template"
	"net/http"
)

func Main() {
	http.HandleFunc("/", escapeTest)
	err := http.ListenAndServe(":9090", nil)
	if err != nil {
		fmt.Println("ListenAndServe Err:", err)
	}
}

func escapeTest(w http.ResponseWriter, r *http.Request) {
	text := template.HTMLEscapeString("<script>alert('aaa');</script>")
	// 输出到客户端
	w.Write([]byte(text))
	// 或者
	//template.HTMLEscape(w, []byte("<script>alert('aaa');</script>"))
}
```

输出为：

```
&lt;script&gt;alert(&#39;aaa&#39;);&lt;/script&gt;
```

	t, _ := template.New("foo").Parse(`{{define "T"}}hello,{{.}}!{{end}}`)
	t.ExecuteTemplate(w, "T", "<script>alert('aaa');</script>")

输出为：

```
hello,&lt;script&gt;alert(&#39;aaa&#39;);&lt;/script&gt;!
```

但如果使用template.HTML进行强转后则：

```
t.ExecuteTemplate(w, "T", template.HTML("<script>alert('aaa');</script>"))
```

输出为：

```
hello,<script>alert('aaa');</script>!
```

```
template.HTMLEscape(w, []byte("<script>alert('aaa');</script>"))
```

这种方式得到的结果是转义了的。

但是下面得到的却是弹窗：

```
w.Write([]byte(template.HTML("<script>alert('aaa');</script>")))
```

# 文件上传
## 表单上传文件

```
package upload

import (
	"fmt"
	"html/template"
	"io"
	"net/http"
	"os"
)

func Main() {
	http.HandleFunc("/", index)
	http.HandleFunc("/upload", upload)
	err := http.ListenAndServe(":9090", nil)
	if err != nil {
		fmt.Println("ListenAndServe Err:", err)
	}
}

func index(w http.ResponseWriter, r *http.Request) {
	t, err := template.ParseFiles("upload/upload.gtpl")
	if err != nil {
		fmt.Println("index() Err:", err)
		return
	}
	t.Execute(w, nil)
}

func upload(w http.ResponseWriter, r *http.Request) {
	if r.Method != "POST" {
		return
	}
	r.ParseMultipartForm(32 << 20) // 32MB
	// 获得上传文件的句柄
	// FormFile会自动调用ParseMultipartForm(),默认内存32MB
	file, header, err := r.FormFile("uploadfile") // uploadfile是<input>的name
	if err != nil {
		fmt.Println("upload() Err:", err)
		return
	}
	defer file.Close()

	// 输出header
	fmt.Fprintf(w, "%v", header.Header)

	// 转储文件[upload目录必须存在]
	f, err := os.OpenFile("./upload/"+header.Filename, os.O_WRONLY|os.O_CREATE, 0666)
	if err != nil {
		fmt.Println("os.OpenFile() Err:", err)
		return
	}
	defer f.Close()
	io.Copy(f, file)
}
```

## 客户端上传文件

```go
package upload

import (
	"bytes"
	"fmt"
	"io"
	"io/ioutil"
	"mime/multipart"
	"net/http"
	"os"
)

func postFile(filename, url string) error {
	bodyBuf := &bytes.Buffer{}
	bodyWriter := multipart.NewWriter(bodyBuf)

	// 关键的一步
	fileWriter, err := bodyWriter.CreateFormFile("uploadfile", filename)
	if err != nil {
		fmt.Println("error writing to buffer")
		return err
	}

	//打开文件句柄操作
	fh, err := os.Open(filename)
	if err != nil {
		fmt.Println("error opening file")
		return err
	}
	defer fh.Close()

	//iocopy
	_, err = io.Copy(fileWriter, fh)
	if err != nil {
		return err
	}

	contentType := bodyWriter.FormDataContentType()
	bodyWriter.Close()

	resp, err := http.Post(url, contentType, bodyBuf)
	if err != nil {
		return err
	}

	defer resp.Body.Close()
	resp_body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		return err
	}

	fmt.Println(resp.Status)
	fmt.Println(string(resp_body))
	return nil
}

func ClientPostFile() {
	target_url := "http://localhost:9090/upload"
	filename := "./魔方公式.txt"
	postFile(filename, target_url)
}
```

# 文件下载

客户端：

```js
window.open("/admin/account/download/" + time[0] + "/" + time[1], "_blank");
```

服务端：

```go
c.Header("Content-Disposition", `attachment;filename="`+url.QueryEscape(fileName)+`"`)
c.File("recordStatisFiles/" + fileName)
```

Content-Type在ServerFile中会自动添加。

# Session和Cookie
## 设置和读取Cookie
```go
import (
	"log"
	"net/http"
	"time"
)

func Main() {
	http.HandleFunc("/w", writeCookie)
	http.HandleFunc("/r", readCookie)
	err := http.ListenAndServe(":9090", nil)
	if err != nil {
		log.Fatal("listener:", err)
	}
}

func writeCookie(w http.ResponseWriter, r *http.Request) {
	expiration := time.Now()
	expiration = expiration.AddDate(0, 0, 1)
	cookie := http.Cookie{Name: "name", Value: "chen", Expires: expiration}
	http.SetCookie(w, &cookie)
}

func readCookie(w http.ResponseWriter, r *http.Request) {
	cookies := r.Cookies()
	for _, cookie := range cookies {
		w.Write([]byte(cookie.String() + "\n"))
	}
	cookie, _ := r.Cookie("name")
	w.Write([]byte("name的cookie:" + cookie.String()))
}
```
## Session
- 目前Go标准包没有为session提供支持




