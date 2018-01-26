<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Worker Pool](#worker-pool)
- [随机数](#随机数)
	- [随机byte数组](#随机byte数组)

<!-- /TOC -->


# Worker Pool

```go
// Here's the worker, of which we'll run several
// concurrent instances. These workers will receive
// work on the `jobs` channel and send the corresponding
// results on `results`. We'll sleep a second per job to
// simulate an expensive task.
func worker(id int, jobs <-chan int, results chan<- int) {
	for j := range jobs {
		fmt.Println("worker", id, "processing job", j)
		time.Sleep(time.Second)
		results <- j * 2
	}
}

func main() {

	// In order to use our pool of workers we need to send
	// them work and collect their results. We make 2
	// channels for this.
	jobs := make(chan int, 100)
	results := make(chan int, 100)

	// This starts up 3 workers, initially blocked
	// because there are no jobs yet.
	for w := 1; w <= 3; w++ {
		go worker(w, jobs, results)
	}

	// Here we send 9 `jobs` and then `close` that
	// channel to indicate that's all the work we have.
	for j := 1; j <= 9; j++ {
		jobs <- j
	}
	close(jobs)

	// Finally we collect all the results of the work.
	for a := 1; a <= 9; a++ {
		re := <-results
		fmt.Println(re)
	}
}
```

可能的运行结果：

```
worker 1 processing job 1
worker 2 processing job 2
worker 3 processing job 3
worker 1 processing job 4
2
worker 2 processing job 5
worker 3 processing job 6
4
6
worker 1 processing job 7
worker 2 processing job 8
worker 3 processing job 9
8
10
12
14
16
18
```



# 随机数

## 随机byte数组

```go
import (
	"fmt"
	"math/rand"
	"time"
)

func randomBytes(d []byte) {
	r := rand.New(rand.NewSource(time.Now().UnixNano()))
	n, err := r.Read(d)
	if err != nil {
		panic(err)
	}
	if len(d) != n {
		fmt.Printf("Read %d bytes, want %d bytes\n", n, len(d))
	}
}
```

或者使用 `crypto/rand` 包，这个包生成的随机数更加复杂并且不可预测:

```go
import (
	"crypto/rand"
	"io"
)

func randomBytes(d []byte) {
	io.ReadFull(rand.Reader, d)
}
```
