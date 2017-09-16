<!-- TOC -->

- [md5](#md5)
- [sha3](#sha3)

<!-- /TOC -->


# md5

- 方式1

 ```go
 md5.Sum([]byte)
 ```

- 方式2

 ```go
 hash := md5.New()
 hash.Write([]byte)
 hash.Sum(nil)
 ```



不同：

1. 方式1返回的是 `[Size]byte` 类型的数组而不是切片
2. 方式2: Write函数会把MD5对象内部的字符串clear掉，然后把其参数作为新的内部字符串。而Sum函数则是先计算出内部字符串的MD5值，而后把输入参数附加到内部字符串后面。



坑：

```go
md5.New().Sum([]byte)
```

这种方式是先加密空字符串，然后将结果附加到了Sum的参数中，和没有加密一样。



# sha3

```go
import (
	"encoding/base64"
	"golang.org/x/crypto/sha3"
)

// 对用户密码进行编码。若密码明文为空字符串，则也会返回空字符串。
func EncodePassword(cleartext string) string {
	if cleartext == "" {
		return ""
	}
	h := make([]byte, 64)
	sha3.ShakeSum256(h, []byte(cleartext))
	return base64.StdEncoding.EncodeToString(h)
}
```

