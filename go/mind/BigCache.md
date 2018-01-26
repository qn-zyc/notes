

# BigCache

## 简介
- [GitHub](https://github.com/allegro/bigcache)


## 结构

```
+------------------------------------------------------------+
|                                                            |
|        +-----------------+    +-----------------+          |
|        |                 |    |                 |          |
|        | +-------------+ |    | +-------------+ |          |
|        | |             | |    | |             | |          |
|        | |   hashmap   | |    | |   hashmap   | |          |
|        | |             | |    | |             | |          |
|        | +-------------+ |    | +-------------+ |   ...    |
|        |                 |    |                 |          |
|        | +-------------+ |    | +-------------+ |          |
|        | |             | |    | |             | |          |
|        | |  entries    | |    | |  entries    | |          |
|        | |             | |    | |             | |          |
|        | +-------------+ |    | +-------------+ |          |
|        |                 |    |                 |          |
|        +-----------------+    +-----------------+          |
|                                                            |
|           cacheShard             cacheShard                |
|                                                            |
|                                                            |
+------------------------------------------------------------+

                     BigCache

```

hashmap 是一个 `map[uint64]uint32` 结构， key 是 `hash(key)` 的结果， value 是储存的数据的起始索引。

entries 是一个 BytesQueue, 用于存储数据, 包括 key, value, 时间戳等。

cacheShard 主要包括的是 hashmap 和 entries, 一个 BigCache 中可以包含多个 cacheShard, 数据具体存放在哪个 cacheShard 由 key 和 hash 函数决定(`hash(key)`)。


## cacheShard


### 初始化

主要是 entries 的初始化, entries 的结构如下:

```go
type BytesQueue struct {
	array           []byte
	capacity        int
	maxCapacity     int
	head            int
	tail            int
	count           int
	rightMargin     int
	headerBuffer    []byte
	verbose         bool
	initialCapacity int
}
```

array 的初始化大小是这样计算的: `(conf.MaxEntriesInWindow / conf.Shards) * conf.MaxEntrySize`, 每个 shard 在声明周期内能存放的最大entry数量乘以entry最大大小， 每个 shard 最少存放 10 个 entry。

array 在使用过程中还会根据需要扩充大小, 如果没有配置 HardMaxCacheSize 的话是没有限制的, 配置了的话每个 cacheShard 分配到的最大大小是: `conf.HardMaxCacheSize / conf.Shards`。

head 和 tail 初始时都为 1 (为什么不是0: hashmap 中存储了每个 key 的索引, 当返回 0 时认为数据不存在, 所以有效数据的索引必须大于0)

rightMargin 是记录有效数据在右边界的位置（初始时为1）。 假如容量是 10, 当前写到第 7 的位置, 下次从第 8 的位置开始写, 但是新数据要占用 5 个空间, 则需要从头开始写, 此时 rightMargin 就是 8, 9 和 10 为无效空间, 在扩充容量等场景下会忽视掉。

count 是存储的 entry 的数目。


### entry 的结构

entry 是 entries 存放的基本单位, 结构如下。 entry 在被放入 BytesQueue 时还会在前面加上 entry 的长度。

```
+---+
|   |
| 8 |   timestamp
|   |
|   |
+---+
|   |
| 8 |   hash(key)
|   |
|   |
+---+
| 2 |   len(key)
|   |
+---+
|   |
|   |   key
|   |
|   |
+---+
|   |
|   |
|   |
|   |
|   |   value
|   |
|   |
|   |
+---+
```


### set()

首先, `set()` 会加锁的。

假设初始时 array 的结构如下:

```
 0 1                                       100
+-+-+---------------------------------------+
| | |                                       |
+-+-+---------------------------------------+
   ^
   |
   +
  head
  tail
rightMargin
```

现在添加一条数据: key 是 `key-0`, value 是 `value - 0`, 序列化成 entry 后大小是 32(8+8+2+5+9)。除了 entry 外还需要用 4 字节标记 entry 的大小, 所以需要占用的大小是 36, 写入后 tail 和 rightMargin 移动到 37 的位置：

```
 0 1                37                     100
+-+-+--------------+-+----------------------+
| | |              | |                      |
+-+-+--------------+-+----------------------+
   ^                ^
   |                |
   +                +
  head             tail
               rightMargin
```


假如现在要写一条数据, 占用 100 字节大小, 此时剩余空间不够用了, 需要进行扩容。 扩容的伪代码如下:

```go
// minumum 是新数据占的大小， 如果 capacity < minimum, 那么再多 capacity 的大小也是不够的, 所以需要先 += minimum。
if capacity < minimum { capacity += minimum }
capacity = capacity * 2
// 如果设置了最大容量，则不能超过最大容量
if maxCapacity > 0 && (capacity > maxCapacity) { capacity = maxCapacity }
if rightMargin != 1 { // == 1 说明没有数据
    copy(array, oldArray[:rightMargin]) // 拷贝有效数据
}
```

分配了新内存后就可以继续添加数据了。


------------

在调用 `set()` 时, 每次都会获取 head 指向的 entry, 检查其是否过期了, 如果过期了, 则将其删除, 并且 head 会向后移, 当 head 移动到 rightMargin 的位置时, 说明 head 之后没有有效数据了, 会将 head 移动到 1 的位置, 如果 tail 也和 rightMargin 相同, 说明所有数据都空了, 也会将 tail 移动到 1 的位置。 重置位置后, rightMargin 和 tail 保持一致。 伪代码如下(`BytesQueue.Pop()`):

```go
head += headerEntrySize + size // 后移, headerEntrySize 是 entry 大小占的空间, 为 4 字节
count--                        // entry 数量自减

if head == rightMargin { // head 之后没有有效数据了, 重置 head
	head = 1
	if tail == rightMargin { tail = 1 } // head == tail == rightMargin, 说明没有 entry 存在了
	rightMargin = tail
}
```


--------------

假设空间设置为 100, 添加两个 entry（1个是 `key1, 46B data`, 1个是 `key2, 1B data`）， 此时 tail 移动到100位置， 再添加一个 entry(`key3, 10B data`), 这个 entry 添加时第一个 entry 已经过期被回收了， 此时 BytesQueue 不是直接分配更大的内存， 而是会复用第一个 entry 的空间， 把 tail 指针直接移动到 1 的位置, 第三个 entry 就从这个位置开始写。写完第三个 entry 后的指针分布是这样的:

```
  1       37                   73            100
+--------------------------------------------+
| |ddddddd|                     |dddddddddddd|
+--------------------------------------------+
          ^                     ^            ^
          |                     |            |
          +                     +            +
         tail                 head        rightMargin
```


---------------

如果是相同的 key 连续两次被 set， 则前面的 key 对应的 entry 的 `hash(key)` 部分会被置为 0， 除此之外没有其他操作了。




### get()

从 hashmap 中查询 index， 根据 index 获取 entry， 比较 key 是否相同。


### del()

从 hashmap 中查询 index, 根据 index 获取 entry, 从 hashmap 中删除 key， 调用用户的函数， 将 entry 的 hash 部分置为0.


## 淘汰算法

这个没有淘汰算法， 被删除的永远是最先放进去的， 不管是不是被频繁访问。
