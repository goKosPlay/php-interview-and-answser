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

### 编码与字符集进阶面试题

#### 1. Unicode 和 UTF-8 有什么区别？
* **Unicode**：字符集（Character Set），为每个字符分配唯一的 ID（Code Point，码点），如 `U+4E2D`。它不规定怎么存。
* **UTF-8**：编码规则（Encoding），是 Unicode 的一种实现。它规定了如何将 Code Point 转换为二进制字节序列（1~4 字节变长）。

#### 2. UTF-8 的编码规则是怎样的？
* **1 字节**：`0xxxxxxx` (兼容 ASCII)。
* **2 字节**：`110xxxxx 10xxxxxx`。
* **3 字节**：`1110xxxx 10xxxxxx 10xxxxxx` (汉字通常在此)。
* **4 字节**：`11110xxx 10xxxxxx 10xxxxxx 10xxxxxx` (Emoji 通常在此)。
* **规律**：首字节有多少个 1 开头决定总字节数，后续字节均以 10 开头。

#### 3. 为什么 Base64 会比原数据大 1/3？
* **原理**：Base64 将 3 个字节（24 位）的数据，重新划分为 4 个组（每组 6 位）。
* **映射**：每组 6 位对应 Base64 索引表（A-Z, a-z, 0-9, +, /）中的一个字符（1 个字符占 1 字节）。
* **结果**：3 字节变成 4 字节，所以体积增加约 33%。
* **Padding**：如果数据长度不是 3 的倍数，用 `=` 填充。

#### 4. URL Encode (Percent-Encoding) 的原理？为什么空格变成 %20 或 +？
* **原理**：将非安全字符（如中文、特殊符号）转换为 `%` 后跟两个十六进制数（如 `中` -> `%E4%B8%AD`）。
* **空格问题**：
    * 标准规定空格转为 `%20`。
    * 但 `application/x-www-form-urlencoded`（表单提交）历史遗留将空格转为 `+`。
    * **建议**：使用 `rawurlencode` (PHP) / `encodeURIComponent` (JS) 生成 `%20`，更符合 RFC 3986 标准。


### 编码与加密面试题库大全 (补充精选)

#### 1. 基础编码 (Encoding)
1.  **ASCII 码占用几个字节？** 1 个字节（最高位为 0，范围 0-127）。
2.  **Unicode 和 UTF-8 的关系？** Unicode 是字符集（码点），UTF-8 是编码方式（存储格式）。
3.  **UTF-8 BOM 是什么？** Byte Order Mark，文件头的 `EF BB BF`，用于标识编码，PHP `include` 含 BOM 文件会产生隐形输出。
4.  **GBK 编码占用几个字节？** 中文 2 字节，英文 1 字节。
5.  **Base64 编码原理？** 3 字节转 4 字符（6bit 一个字符），体积增加 33%。
6.  **Base64 的应用场景？** 邮件附件、Data URL (图片转字符串)、JWT Payload。
7.  **URL Encode (Percent-Encoding)？** 非安全字符转 `%XX`，空格转 `%20` 或 `+`。
8.  **Protobuf 为什么比 JSON 小？** 二进制存储，去除字段名（元数据分离），Varint 压缩数字。
9.  **Huffman 编码原理？** 频率高的字符用短码，频率低的用长码，无损压缩。
10. **什么是乱码？** 解码字符集与编码字符集不一致（如 UTF-8 用 GBK 打开）。

#### 2. 哈希算法 (Hashing)
11. **Hash 函数的特性？** 不可逆、定长输出、雪崩效应（输入微变输出剧变）。
12. **MD5 长度？** 128 位（16 字节），通常表示为 32 位 16 进制字符串。
13. **SHA-1 长度？** 160 位（20 字节）。
14. **SHA-256 长度？** 256 位（32 字节）。
15. **什么是 Hash 碰撞？** 不同的输入产生相同的输出。
16. **Hash 碰撞攻击（Hash DoS）？** 构造大量碰撞键值，使哈希表退化为链表，消耗 CPU。
17. **如何存储密码？** 禁止明文，禁止单纯 MD5。应使用 `Salt` + 慢哈希（Bcrypt/Argon2）。
18. **加盐（Salt）的作用？** 防止彩虹表攻击，保证相同密码 Hash 不同。
19. **HMAC 是什么？** Hash-based Message Authentication Code，带密钥的 Hash，用于验签。
20. **一致性 Hash (Consistent Hashing)？** 解决分布式缓存扩容数据迁移问题（Hash 环）。

#### 3. 对称加密 (Symmetric Encryption)
21. **对称加密特点？** 加解密同钥，速度快。
22. **常见算法？** DES, 3DES, AES, RC4, ChaCha20。
23. **AES 密钥长度？** 128, 192, 256 位。
24. **分组密码工作模式？** ECB, CBC, CFB, OFB, CTR, GCM。
25. **ECB 模式缺点？** 相同明文块产生相同密文，不安全，无法隐藏数据模式。
26. **CBC 模式特点？** 引入 IV（初始化向量），前一块密文与当前明文异或。串行加密，无法并行。
27. **GCM 模式？** 计数器模式（CTR）+ Galois 认证。支持并行，提供完整性校验（AEAD）。
28. **填充方式（Padding）？** PKCS#5 / PKCS#7（缺几位补几个几）。
29. **IV（初始化向量）的作用？** 随机化初始状态，防止相同首块明文产生相同密文。IV 无需保密。
30. **流加密（Stream Cipher）？** 对数据流逐位/逐字节加密（如 RC4），不需填充。

#### 4. 非对称加密 (Asymmetric Encryption)
31. **非对称加密特点？** 公钥加密私钥解密（或反之），速度慢，密钥管理方便。
32. **常见算法？** RSA, ECC (Elliptic Curve), ElGamal, SM2 (国密)。
33. **RSA 原理？** 基于大整数分解难题。
34. **ECC 原理？** 基于椭圆曲线离散对数难题。
35. **ECC 与 RSA 相比优势？** 相同安全性下，ECC 密钥更短，计算更快，带宽占用小。
36. **公钥和私钥的作用？**
    *   加密通信：接收方公钥加密，接收方私钥解密。
    *   数字签名：发送方私钥签名，接收方公钥验签。
37. **数字签名的作用？** 防篡改（完整性）、防抵赖（不可否认性）、身份认证。
38. **DH 密钥交换 (Diffie-Hellman)？** 在不安全的信道协商出共享密钥，不传递密钥本身。
39. **ECDH？** 基于椭圆曲线的 DH 交换。
40. **PFS (前向安全性)？** 长期私钥泄露不影响历史会话密钥（每次会话生成临时 DH 密钥）。

#### 5. PKI 与 证书 (Certificates)
41. **PKI 是什么？** Public Key Infrastructure，公钥基础设施。
42. **CA 是什么？** Certificate Authority，证书颁发机构。
43. **数字证书包含什么？** 实体信息（域名/公司）、公钥、有效期、CA 签名。
44. **X.509 标准？** 证书的格式标准。
45. **证书链（Certificate Chain）？** 根证书 -> 中间证书 -> 用户证书。
46. **如何验证证书有效性？** 用上级 CA 公钥解密签名，比对 Hash；检查有效期；检查吊销列表。
47. **CRL 和 OCSP？**
    *   CRL：证书吊销列表（离线文件）。
    *   OCSP：在线证书状态协议（实时查询）。
48. **OCSP Stapling？** 服务端定期查询 OCSP 并随握手下发，减轻 CA 压力，保护隐私。
49. **自签名证书？** 自己签发，不受信任（浏览器报红），仅用于内部测试。
50. **CSR 是什么？** Certificate Signing Request，证书签名请求，包含公钥和主体信息发给 CA。

#### 6. 传输层安全 (SSL/TLS)
51. **SSL 和 TLS 的关系？** SSL 是 TLS 的前身。现在统称 TLS。
52. **TLS 握手流程 (1.2)？**
    1. ClientHello (随机数1, 套件列表)。
    2. ServerHello (随机数2, 选定套件, 证书)。
    3. KeyExchange (交换预主密钥)。
    4. Finished (验证)。
53. **Session Resumption (会话复用)？** Session ID 或 Session Ticket，减少握手 RTT。
54. **TLS 1.3 的改进？** 废除不安全算法（RSA 密钥交换），握手只需 1-RTT，支持 0-RTT。
55. **SNI (Server Name Indication)？** 握手时客户端发送域名，服务端根据域名返回对应证书（解决单 IP 多 HTTPS）。
56. **ALPN？** 协商应用层协议（如 HTTP/2）。
57. **HTTPS 会变慢吗？** 握手有延迟（TLS 1.3 优化后很小），对称加密 CPU 消耗可忽略（AES-NI 指令集）。
58. **HSTS？** 强制客户端使用 HTTPS。
59. **什么是中间人攻击 (MITM)？** 攻击者插入通信双方，伪造证书劫持流量。防御：校验证书链。
60. **SSL Pinning？** App 内置证书指纹，拒绝非预期 CA 签发的证书（防抓包）。

#### 7. 认证与令牌 (Token)
61. **JWT (JSON Web Token) 结构？** Header.Payload.Signature。
62. **JWT 签名原理？** `HMACSHA256(base64(Header) + "." + base64(Payload), secret)`。
63. **JWT 为什么不安全？** Payload 仅 Base64 编码，可被解码看到信息（不可存敏感数据）。
64. **JWT 的 `exp`？** 过期时间。
65. **Refresh Token 的作用？** Access Token 短期有效，过期后用 Refresh Token 换新，兼顾安全与体验。
66. **JWE (JSON Web Encryption)？** 对 JWT 内容进行加密（Payload 不可见）。
67. **Session ID 安全性？** 需足够长、随机、HttpOnly、Secure。
68. **OAuth 2.0 的 state 参数？** 防止 CSRF 攻击。
69. **OpenID Connect (OIDC)？** 基于 OAuth 2.0 的身份认证层（增加了 ID Token）。
70. **SAML？** 基于 XML 的企业级 SSO 标准。

#### 8. 国密算法 (SM Series)
71. **SM1？** 对称加密（硬件实现），类似 AES。
72. **SM2？** 非对称加密，基于 ECC，替代 RSA/ECDSA。
73. **SM3？** 哈希算法，类似 SHA-256。
74. **SM4？** 分组对称加密，类似 AES-128。
75. **国密的应用场景？** 金融、政务、信创领域。

#### 9. 编码实战与坑
76. **如何生成安全的随机数？** `random_bytes` / `openssl_random_pseudo_bytes` (CSPRNG)，不要用 `rand` / `mt_rand`。
77. **UUID 版本？** v1 (时间+MAC, 泄露隐私), v4 (完全随机, 最常用)。
78. **ULID / Snowflake？** 有序 UUID，适合数据库主键（索引友好）。
79. **大端序 (Big-Endian) 与小端序 (Little-Endian)？** 网络字节序通常是大端（高位在前）。
80. **BOM 头问题？** 文本文件开头多出的 3 字节，导致 JSON 解析失败或 Header 发送错误。
81. **换行符问题？** Windows (`\r\n`), Linux (`\n`), Mac旧 (`\r`)。
82. **PHP `json_encode` 中文问题？** 默认转义 Unicode (`\uXXXX`)，用 `JSON_UNESCAPED_UNICODE` 不转义。
83. **浮点数精度问题？** 二进制无法精确表示 0.1，需用 BCMath 或 乘整数处理。
84. **摩尔斯电码？** 点划编码。
85. **二维码原理？** 矩阵二维码，Reed-Solomon 纠错算法。

#### 10. 综合
86. **HTTPS 为什么比 HTTP 安全？** 数据加密（防窃听）、身份验证（防伪装）、完整性校验（防篡改）。
87. **密码学中的盐（Salt）和胡椒（Pepper）？** Salt 存数据库（每个用户不同），Pepper 存代码/配置（所有用户相同）。
88. **彩虹表是什么？** 预先计算好的 Hash 链表，用于逆向破解 Hash。
89. **什么是零知识证明？** 证明者让验证者相信自己知道秘密，而不泄露秘密本身（如 ZK-SNARKs）。
90. **同态加密？** 对密文进行计算，结果解密后等于对明文进行计算（云隐私计算）。
91. **量子计算对加密的威胁？** Shor 算法能快速破解 RSA/ECC（非对称），Grover 算法减半对称加密强度。
92. **后量子密码学 (PQC)？** 抗量子攻击的算法（如格密码）。
93. **什么是多方计算 (MPC)？** 多个参与方联合计算，互不泄露输入数据。
94. **区块链中的 Merkle Tree？** 快速验证数据完整性，由 Hash 构成的树。
95. **TOTP (Time-based One-Time Password)？** 谷歌验证器原理，基于时间 + 共享密钥生成动态码。
96. **什么是熵 (Entropy)？** 随机性的度量，熵越高越难预测。
97. **Base58 编码？** 比 Base64 少了 `0, O, I, l, +, /`，用于比特币地址，更易读。
98. **WebAuthn？** 基于 OIDC 的 Web 认证标准。
99. **FIDO2 / WebAuthn (无密码登录)？** 利用指纹/FaceID/YubiKey 进行公钥认证。
100. **如何安全传输密码？** 前端 Hash (防传输明文) + HTTPS + 后端 Hash (防拖库)。