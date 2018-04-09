<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [创建和删除目录](#创建和删除目录)
- [linux中文件权限的表示](#linux中文件权限的表示)
- [创建并写文件](#创建并写文件)
- [打开并读文件](#打开并读文件)
	- [按行读取文件](#按行读取文件)
- [删除文件](#删除文件)
- [覆盖式写文件（不存在时创建）](#覆盖式写文件不存在时创建)
- [遍历目录](#遍历目录)
	- [遍历所有后代文件或目录](#遍历所有后代文件或目录)
		- [跳过目录](#跳过目录)
	- [遍历子文件或目录](#遍历子文件或目录)
	- [以树形显示目录树](#以树形显示目录树)
	- [使用正则搜索文件](#使用正则搜索文件)
- [当前程序运行的目录](#当前程序运行的目录)
- [判断文件是否存在](#判断文件是否存在)
- [以列表显示目录树，可自定义根节点名称 #](#以列表显示目录树可自定义根节点名称-)
- [复制文件](#复制文件)

<!-- /TOC -->



# 创建和删除目录

```go
os.Mkdir("file", os.ModePerm)
os.MkdirAll("file2/sub1/sub2", os.ModePerm)
err := os.Remove("file2/sub1") // cann't remove
if err != nil {
	fmt.Println(err)
}
os.RemoveAll("file2")
os.Remove("file")
```

# linux中文件权限的表示
可读-可写-可执行表示为111，没有权限使用0

分组：自己-同组用户-其他用户，每组3位数，如110 110 100即664，表示自己可读写，同组用户可读写，其他用户可读。

# 创建并写文件

```go
fout, err := os.Create("t1.txt")
if err != nil {
	fmt.Println(err)
	return
}
defer fout.Close()

for i := 0; i < 10; i++ {
	fout.WriteString("just 测试 string.\r\n")
	fout.Write([]byte("just 测试 []byte.\r\n"))
}
```

# 打开并读文件

```go
f, err := os.Open("t1.txt")
if err != nil {
	fmt.Println(err)
	return
}
defer f.Close()

buf := make([]byte, 1024)
for {
	n, _ := f.Read(buf)
	if n == 0 {
		break
	}
	os.Stdout.Write(buf[:n])
}
```

也可以使用 `ioutil.ReadAll(f)`，它会把整个文件都读到内存中。

## 按行读取文件

```go
func FileRead(filePath string) (err error) {
	file, err := os.OpenFile(filePath, os.O_RDONLY, os.ModePerm)
	if err != nil {
		return
	}
	defer file.Close()

	reader := bufio.NewReader(file)
	var (
		line      string
		lineCount int
	)
	for err == nil {
		line, err = reader.ReadString('\n')
		line = strings.TrimSpace(line) // line包含换行符
		if line == "" {
			continue
		}
		lineCount++
		if lineCount > 10 {
			break
		}
		fmt.Println(lineCount, line)
	}
	if err == io.EOF {
		err = nil
	}
	return
}
```

使用bufio.NewScanner

```go
input := bufio.NewScanner(f)
for input.Scan() {
	counts[input.Text()]++ // input.Text()不包含换行符
}
```

```go
// readFromFile will read all lines from the given filename and write them to a
// string array, if filename is empty readFromFile returns and empty string
// array
func readLinesFromFile(filename string) ([]string, error) {
	if filename == "" {
		return nil, nil
	}

	file, err := os.Open(filename)
	if err != nil {
		return nil, err
	}
	defer file.Close()

	var lines []string

	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		lines = append(lines, scanner.Text())
	}

	if err := scanner.Err(); err != nil {
		return nil, err
	}

	return lines, nil
}
```



# 删除文件
删除文件和删除目录是同一个方法

```go
err := os.Remove("t1.txt")
if err != nil {
	fmt.Println(err)
} else {
	fmt.Println("removed.")
}
```

删除目录时要求目录是空的。

# 覆盖式写文件（不存在时创建）

```go
file, err := os.OpenFile(fileName, os.O_WRONLY|os.O_CREATE|os.O_TRUNC, os.ModePerm)
if err != nil {
	return
}
defer file.Close()

_, err = file.Write(data)
```

# 遍历目录

## 遍历所有后代文件或目录

```go
filepath.Walk("/a/b/c", func(path string, info os.FileInfo, err error) error {
	// do something...
	return nil
})
```

path是文件的全路径

### 跳过目录

```go
return filepath.SkipDir
```



## 遍历子文件或目录

```go
file, _ := os.Open(path)
children, _ := file.Readdir(-1)
```

或者使用`ioutil.ReadDir()`

## 以树形显示目录树

```go
func WalkSubDir() (t *tree.TreeNode) {
	root := "/a/b/c"
	t = tree.NewTreeNode(nil, root)

	walkDirToTree(t, root)
	return t
}

func walkDirToTree(t *tree.TreeNode, path string) {
	file, _ := os.Open(path)
	fileInfo, _ := file.Stat()
	node, _ := t.AppendChild(fileInfo.Name())

	// 文件
	if !fileInfo.IsDir() {
		return
	}

	// 目录
	children, _ := file.Readdir(-1)
	for _, child := range children {
		walkDirToTree(node, filepath.Join(path, child.Name()))
	}
}
```

## 使用正则搜索文件

```go
dir := "/a"
files, err := filepath.Glob(dir + "/*.md")
if err != nil {
	panic(err)
}

for _, d := range files {
	fmt.Printf("match %10s \n", d)
}
```

输出示例：

```go
match /a/a.md
```

可用的正则：

```
pattern:
	{ term }
term:
	'*'         matches any sequence of non-Separator characters
	'?'         matches any single non-Separator character
	'[' [ '^' ] { character-range } ']'
	            character class (must be non-empty)
	c           matches character c (c != '*', '?', '\\', '[')
	'\\' c      matches character c

character-range:
	c           matches character c (c != '\\', '-', ']')
	'\\' c      matches character c
	lo '-' hi   matches character c for lo <= c <= hi
```

查看文档和test文件了解更多。




# 当前程序运行的目录

```go
workPath, _ = os.Getwd()
workPath, _ = filepath.Abs(workPath)
AppPath, _ = filepath.Abs(filepath.Dir(os.Args[0]))
```

- 摘自beego
- os.Args[0] 是运行文件的全路径
- workPath 和 AppPath 应该是一样的，不一样的情况还不知道是怎么样的？


# 判断文件是否存在

```go
func FileExists(name string) bool {
	if _, err := os.Stat(name); err != nil {
		if os.IsNotExist(err) {
			return false
		}
	}
	return true
}
```

- 摘自beego
- os.IsNotExist(err) 可判断err是不是表示不存在
- 与之相反的是：os.IsExist(err)
- 也可以判断目录是否存在

# 以列表显示目录树，可自定义根节点名称 #

```go
func GetHugoContentDirList() (list []string) {
	root, err := os.Open("D:/work/hugo/mysite/content")
	if err != nil {
		return
	}
	list = append(list, "root")
	children := getChildren("root", root)
	list = append(list, children...)
	return
}

func getChildren(prefix string, parent *os.File) (children []string) {
	cFiles, err := parent.Readdir(-1)
	if err != nil {
		return
	}

	parentAbsPath, err := filepath.Abs(parent.Name())
	if err != nil {
		fmt.Println(err)
		return
	}

	for _, c := range cFiles {
		fullPath := prefix + "/" + c.Name()
		children = append(children, fullPath)
		if c.IsDir() {
			cFile, err := os.Open(filepath.Join(parentAbsPath, c.Name()))
			if err != nil {
				fmt.Println(err, c.Name())
				continue
			}
			subChildren := getChildren(fullPath, cFile)
			children = append(children, subChildren...)
		}
	}
	return
}
```

输出示例：

```
[root root/about.md root/post root/post/first.md root/post/second.md root/post/THREE.md]
```

# 复制文件

```go
// copyFile creates dst from src, preserving file attributes and timestamps.
func copyFile(dst, src string) error {
	fi, err := os.Stat(src)
	if err != nil {
		return err
	}

	fsrc, err := os.Open(src)
	if err != nil {
		return err
	}

	if err = os.MkdirAll(filepath.Dir(dst), 0755); err != nil {
		fmt.Printf("MkdirAll(%v)\n", filepath.Dir(dst))
		return err
	}

	fdst, err := os.Create(dst)
	if err != nil {
		return err
	}

	if _, err = io.Copy(fdst, fsrc); err != nil {
		return err
	}

	if err == nil {
		err = fsrc.Close()
	}

	if err == nil {
		err = fdst.Close()
	}

	if err == nil {
		err = os.Chmod(dst, fi.Mode())
	}

	if err == nil {
		err = os.Chtimes(dst, fi.ModTime(), fi.ModTime())
	}

	return nil
}
```
