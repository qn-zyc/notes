# 生成xml

## 生成root

```python
from xml.dom.minidom import getDOMImplementation

root = getDOMImplementation().createDocument(None, 'root', None).documentElement
print root.toxml()  # <root/>
```

## 创建文本节点

```python
from xml.dom.minidom import getDOMImplementation

doc = getDOMImplementation().createDocument(None, 'root', None)
root = doc.documentElement
child = doc.createTextNode('hello world')
root.appendChild(child)
print root.toxml()
```

```xml
<root>hello world</root>
```

## 创建子节点

```python
from xml.dom.minidom import getDOMImplementation

doc = getDOMImplementation().createDocument(None, 'root', None)
root = doc.documentElement

tag1 = doc.createElement('tag1')
tag1.setAttribute('attr', 'val')  # 设置属性
root.appendChild(tag1)
print root.toxml()
```

```xml
<root><tag1 attr="val"/></root>
```

