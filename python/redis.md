<!-- TOC -->

- [redis-py](#redis-py)
    - [安装](#安装)
    - [getting started](#getting-started)
    - [错误处理](#错误处理)
    - [操作](#操作)

<!-- /TOC -->


# redis-py
* [github](https://github.com/andymccurdy/redis-py)

## 安装
* `sudo pip install redis`


## getting started

```py
>>> import redis
>>> r = redis.StrictRedis(host='localhost', port=6379, db=0)
>>> r.set('foo', 'bar')
True
>>> r.get('foo')
'bar'
```


## 错误处理
* 操作错误的类型(比如在hash上使用get): `ResponseError: WRONGTYPE Operation against a key holding the wrong kind of value`


## 操作

```py
# keys
r.keys('*') # list
r.delete(key) # 被删除的键的个数

# string
r.set(key, value) # True
r.get(key) # value or None

# hash
r.hset(key, field, value) # 成功个数
r.hget(key, field) # value or None
```