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
