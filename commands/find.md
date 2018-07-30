
# find

```
find 路径 参数
```

在当前文件夹下搜索目录或文件： `find ./ -iname "name"`, 只会搜索全名是name的目录或文件，会递归搜索。名称可以使用通配符 `*`。

```bash
# 找出并打印每个文件
find /
# 在主目录中找到所有 jpg 文件
find ~ -name "*.jpg"
# 在主目录中找到所有 jpg 文件, 不区分大小写
find ~ -iname "*.jpg"
# 在当前目录查找 jpg 后缀或者 txt 后缀(-o 表示或, 括号需要用反斜杠转义)
find ./ \( -iname "*.jpg" -o -iname "*.txt" \)
# 只查找文件
find ./ \( -iname "*.jpg" -o -iname "*.txt" \) -type f
# 只查找目录
find ./ \( -iname "*.jpg" -o -iname "*.txt" \) -type d
# 查找修改时间(mtime)在过去7天内的文件(类似的还有 更改时间(ctime), 访问时间(atime))
find ./ \( -iname "*.jpg" -o -iname "*.txt" \) -type f -mtime -7
# 在 log 目录找到大小大于 1G 的文件
find /var/log -size +1G
# 找到 root 拥有的所有文件
find /data -owner root
# 在主目录中找到对所有人可读的文件
find ~ -perm -o=r
```


## 搜索包含指定内容的文件

```bash
find ./ | xargs grep "content"
```
