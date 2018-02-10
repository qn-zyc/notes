

# go 代码风格

## import

导入包的顺序： 标准库包， 程序内部包， 第三方包：

```go
import (
    "encoding/json"
    "strings"

    "myproject/models"
    "myproject/controller"
    "myproject/utils"

    "github.com/astaxie/beego"
    "github.com/go-sql-driver/mysql"
)
```

不用使用相对路径引入包：

```go
// 这是不好的导入
import "../net"

// 这是正确的做法
import "github.com/repo/proj/src/net"
```


## error

错误信息首字母不要大写。




# CodeReview

- 在代码上执行 gofmt 或 goimports。
- 注释应该是一个完整的句子，虽然有时看起来有点啰嗦。注释应该以要描述的事物开头，以句号结尾。参考 https://golang.org/doc/effective_go.html#commentary
```go
```




# 参考
- [Go编码规范指南](https://gocn.io/article/1)
- [CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments)
