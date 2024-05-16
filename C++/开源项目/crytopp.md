*https://github.com/weidai11/cryptopp*

*https://www.cryptopp.com/*

`Crypto++`（也称为`cryptopp`）是一个开源的C++加密库，它提供了许多加密算法和技术的实现，包括对称加密（如`AES`和`DES`）、非对称加密（如`RSA`）、散列算法（如`SHA-1`和`SHA-256`）、消息认证码（`MAC`）以及其他加密相关的功能。这个库由Wei Dai在1995年创建，目的是为了提供一个安全、高效、且易于使用的加密工具集。它广泛用于需要数据加密的软件项目中，支持多种操作系统和平台。

# MD5加密

[[MD5]]

*https://www.cryptopp.com/wiki/MD5*

## 查询digest与block大小

```cpp
#include <iostream>
#include <string>

#include "cryptlib.h"

#define CRYPTOPP_ENABLE_NAMESPACE_WEAK 1
#include "md5.h"

int main()
{
    CryptoPP::Weak::MD5 hash;
    std::cout << "Name: " << hash.AlgorithmName() << std::endl;
    std::cout << "Digest size: " << hash.DigestSize() << std::endl;
    std::cout << "Block size: " << hash.BlockSize() << std::endl;

	return 0;
}
```

```cpp
Name: MD5
Digest size: 16
Block size: 64
```

## MD5生成

```cpp
#include <iostream>
#include <string>

#include "cryptlib.h"

#define CRYPTOPP_ENABLE_NAMESPACE_WEAK 1
#include "md5.h"

#include "hex.h"
#include "files.h"

int main()
{
	using namespace CryptoPP;
	HexEncoder encoder(new FileSink(std::cout));

	std::string msg = "Yoda said, Do or do not. There is no try.";
	std::string digest;

	Weak::MD5 hash;
	hash.Update((const byte*)&msg[0], msg.size());
	digest.resize(hash.DigestSize());
	hash.Final((byte*)&digest[0]);

	std::cout << "Message: " << msg << std::endl;

	std::cout << "Digest: ";
	StringSource(digest, true, new Redirector(encoder));
	std::cout << std::endl;

	return 0;
}
```

```cpp
Message: Yoda said, Do or do not. There is no try.
Digest: CF4A16604182539B5CE297092A034625
```
# AES加密

[[AES]]

*https://www.cryptopp.com/wiki/Advanced_Encryption_Standard*

```cpp
#include <iostream>
#include <string>

#include "aes.h"
#include "modes.h"
#include "filters.h"

int main()
{
	using namespace CryptoPP;
	using namespace std;
	
	cout << "key length: " << AES::DEFAULT_KEYLENGTH << endl;
	cout << "key length (min): " << AES::MIN_KEYLENGTH << endl;
	cout << "key length (max): " << AES::MAX_KEYLENGTH << endl;
	cout << "block size: " << AES::BLOCKSIZE << endl;

    // 密钥和初始化向量
    unsigned char key[AES::DEFAULT_KEYLENGTH], iv[AES::BLOCKSIZE];
    memset(key, 0x00, AES::DEFAULT_KEYLENGTH);
    memset(iv, 0x00, AES::BLOCKSIZE);

    // 待加密的字符串
    std::string plaintext = "Hello, world!";
    std::string ciphertext;

    // 加密器
    CBC_Mode<AES>::Encryption encryptor;
    encryptor.SetKeyWithIV(key, sizeof(key), iv);

    // 使用StringSink和StreamTransformationFilter进行加密
    StringSource ss(plaintext, true,
        new StreamTransformationFilter(encryptor,
            new StringSink(ciphertext)
        ) // StreamTransformationFilter
    ); // StringSource

    // 输出加密后的数据
    std::cout << "Ciphertext: ";
    for (unsigned char c : ciphertext) {
        std::cout << std::hex << (int)c;
    }
    std::cout << std::endl;

	return 0;
}
```

