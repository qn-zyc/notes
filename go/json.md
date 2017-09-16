<!-- TOC -->

- [解析json](#解析json)
    - [遇到的问题](#遇到的问题)
        - [解析到切片](#解析到切片)
- [解析到interface](#解析到interface)
- [生成JSON](#生成json)
- [使用json标签](#使用json标签)
- [Unmarshal 中没有的字段不会覆盖成默认值](#unmarshal-中没有的字段不会覆盖成默认值)
- [自定义解析过程](#自定义解析过程)

<!-- /TOC -->


# 解析json

```go
type Server struct {
	ServerName, ServerIP string
}

type ServerSlice struct {
	Servers []Server
}

func Read() {
	str := `{"servers":[{"serverName":"Shanghai_VPN","serverIP":"127.0.0.1"},{"serverName":"Beijing_VPN","serverIP":"127.0.0.2"}]}`
	var s ServerSlice
	json.Unmarshal([]byte(str), &s)
	fmt.Println(s)
}
```

输出：

```
{[{Shanghai_VPN 127.0.0.1} {Beijing_VPN 127.0.0.2}]}
```

例如JSON的key是Foo，那么怎么找对应的字段呢？

1. 首先查找tag含有Foo的可导出的struct字段(首字母大写)
2. 其次查找字段名是Foo的导出字段
3. 最后查找类似FOO或者FoO这样的除了首字母之外其他大小写不敏感的导出字段,能够被赋值的字段必须是可导出字段(即首字母大写），同时JSON解析的时候只会解析能找得到的字段，找不到的字段会被忽略
4. json字符串的`name`可以被解析到`Name`字段(没有加tag)


## 遇到的问题

### 解析到切片

有一个类型是 `[]*Tree` 的字段，如果需要解析到这个字段的话需要使用 `&`

```go
json.Unmarshal(bytes, &dirData.root.Children)
```

Children的类型是 

```go
Children []*Tree     `json:"children"`
```


# 解析到interface

现在我们假设有如下的JSON数据

```go
b := []byte(`{"Name":"Wednesday","Age":6,"Parents":["Gomez","Morticia"]}`)
```

如果在我们不知道他的结构的情况下，我们把他解析到interface{}里面

```go
b := []byte(`{"Name":"Wednesday","Age":6,"Parents":["Gomez","Morticia"]}`)
var f interface{}
json.Unmarshal(b, &f) // 这个时候f里面存储了一个map类型，他们的key是string，值存储在空的interface{}里
if m, ok := f.(map[string]interface{}); ok {
	for k, v := range m {
		switch vv := v.(type) {
		case string:
			fmt.Println(k, "is string", vv)
		case int:
			fmt.Println(k, "is int", vv)
		case float64:
			fmt.Println(k, "is float64", vv)
		case []interface{}:
			fmt.Println(k, "is an array:")
			for i, u := range vv {
				fmt.Println(i, u)
			}
		default:
			fmt.Println(k, "is of a type I don't know how to handle")
		}
	}
}
```

# 生成JSON

结构体定义如上。

```go
var s ServerSlice
s.Servers = append(s.Servers, Server{ServerName: "Shanghai_VPN", ServerIP: "127.0.0.1"})
s.Servers = append(s.Servers, Server{ServerName: "Beijing_VPN", ServerIP: "127.0.0.2"})
b, err := json.Marshal(s)
if err != nil {
	fmt.Println("json err:", err)
}
fmt.Println(string(b))
```

输出：

```json
{"Servers":[{"ServerName":"Shanghai_VPN","ServerIP":"127.0.0.1"},{"ServerName":"Beijing_VPN","ServerIP":"127.0.0.2"}]}
```

# 使用json标签

如果将结构体定义成这样：

```go
type Server struct {
	ServerName string `json:"server-name"`
	ServerIP   string `json:"server-ip"`
}

type ServerSlice struct {
	Servers []Server `json:"servers"`
}
```

输出时：

```json
{"servers":[{"server-name":"Shanghai_VPN","server-ip":"127.0.0.1"},{"server-name":"Beijing_VPN","server-ip":"127.0.0.2"}]}
```

更多细节参考 **json.Marshal**。



- 不导出：ID int `json:"-"`
- 如果为空，不导出：ServerIP   string `json:"serverIP,omitempty"`
- 二次JSON编码：

  ```go
  ServerName  string `json:"serverName"`
  ServerName2 string `json:"serverName2,string"
  ```
  假如值都是 \`Go "1.0" \`，输出：

  ```js
  {"serverName":"Go \"1.0\" ","serverName2":"\"Go \\\"1.0\\\" \""}
  ```


# Unmarshal 中没有的字段不会覆盖成默认值 #

```go
type User struct {
	Name string
	Age  int
}

func main() {
	u := &User{Name: "chen"}
	err := json.Unmarshal([]byte(`{"Age":18}`), u)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(u) // &{chen 18}
	err = json.Unmarshal([]byte(`{"Name":"hello","Age":18}`), u)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(u) // &{hello 18}
}
```


# 自定义解析过程

```go
type DurationConf time.Duration

func (dc *DurationConf) UnmarshalJSON(d []byte) error {
	if len(d) == 0 {
		return errors.New("cann't parse empty string to DurationConf")
	}
	d = bytes.Trim(d, `"`)
	dur, err := time.ParseDuration(string(d))
	if err != nil {
		return err
	}
	*dc = DurationConf(dur)
	return nil
}
```

注意d中可能会包括双引号。


