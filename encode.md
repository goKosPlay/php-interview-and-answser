### 现在加密分为哪几种
```
加密技术主要分为以下几种类型，每种类型下都有多种具体算法，包括 AES（高级加密标准）。以下是分类及常见算法的简要概述：

1. 对称加密（Symmetric Encryption）
特点：加密和解密使用同一密钥，速度快，适合处理大量数据。
常见算法（包含 AES）：
AES（Advanced Encryption Standard）：密钥长度为 128、192 或 256 位，高效、安全，广泛用于 SSL/TLS、VPN 和磁盘加密等场景。
DES（Data Encryption Standard）：56位密钥，安全性较低，已被淘汰。
3DES（Triple DES）：对 DES 进行三次加密，密钥长度更长，但仍较慢。
Blowfish：密钥长度可变（32-448位），适合小规模数据加密。
Twofish：Blowfish 的后继者，密钥长度最高256位，速度快且安全。
RC4：流加密算法，简单但现已被认为不安全（如在 WEP 中使用）。
ChaCha：由 Daniel J. Bernstein 设计，适合移动设备，安全性高且性能优异。
2. 非对称加密（Asymmetric Encryption）
特点：使用公钥加密，私钥解密，安全性高但速度慢，适合小数据量或密钥交换。
常见算法：
RSA：基于大整数分解难题，广泛用于密钥交换和数字签名。
ECC（椭圆曲线加密，Elliptic Curve Cryptography）：基于椭圆曲线数学，比 RSA 更高效，密钥长度短但安全性高。
Diffie-Hellman（DH）：用于安全密钥交换，而非直接加密数据。
ElGamal：基于离散对数问题，用于加密和数字签名。
DSA（Digital Signature Algorithm）：主要用于数字签名，而非数据加密。
3. 哈希函数（Hash Functions）
特点：将任意长度的数据映射为固定长度的值，不可逆，用于数据完整性验证和密码存储。
常见算法：
MD5：128位输出，速度快但安全性低（易受碰撞攻击）。
SHA-1：160位输出，安全性已被削弱，不推荐使用。
SHA-2：包括 SHA-256、SHA-512 等，安全性较高，广泛使用。
SHA-3：基于 Keccak 算法，抗量子攻击潜力更高。
Bcrypt：专为密码哈希设计，加入盐值和计算成本，安全性高。
Argon2：密码哈希竞赛胜者，适合现代密码存储。
4. 其他加密技术
混合加密：结合对称和非对称加密，如 HTTPS 中用 RSA 交换 AES 密钥。
流加密：逐位或逐字节加密数据，如 RC4 或 ChaCha。
分组加密：将数据分成固定大小的块加密，如 AES、DES。
量子加密：基于量子力学原理（如 BB84 协议），未来潜力大但尚未普及。
总结
加密算法的选择取决于场景：AES 是对称加密的标准代表，高效且安全；其他如 RSA 用于非对称场景，SHA-2 用于哈希验证。每种算法有其优缺点，选择时需考虑安全性、性能和兼容性。如果您想了解特定算法的实现细节或应用示例，请提供更多信息！


```

### AES如何加密
```
AES（高级加密标准，Advanced Encryption Standard）是一种对称加密算法，广泛用于数据加密。以下是 AES 加密的简要工作原理和步骤，尽量简洁但覆盖核心内容：
AES 加密概述

类型：对称加密，使用同一密钥进行加密和解密。
密钥长度：128位、192位或256位。
分组长度：固定为128位（16字节）。
模式：常见操作模式包括 ECB、CBC、GCM 等（影响数据块的处理方式）。
用途：保护数据机密性，如 SSL/TLS、VPN、磁盘加密等。

AES 加密步骤
AES 采用分组密码，将输入数据分成 128 位（16 字节）块，逐块加密。核心是一个基于替换-置换网络（Substitution-Permutation Network）的算法，包含多轮变换。以下是加密流程：

初始化：

输入：明文（128位数据块）和密钥（128/192/256位）。
密钥扩展：将输入密钥扩展为一组轮密钥（Round Keys），用于每一轮加密。


初始轮（AddRoundKey）：

将明文与第一轮密钥进行按位异或（XOR）操作。


主轮（重复若干次，根据密钥长度决定轮数）：

128位密钥：10轮
192位密钥：12轮
256位密钥：14轮
每轮包含以下四个步骤：
SubBytes（字节替换）：通过 S-Box（查找表）将每个字节替换为另一个字节，非线性变换增强安全性。
ShiftRows（行移位）：将状态矩阵的行进行循环左移，增加数据混淆。
MixColumns（列混合）：对状态矩阵的列进行线性变换，增强扩散（最后一轮除外）。
AddRoundKey（轮密钥加）：将当前轮密钥与状态矩阵按位异或。


最后一轮：

包括 SubBytes、ShiftRows 和 AddRoundKey，但省略 MixColumns。


输出：

加密后的密文（128位数据块）。



操作模式
AES 本身是分组加密算法，实际应用中结合操作模式处理任意长度数据：

ECB（电子密码本）：每块独立加密，简单但不安全（相同明文块产生相同密文）。
CBC（密码块链接）：每块加密前与前一块密文异或，需初始化向量（IV），安全性较高。
GCM（Galois/Counter Mode）：结合计数器模式和认证，提供加密和完整性验证。

举例（简化）
假设明文为 "Hello, World!"，密钥为 128 位：

填充明文到 16 字节（如 PKCS#5/PKCS#7 填充）。
将明文分为 128 位块。
对每块应用 AES 加密（10轮变换）。
输出密文（不可读的字节序列）。
```

```python
from Crypto.Cipher import AES
key = b'16-byte-key-1234'  # 128位密钥
iv = b'16-byte-iv-1234'   # 初始化向量（CBC模式）
cipher = AES.new(key, AES.MODE_CBC, iv)
plaintext = b'Hello, World!123'  # 16字节明文
ciphertext = cipher.encrypt(plaintext)
# ciphertext 为加密结果

```

```go
package main

import (
	"bytes"
	"crypto/aes"
	"crypto/cipher"
	"crypto/rand"
	"encoding/base64"
	"errors"
	"fmt"
	"io"
)

// PKCS7Padding 实现 PKCS#7 填充
func PKCS7Padding(data []byte, blockSize int) []byte {
	padding := blockSize - len(data)%blockSize
	padText := bytes.Repeat([]byte{byte(padding)}, padding)
	return append(data, padText...)
}

// PKCS7UnPadding 移除 PKCS#7 填充
func PKCS7UnPadding(data []byte) ([]byte, error) {
	length := len(data)
	if length == 0 {
		return nil, errors.New("invalid padding size")
	}
	padding := int(data[length-1])
	if padding > length {
		return nil, errors.New("invalid padding")
	}
	return data[:length-padding], nil
}

// AESEncrypt 加密函数
func AESEncrypt(plaintext, key []byte) (string, error) {
	// 创建 AES 密码器
	block, err := aes.NewCipher(key)
	if err != nil {
		return "", err
	}

	// 填充明文
	plaintext = PKCS7Padding(plaintext, aes.BlockSize)

	// 生成随机 IV
	iv := make([]byte, aes.BlockSize)
	if _, err := io.ReadFull(rand.Reader, iv); err != nil {
		return "", err
	}

	// 创建 CBC 模式加密器
	ciphertext := make([]byte, len(plaintext))
	mode := cipher.NewCBCEncrypter(block, iv)
	mode.CryptBlocks(ciphertext, plaintext)

	// 将 IV 和密文拼接并编码为 Base64
	result := append(iv, ciphertext...)
	return base64.StdEncoding.EncodeToString(result), nil
}

// AESDecrypt 解密函数
func AESDecrypt(ciphertext string, key []byte) ([]byte, error) {
	// Base64 解码
	data, err := base64.StdEncoding.DecodeString(ciphertext)
	if err != nil {
		return nil, err
	}

	// 提取 IV 和密文
	if len(data) < aes.BlockSize {
		return nil, errors.New("invalid ciphertext")
	}
	iv := data[:aes.BlockSize]
	ciphertextBytes := data[aes.BlockSize:]

	// 创建 AES 密码器
	block, err := aes.NewCipher(key)
	if err != nil {
		return nil, err
	}

	// 创建 CBC 模式解密器
	plaintext := make([]byte, len(ciphertextBytes))
	mode := cipher.NewCBCDecrypter(block, iv)
	mode.CryptBlocks(plaintext, ciphertextBytes)

	// 移除填充
	return PKCS7UnPadding(plaintext)
}

func main() {
	key := []byte("16-byte-key-1234") // 128 位密钥
	plaintext := []byte("Hello, World!")

	// 加密
	encrypted, err := AESEncrypt(plaintext, key)
	if err != nil {
		fmt.Println("Encrypt error:", err)
		return
	}
	fmt.Println("Encrypted:", encrypted)

	// 解密
	decrypted, err := AESDecrypt(encrypted, key)
	if err != nil {
		fmt.Println("Decrypt error:", err)
		return
	}
	fmt.Println("Decrypted:", string(decrypted))
}
```