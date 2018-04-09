<!-- TOC -->

- [HTTP客户端](#http客户端)
    - [基本方法](#基本方法)
    - [http.Get()](#httpget)
    - [http.Post()](#httppost)
    - [http.PostForm()](#httppostform)
    - [http.Head()](#httphead)
    - [(*http.Client).Do()](#httpclientdo)
    - [高级封装](#高级封装)
        - [自定义http.Client](#自定义httpclient)
        - [自定义http.Transport](#自定义httptransport)
        - [灵活的http.RoundTripper 接口](#灵活的httproundtripper-接口)
        - [设计优雅的HTTP Client](#设计优雅的http-client)
- [HTTP服务端](#http服务端)
    - [处理HTTP请求](#处理http请求)
    - [处理HTTPS请求](#处理https请求)
- [测试](#测试)
- [HTTPS](#https)
    - [服务端](#服务端)
    - [客户端](#客户端)
        - [客户端跳过证书验证](#客户端跳过证书验证)

<!-- /TOC -->


# HTTP客户端

## 基本方法

net/http包的Client类型提供了如下几个方法，让我们可以用最简洁的方式实现HTTP请求：

```go
func(c *Client) Get(url string) (r *Response, err error)
func(c *Client) Post(url string, bodyType string, body io.Reader) (r *Response, err error)
func(c *Client) PostForm(url string, data url.Values) (r *Response, err error)
func(c *Client) Head(url string) (r *Response, err error)
func(c *Client) Do(req *Request) (resp *Response, err error)
```


## http.Get()

等价于http.DefaultClient.Get()

示例【将网页内容打印到输出流中】：

```go
	import (
		"fmt"
		"io"
		"net/http"
		"os"
	)
	resp, err := http.Get("http://www.baidu.com") // 需要带http协议名
	if err != nil {
		fmt.Println("E:", err)
		return
	}
	defer resp.Body.Close()
	io.Copy(os.Stdout, resp.Body) // 输出到控制台
```

如果不想要内容只想要内容长度，可以使用：

```go
nbytes, err := io.Copy(ioutil.Discard, resp.Body)
```


## http.Post()


Post()需要以下3个参数：

1. 请求的目标URL
2. 将要POST 数据的资源类型（MIMEType）
3. 数据的比特流（[]byte形式）

下面的示例代码演示了如何上传一张图片：

```go
resp, err := http.Post("http://example.com/upload", "image/jpeg", &imageDataBuf)
if err != nil{
	// 处理错误
	return
}
if resp.StatusCode != http.StatusOK {
	// 处理错误
	return
}
// ...
```


## http.PostForm()

http.PostForm()方法实现了标准编码格式为application/x-www-form-urlencoded的表单提交。

下面的示例代码模拟HTML表单提交一篇新文章：

```go
resp, err := http.PostForm("http://example.com/posts", url.Values{"title": {"article title"}, "content": {"article body"}})
if err != nil{
	// 处理错误
	return
}
// ...
```


## http.Head()

HTTP 中的Head 请求方式表明只请求目标URL 的头部信息，即HTTP Header, 而不返回HTTP Body

```go
resp, err := http.Head("http://www.baidu.com")
if err != nil {
	fmt.Println("E:", err)
	return
}
fmt.Println(resp.Header)
```

## http.Client.Do()

在多数情况下，http.Get()和http.PostForm() 就可以满足需求，但是如果我们发起的HTTP 请求需要更多的定制信息，我们希望设定一些自定义的Http Header 字段，比如：

1. 设定自定义的"User-Agent"，而不是默认的"Go http package"
2. 传递Cookie

```go
request, err := http.NewRequest("GET", "http://www.baidu.com", nil)
request.Header.Add("User-Agent", "default")
request.Host = "www.baidu.com" // 自定义 Host 只能这样设置
client := &http.Client{}
resp, err := client.Do(request)
```

## 高级封装

### 自定义http.Client

前面我们使用的http.Get()、http.Post()、http.PostForm()和http.Head()方法其实都是在http.DefaultClient的基础上进行调用的，比如http.Get()等价于http.DefaultClient.Get()，依次类推

让我们来看一看http.Client类型的结构：

```go
type Client struct{
	// Transport用于确定HTTP请求的创建机制。
	// 如果为空，将会使用DefaultTransport
	Transport RoundTripper

	// CheckRedirect定义重定向策略。
	// 如果CheckRedirect不为空，客户端将在跟踪HTTP重定向前调用该函数。
	// 两个参数req和via分别为即将发起的请求和已经发起的所有请求，最早的
	// 已发起请求在最前面。
	// 如果CheckRedirect返回错误，客户端将直接返回错误，不会再发起该请求。
	// 如果CheckRedirect为空，Client将采用一种确认策略，将在10个连续
	// 请求后终止
	CheckRedirect func(req *Request, via []*Request) error

	// 如果Jar为空，Cookie将不会在请求中发送，并会
	// 在响应中被忽略
	Jar CookieJar
}
```

在Go语言标准库中，http.Client类型包含了3个公开数据成员：

1. `Transport RoundTripper `
2. `CheckRedirect func(req *Request, via []*Request) error `
3. `Jar CookieJar `

其中Transport类型必须实现http.RoundTripper接口。Transport指定了执行一个HTTP 请求的运行机制，倘若不指定具体的Transport，默认会使用http.DefaultTransport，这意味着http.Transport也是可以自定义的。net/http包中的http.Transport类型实现了http.RoundTripper接口

CheckRedirect函数指定处理重定向的策略。当使用HTTP Client 的Get()或者是Head()方法发送HTTP 请求时，若响应返回的状态码为30x （比如301 / 302 / 303 / 307），HTTP Client 会在遵循跳转规则之前先调用这个CheckRedirect函数。

Jar可用于在HTTP Client 中设定Cookie，Jar的类型必须实现了 http.CookieJar 接口，该接口预定义了 SetCookies()和Cookies()两个方法。如果HTTP Client 中没有设定 Jar，Cookie将被忽略而不会发送到客户端。实际上，我们一般都用 http.SetCookie() 方法来设定Cookie。

使用自定义的http.Client及其Do()方法，我们可以非常灵活地控制HTTP 请求，比如发送自定义HTTP Header 或是改写重定向策略等。创建自定义的HTTP Client 非常简单，具体代码如下：

```go
client := &http.Client {
	CheckRedirect: redirectPolicyFunc,
}
resp, err := client.Get("http://example.com")
// ...
req, err := http.NewRequest("GET", "http://example.com", nil)
// ...
req.Header.Add("User-Agent", "Our Custom User-Agent")
req.Header.Add("If-None-Match", `W/"TheFileEtag"`)
resp, err := client.Do(req)
// ...
```

### 自定义http.Transport

在http.Client 类型的结构定义中，我们看到的第一个数据成员就是一个 http.Transport对象，该对象指定执行一个HTTP 请求时的运行规则。下面我们来看看 http.Transport 类型的具体结构：

```go
type Transport struct{
	// Proxy指定用于针对特定请求返回代理的函数。
	// 如果该函数返回一个非空的错误，请求将终止并返回该错误。
	// 如果Proxy为空或者返回一个空的URL指针，将不使用代理
	Proxy func(*Request) (*url.URL, error)

	// Dial指定用于创建TCP连接的dail()函数。
	// 如果Dial为空，将默认使用net.Dial()函数
	Dial func(net, addr string) (c net.Conn, err error)

	// TLSClientConfig指定用于tls.Client的TLS配置。
	// 如果为空则使用默认配置
	TLSClientConfig *tls.Config
	DisableKeepAlives  bool
	DisableCompression bool

	// 如果MaxIdleConnsPerHost为非零值，它用于控制每个host所需要
	// 保持的最大空闲连接数。如果该值为空，则使用	DefaultMaxIdleConnsPerHost
	MaxIdleConnsPerHost int
	// ...
}
```

在上面的代码中，我们定义了http.Transport 类型中的公开数据成员，下面详细说明其中的各行代码。

```go
Proxy func(*Request) (*url.URL, error)
```

Proxy 指定了一个代理方法，该方法接受一个`*Request` 类型的请求实例作为参数并返回一个最终的HTTP 代理。如果 Proxy 未指定或者返回的*URL 为零值，将不会有代理被启用。

```go
Dial func(net, addr string) (c net.Conn, err error)
```

Dial 指定具体的dial()方法来创建TCP 连接。如果不指定，默认将使用 net.Dial() 方法。

```go
TLSClientConfig *tls.Config
```

SSL连接专用，TLSClientConfig 指定tls.Client 所用的TLS 配置信息，如果不指定，也会使用默认的配置。

```go
DisableKeepAlives bool
```

是否取消长连接，默认值为false，即启用长连接。

```go
DisableCompression bool
```

是否取消压缩（GZip），默认值为 false，即启用压缩。

```go
MaxIdleConnsPerHost int
```

指定与每个请求的目标主机之间的最大非活跃连接（keep-alive）数量。如果不指定，默认使用 DefaultMaxIdleConnsPerHost 的常量值。

除了 http.Transport 类型中定义的公开数据成员以外，它同时还提供了几个公开的成员方法。

```go
func(t *Transport) CloseIdleConnections()
```

该方法用于关闭所有非活跃的连接。

```
func(t *Transport) RegisterProtocol(scheme string, rt RoundTripper)。
```

该方法可用于注册并启用一个新的传输协议，比如WebSocket 的传输协议标准（ws），或者FTP、File 协议等。

```go
func(t *Transport) RoundTrip(req *Request) (resp *Response, err error)。
```

用于实现 http.RoundTripper 接口。

自定义http.Transport也很简单，如下列代码所示：

```go
tr := &http.Transport{
	TLSClientConfig: &tls.Config{RootCAs: pool},
	DisableCompression: true,
}
client := &http.Client{Transport: tr}
resp, err := client.Get("https://example.com")
```

Client和Transport在执行多个goroutine 的并发过程中都是安全的，但出于性能考虑，应当创建一次后反复使用。


### 灵活的http.RoundTripper 接口

在前面的两小节中，我们知道HTTP Client 是可以自定义的，而 http.Client 定义的第一个公开成员就是一个  http.Transport  类型的实例，且该成员所对应的类型必须实现http.RoundTripper 接口。下面我们来看看http.RoundTripper 接口的具体定义

```go
typeRoundTripper interface{
	// RoundTrip执行一个单一的HTTP事务，返回相应的响应信息。
	// RoundTrip函数的实现不应试图去理解响应的内容。如果RoundTrip得到一个响应，
	// 无论该响应的HTTP状态码如何，都应将返回的err设置为nil。非空的err
	// 只意味着没有成功获取到响应。
	// 类似地，RoundTrip也不应试图处理更高级别的协议，比如重定向、认证和
	// Cookie等。
	//
	// RoundTrip不应修改请求内容, 除非了是为了理解Body内容。每一个请求
	// 的URL和Header域都应被正确初始化
	RoundTrip(*Request) (*Response, error)
}
```

从上述代码中可以看到，http.RoundTripper接口很简单，只定义了一个名为RoundTrip的方法。任何实现了 RoundTrip() 方法的类型即可实现http.RoundTripper接口。前面我们看到的http.Transport类型正是实现了 RoundTrip() 方法继而实现了该接口。

http.RoundTripper 接口定义的 RoundTrip() 方法用于执行一个独立的HTTP 事务，接受传入的`*Request` 请求值作为参数并返回对应的`*Response` 响应值，以及一个 error 值。在实现具体的 RoundTrip() 方法时，不应该试图在该函数里边解析HTTP 响应信息。若响应成功，error 的值必须为nil，而与返回的HTTP 状态码无关。若不能成功得到服务端的响应，error必须为非零值。类似地，也不应该试图在 RoundTrip() 中处理协议层面的相关细节，比如重定向、认证或是cookie 等。

非必要情况下，不应该在 RoundTrip() 中改写传入的请求体（\*Request），请求体的内容（比如URL 和Header 等）必须在传入 RoundTrip() 之前就已组织好并完成初始化。通常，我们可以在默认的 http.Transport 之上包一层 Transport 并实现 RoundTrip()方法：

```go
package main
import(
	"net/http"
)
type OurCustomTransport struct{
	Transport http.RoundTripper
}
func(t *OurCustomTransport) transport() http.RoundTripper {
	if t.Transport != nil{
		return t.Transport
	}
	return http.DefaultTransport
}
func(t *OurCustomTransport) RoundTrip(req *http.Request) (*http.Response, error) {
	// 处理一些事情...
	// 发起HTTP请求
	// 添加一些域到req.Header中
	return t.transport().RoundTrip(req)
}
func(t *OurCustomTransport) Client() *http.Client {
	return &http.Client{Transport: t}
}
func main() {
	t := &OurCustomTransport{
	//...
	}
	c := t.Client()
	resp, err := c.Get("http://example.com")
	// ...
}
```

因为实现了http.RoundTripper 接口的代码通常需要在多个goroutine中并发执行，因此我们必须确保实现代码的线程安全性


### 设计优雅的HTTP Client

综上示例讲解可以看到，Go语言标准库提供的HTTP Client 是相当优雅的。一方面提供了极其简单的使用方式，另一方面又具备极大的灵活性。

Go语言标准库提供的HTTP Client 被设计成上下两层结构。一层是上述提到的 http.Client类及其封装的基础方法，我们不妨将其称为“业务层”。之所以称为业务层，是因为调用方通常只需要关心请求的业务逻辑本身，而无需关心非业务相关的技术细节，这些细节包括：

- HTTP 底层传输细节
- HTTP 代理
- gzip 压缩
- 连接池及其管理
- 认证（SSL或其他认证方式）

之所以HTTP Client可以做到这么好的封装性，是因为HTTP Client 在底层抽象了http.RoundTripper 接口，而http.Transport 实现了该接口，从而能够处理更多的细节，我们不妨将其称为“传输层”。HTTP Client 在业务层初始化HTTP Method、目标URL、请求参数、请求内容等重要信息后，经过“传输层”，“传输层”在业务层处理的基础上补充其他细节，然后再发起HTTP 请求，接收服务端返回的HTTP 响应。


# HTTP服务端

## 处理HTTP请求

使用net/http 包提供的 http.ListenAndServe() 方法，可以在指定的地址进行监听，开启一个HTTP，服务端该方法的原型如下

	func ListenAndServe(addr string, handler Handler) error

该方法用于在指定的TCP 网络地址addr 进行监听，然后调用服务端处理程序来处理传入的连接请求。该方法有两个参数：第一个参数addr 即监听地址；第二个参数表示服务端处理程序，通常为空，这意味着服务端调用 http.DefaultServeMux 进行处理，而服务端编写的业务逻辑处理程序 http.Handle() 或 http.HandleFunc() 默认注入 http.DefaultServeMux 中，具体代码如下：

```go
http.Handle("/foo", fooHandler)
http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})
http.ListenAndServe(":8080", nil)
```

当访问localhost:8080/bar时，会返回Hello,”/bar”

如果想更多地控制服务端的行为，可以自定义 http.Server，代码如下：

```go
s := &http.Server{
	Addr: ":8080",
	Handler: myHandler,
	ReadTimeout: 10 * time.Second,
	WriteTimeout: 10 * time.Second,
	MaxHeaderBytes: 1 << 20,
}
s.ListenAndServe()
```


## 处理HTTPS请求

net/http 包还提供 http.ListenAndServeTLS() 方法，用于处理HTTPS 连接请求：

```go
func ListenAndServeTLS(addr string, certFile string, keyFile string, handler Handler)  error
```

ListenAndServeTLS() 和 ListenAndServe()的行为一致，区别在于只处理HTTPS请求。

此外，服务器上必须存在包含证书和与之匹配的私钥的相关文件，比如certFile对应SSL证书文件存放路径，keyFile对应证书私钥文件路径。如果证书是由证书颁发机构签署的，certFile参数指定的路径必须是存放在服务器上的经由CA认证过的SSL证书。

开启SSL 监听服务也很简单，如下列代码所示：

```go
http.Handle("/foo", fooHandler)
http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})
log.Fatal(http.ListenAndServeTLS(":10443", "cert.pem", "key.pem", nil))
```

或者是：

```go
ss := &http.Server{
	Addr: ":10443",
	Handler: myHandler,
	ReadTimeout: 10 * time.Second,
	WriteTimeout: 10 * time.Second,
	MaxHeaderBytes: 1 << 20,
}
log.Fatal(ss.ListenAndServeTLS("cert.pem", "key.pem"))
```


# 测试

```go
body := url.Values{}
req, err := http.NewRequest("POST", "/api", bytes.NewBufferString(body.Encode()))
w := httptest.NewRecorder()
router.ServeHTTP(w, req)
data, err := ioutil.ReadAll(w.Body)
```



# HTTPS

## 服务端

```go
http.HandleFunc("/", handler)
http.ListenAndServeTLS(":8081", "server.crt", "server.key", nil)
```


## 客户端

### 客户端跳过证书验证

```go
func main() {
    tr := &http.Transport{
        TLSClientConfig:    &tls.Config{InsecureSkipVerify: true}, // 跳过验证
    }
    client := &http.Client{Transport: tr}
    resp, err := client.Get("https://localhost:8081")

    if err != nil {
        fmt.Println("error:", err)
        return
    }
    defer resp.Body.Close()
    body, err := ioutil.ReadAll(resp.Body)
    fmt.Println(string(body))
}
```
