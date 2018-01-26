<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [md5](#md5)
- [sha256](#sha256)
- [sha3](#sha3)
- [hmac](#hmac)
- [DES 加密](#des-加密)
	- [3DES](#3des)
- [AES](#aes)
- [AEAD](#aead)
- [参考](#参考)

<!-- /TOC -->


# md5

- 方式1

 ```go
 md5.Sum([]byte)
 ```

- 方式2

 ```go
 hash := md5.New()
 hash.Write([]byte)
 hash.Sum(nil)
 ```



不同：

1. 方式1返回的是 `[Size]byte` 类型的数组而不是切片
2. 方式2: Write函数会把MD5对象内部的字符串clear掉，然后把其参数作为新的内部字符串。而Sum函数则是先计算出内部字符串的MD5值，而后把输入参数附加到内部字符串后面。



坑：

```go
md5.New().Sum([]byte)
```

这种方式是先加密空字符串，然后将结果附加到了Sum的参数中，和没有加密一样。


# sha256

安全散列算法SHA（Secure Hash Algorithm）是美国国家安全局 （NSA） 设计，美国国家标准与技术研究院（NIST） 发布的一系列密码散列函数，包括 SHA-1、SHA-224、SHA-256、SHA-384 和 SHA-512 等变体。主要适用于数字签名标准（DigitalSignature Standard DSS）里面定义的数字签名算法（Digital Signature Algorithm DSA）。


摘抄自 golang 文档中的示例：

```go
import (
	"crypto/sha256"
	"fmt"
)

func main() {
	h := sha256.New()
	h.Write([]byte("hello world\n"))
	fmt.Printf("%x", h.Sum(nil))
}
```

对文件：

```go
import (
	"crypto/sha256"
	"fmt"
	"io"
	"log"
	"os"
)

func main() {
	f, err := os.Open("file.txt")
	if err != nil {
		log.Fatal(err)
	}
	defer f.Close()

	h := sha256.New()
	if _, err := io.Copy(h, f); err != nil {
		log.Fatal(err)
	}

	fmt.Printf("%x", h.Sum(nil))
}
```






# sha3

```go
import (
	"encoding/base64"
	"golang.org/x/crypto/sha3"
)

// 对用户密码进行编码。若密码明文为空字符串，则也会返回空字符串。
func EncodePassword(cleartext string) string {
	if cleartext == "" {
		return ""
	}
	h := make([]byte, 64)
	sha3.ShakeSum256(h, []byte(cleartext))
	return base64.StdEncoding.EncodeToString(h)
}
```


# hmac

HMAC是密钥相关的哈希运算消息认证码，HMAC运算利用哈希算法，以一个密钥和一个消息为输入，生成一个消息摘要作为输出。

```go
// CheckMAC reports whether messageMAC is a valid HMAC tag for message.
func CheckMAC(message, messageMAC, key []byte) bool {
	mac := hmac.New(sha256.New, key)
	mac.Write(message)
	expectedMAC := mac.Sum(nil)
	return hmac.Equal(messageMAC, expectedMAC)
}
```



# DES 加密

DES(Data Encryption Standard), 对称加密算法，加密和解密使用同一密钥。

DES加密，涉及到加密模式和填充方式，所以，和其他语言加解密时，应该约定好加密模式和填充方式。（模式定义了Cipher如何应用加密算法。改变模式可以容许一个块加密程序变为流加密程序。）

DES算法的入口参数有三个：Key、Data、Mode。
- Key 必须为 8 个字节共 64 位，是 DES 算法的工作密钥, 实际只用到 56 位, 有 8 位是奇偶校验位。
- Data 也为 8 个字节 64 位的倍数，是要被加密或被解密的数据；
- Mode 为 DES 的工作方式，有两种：加密或解密。

DES 算法主要采用替换和移位的方式进行加密，它用56位（64位密钥只有56位有效）对64位二进制数据块进行加密，每次加密对64位的输入数据进行16轮编码，经过一系列替换和移位后，输入的64位原数据转换成完全不同的64位输出数据。

DES 工作模式：
- ECB(电子密码密码本模式): 这是最原始的一种加密工作模式，将明文分组成64位，与密钥长度相同，然后按照加密算法加密，得到 64位密文，最后将所有加密后的密文连接在一起即可，各段之间互不影响（当最后一段不足64位是要补足64位在进行计算）。
    - 明文中重复的排列会反映在密文中, 并且，当密文被篡改时，解密后对应的明文分组也会出错，且解密者察觉不到密文被篡改了。也就是说，ECB不能提供对密文的完整性校验。因此，在任何情况下都不推荐使用ECB模式。
- CBC(密文链接模式): 加密过程如下：
    1. 首先将数据按照8个字节一组进行分组得到D1,D2,...,Dn（若数据不是8的整数倍，用指定的PADDING数据补位）
    2. 第一组数据D1与初始化向量I异或后的结果进行DES加密得到第一组密文C1（初始化向量I为全零）
    3. 第二组数据D2与第一组的加密结果C1异或以后的结果进行DES加密，得到第二组密文C2
    4. 之后的数据以此类推，得到Cn
    5. 按顺序连为C1C2C3......Cn即为加密结果
- CFB(加密反馈模式)
- OFB(输出反馈模式)
- CTR(CounTeR 计数器模式):
    - 在计数器模式下，我们不再对密文进行加密，而是对一个逐次累加的计数器进行加密，用加密后的比特序列与明文分组进行 XOR 得到密文。
    - 每次与明文分组进行XOR的比特序列是不同的，因此，计数器模式解决了ECB模式中，相同的明文会得到相同的密文的问题。
- MAC(Message Authentication Code 消息验证码):
    - 密文的收发双发需要提前共享一个秘钥。密文发送者将密文的MAC值随密文一起发送，密文接收者通过共享秘钥计算收到密文的MAC值，这样就可以对收到的密文做完整性校验。当篡改者篡改密文后，没有共享秘钥，就无法计算出篡改后的密文的MAC值。
    - 如果生成密文的加密模式是CTR，或者是其他有初始IV的加密模式，别忘了将初始的计时器或初始向量的值作为附加消息与密文一起计算MAC。
- GMAC(Galois message authentication code mode, 伽罗瓦消息验证码):
    - GMAC就是利用伽罗华域(Galois Field，GF，有限域)乘法运算来计算消息的MAC值。假设秘钥长度为128bits, 当密文大于128bits时，需要将密文按128bits进行分组。
- GCM(Galois/Counter Mode):
    - GCM 中的 G 就是指 GMAC，C 就是指 CTR。
    - GCM 可以提供对消息的加密和完整性校验，另外，它还可以提供附加消息的完整性校验。在实际应用场景中，有些信息是我们不需要保密，但信息的接收者需要确认它的真实性的，例如源IP，源端口，目的IP，IV，等等。因此，我们可以将这一部分作为附加消息加入到MAC值的计算当中。


DES 填充模式：
- NoPadding: API或算法本身不对数据进行处理，加密数据由加密双方约定填补算法。例如若对字符串数据进行加解密，可以补充`\0`或者空格，然后trim。
- PKCS5Padding: 当明文不是8字节的倍数时需要需要补充为8字节的倍数。比如差3字节才构成 `8*n` 字节, 则在最后加上 3个 **3**.



Go 中 `crypto/des` 包实现了 Data Encryption Standard (DES) 和 the Triple Data Encryption Algorithm (TDEA)。

初始化使用下面两个函数中的一个：

```go
func NewCipher(key []byte) (cipher.Block, error)          // DES
func NewTripleDESCipher(key []byte) (cipher.Block, error) // 3DES
```

`crypto/cipher` 包实现了标准的块加密模式。 `Block` 是一个接口：

```go
type Block interface {
    // BlockSize returns the cipher's block size.
    BlockSize() int

    // Encrypt encrypts the first block in src into dst.
    // Dst and src may point at the same memory.
    Encrypt(dst, src []byte)

    // Decrypt decrypts the first block in src into dst.
    // Dst and src may point at the same memory.
    Decrypt(dst, src []byte)
}
```

而 `BlockMode` 代表各种模式：

```go
type BlockMode interface {
    // BlockSize returns the mode's block size.
    BlockSize() int

    // CryptBlocks encrypts or decrypts a number of blocks. The length of
    // src must be a multiple of the block size. Dst and src may point to
    // the same memory.
    CryptBlocks(dst, src []byte)
}
```

获取 `BlockMode` 实例的两种方法：

```go
func NewCBCDecrypter(b Block, iv []byte) BlockMode // CBC 解密
func NewCBCEncrypter(b Block, iv []byte) BlockMode // CBC 加密
```

按流方式加密也定义了一个接口：

```go
type Stream interface {
    // XORKeyStream XORs each byte in the given slice with a byte from the
    // cipher's key stream. Dst and src may point to the same memory.
    XORKeyStream(dst, src []byte)
}
```

以下是 DES 加解密的示例：

```go
func Demo() {
	data := "hello, sldfjdklsdfjsldkfjsldkfjlskdfjsld"
	key := []byte("12345678") // 必须是 8 个字节，64 bit
	encryptedData, err := DesEncrypt([]byte(data), key)
	if err != nil {
		log.Fatal(err)
	}

	data2, err := DesDecrypt(encryptedData, key)
	if err != nil {
		log.Fatal(err)
	}
	data2Text := string(data2)
	if data2Text != data {
		fmt.Println("failed!")
	}
	fmt.Println(data)
	fmt.Println(data2Text)
}

// DesEncrypt des加密
func DesEncrypt(origData, key []byte) ([]byte, error) {
	block, err := des.NewCipher(key)
	if err != nil {
		return nil, err
	}
	origData = PKCS5Padding(origData, block.BlockSize())
	blockMode := cipher.NewCBCEncrypter(block, key) // 这里使用 key 作为初始向量, 最好用随机的数据
	crypted := make([]byte, len(origData))
	// 根据CryptBlocks方法的说明，如下方式初始化crypted也可以
	// crypted := origData
	blockMode.CryptBlocks(crypted, origData)
	return crypted, nil
}

// DesDecrypt des解密
func DesDecrypt(crypted, key []byte) ([]byte, error) {
	block, err := des.NewCipher(key)
	if err != nil {
		return nil, err
	}
	blockMode := cipher.NewCBCDecrypter(block, key)
	origData := make([]byte, len(crypted))
	// origData := crypted
	blockMode.CryptBlocks(origData, crypted)
	origData = PKCS5UnPadding(origData)
	// origData = ZeroUnPadding(origData)
	return origData, nil
}

// PKCS5Padding padding
func PKCS5Padding(ciphertext []byte, blockSize int) []byte {
	padding := blockSize - len(ciphertext)%blockSize
	padtext := bytes.Repeat([]byte{byte(padding)}, padding)
	return append(ciphertext, padtext...)
}

// PKCS5UnPadding unpadding
func PKCS5UnPadding(origData []byte) []byte {
	length := len(origData)
	// 去掉最后一个字节 unpadding 次
	unpadding := int(origData[length-1])
	return origData[:(length - unpadding)]
}
```


## 3DES

3DES 是使用 168 位密钥对数据进行三次加密。

代码示例：

```go
// 3DES加密
func TripleDesEncrypt(origData, key []byte) ([]byte, error) {
     block, err := des.NewTripleDESCipher(key) // 密钥长度必须是 24byte.
     if err != nil {
          return nil, err
     }
     origData = PKCS5Padding(origData, block.BlockSize())
     blockMode := cipher.NewCBCEncrypter(block, key[:8]) // 初始化向量长度必须是 8byte。
     crypted := make([]byte, len(origData))
     blockMode.CryptBlocks(crypted, origData)
     return crypted, nil
}
```



# AES

AES(Advanced Encryption Standard, 高级加密标准), 又称Rijndael加密法, 对称加密算法，用来替代原来的 DES 算法(DES 算法实际使用 56bit 密钥，安全性不够高)。

AES加密数据块分组长度必须为128bit(16byte)，密钥长度可以是128bit、192bit、256bit中的任意一个。

AES 加解密示例：

```go
func demo() {
	data := "hello, sldfjdklsdfjsldkfjsldkfjlskdfjsld"
	key := []byte("1234567812345678")
	encryptedData, err := AesEncrypt([]byte(data), key)
	if err != nil {
		log.Fatal(err)
	}

	data2, err := AesDecrypt(encryptedData, key)
	if err != nil {
		log.Fatal(err)
	}
	data2Text := string(data2)
	if data2Text != data {
		fmt.Println("failed!")
	}
	fmt.Println(data)
	fmt.Println(data2Text)
}

// AesEncrypt AES 加密。
func AesEncrypt(origData, key []byte) ([]byte, error) {
	block, err := aes.NewCipher(key)
	if err != nil {
		return nil, err
	}
	blockSize := block.BlockSize()
	origData = PKCS5Padding(origData, blockSize)
	blockMode := cipher.NewCBCEncrypter(block, key[:blockSize])
	crypted := make([]byte, len(origData))
	// 根据CryptBlocks方法的说明，如下方式初始化crypted也可以
	// crypted := origData
	blockMode.CryptBlocks(crypted, origData)
	return crypted, nil
}

// AesDecrypt AES 解密。
func AesDecrypt(crypted, key []byte) ([]byte, error) {
	block, err := aes.NewCipher(key)
	if err != nil {
		return nil, err
	}
	blockSize := block.BlockSize()
	blockMode := cipher.NewCBCDecrypter(block, key[:blockSize])
	origData := make([]byte, len(crypted))
	// origData := crypted
	blockMode.CryptBlocks(origData, crypted)
	origData = PKCS5UnPadding(origData)
	return origData, nil
}

// PKCS5Padding padding
func PKCS5Padding(ciphertext []byte, blockSize int) []byte {
	padding := blockSize - len(ciphertext)%blockSize
	padtext := bytes.Repeat([]byte{byte(padding)}, padding)
	return append(ciphertext, padtext...)
}

// PKCS5UnPadding unpadding
func PKCS5UnPadding(origData []byte) []byte {
	length := len(origData)
	// 去掉最后一个字节 unpadding 次
	unpadding := int(origData[length-1])
	return origData[:(length - unpadding)]
}
```


# AEAD

AEAD 接口是一种提供了使用关联数据进行认证加密的功能的加密模式。接口声明如下：

```go
type AEAD interface {
    // 返回提供给Seal和Open方法的随机数nonce的字节长度
    NonceSize() int
    // 返回原始文本和加密文本的最大长度差异
    Overhead() int
    // 加密并认证明文，认证附加的data，将结果添加到dst，返回更新后的切片。
    // nonce的长度必须是NonceSize()字节，且对给定的key和时间都是独一无二的。
    // plaintext和dst可以是同一个切片，也可以不同。
    Seal(dst, nonce, plaintext, data []byte) []byte
    // 解密密文并认证，认证附加的data，如果认证成功，将明文添加到dst，返回更新后的切片。
    // nonce的长度必须是NonceSize()字节，nonce和data都必须和加密时使用的相同。
    // ciphertext和dst可以是同一个切片，也可以不同。
    Open(dst, nonce, ciphertext, data []byte) ([]byte, error)
}
```

要得到 AEAD 可以使用下面的方法：
- `func NewGCM(cipher Block) (AEAD, error)`: 用迦洛瓦计数器模式包装提供的128位Block接口，并返回AEAD接口。


加解密示例：

```go
func main() {
	// The key argument should be the AES key, either 16 or 32 bytes
	// to select AES-128 or AES-256.
	key := []byte("AES256Key-32Characters1234567890")
	plainText := []byte("exampleplaintext")
	cipherText, nonce, err := GCMEncrypt(key, plainText)
	if err != nil {
		panic(err)
	}
	fmt.Printf("cipherText: %x, nonce: %x\n", cipherText, nonce)
	plainText2, err := GCMDecrypt(key, cipherText, nonce)
	if err != nil {
		panic(err)
	}
	fmt.Printf("plainText: %s, plainText2: %s\n", string(plainText), string(plainText2))
}

// GCMEncrypt gcm 加密
func GCMEncrypt(key, plainText []byte) (cipherText, nonce []byte, err error) {
	block, err := aes.NewCipher(key)
	if err != nil {
		return
	}

	aesgcm, err := cipher.NewGCM(block)
	if err != nil {
		return
	}

	// Never use more than 2^32 random nonces with a given key because of the risk of a repeat.
	nonce = make([]byte, aesgcm.NonceSize())
	if _, err = io.ReadFull(rand.Reader, nonce); err != nil {
		return
	}

	cipherText = aesgcm.Seal(nil, nonce, plainText, nil) // 附件数据这里没有加
	return
}

// GCMDecrypt gcm 解密
func GCMDecrypt(key, cipherText, nonce []byte) (plainText []byte, err error) {
	block, err := aes.NewCipher(key)
	if err != nil {
		return
	}

	aesgcm, err := cipher.NewGCM(block)
	if err != nil {
		return
	}

	plainText, err = aesgcm.Open(nil, nonce, cipherText, nil) // 附件数据这里没有加
	return
}
```


# 参考
- [什么是DES加密？知乎](https://www.zhihu.com/question/36767829)
- [DES算法的详细使用-慕课网](http://www.imooc.com/article/15997)
- [GO加密解密之DES-studygolang](http://blog.studygolang.com/2013/01/go%E5%8A%A0%E5%AF%86%E8%A7%A3%E5%AF%86%E4%B9%8Bdes/)
- [GO加密解密之AES-studygolang](http://blog.studygolang.com/2013/01/go%E5%8A%A0%E5%AF%86%E8%A7%A3%E5%AF%86%E4%B9%8Baes/)
- [什么是 AES-GCM加密算法](http://blog.csdn.net/t0mato_/article/details/53160772)
- [对称加密和分组加密中的四种模式(ECB、CBC、CFB、OFB)](http://www.cnblogs.com/adylee/archive/2007/09/14/893438.html)
- [什么是安全散列算法SHA256？](http://8btc.com/article-136-1.html)
