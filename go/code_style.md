

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



## log




# 参考
- [Go编码规范指南](https://gocn.io/article/1)
