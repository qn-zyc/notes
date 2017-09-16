<!-- TOC -->

- [参考](#参考)
- [逻辑](#逻辑)
    - [条件](#条件)
    - [遍历](#遍历)
- [结构体](#结构体)
    - [调用结构体的方法获取值](#调用结构体的方法获取值)

<!-- /TOC -->

# 参考 #

* <http://gohugo.io/templates/go-templates/>

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

