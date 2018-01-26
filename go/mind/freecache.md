<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [freecache](#freecache)
	- [结构](#结构)
	- [segment](#segment)
		- [set()](#set)
			- [淘汰算法](#淘汰算法)
		- [get()](#get)
	- [RingBuf](#ringbuf)
		- [Write()](#write)
		- [ReadAt()](#readat)
		- [Evacuate()](#evacuate)

<!-- /TOC -->

# freecache


## 结构

一个 Cache 有 256 个 segment, 每个 segment 有一把锁。

一个 segment 有 `256 * n` 个 slot, 每个 slot 存放的是 entry 的指针信息。槽的数量是动态调整的, 如果同一 slotID 对应的槽的数量超过或等于 n 了, 那么槽的数量就会增多一倍。

segment 的结构如下:

```go
type segment struct {
	rb            RingBuf // 存放具体的数据
	segId         int
	entryCount    int64      // entry 的数量
	totalCount    int64      // entry 的数量, 包括删除的entry
	totalTime     int64      // 时间总和, 用于计算平均时间
	vacuumLen     int64      // 空闲空间, 空闲空间不够的话需要覆盖旧数据
	slotLens      [256]int32 // 每个槽的实际长度
	slotCap       int32      // 每个指针在槽中占用多大空间, 初始时是1
	slotsData     []entryPtr // 指针数组
}
```

指针记录了 entry 在 RingBuf 中的信息, 指针结构如下:

```go
type entryPtr struct {
	offset   int64  // 在 RingBuf 中的偏移位置
	hash16   uint16 // 用于排序的, RingBuf 中的 entry 根据这个字段排序, 查找时使用二分查找。
	keyLen   uint16 // key 长度, 用于比较指针指向的 entry 是不是想要的(主要是可以在不查找 entry 的情况下排除掉一些不匹配的)
	reserved uint32
}
```

具体的数据放在了 RingBuf 中, RingBuf 是一个环形缓存, 底层是一个字节数组, 数组按序写入, 写满后重新从0位置开始写。


## segment


### set()

slotID 和 hash16 的计算(hashVal 是 `hash(key)` 的值):

```go
slotID := uint8(hashVal >> 8)
hash16 := uint16(hashVal >> 16)
```

在一个初始状态的 segment 中写入数据的过程是这样的:

```go
// 查找具有相同 slotID 的 slot, 因为相同 id 的都放在一起了, 像这样: `| idx | idx | id-ok | id-ok | idx | idx | ...`, 所以可以直接根据 slotID 来计算出偏移量
slotOff := slotID * slotCap
slot := findSlotsSameID(slotsData, slotID, slotOff)
// 查找新的 entry 应该在哪个索引位置, 这里使用了二分查找来加速, 根据 hash16 加速
idx := lookupSlotIndex(slot, key, hash16)
// 把新的 entryPtr 插入到 idx 位置, 保证在 slot 中是按照 hash16 排序的。
insertEntryPtr(idx)
// 写入到 RingBuf 中的数据按先后顺序分别是: hdr, key, value, [一些空格], 这里的空格可能发生的情况是 value 为空时。
insertDataToRingBuf()
```

- 怎么按照 hash16 排序?
    - 先根据 key 和 hash16 通过二分查找来找到应该放在哪个位置 idx, 然后插入数据的时候插入到 idx 中, 这样就保证了相同 slotID 的那一块数据是按照 hash16 来排序的。
- 插入 entryPtr 时的扩容问题?
    - 插入数据时会检测 slotID 对应的槽中有多少 ptr 了, 如果 `slotLen[id] == slotCap` 了, 再插入 ptr 时就会覆盖旧的 ptr, 所以这里会有一个扩容的操作。
    - 扩容很粗暴, 直接将 `slotCap * 2`, 然后原来的 ptr 放到正确的位置上。 比如现在是 `|id1|id1|id2|id2|`, 扩容后 slotCap 变为 4, 变成 `|id1|id1| | |id2|id2| | |`。


hdr 的结构如下:

```go
type entryHdr struct {
	accessTime uint32
	expireAt   uint32
	keyLen     uint16
	hash16     uint16
	valLen     uint32
	valCap     uint32
	deleted    bool
	slotId     uint8
	reserved   uint16
}
```


如果前后两个 entry 的 key 相同, 那么前一个 entry 会被删除。 这里的删除分两种情况:
1. 新的 value 长度小于等于旧的 value 长度。 这意味着旧 entry 在 RingBuf 中的空间可以被复用, 只需要修改 entryHdr 的信息以及用新 value 覆盖旧 value 即可。
2. 新的 value 长度大于旧的 value 长度。此时会将旧 entry 软删除(entryHdr的deleted字段置为true, entryPtr从slotsData中删除)。然后再将新 entry 插入到 RingBuf 中, 将 entryHdr 插入到 slotsData 中。


#### 淘汰算法

淘汰发生在 `set()` 被调用时, 会检查空闲空间是否能容得下新数据。

```go
func (seg *segment) evacuate(entryLen int64, slotId uint8, now uint32) (slotModified bool) {
	consecutiveEvacuate := 0 // 连续将有效 entry 重写的次数
	for seg.vacuumLen < entryLen {
		// 软删除的 和 过期的, vacuumLen 会增加被删除的 entryLen, 所以下次 oldOff 可以指向下一个 entry.
		// 重写 entry 的话会修改 end, 但 vacuumLen 不变, 所以下次 oldOff 也可以指向下一个 entry.
		oldOff := seg.rb.End() + seg.vacuumLen - seg.rb.Size()
        oldEntry := readEntryByOff(oldOff)
		// 被软删除了， 回收空间
		if oldEntry.deleted {
			consecutiveEvacuate = 0
			seg.vacuumLen += oldEntryLen
			continue
		}
		// 过期的, 或者最后一次访问时间小于等于平均访问时间的, 或者连续重写 entry 的次数超过 5 次
		expired := oldHdr.expireAt != 0 && oldHdr.expireAt < now
		leastRecentUsed := int64(oldHdr.accessTime)*seg.totalCount <= seg.totalTime // accessTime <= totalTime / totalCount
		if expired || leastRecentUsed || consecutiveEvacuate > 5 {
            // 删除 ptr 并且标记 entry 被删除了
			seg.delEntryPtr(oldHdr.slotId, oldHdr.hash16, oldOff)
			if oldHdr.slotId == slotId {
				slotModified = true
			}
			consecutiveEvacuate = 0
			seg.vacuumLen += oldEntryLen
		} else {
			// evacuate an old entry that has been accessed recently for better cache hit rate.
			// 将 oldOff 对应的 entry 追加到 ringBuf 后, 相当于重新写了一遍。
			newOff := seg.rb.Evacuate(oldOff, int(oldEntryLen))
			// 更新指针的偏移
			seg.updateEntryPtr(oldHdr.slotId, oldHdr.hash16, oldOff, newOff)
			consecutiveEvacuate++
		}
	}
	return
}
```

在检查空余空间时会将没有被删除的且没有过期的 entry 从前面重写到 RingBuf 后面, 防止被新的数据覆盖。




### get()

```go
hdr := lookupHdr(slotsData, key, hash16) // 二分查找
if expired(hdr) {
    // 软删除, 标记 hdr.deleted, 删除 entryPtr
    delEntryPtr()
    return
}
// 设置 hdr.accessTime 为当前时间
setHdrAccessTime(now)
return readValue()
```



## RingBuf

### Write()

代码:

```go
func (rb *RingBuf) Write(p []byte) (n int, err error) {
	if len(p) > len(rb.data) {
		err = ErrOutOfRange
		return
	}
	for n < len(p) {
		written := copy(rb.data[rb.index:], p[n:])
		rb.end += int64(written)
		n += written
		rb.index += written
		if rb.index >= len(rb.data) {
			rb.index -= len(rb.data) // 只重置了 index， 没有重置 end
		}
	}
    // begin 和 end 最多相距 len(data)
	if int(rb.end-rb.begin) > len(rb.data) {
		rb.begin = rb.end - int64(len(rb.data))
	}
	return
}
```

假设 RingBuf 大小是 16， 初始状态：

```
0                                                 16
+--------------------------------------------------+
|                                                  |
+--------------------------------------------------+
^
+
begin, end, index
```

写了 10 个字节后：

```
0                               10                16
+--------------------------------+-----------------+
|                                |                 |
+--------------------------------+-----------------+
^                                ^
+                                +
begin                          end, index
```

再写 10 个字节后:

```
0      4                                          16             20
+------+-------------------------------------------+              +
|      |                                           |              |
+------+-------------------------------------------+              +
       ^                                                          ^
       +                                                          +
    begin, index                                                 end
```


### ReadAt()

代码:

```go
func (rb *RingBuf) ReadAt(p []byte, off int64) (n int, err error) {
	if off > rb.end || off < rb.begin {
		err = ErrOutOfRange
		return
	}
	// off 是相对于 begin 而言的, off - begin 就是从有效数据的某个偏移位置开始读
	var readOff int
	if rb.end-rb.begin < int64(len(rb.data)) { // 说明 end 还没有超出 data 的范围
		readOff = int(off - rb.begin) // 有效数据从 0 开始的
	} else {
		readOff = rb.index + int(off-rb.begin) // 有效数据从 index 开始, index 是有效数据的第一个位置, 也是下一个要覆盖写的位置
	}
	if readOff >= len(rb.data) {
		readOff -= len(rb.data)
	}
	readEnd := readOff + int(rb.end-off) // 可被读取的数据最长为 end-off.
	if readEnd <= len(rb.data) { // 没有超出实际边界
		n = copy(p, rb.data[readOff:readEnd])
	} else { // 超出实际边界了要分两次读取
		n = copy(p, rb.data[readOff:])
		if n < len(p) {
			// readEnd - len(data) 是从 0 开始的有效数据的长度
			n += copy(p[n:], rb.data[:readEnd-len(rb.data)])
		}
	}
	if n < len(p) {
		err = io.EOF
	}
	return
}
```

假设现在写了 10 个字节, 要全部读取出来, 可能会调用 `ReadAt(p, 0)`, 其中 `len(p) == 10`:

```
0                               10                16
+--------------------------------+-----------------+
|                                |                 |
+--------------------------------+-----------------+
^                                ^
+                                +
begin                          end, index
```

此时读取的位置是 `data[0:10]`

再次放入 10 个字节后, 状态变为:

```
0      4                                          16             20
+------+-------------------------------------------+              +
|      |                                           |              |
+------+-------------------------------------------+              +
       ^                                                          ^
       +                                                          +
    begin, index                                                 end
```

将刚才写入的 10 个字节读出来, 调用 `ReadAt(p, 10)`, 其中 `len(p) == 10`, 此时读取的位置是 `data[10: 20]`, 分两次读取, 先读取 `data[10:16]`, 再读取 `data[0:4]`。



### Evacuate()

Evacuate 读取特定位置的数据重新写到 RingBuffer 的后面。

```go
func (rb *RingBuf) Evacuate(off int64, length int) (newOff int64) {
	if off+int64(length) > rb.end || off < rb.begin {
		return -1
	}
    // 转换 off
	var readOff int
	if rb.end-rb.begin < int64(len(rb.data)) {
		readOff = int(off - rb.begin)
	} else {
		readOff = rb.index + int(off-rb.begin)
	}
	if readOff >= len(rb.data) {
		readOff -= len(rb.data)
	}

	if readOff == rb.index {
		// 读取的位置正好是有效数据开始的位置, 不需要复制了, 直接移动指针就可以了
		rb.index += length
		if rb.index >= len(rb.data) {
			rb.index -= len(rb.data)
		}
	} else if readOff < rb.index {
		// 要读取的数据在 index 前面
		// 如果 readOff + length 超过了有效数据的范围了呢? 不会的, 上面会检测 off+length 是否在有效数据范围内的
		var n = copy(rb.data[rb.index:], rb.data[readOff:readOff+length])
		rb.index += n
		if rb.index == len(rb.data) {
			// data 已经写到底了, 读取的数据还不够 length, 读取剩余的数据到 data 开始的位置
			rb.index = copy(rb.data, rb.data[readOff+n:readOff+length])
		}
	} else {
		// 读取的位置在 index 后面
		var readEnd = readOff + length
		var n int
		if readEnd <= len(rb.data) {
			// 不用转弯, 要读取的数据在 index - len(data) 之间
			n = copy(rb.data[rb.index:], rb.data[readOff:readEnd])
			rb.index += n
			if rb.index == len(rb.data) {
				// 既然 readOff > index, readEnd <= len(data), 那这里 index 不可能达到 len(data) 的位置啊
				rb.index = copy(rb.data, rb.data[readOff+n:readEnd])
			}
		} else {
			// 需要转弯
			n = copy(rb.data[rb.index:], rb.data[readOff:])
			rb.index += n
			var tail = length - n // 还有多少没读的
			n = copy(rb.data[rb.index:], rb.data[:tail])
			rb.index += n
			if rb.index == len(rb.data) { // 写到底了还没有写完
				rb.index = copy(rb.data, rb.data[n:tail])
			}
		}
	}
	// 更新 end 和 begin, 确保 begin 和 end 最多相差 len(data) 的长度
	newOff = rb.end
	rb.end += int64(length)
	if rb.begin < rb.end-int64(len(rb.data)) {
		rb.begin = rb.end - int64(len(rb.data))
	}
	return
}
```

基本思路是确定从 data 的哪个位置开始读取数据, 然后写到 index 后面。
