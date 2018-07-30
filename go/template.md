<!-- TOC -->

- [参考](#参考)
- [逻辑](#逻辑)
    - [条件](#条件)
    - [遍历](#遍历)
- [变量](#变量)
    - [外部变量](#外部变量)
- [结构体](#结构体)
    - [调用结构体的方法获取值](#调用结构体的方法获取值)
- [函数](#函数)
    - [内置函数](#内置函数)
    - [添加自定义函数](#添加自定义函数)

<!-- /TOC -->

# 参考 #

* <http://gohugo.io/templates/go-templates/>


------------------------------
# 逻辑 #

## 条件 ##

Conditionals

if, else, with, or & and provide the framework for handling conditional logic in Go Templates. Like range, each statement is closed with end.

Go Templates treat the following values as false:

* false
* 0
* any array, slice, map, or string of length zero

Example 1: if

```html
{{ if isset .Params "title" }}<h4>{{ index .Params "title" }}</h4>{{ end }}
```

Example 2: if … else

```html
{{ if isset .Params "alt" }}
    {{ index .Params "alt" }}
{{else}}
    {{ index .Params "caption" }}
{{ end }}
```

Example 3: and & or

```html
{{ if and (or (isset .Params "title") (isset .Params "caption")) (isset .Params "attr")}}
```

Example 4: with

An alternative way of writing “if” and then referencing the same value is to use “with” instead. with rebinds the context . within its scope, and skips the block if the variable is absent.

The first example above could be simplified as:

```html
{{ with .Params.title }}<h4>{{ . }}</h4>{{ end }}
```

Example 5: if … else if

```html
{{ if isset .Params "alt" }}
    {{ index .Params "alt" }}
{{ else if isset .Params "caption" }}
    {{ index .Params "caption" }}
{{ end }}
```


## 遍历 ##

Example 1: Using Context

```go
{{ range array }}
    {{ . }}
{{ end }}
```

Example 2: Declaring value variable name

```
{{range $element := array}}
    {{ $element }}
{{ end }}
```

Example 2: Declaring key and value variable name

```
{{range $index, $element := array}}
    {{ $index }}
    {{ $element }}
{{ end }}
```

------------------------------
# 变量

```go
t := template.New("")
t, err := t.Parse(`{{with $x := .v.x}}output: {{$x}}{{end}}`)
if err != nil {
	panic(err)
}
w := bytes.NewBuffer(nil)
err = t.Execute(w, map[string]interface{}{
	"v": map[string]interface{}{
		"x": 123,
	},
})
if err != nil {
	panic(err)
}
fmt.Println(w.String()) // output: 123
```

## 外部变量

在局部作用域内访问外部变量, 比如循环内.

```go
{{range .slice}}
	{{$.GlobalVal}}
{{end}}
```


------------------------------
# 结构体

## 调用结构体的方法获取值

```go
package main

import (
	"fmt"
	"html/template"
	"log"
	"net/http"
)

type User struct {
	name string
}

func (this *User) GetName() string {
	return this.name
}

func test(w http.ResponseWriter, r *http.Request) {
	user := &User{
		name: "hello world!",
	}
	m := map[string]interface{}{
		"user": user,
	}
	t, err := template.ParseFiles("views/test.tpl")
	if err != nil {
		fmt.Println("parseFiles:", err)
		return
	}
	t.Execute(w, m)
}

func main() {
	http.HandleFunc("/", test)
	err := http.ListenAndServe(":9090", nil)
	if err != nil {
		log.Fatal("listenAndServe:", err)
	}
}
```

现在map中有一个user对象，想在模板中调用user.GetName()：

```html
    <div>
        {{.user.GetName}}
    </div>
```



------------------------------
# 函数

## 内置函数

示例:

```go
// printf
t, err := t.Parse(`{{printf "%.4f" .value}}`)
```

## 添加自定义函数

* `Funcs` 需要在 `Parse` 之前调用.
* 自定义函数可以有一个或者两个返回值, 如果有两个返回值, 第二个返回值应该是 error 类型, 当返回的 error 不为空时, 会将 error 通过 `Execute` 返回.

```go
t := template.New("")
t.Funcs(template.FuncMap{
	"myfunc": func(name string, value int) string {
		return fmt.Sprintf("hello %s, value is %d", name, value)
	},
})
t, err := t.Parse(`output: {{myfunc "name" .value}}`)
if err != nil {
	panic(err)
}
w := bytes.NewBuffer(nil)
err = t.Execute(w, map[string]interface{}{
	"value": 5,
})
if err != nil {
	panic(err)
}
fmt.Println(w.String()) // output: hello name, value is 5
```