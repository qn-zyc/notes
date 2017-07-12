<!-- TOC -->

- [当前时间](#当前时间)
- [函数执行时间](#函数执行时间)
- [格式化时间](#格式化时间)
- [timezone](#timezone)
- [定时器](#定时器)
- [Ticker](#ticker)

<!-- /TOC -->



# 当前时间

可以使用 `time.Now()` 获取，或者使用 `t.Day()`、`t.Minute()` 等等来获取时间的一部分；你甚至可以自定义时间格式化字符串，例如： `fmt.Printf("%02d.%02d.%4d\n”, t.Day(), t.Month(), t.Year())` 将会输出 21.07.2011。

`time.Nanosecond()` 并不是从70年开始的纳秒，`UnixNano()` 才是。


# 函数执行时间

```go
start := time.Now()
doSomeThing()
end := time.Now()
delta := end.Sub(start)
fmt.Printf("doSomeThing took this amount of time: %s\n", delta) // 纳秒
```

# 格式化时间

```go
time.Now().Format("2006-01-02 15:04:05") // 2015-03-15 09:48:34
```

```go
y, m, d := time.Now().Date()
h, mi, s := time.Now().Clock()
fmt.Printf("现在是：%d年%d月%d日 %d时%d分%d秒 \n", y, m, d, h, mi, s) // 现在是：2015年3月15日 9时55分12秒
```

| 含义 | 格式化字符                             |
| ---- | -------------------------------------- |
| 月   | 1, 01, Jan, January                    |
| 日   | 2, 02, _2                              |
| 时   | 3, 03, 15, PM, pm, AM, am              |
| 分   | 4, 04                                  |
| 秒   | 5, 05                                  |
| 年   | 06, 2006                               |
| 时区 | -07, -0700, Z0700, Z07:00, -07:00, MST |
| 周   | Mon, Monday                            |

* 以 `Mon Jan 2 15:04:05 MST 2006` 时间来设定的, Unix 时间是1136239445.
* `_2` 表示如果日期只有一位, 则在前面加空格, 比如 7号 在格式化字符串 `<_2>` 下是 `< 7>`.
* 秒(5)后可以跟小数点加0或9, 比如 `05.00000` 可以格式化成 `39.12340`, 通过在后面补0来凑位数, 而 `05.99999` 可以格式化 `39.1234`, 后面的0被删除了.
* 时区, 比如东八区用 `-0700` 格式化为 `+0800`
	* `-0700  ±hhmm`
	* `-07:00 ±hh:mm`
	* `-07    ±hh`
	使用 `Z` 代替 `-`(UTC时间显示成Z, 其他地区的和上面一样)
	* `Z0700  Z or ±hhmm`
	* `Z07:00 Z or ±hh:mm`
	* `Z07    Z or ±hh`
	`MST` 表示时区的英文, 比如 `UTC`, `CST` 等. `MST` 是 `-0700` 时区.
* 字符串中未被识别的将原样输出.



# timezone

```go
	now := time.Now()
	s := now.Format("20060102150405")
	t, err := time.Parse("20060102150405", s)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(now, t)
```

上面输出：2016-01-25 19:15:31.21773073 +0800 CST 2016-01-25 19:15:31 +0000 UTC

Parse是按照UTC来解析的。

如果想按照本地时区来解析的话使用：`time.ParseInLocation("20060102150405", s, time.Local)`



# 定时器

```go
time.NewTimer(time.Second * 2)
```

NewTimer创建一个Timer，它会在最少过去时间段d后到期，向其自身的C字段发送当时的时间。

```go
	// Timers represent a single event in the future. You
	// tell the timer how long you want to wait, and it
	// provides a channel that will be notified at that
	// time. This timer will wait 2 seconds.
	timer1 := time.NewTimer(time.Second * 2)

	// The `<-timer1.C` blocks on the timer's channel `C`
	// until it sends a value indicating that the timer
	// expired.
	<-timer1.C
	fmt.Println("Timer 1 expired")

	// If you just wanted to wait, you could have used
	// `time.Sleep`. One reason a timer may be useful is
	// that you can cancel the timer before it expires.
	// Here's an example of that.
	timer2 := time.NewTimer(time.Second)
	go func() {
		<-timer2.C
		fmt.Println("Timer 2 expired")
	}()
	stop2 := timer2.Stop()
	if stop2 {
		fmt.Println("Timer 2 stopped")
	}
```

运行结果

```
Timer 1 expired
Timer 2 stopped
```



# Ticker

```go
time.NewTicker(time.Millisecond * 500)
```

NewTicker返回一个新的Ticker，该Ticker包含一个通道字段，并会每隔时间段d就向该通道发送当时的时间。它会调整时间间隔或者丢弃tick信息以适应反应慢的接收者。如果d<=0会panic。关闭该Ticker可以释放相关资源。

```go
	// Tickers use a similar mechanism to timers: a
	// channel that is sent values. Here we'll use the
	// `range` builtin on the channel to iterate over
	// the values as they arrive every 500ms.
	ticker := time.NewTicker(time.Millisecond * 500)
	go func() {
		for t := range ticker.C {
			fmt.Println("Tick at", t)
		}
	}()

	// Tickers can be stopped like timers. Once a ticker
	// is stopped it won't receive any more values on its
	// channel. We'll stop ours after 1500ms.
	time.Sleep(time.Millisecond * 1500)
	ticker.Stop()
	fmt.Println("Ticker stopped")
```

运行结果

```
Tick at 2015-04-22 09:58:52.048046619 +0800 CST
Tick at 2015-04-22 09:58:52.548034081 +0800 CST
Tick at 2015-04-22 09:58:53.048019367 +0800 CST
Ticker stopped
```


