# 编码

```python
import json

data = {"a":"hello", "b":123, "c":[2,5,0]}
jsonStr = json.dumps(data)
print jsonStr  # {"a": "hello", "c": [2, 5, 0], "b": 123}
```

# 解码

```python
import json

jsonStr = '{"a": "hello", "c": [2, 5, 0], "b": 123}'
data = json.loads(jsonStr)
print data  # {u'a': u'hello', u'c': [2, 5, 0], u'b': 123}
print data["a"]  # hello
print data["c"]  # [2, 5, 0]
```



# 自定义对象

## 编码

```python
def convert_to_builtin_type(obj):
    # d = {
    #     '__class__': obj.__class__.__name__,
    #     '__module__': obj.__module__,
    # }
    # d.update(obj.__dict__)
    # return d
    return obj.__dict__


def marshal(obj):
    return json.dumps(obj, default=convert_to_builtin_type)

```

