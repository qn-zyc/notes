# 简介

正则表达式比纯粹的文本匹配效率低，但是它却更灵活

Go语言通过regexp标准包为正则表达式提供了官方支持，Go实现的是RE2标准

字符串处理我们可以使用strings包来进行搜索(Contains、Index)、替换(Replace)和解析(Split、Join)等操作，但是这些都是简单的字符串操作，他们的搜索都是大小写敏感，而且固定的字符串，如果我们需要匹配可变的那种就没办法实现了，当然如果strings包能解决你的问题，那么就尽量使用它来解决。因为他们足够简单、而且性能和可读性都会比正则好

在使用中需要注意的一点就是：所有的字符都是UTF-8编码的

# 判断是否匹配

Match, MatchReader, MatchString

```go
ip := "127.0.0.1"
match, err := regexp.MatchString("^[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}$", ip)
```

# Find

```go
	a := "I am learning Go language."
	re, err := regexp.Compile("[a-z]{2,4}")
	if err != nil {
		fmt.Println(err)
		return
	}

	// 查找符合正则的第一个
	one := re.Find([]byte(a))
	fmt.Println("Find:", string(one))

	// 查找符合正则的所有slice,n小于0表示返回全部符合的字符串，不然就是返回指定的长度
	all := re.FindAllString(a, -1)
	fmt.Println("FindAll:", all)

	// 查找符合条件的index位置,开始位置和结束位置
	index := re.FindIndex([]byte(a))
	fmt.Println("FindIndex:", index)

	// 查找符合条件的所有的index位置，n同上
	allindex := re.FindAllIndex([]byte(a), -1)
	fmt.Println("FindAllIndex:", allindex)

	re2, _ := regexp.Compile("am(.*)lang(.*)")

	// 查找submatch
	sub := re2.FindStringSubmatch(a)
	fmt.Println("FindStringSubmatch:", sub)

	allsub := re2.FindAllStringSubmatch(a, -1)
	fmt.Println("FindAllStringSubmatch:", allsub)
```

输出：

```
Find: am
FindAll: [am lear ning lang uage]
FindIndex: [2 4]
FindAllIndex: [[2 4] [5 9] [9 13] [17 21] [21 25]]
FindStringSubmatch: [am learning Go language.  learning Go  uage.]
FindAllStringSubmatch: [[am learning Go language.  learning Go  uage.]]
```

# Expand

Expand可以替换模板中$name或${name}，name是分组名

```go
	src := []byte(`
        call hello alice
        hello bob
        call hello eve
	`)
	// (?m)开启m标志，后面的$还可以匹配行尾
	// 3个分组，2个命名分组
	re := regexp.MustCompile(`(?m)(call)\s+(?P<cmd>\w+)\s+(?P<arg>\w+)\s*$`)

	allsub := re.FindAllStringSubmatch(string(src), -1)
	for _, v := range allsub {
		for _, vv := range v {
			fmt.Printf("%q, ", vv)
		}
		fmt.Print("\n")
	}

	res := []byte{}
	for _, s := range re.FindAllSubmatchIndex(src, -1) {
		res = re.Expand(res, []byte("$cmd('$arg')\n"), src, s)
	}
	fmt.Println(string(res))
```

输出：

```
"call hello alice", "call", "hello", "alice", 
"call hello eve\n\t", "call", "hello", "eve", 
hello('alice')
hello('eve')
```

