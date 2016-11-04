# Redis Rate Limiter

`redis` 实现限速器的几种方式。

## GET + INCR + EXPIRE

先获取 `key` 的当前值，如果没有超出限制再执行 `INCR` 增1，如果 `key` 不存在，使用 `redis` 的事务初始化 `key` 和过期时间。

伪代码：

```go
count = redis.GET(key)
if redis return nil {
  redis.MULTI
  	redis.INCR(key)
  	redis.EXPIRE(key, expire_time)
  redis.EXEC
  count = 1
}
if count > limit {
  return 超出限制
} else {
  redis.INCR(key)
}
```

高并发下的问题：

如果同时10个并发程序执行 `GET` 返回了 `nil`, 那么这10个并发程序都会执行 `redis` 的事务将 `key` 增一，但每个程序的 `count` 值都为1，如果 `limit` 设置的值小于10，那么真正执行的程序就超过限制了。如果执行完事务后再查一次 `redis` 赋值给 `count`，那么每个程序可能都会返回10，从而没有程序能够继续执行。

`key` 已经存在的情况下，先 `GET` 后 `INCR` 的逻辑也可能会出现实际执行的程序数多于 `limit` 的情况。



## INCR + EXPIRE

先 `INCR`, 如果值为1说明是 `key` 刚设置的，此时再执行 `EXPIRE`

伪代码：

```go
count = redis.INCR(key)
if count == 1 {
  redis.EXPIRE(key, expire_time)
}
if count > limit {
  return 超出限制
}
```

**慎用**

如果 `INCR` 之后程序挂掉了，没有执行 `EXPIRE`, 那么这个 `key` 就没有过期时间了，具体的影响视需求而定。



## lua脚本

摘自 <http://redisdoc.com/string/incr.html>, 我不会lua

```lua
local current
current = redis.call("incr",KEYS[1])
if tonumber(current) == 1 then
    redis.call("expire",KEYS[1],1)
end
```

