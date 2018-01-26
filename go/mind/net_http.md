<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [client](#client)
- [Transport](#transport)
	- [readLoop](#readloop)
	- [writeLoop](#writeloop)
	- [putOrCloseIdleConn](#putorcloseidleconn)

<!-- /TOC -->

go version: 1.9.2

# client


# Transport

Transport 最主要的函数就是 `RoundTrip()`, 它接受一个 `*Request` 返回一个 `*Response`。

下面的流程只描述 http 相关的逻辑。

- 首先如果 `req.URL.Scheme` 是 `http` 或 `https`，会先检查 `req.Header` 是否合法(`httplex.ValidHeaderFieldName()`, `httplex.ValidHeaderFieldValue()`)。
- 根据 `req.URL.Scheme` 从 altProto(`map[string]RoundTripper`) 中获取 RoundTripper, 如果有就调用其 `RoundTrip()`。
- 下面的逻辑放到一个 `for` 无限循环中:
    - 首先构建一个 `transportRequest` 和 `connectMethod`。
    - 根据上面构建的参数来获取连接 persistConn:
        - 创建一个 goroutine 调用 `dialConn()` 或创建连接。 dialConn 返回一个持久连接 `persistConn`。
            - 调用 `dial()` 来获得一个 `net.Conn`，这里有两种实现方法，一种是 `DialContext`, 一种是 `Dial`。`pconn.conn` 保存的就是这个。 TODO
            - 在 pconn 上启动两个 goroutine, 一个读循环，一个写循环。
        - 如果上面的 goroutine 及时返回了一个 persistConn，那就使用这个连接，否则：
        - 通过 `<-idleConnCh` 获取了一个空闲连接, 那么就使用这个空闲连接，但是上面的 goroutine 还在跑着, 那么就得再启动一个 goroutine，等到上面的 goroutine 返回了连接，就将返回的连接放到空闲队列中(参考 #putOrCloseIdleConn)。
        - 如果请求被取消或者 ctx.Done() 了，直接返回，并且开启一个协程调用 `putOrCloseIdleConn()`。
    - 调用 `pconn.roundTrip()` 来获取 resp:
        - 是否需要 gzip(没有禁用(`transport.DisableCompression`) && `Accept-Encoding, Range` 为空 && 不是 HEAD), 需要 gzip, 设置 `Accept-Encoding = gzip`。
        - 如果设置了 `transport.DisableKeepAlives == true`，设置 `Connection = close`。
        - 生成一个 `writeRequest`(带着 transportRequest) 放入 `pc.writech`, 写循环会处理它(#writeLoop)。
        - 生成一个 `requestAndChan` 放入 `persistConn.reqch` 中，读循环会处理它(#readLoop)。
        - 之后就是等待响应或者错误了。
    - 如果出错了，但是需要重试，则构建一个新的 req, 重新走这个 for 内的逻辑。


总体来讲就是：一个连接上有两个 goroutine 跑循环，读循环和写循环，当我们构建好一个 Request 之后，写循环会将请求写到连接中，然后读请求会从连接中读取到响应。


## readLoop

- 从 `persistConn.reqch` 中获取一个 `requestAndChan`, 交给 `pc.readResponse(rc, trace)`:
    - 调用 `resp, err = ReadResponse(pc.br, rc.req)` 来获取 resp:
        - 先读取第一行，解析
        - 读取 Header.
        - `readTransfer`: body, content-length, transfer-encoding, trailer, close
- 上面只读取了 header, 下面会读取 body, 首先判断是否有 body: 不是 HEAD 并且 ContentLength != 0.
- 如果没有 body, 就将 resp 放入 `requestAndChan.ch` 中，结束。
- 有 body 的话，并不立即就读取 body，而是用 `bodyEOFSignal` 保证了 resp.Body, 如果 `Content-Encoding == gzip`, 还用 gzipReader 再包装一层。bodyEOFSignal 的作用是决定还要不要从这个连接中读取下一个请求。首先将 responseAndError 放入 `requestAndChan.ch` 后，会监听 body 关闭的情况，如果 body 被读取完了(EOF) && 写入操作没有错误(50ms内) && 成功放入空闲队列中，那么就会继续监听下一个请求。否则，除了上面说的情况之外，像请求被 cancel, ctx.Done() 这样的就会导致该连接被关闭。


## writeLoop

- 从 `persistConn.writech` 中读取 writeRequest(wr), 调用 `wr.req.Request.write()` 将请求的 header 和 body 写入到连接的 bw(buffer writer) 中。



## putOrCloseIdleConn
tag: putOrCloseIdleConn

putOrCloseIdleConn 的作用是将一个 persistConn 放到空闲连接队列中。

- 如果 transport 禁用了 keepAlive(`DisableKeepAlives == true`) 或者 每个 host 最大空闲连接小于0(`MaxIdleConnsPerHost < 0`)，那么这个连接就不放到空闲队列中了。
- 如果连接断开了(`pconn.isBroken() == true`), 也丢掉这个连接。
- 将这个连接的 `reused` 字段置为 true。
- 尝试着将连接放入 `idleConnCh[key]` 中，key 就是 `connectMethod.key()`, 能放进去说明有其他请求正在等着连接。放不进去说明没有其他请求在等着，就 `delete(t.idleConnCh, key)`。
- TODO: transport.wantIdle ????
- 放入空闲队列 `transport.idleConn[key]`, 值是一个 `[]*persistConn`, 表示每个 host 上的持久连接，如果超过最大连接数(`MaxIdleConnsPerHost`, 默认2个)，就丢弃连接。
- 把连接也放入 `transport.idleLRU` 中, 如果 lru 中的连接数超过 `transport.MaxIdleConns`(如果为0则没有限制)，就删除最久的连接(从 lru 中删除，也从 idleConn 中删除)。
- 如果配置了 `transport.IdleConnTimeout`, 则给这个 persistConn 设置一个 timer, 时间到了会执行 `pconn.closeConnIfStillIdle`, 这个函数先检查是否是空闲的(是否在 idleLRU 中), 如果空闲就从 `idleLRU` 和 `idleConn` 中删除, 并把这个连接关闭。
