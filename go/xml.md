<!-- TOC -->

- [解析xml](#解析xml)
- [输出xml](#输出xml)

<!-- /TOC -->

# 解析xml

示例xml:xml_test.xml

```
<?xml version="1.0" encoding="utf-8"?>
<servers version="1">
    <server>
        <serverName>Shanghai_VPN</serverName>
        <serverIP>127.0.0.1</serverIP>
    </server>
    <server>
        <serverName>Beijing_VPN</serverName>
        <serverIP>127.0.0.2</serverIP>
    </server>
</servers>
```

```go
type Servers struct {
	XMLName     xml.Name `xml:"servers"`
	Version     string   `xml:"version,attr"`
	Svs         []Server `xml:"server"`
	Description string   `xml:",innerxml"`
}

type Server struct {
	XMLName    xml.Name `xml:"server"`
	ServerName string   `xml:"serverName"`
	ServerIP   string   `xml:"serverIP"`
}

func ReadXml() {
	f, err := os.Open("xml_test.xml")
	if err != nil {
		fmt.Println(err)
		return
	}
	defer f.Close()

	data, err := ioutil.ReadAll(f)
	if err != nil {
		fmt.Println(err)
		return
	}

	servers := Servers{}
	err = xml.Unmarshal(data, &servers)
	if err != nil {
		fmt.Println(err)
		return
	}

	fmt.Printf("%#v\n", servers)
}
```
输出：
```
xml.Servers{XMLName:xml.Name{Space:"", Local:"servers"}, Version:"1", Svs:[]xml.Server{xml.Server{XMLName:xml.Name{Space:"", Local:"server"}, ServerName:"Shanghai_VPN", ServerIP:"127.0.0.1"}, xml.Server{XMLName:xml.Name{Space:"", Local:"server"}, ServerName:"Beijing_VPN", ServerIP:"127.0.0.2"}}, Description:"\r\n    <server>\r\n        <serverName>Shanghai_VPN</serverName>\r\n        <serverIP>127.0.0.1</serverIP>\r\n    </server>\r\n    <server>\r\n        <serverName>Beijing_VPN</serverName>\r\n        <serverIP>127.0.0.2</serverIP>\r\n    </server>\r\n"}
```
结构体中tag标签的写法参考xml.Unmarshal上的注释。

# 输出xml

这里省略了Servers和Server两个的声明，见上面的定义。

```go
	servers := &Servers{Version: "1"}
	servers.Svs = append(servers.Svs, Server{ServerName: "Shanghai_VPN", ServerIP: "127.0.0.1"})
	servers.Svs = append(servers.Svs, Server{ServerName: "Beijing_VPN", ServerIP: "127.0.0.2"})

	//data, err := xml.Marshal(servers)
	data, err := xml.MarshalIndent(servers, "", "  ")
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(xml.Header)
	fmt.Println(string(data))
```



