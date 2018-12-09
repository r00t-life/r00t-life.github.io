---
title: Deffie-Hellman
date: 2017-08-26 13:59:03
tags: cryptography
---

##### 数论基础
`Deffie-Hellman`属于密钥交换算法，要理解DH算法，前提要了解数论知识
假设A,P都是素数，那么下面两个集合相等
```
{a^1 mod p, a^2 mod p, a^3 mod p, ..., a^(p-1) mod p} = {1, 2, 3, ..., p-1}
```
对于`1 <=x, y <= p-1`上面的等式可以概括为
 - `a^x mod p` 一定属于 `{1,2,3,...,p-1}`
 - 如果 `x!=y`，则`a^x mod p != a^y mod p`
 - 对于`1 <= b <= p-1`，一定存在唯一的`1 <= x <= p-1`，使得`b = a^x mod p`

##### 求模公式
假设p为素数，对于正整数a, x, y有：
```
(a^x mod p)^y mod p = a^(xy) mod p
```
证明：
```
令 a^x = mp + n,其中m, n为自然数, 0 <= n < p,则有
C = (a^x mod p)^y mod p
  = ((mp + n) mod p)^y mod p
  = (mp mod p + n mod p)^y mod p
  = n^y mod p
  = (mp + n)^y mod p
  = a^(xy) mod p
```

##### DH算法原理
A,B双方在通信之前需要交换密钥，共同选取了a, p两个素数，其中p和a均公开。A选取自然数Xa，计算出Ya，Xa保密，Ya公开。同理B选择Xb，计算出Yb。然后A使用Yb和Xa计算出密钥k，B使用Ya和Xb计算出密钥K，两者计算出的密钥K是相同的。
 - 选择共同的a, p素数
 - 用户A操作: Xa < p, Ya = a^Xa mod p
 - 用户B操作: Xb < p, Yb = a^Xb mod p
 - 用户A计算K：K = Yb^Xa mod p => (a^Xb mod p)^Xa mod p
 - 用户B计算K: K = Ya^Xb mod p => (a^Xa mod p)^Xb mod p

通过上面的求模公式，可以得知A, B用户所计算出来的K是相同的，根据DH算法，可以得知Xa, Xb是密钥保护的关键，所以需要素数P的数值非常大，才能保证该算法的安全。

##### DH密钥交换通信例子
```
from Crypto.Cipher import AES
from Crypto.Hash import SHA256
from Crypto.Util.number import bytes_to_long, long_to_bytes
from Crypto import Random
import os


def pad(s):
    return s + b'\0' * (AES.block_size - len(s) % AES.block_size)


def encrypt(plaintext, key):
    plaintext = pad(plaintext)
    iv = Random.new().read(AES.block_size)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    return iv + cipher.encrypt(plaintext)


def decrypt(ciphertext, key):
    iv = ciphertext[:AES.block_size]
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = cipher.decrypt(ciphertext[AES.block_size:])
    return plaintext.rstrip(b'\0')


def diffiehellman(sock, bits=2048):
    p = 0xFFFFFFFFFFFFFFFFC90FDAA22168C234C4C6628B80DC1CD129024E088A67CC74020BBEA63B139B22514A08798E3404DDEF9519B3CD3A431B302B0A6DF25F14374FE1356D6D51C245E485B576625E7EC6F44C42E9A637ED6B0BFF5CB6F406B7EDEE386BFB5A899FA5AE9F24117C4B1FE649286651ECE45B3DC2007CB8A163BF0598DA48361C55D39A69163FA8FD24CF5F83655D23DCA3AD961C62F356208552BB9ED529077096966D670C354E4ABC9804F1746C08CA18217C32905E462E36CE3BE39E772C180E86039B2783A2EC07A28FB5C55DF06F4C52C9DE2BCBF6955817183995497CEA956AE515D2261898FA051015728E5A8AACAA68FFFFFFFFFFFFFFFF;
    a = 2
    x = bytes_to_long(os.urandom(32))
    Ya = pow(a, x, p)

    # send ya to user B
    sock.send(long_to_bytes(Ya))
    # user B recv ya(b)
    b = bytes_to_long(sock.recv(256))

    Key = pow(b, x, p)
    return SHA256.new(long_to_bytes(Key)).digest()
```


