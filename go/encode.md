<!-- TOC -->

- [使用golang.org/x/text/encoding和golang.org/x/text/transform编解码GBK](#使用golangorgxtextencoding和golangorgxtexttransform编解码gbk)

<!-- /TOC -->

# 使用golang.org/x/text/encoding和golang.org/x/text/transform编解码GBK

```go
func DecodeToGBK(utf8Str string) (dst string, err error) {
	var trans transform.Transformer = simplifiedchinese.GBK.NewEncoder()
	var reader *strings.Reader = strings.NewReader(utf8Str)
	var transReader *transform.Reader = transform.NewReader(reader, trans)
	bytes, err := ioutil.ReadAll(transReader)
	if err != nil {
		return
	}
	dst = string(bytes)
	return
}

func EncodeFromGBK(gbkStr string) (utf8Str string, err error) {
	var trans transform.Transformer = simplifiedchinese.GBK.NewDecoder()
	var reader *strings.Reader = strings.NewReader(gbkStr)
	var transReader *transform.Reader = transform.NewReader(reader, trans)
	bytes, err := ioutil.ReadAll(transReader)
	if err != nil {
		return
	}
	utf8Str = string(bytes)
	return
}
```
