# md5

```python
import hashlib


def md5(src):
    return hashlib.md5(src).hexdigest()
```


# base64

```python
import base64

base64.b64encode("message1")
base64.b64decode("message2")

# url安全的base64编码
base64.urlsafe_b64encode("message")
```

# hmac_sha1

```python
from hashlib import sha1
import hmac

hm = hmac.new(key=sk, msg=str, digestmod=sha1)
return hm.digest()
```
