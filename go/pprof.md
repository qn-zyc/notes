<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [性能分析](#性能分析)
	- [pprof](#pprof)
		- [runtime/pprof](#runtimepprof)
		- [net/http/pprof](#nethttppprof)
- [优化代码](#优化代码)
	- [二进制IP地址](#二进制ip地址)

<!-- /TOC -->

# 性能分析

## pprof

- [文档](https://github.com/google/pprof/blob/master/doc/pprof.md)



### runtime/pprof

如果程序为非 httpserver 类型，使用此方式；在 main 函数中嵌入如下代码:

```go
import "runtime/pprof"

var cpuprofile = flag.String("cpuprofile"， ""， "write cpu profile `file`")
var memprofile = flag.String("memprofile"， ""， "write memory profile to `file`")

func main() {
    flag.Parse()
    if *cpuprofile != "" {
        f， err := os.Create(*cpuprofile)
        if err != nil {
            log.Fatal("could not create CPU profile: "， err)
        }
        if err := pprof.StartCPUProfile(f); err != nil {
            log.Fatal("could not start CPU profile: "， err)
        }
        defer pprof.StopCPUProfile()
    }

    // ... rest of the program ...

    if *memprofile != "" {
        f， err := os.Create(*memprofile)
        if err != nil {
            log.Fatal("could not create memory profile: "， err)
        }
        runtime.GC() // get up-to-date statistics
        if err := pprof.WriteHeapProfile(f); err != nil {
            log.Fatal("could not write memory profile: "， err)
        }
        f.Close()
    }
}
```

运行程序: `./logger -cpuprofile cpu.prof -memprofile mem.prof`, 可以得到 cpu.prof 和 mem.prof 文件，使用 go tool pprof 分析。

```
go tool pprof logger cpu.prof
go tool pprof logger mem.prof
```

### net/http/pprof

如果程序直接使用 http 包下的服务(比如 `http.ListenAndServe()`)， 则只需要导入该包: `import _ "net/http/pprof"`, pprof 包会在 init 函数中将路由注册到 http 包默认的路由中。

如果是自定义的 `http.Server`， 可以使用下面的代码注册路由:

```go
mux := http.NewServeMux()
mux.HandleFunc("/debug/pprof/", pprof.Index)
mux.Handle("/debug/pprof/heap", pprof.Handler("heap"))
mux.Handle("/debug/pprof/goroutine", pprof.Handler("goroutine"))
mux.Handle("/debug/pprof/block", pprof.Handler("block"))
mux.Handle("/debug/pprof/threadcreate", pprof.Handler("threadcreate"))
mux.Handle("/debug/pprof/mutex", pprof.Handler("mutex"))
mux.HandleFunc("/debug/pprof/cmdline", pprof.Cmdline)
mux.HandleFunc("/debug/pprof/profile", pprof.Profile)
mux.HandleFunc("/debug/pprof/symbol", pprof.Symbol)
mux.HandleFunc("/debug/pprof/trace", pprof.Trace)
return &http.Server{
	Addr:    addr,
	Handler: mux,
}
```

如果 httpserver 使用 go-gin 包，而不是使用默认的 http 包启动，则需要手动添加 /debug/pprof 对应的 handler，github 有封装好的模版:

```
import "github.com/DeanThompson/ginpprof"
...
router := gin.Default()
ginpprof.Wrap(router)
...
```

导入包重新编译程序后运行，在浏览器中访问 `http://host:port/debug/pprof` 可以看到如下信息:

```
/debug/pprof/

profiles:
0	block
62	goroutine
427	heap
0	mutex
12	threadcreate

full goroutine stack dump
```

点击对应的 profile 可以查看具体信息，通过浏览器查看的数据不能直观反映程序性能问题，go tool pprof 命令行工具提供了丰富的工具集:

- 查看 heap 信息: `go tool pprof http://127.0.0.1:4500/debug/pprof/heap`
- 查看 30s 的 CPU 采样信息: `go tool pprof http://127.0.0.1:4500/debug/pprof/profile`
- 其他功能使用参见 官方 [net/http/pprof](https://golang.org/pkg/net/http/pprof/) 库


### 内存占用

`http://localhost:9090/debug/pprof/heap?debug=1`

输出示例:

```
heap profile: 3190(inused objects): 77516056(inused bytes) [54762(alloc objects): 612664248(alloc bytes)] @ heap/1048576(2*MemProfileRate)
1: 29081600 [1: 29081600] (前面4个数跟第一行的一样，此行以后是每次记录的，后面的地址是记录中的栈指针)@ 0x89368e 0x894cd9 0x8a5a9d 0x8a9b7c 0x8af578 0x8b4441 0x8b4c6d 0x8b8504 0x8b2bc3 0x45b1c1
...
# runtime.MemStats
# Alloc = 6951912 
# TotalAlloc = 5619604376 
# Sys = 19249400 (进程从系统获得的内存空间，虚拟地址空间)
# Lookups = 3656
# Mallocs = 43503252
# Frees = 43445133
# HeapAlloc = 6951912 (进程堆内存分配使用的空间，通常是用户new出来的堆对象，包含未被gc掉的)
# HeapSys = 12386304 (进程从系统获得的堆内存，因为golang底层使用TCmalloc机制，会缓存一部分堆内存，虚拟地址空间。)
# HeapIdle = 4325376
# HeapInuse = 8060928
# HeapReleased = 0
# HeapObjects = 58119
# Stack = 3276800 / 3276800
# MSpan = 158080 / 229376
# MCache = 6944 / 16384
# BuckHashSys = 1474607
# GCSys = 782336
# OtherSys = 1083593
# NextGC = 7960256
# LastGC = 1524025669886421880
# PauseNs = [89496 119498 ...] (记录每次gc暂停的时间(纳秒)，最多记录256个最新记录。)
# PauseEnd = [1524025666427497151 ...]
# NumGC = 1988 (记录gc发生的次数。)
# NumForcedGC = 0
# GCCPUFraction = -8.074638565876587e-06
# DebugGC = false
```


# 优化代码

## 二进制IP地址

```go
func binaryIP1(ipStr string) string {
	ip, _, err := net.ParseCIDR(ipStr)
	if err != nil {
		ip = net.ParseIP(ipStr)
	}
	if ip4 := ip.To4(); ip4 != nil {
		ip = ip4
	}
	bitIP := make([]byte, len(ip)*8)
	for i, v := range ip {
		copy(bitIP[i*8:], fmt.Sprintf("%08s", strconv.FormatInt(int64(v), 2)))
	}
	return string(bitIP)
}

func binaryIP2(ipStr string) string {
	ip, _, err := net.ParseCIDR(ipStr)
	if err != nil {
		ip = net.ParseIP(ipStr)
	}
	if ip4 := ip.To4(); ip4 != nil {
		ip = ip4
	}
	bitIP := make([]byte, len(ip)*8)
	for i, v := range ip {
        binaryIP := strconv.FormatInt(int64(v), 2)
        if n := len(binaryIP); n >= 8 {
            copy(bitIP[i*8:], binaryIP[:8])
        } else {
            j := i * 8
            for k := 0; k < 8-n; k++ {
                bitIP[j] = '0'
                j++
            }
            copy(bitIP[j:], binaryIP[:n])
        }
	}
	return string(bitIP)
}

func binaryIP3(ipStr string) string {
	ip, _, err := net.ParseCIDR(ipStr)
	if err != nil {
		ip = net.ParseIP(ipStr)
	}
	if ip4 := ip.To4(); ip4 != nil {
		ip = ip4
	}
	bitIP := make([]byte, len(ip)*8)
	for i, v := range ip {
		iv := int(v)
		for j := 0; j < 8; j++ {
			if (1<<uint(7-j))&iv == 0 {
				bitIP[i*8+j] = '0'
			} else {
				bitIP[i*8+j] = '1'
			}
		}
	}
	return string(bitIP)
}
```

运行 benchmark:

```go
func BenchmarkBinaryIP1(b *testing.B) {
	for i := 0; i < b.N; i++ {
		binaryIP1("127.0.0.1")
	}
}

func BenchmarkBinaryIP2(b *testing.B) {
	for i := 0; i < b.N; i++ {
		binaryIP2("127.0.0.1")
	}
}

func BenchmarkBinaryIP3(b *testing.B) {
	for i := 0; i < b.N; i++ {
		binaryIP3("127.0.0.1")
	}
}
```

```
BenchmarkBinaryIP1-4   	 1000000	      1118 ns/op
BenchmarkBinaryIP2-4   	 5000000	       393 ns/op
BenchmarkBinaryIP3-4   	 5000000	       239 ns/op
```
