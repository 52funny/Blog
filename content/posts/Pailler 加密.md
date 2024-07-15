---
title: 'Pailler 加密方案'
date: 2024-07-09T16:24:16+08:00
draft: true
tags: ['cryptography']
---

Pailler 加密是 [Pascal Paillier](https://scholar.google.com/citations?user=xwzhjfoAAAAJ&hl=en) 在 1999 年发表的论文「Public-Key Cryptosystems Based on Composite Degree Residuosity Classes」中提出的公钥密码学。

<!--more-->

## 算法描述

Pailler 加密主要分为分为三个步骤：

- 密钥生成（Key Generation）
- 加密（Encryption）
- 解密（Decryption）

密钥生成过程主要生成 Pailler 加密的 **公钥** 和 **私钥**，用于后期的加密和解密过程。加密阶段使用公钥对明文$$m$$进行加密获得密文$$c$$。解密阶段使用私钥对密文$$c$$进行解密获得明文$$m$$。同时 Pailler 加密并且满足同态密码性质：给定密文$$c_1$$和$$c_2$$，对密文进行计算可以获得在明文$$m_1$$和$$m_2$$上的操作。

## 密钥生成

密钥生成过程主要有以下几步操作：

1. 随机选择两个 **长度相等** 的随机素数$$p$$和$$q$$
2. 计算$$n = p * q$$，并且计算$$\lambda = lcm(p-1, q-1)$$
3. 选择一个随机数$$g \stackrel{\rm \\$}{\leftarrow} \mathbb{Z}_{n^2}^*$$
4. 然后计算一个$$\mu = (L(g ^ {\lambda} \bmod n ^ {2})) ^ {-1} \bmod n$$

其中$$L(x) = \lfloor \frac{x-1}{n} \rfloor$$, 并且结果是一个整数。

```python
import libnum
import random
from collections import namedtuple

# L(x) fucntion
def L(x: int, n: int) -> int:
    return (x - 1) // n

# generate random number r
# ensure lcd(r, n-1) = 1
def random_number(n: int) -> int:
    while True:
        g = random.randint(1, n - 1)
        if libnum.gcd(n, g) == 1:
            return g
        else:
            print(g, n)

# quickly power mod
def pow_mod(a: int, b: int, p: int) -> int:
    r = 1
    while b:
        r = r * a % p if b & 1 else r
        a = a * a % p
        b >>= 1
    return r

# generate 256 bits prime number
p = libnum.generate_prime(256)
q = libnum.generate_prime(256)
n = p * q
n2 = n ** 2

phi_n = (p - 1) * (q - 1)
lambd = libnum.lcm(p - 1, q - 1)

# random number
g = random_number(n2)
# mu
mu = pow_mod(L(pow_mod(g, lambd, n2), n), phi_n - 1, n)

PK = namedtuple("PK", ["n", "g"])
SK = namedtuple("SK", ["lambd", "mu"])

# public key
pk = PK(n, g)
# secret key
sk = SK(lambd, mu)
```

## 加密

加密过程主要有以下过程：

1. 首先待加密的消息 $$m$$ 满足 $$0 <= m < n$$
2. 随机选择一个整数 $$r \stackrel{\\$}{\leftarrow} \mathbb{Z}_{n}^{*}$$
3. 计算一个一个密文 $$c = g^{m} \cdot r^{n} \bmod n^{2}$$

```python
def encrypt(m: int, pk: PK) -> int:
    # return cipher text c = (g ** m)  * (r ** n) mod (n ** 2)
    r = random_number(n)
    return pow_mod(g, m, n2) * pow_mod(r, pk.n, n2) % n2
```

## 解密

解密主要有以下过程：

1. 密文 $$c$$ 是要被进行解密的内容
2. 计算明文消息 $$m$$ 通过式子 $$m = L(c^{\lambda} \bmod n^{2}) \cdot \mu \bmod n$$

```python
def decrypt(c: int, sk: SK) -> int:
    return L(pow_mod(c, sk.lambd, n2), n) * mu % n
```

## Encrypt & Decrypt Demo

下面给出 **加密** 和 **解密** 的代码：

```codapitmpl{id="main.py"}
import libnum
import random
from collections import namedtuple

# L(x) fucntion
def L(x: int, n: int) -> int:
    return (x - 1) // n

# generate random number r
# ensure lcd(r, n-1) = 1
def random_number(n: int) -> int:
    while True:
        g = random.randint(1, n - 1)
        if libnum.gcd(n, g) == 1:
            return g
        else:
            print(g, n)

# quickly power mod
def pow_mod(a: int, b: int, p: int) -> int:
    r = 1
    while b:
        r = r * a % p if b & 1 else r
        a = a * a % p
        b >>= 1
    return r

# generate 256 bits prime number
p = libnum.generate_prime(256)
q = libnum.generate_prime(256)
n = p * q
n2 = n ** 2

phi_n = (p - 1) * (q - 1)
lambd = libnum.lcm(p - 1, q - 1)

# random number
g = random_number(n2)
# mu
mu = pow_mod(L(pow_mod(g, lambd, n2), n), phi_n - 1, n)

PK = namedtuple("PK", ["n", "g"])
SK = namedtuple("SK", ["lambd", "mu"])

# public key
pk = PK(n, g)
# secret key
sk = SK(lambd, mu)


def encrypt(m: int, pk: PK) -> int:
    # return cipher text c = (g ** m)  * (r ** n) mod (n ** 2)
    r = random_number(n)
    return pow_mod(g, m, n2) * pow_mod(r, pk.n, n2) % n2

def decrypt(c: int, sk: SK) -> int:
    return L(pow_mod(c, sk.lambd, n2), n) * mu % n

##CODE##
```

```codapi{lang="python" editor="basic" template="main.py"}
m = 520
c = encrypt(m, pk)
plain = decrypt(c, sk)
print("pk:", pk)
print("sk:", sk)
print("m: ", m)
print("c: ", c)
print("m*:", plain)
```

## 同态性质

Pailler 密码可以满足同态密码学的性质。通常所说的 **同态加法** 和 **同态乘法** 是对应 **明文** 上的操作，并不要求在密文上也需要满足加法和乘法的性质。如下式所示，当满足同态加法或同态乘法的时候，是对应运算 $$\bullet$$ 的操作，而并不是运算 $$\circ$$ 也要满足加法或乘法操作。

$$D(E(m_1) \circ E(m_2)) = m_1 \bullet m_2 $$

Pialler 满足同态加法，也就是对密文操作的时候相对于对明文做加法操作。

$$D(E(m_1, r_1) \cdot E(m_2, r_2) \bmod n^{2}) = (m_1 + m_2) \bmod n $$

```codapi{lang="python" editor="basic" template="main.py"}
m1 = 520
m2 = 1314
c1 = encrypt(m1, pk)
c2 = encrypt(m2, pk)
c = c1 * c2 % n2
m = decrypt(c, sk)

print("m1: ", m1)
print("m2: ", m2)
print("m : ", m)

assert m == m1 + m2
```

同时 Pailler 加密满足 **同态乘法**。

$$D(E(m_1, r_1)^{m_2} \bmod n^2) = m_1m_2 \bmod n$$

此外，我们可以式子变换为更为普通的形式。

$$D(E(m_1, r_1)^k \bmod n^2) = km_1 \bmod n$$

```codapi{lang="python" editor="basic" template="main.py"}
m1 = 520
k = 1314
c1 = encrypt(m1, pk)
c = pow_mod(c1, k, n2)
m = decrypt(c, sk)
print("m1:", m1)
print("k: ", k)
print("m: ", m)
assert m == m1 * k
```

## Pailler Correctness

在了解 Pailler 加密 & 解密的正确性之前，我们需要了解一个结论 [Carmichael theorem](https://en.wikipedia.org/wiki/Carmichael_function)

> [Wikipedia: Paillier cryptosystem](https://en.wikipedia.org/wiki/Paillier_cryptosystem)
>
> [Wikipedia: Carmichael function](https://en.wikipedia.org/wiki/Carmichael_function)
>
> [Public-Key Cryptosystems Based on Composite Degree Residuosity Classes](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=592dd02703a7e2b5776f0467026b8f4c1bad9d26)
