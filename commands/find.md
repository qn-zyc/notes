
# find

```
find 路径 参数
```

在当前文件夹下搜索目录或文件： `find ./ -iname "name"`, 只会搜索全名是name的目录或文件，会递归搜索。名称可以使用通配符 `*`。


## 搜索包含指定内容的文件

```bash
find ./ | xargs grep "content"
```
