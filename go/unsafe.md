<!-- TOC -->

- [[]byte ==> struct](#byte--struct)

<!-- /TOC -->


# []byte ==> struct

分配[]byte，转成struct（from free-cache）

```go
type entryHdr struct {
	accessTime uint32
	expireAt   uint32
	keyLen     uint16
	hash16     uint16
	valLen     uint32
	valCap     uint32
	deleted    bool // 占8bit，一个字节
	slotId     uint8
	reserved   uint16
}

func main() {
	var hdrBuf [24]byte // entryHdr 一共占24字节
	hdr := (*entryHdr)(unsafe.Pointer(&hdrBuf[0]))
	hdr.accessTime = 1
	hdr.expireAt = 2
	hdr.keyLen = 3
	hdr.hash16 = 4
	hdr.valLen = 5
	hdr.valCap = 6
	hdr.deleted = true
	hdr.slotId = 8
	hdr.reserved = 9
	fmt.Println(hdrBuf) // [1 0 0 0 2 0 0 0 3 0 4 0 5 0 0 0 6 0 0 0 1 8 9 0]
}
```

