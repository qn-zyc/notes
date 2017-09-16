<!-- TOC -->

- [os包](#os包)
    - [itoa(int) string](#itoaint-string)
- [strings](#strings)
    - [Split与Field的区别](#split与field的区别)
- [strconv](#strconv)
    - [IntSize int类型的位数](#intsize-int类型的位数)

<!-- /TOC -->

# os包

## itoa(int) string

位置：os/str.go

```go
func itoa(val int) string { // do it here rather than with fmt to avoid dependency
	if val < 0 {
		return "-" + itoa(-val)
	}
	var buf [32]byte // big enough for int64
	i := len(buf) - 1
	for val >= 10 {
		buf[i] = byte(val%10 + '0') // 比如：将数字3转换为ASCII中的字符'3'
		i--
		val /= 10
	}
	buf[i] = byte(val + '0')
	return string(buf[i:])
}
```

# strings

## Split与Field的区别

```go
var ret []string = strings.Split("a  a", " ") //a之间有两个空格
for _, s := range ret {
	fmt.Printf("%q,", s)
}
fmt.Print("\n")
ret = strings.Fields("a  a")
for _, s := range ret {
	fmt.Printf("%q,", s)
}
fmt.Print("\n")
```

输出

```
"a","","a",
"a","a",
```

当两个a之间只有一个空格时，输出都一样”a”,”a”,但有两个空格时Split比Fields多出一个””，这是因为Split是根据第二个参数在第一个参数中出现的次数来决定最后返回的数组的大小，而Fields则是根据空格将字符串分成了几个段，如果有相连的空格，则会跳过（可以当成一个空格处理）。


# strconv

## IntSize int类型的位数

```go
const intSize = 32 << uint(^uint(0)>>63)
```

`^uint(0)`是将0取反，是全1，`>>63`是得到第64位，32为OS下得到的是0,64位下得到的是1,`32<<0`还是32,`32<<1`就是64

