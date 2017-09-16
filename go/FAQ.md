<!-- TOC -->

- [`:=` 左边出现结构体字段问题](#-左边出现结构体字段问题)
- [测试时](#测试时)
    - [benchmark测试时调用顺序不同影响了执行时间](#benchmark测试时调用顺序不同影响了执行时间)
- [install](#install)
    - [go install: no install location for directory /Users/zhangyuchen/chen/src outside GOPATH](#go-install-no-install-location-for-directory-userszhangyuchenchensrc-outside-gopath)

<!-- /TOC -->



# `:=` 左边出现结构体字段问题

问题还原：

```go
type stu struct {
	a int64
	b int64
}

func Add(a, b int64) (int64, error) {
	return a + b, nil
}
func (p *stu) Add(a, b int64) (int64, error) {
//	var err error
	p.a, err := Add(a, b) //此处为什么会提示错误，p.a已经存在，但err不存在。感觉应该可以啊
	fmt.Println(p.a)
	if err != nil {
		fmt.Println(err.Error())
	}
	return 0, nil
}

func main() {
	s := stu{10, 20}
	s.Add(20, 30)
}
```

提示错误：

	non-name p.a on left side of :=

使用下面的方式是可以的：

```go
var err error
p.a, err = Add(a, b)
```


# 测试时

## benchmark测试时调用顺序不同影响了执行时间

测试二叉搜索树的搜索算法时，每一个测试方法中都是先生成了一棵树（数据是随机的），然后用不同的搜索方法搜索，测试的结果总是谁在前面谁执行的时间短。

后来在群里大神的帮助下，将树的产生放到init()中，测试的结果和预期一样了。

经过仔细讨论发现，问题的原因是rand引起的。之前我以为rand是每次运行时都会产生不同的数据，但是go的随机数不是这样的，默认情况下是将1作为种子的：

```go
var globalRand = New(&lockedSource{src: NewSource(1)})
```

这样造成的结果就是每次运行产生的随机数序列都是一样的，所以生成的两棵树每次运行都是一样的，比如第一个每次都是[1,2,3],第二颗每次都是[4,5,6]，这样在搜索的时候当然是谁先运行谁执行的时间短了（这是随机出来的数决定的，也有可能总是第二棵树执行的时间短）。

总之，这次是栽在了rand上，也怪自己粗心和测试不严谨，以后吸取教训。

# install

## go install: no install location for directory /Users/zhangyuchen/chen/src outside GOPATH

这是直接安装的main.go，如果按照其他的文件就可以了。。。