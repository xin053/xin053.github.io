---
title: cryptography加密库使用详解
date: 2016-12-20 20:59:43
categories:
- Python模块学习
tags:
- Python
- cryptography
---

## cryptography简介

cryptography模块主要分为两类，一类是高层次的加密配方，也就是我们只用关心如何使用它提供的api，并不用关心具体加密过程等细节，这也是我们经常使用的。另一类是低层次的加密原语，如果对密码学不是很了解的话，使用加密原语构造自己的加密算法是很危险的。本片文章介绍高层次的对称加密api和低层次非对称的公钥私钥以及证书

<!-- more -->

## cryptography使用

### Fernet(对称加密)

```python
from cryptography.fernet import Fernet

key = Fernet.generate_key()
key  # A URL-safe base64-encoded 32-byte key
# b'7A7idpk7MjmvTWqZf4_vWwvXwAJmmi4SFRnomqKTrB8='
f = Fernet(key)
token = f.encrypt(b"my deep dark secret")
token
# b'gAAAAABYWUWYZywJx9l3UrSUMGa5OS3dlz15NpUuOu-Wk6UNsLnQmtDx2hGdRRhwe62EhzT7OuvLafjzwjf7fASFRLMBQPhq3fa2U_WsFcEUzCFR0ZcxJC8='
f.decrypt(token)
# b'my deep dark secret'
```

#### Using passwords with Fernet

```python
>>> import base64
>>> import os
>>> from cryptography.fernet import Fernet
>>> from cryptography.hazmat.backends import default_backend
>>> from cryptography.hazmat.primitives import hashes
>>> from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
>>> password = b"password"
>>> salt = os.urandom(16)
>>> kdf = PBKDF2HMAC(
...     algorithm=hashes.SHA256(),
...     length=32,
...     salt=salt,
...     iterations=100000,
...     backend=default_backend()
... )
>>> key = base64.urlsafe_b64encode(kdf.derive(password))
>>> f = Fernet(key)
>>> token = f.encrypt(b"Secret message!")
>>> token
'...'
>>> f.decrypt(token)
'Secret message!'
```

为了以后根据`password`得到`token`，需要保存好`salt`

### X.509(数字证书标准)

数字证书是CA机构签名的含有服务器公钥以及其他网站相关信息的一种电子证书，用来说明该服务器(网站)确实是真的(官方的)，而不是伪造的

这里主要使用的是非对称加密，也就是公钥和私钥(RSA)，私钥用来签名，公钥用来验签

#### Creating a Certificate Signing Request (CSR)

When obtaining a certificate from a certificate authority (CA), the usual flow is:

1. You generate a private/public key pair.
2. You create a request for a certificate, which is signed by your key (to prove that you own that key).
3. You give your CSR to a CA (but *not* the private key).
4. The CA validates that you own the resource (e.g. domain) you want a certificate for.
5. The CA gives you a certificate, signed by them, which identifies your public key, and the resource you are authenticated for.
6. You configure your server to use that certificate, combined with your private key, to server traffic.

所以首先要生成密钥对:

```python
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import rsa

key = rsa.generate_private_key(
    public_exponent=65537,
    key_size=2048,
    backend=default_backend()
)
```

关于生成certificate signing request，请看[官方文档](https://cryptography.io/en/latest/x509/tutorial/#creating-a-certificate-signing-request-csr),然后就可以将生成的证书发送给CA机构，待CA机构处理完，就会返回给你经过他们签名的数字证书，该数字证书也是用户用来核实我们网站的证书。

#### RSA 常用操作

##### 生成

```python
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.asymmetric import rsa

private_key = rsa.generate_private_key(
    public_exponent=65537,
    key_size=2048,
    backend=default_backend()
)
```

这样就生成了一个`RSAPrivateKey`对象。参数保持上面就可以了，具体参数解析看[官方文档](https://cryptography.io/en/latest/hazmat/primitives/asymmetric/rsa/#generation)

**私钥公钥是成对生成的，所以当我们使用`generate_private_key`生成`RSAPrivateKey`对象时，我们可以通过生成的对象获取到`RSAPublicKey`对象**

```python
public_key = private_key.public_key()
```

当然，肯定是不可以从`RSAPublicKey`对象中获取到`RSAPrivateKey`对象的。

##### 从pem文件导入

也可以从一个pem格式的文件导入一个`RSAPrivateKey`对象

pem格式文件就是类似:

```
-----BEGIN CERTIFICATE-----
MIICKjCCAZMCCQDQ8o4kHKdCPDANBgkqhkiG9w0BAQUFADB6MQswCQYDVQQGEwJV
UzELMAkGA1UECBMCQ0ExCzAJBgNVBAcTAlNGMQ8wDQYDVQQKEwZKb3llbnQxEDAO
BgNVBAsTB05vZGUuanMxDDAKBgNVBAMTA2NhMTEgMB4GCSqGSIb3DQEJARYRcnlA
dGlueWNsb3Vkcy5vcmcwHhcNMTEwMzE0MTgyOTEyWhcNMzgwNzI5MTgyOTEyWjB9
MQswCQYDVQQGEwJVUzELMAkGA1UECBMCQ0ExCzAJBgNVBAcTAlNGMQ8wDQYDVQQK
EwZKb3llbnQxEDAOBgNVBAsTB05vZGUuanMxDzANBgNVBAMTBmFnZW50MTEgMB4G
CSqGSIb3DQEJARYRcnlAdGlueWNsb3Vkcy5vcmcwXDANBgkqhkiG9w0BAQEFAANL
ADBIAkEAnzpAqcoXZxWJz/WFK7BXwD23jlREyG11x7gkydteHvn6PrVBbB5yfu6c
bk8w3/Ar608AcyMQ9vHjkLQKH7cjEQIDAQABMA0GCSqGSIb3DQEBBQUAA4GBAKha
HqjCfTIut+m/idKy3AoFh48tBHo3p9Nl5uBjQJmahKdZAaiksL24Pl+NzPQ8LIU+
FyDHFp6OeJKN6HzZ72Bh9wpBVu6Uj1hwhZhincyTXT80wtSI/BoUAW8Ls2kwPdus
64LsJhhxqj2m4vPKNRbHB2QxnNrGi30CUf3kt3Ia
-----END CERTIFICATE-----
```

***A PEM block which starts with `-----BEGIN CERTIFICATE-----` is not a public or private key, it’s an[X.509 Certificate](https://cryptography.io/en/latest/x509/). You can load it using [`load_pem_x509_certificate()`](https://cryptography.io/en/latest/x509/reference/#cryptography.x509.load_pem_x509_certificate) and extract the public key with [`Certificate.public_key`](https://cryptography.io/en/latest/x509/reference/#cryptography.x509.Certificate.public_key)***

当然这个文件也可以被加密，我们使用如下方法从pem文件中导入`RSAPrivateKey`对象

```python
from cryptography.hazmat.primitives import serialization

with open("path/to/key.pem", "rb") as key_file:
    private_key = serialization.load_pem_private_key(
        key_file.read(),
        password=None,
        backend=default_backend()
    )
```

同理也可以从cer文件和ssh格式文件中导入私钥或公钥。

##### 序列化

`RSAPrivateKey`对象和`RSAPublicKey`对象都可以序列化为pem文件

```python
from cryptography.hazmat.primitives import serialization

pem = private_key.private_bytes(
   encoding=serialization.Encoding.PEM,
   format=serialization.PrivateFormat.PKCS8,
   encryption_algorithm=serialization.BestAvailableEncryption(b'mypassword')
)

pem.splitlines()
# [b'-----BEGIN ENCRYPTED PRIVATE KEY-----',
#  b'MIIFHzBJBgkqhkiG9w0BBQ0wPDAbBgkqhkiG9w0BBQwwDgQI4LyuGo+hDoACAggA',
#  b'MB0GCWCGSAFlAwQBKgQQGuA8UxHCt7qLEF29noqffQSCBNBH0rZH59FTTWaPWEV/',
#  ......
#  b'Y6Dt0ACOPHcd8Z2Y9MTJ0QFY8A==',
#  b'-----END ENCRYPTED PRIVATE KEY-----']
```

强烈建议对私钥进行序列化的时候用自己的密钥进行加密，这样不会将私钥完全暴露

**我们之所以说上述过程是序列化，而不是保存私钥，是因为该pem文件不止包含私钥，还包括一些有关私钥的重要信息，具体pem格式请查阅相关文档。而且实际上用的时候并不需要我们手动对pem文件进行解析，只用使用库提供的api就行**

也可以不加密，改变如下

```python
encryption_algorithm=serialization.NoEncryption()
```

对于公钥的序列化，如下:

```python
from cryptography.hazmat.primitives import serialization
public_key = private_key.public_key()

pem = public_key.public_bytes(
   encoding=serialization.Encoding.PEM,
   format=serialization.PublicFormat.SubjectPublicKeyInfo
)

pem.splitlines()
# [b'-----BEGIN PUBLIC KEY-----',
#  b'MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAtboyGrCz1JIVru4+eoKG',
#  b'n/adEsavPDb2FQ6/UkIum392ni/Q9H27chliPXEZWZmEorbJvWeHupuL0ld3IWXi',
#  ......
#  b'LwIDAQAB',
#  b'-----END PUBLIC KEY-----']
```

##### 签名

使用私钥可以对一段信息进行签名，然后别人就可以使用公钥进行验证。

```python
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.asymmetric import padding

signer = private_key.signer(
    padding.PSS(
        mgf=padding.MGF1(hashes.SHA256()),
        salt_length=padding.PSS.MAX_LENGTH
    ),
    hashes.SHA256()
)

message = b"A message I want to sign"
signer.update(message)
signature = signer.finalize()

signature
# b'\x19\x87!5\xc0\xe3s\x01M\xa5-\xf3......\xce\xf5\x03=F\xb3\xd5\xd1\xf9\xc2\xf2\xbak'
```

`padding`也就是填充，就是将不够长度的信息填充成指定长度(这里为256)，具体为什么需要填充请参考SHA256算法实现

也可以使用更简单的方法进行签名:

```python
message = b"A message I want to sign"
signature = private_key.sign(
    message,
    padding.PSS(
        mgf=padding.MGF1(hashes.SHA256()),
        salt_length=padding.PSS.MAX_LENGTH
    ),
    hashes.SHA256()
)
```

##### 验证

```python
public_key = private_key.public_key()
verifier = public_key.verifier(
    signature,
    padding.PSS(
        mgf=padding.MGF1(hashes.SHA256()),
        salt_length=padding.PSS.MAX_LENGTH
    ),
    hashes.SHA256()
)

verifier.update(message)
verifier.verify()
```

如果验证不通过，将会触发异常，同样，也有以下简单的方式进行验证:

```python
public_key.verify(
    signature,
    message,
    padding.PSS(
        mgf=padding.MGF1(hashes.SHA256()),
        salt_length=padding.PSS.MAX_LENGTH
    ),
    hashes.SHA256()
)
```

##### 加密

**使用私钥对信息加密没有意义，因为全世界都有你的公钥，毕竟公钥是公开的**，当然，如果你不公开你的公钥，那更失去了意义，所以加密指的是用公钥进行加密，然后我们使用私钥来解密

```python
message = b"encrypted data"
ciphertext = public_key.encrypt(
    message,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA1()),
        algorithm=hashes.SHA1(),
        label=None
    )
)

ciphertext
# b'J\x95\xadC\xa9......\x18\xbb\\\xa3\xb3\x13f_N\x89\x07`\xa1'
```

##### 解密

```python
plaintext = private_key.decrypt(
    ciphertext,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA1()),
        algorithm=hashes.SHA1(),
        label=None
    )
)

plaintext
# b'encrypted data'
```

可以看到目前对公钥私钥的操作很多都是使用固定参数就完全够了，所以可以对此进一步封装，于是就出现了[该项目](https://github.com/istommao/cryptokit/blob/master/cryptokit/rsa.py)

## 参考文档

- [cryptography官方文档](https://cryptography.io/en/latest/)
- [cryptography RSA](https://cryptography.io/en/latest/hazmat/primitives/asymmetric/rsa/)
