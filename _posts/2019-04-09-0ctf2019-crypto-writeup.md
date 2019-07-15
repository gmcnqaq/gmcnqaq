---
layout: post
title: Tctf/0ctf2019 Crypto writeup
date: 2019-04-09
excerpt: "毕业要紧 先咕咕咕"
tag:
- Writeup
- CTF
- Crypto
comments: true
---

---

* TOC
{:toc}

---

## baby rsa

### 题目描述

```python
R.<a> = GF(2^2049)

def encrypt(m):
    global n
    assert len(m) <= 256
    m_int = Integer(m.encode('hex'), 16)
    m_poly = P(R.fetch_int(m_int))
    c_poly = pow(m_poly, e, n)
    c_int = R(c_poly).integer_representation()
    c = format(c_int, '0256x').decode('hex')
    return c
```


### 分析

回顾一下整数域上的RSA

**公约与私钥的产生:**

1. 随机选择两个不同的大素数 $$ p $$和 $$ q $$, 计算 $$ N = p \times q $$
2. 根据欧拉函数, 求得 $$ r = \varphi (N) = \varphi (p) \times \varphi (q) = (p - 1) \times (q - 1) $$
3. 选择一个小于 $$ r $$的整数 $$ e $$, 使得 $$ e $$与 $$ r $$互素. 并求得 $$ e $$关于 $$ r $$的模反元素 $$ d $$, 使得 $$ e \times d \equiv 1\ mod\ r $$

此时 $$ (N, e) $$是公钥, $$ (N, d) $$是私钥

**加密:**

$$ c \equiv m^e\ mod\ N  $$

**解密:**

$$ m \equiv c^d\ mod\ N $$

而这道题是在**有限域 $$ GF(2^n) $$**上的



细节与整数域上的RSA基本一致, 需要注意的点是欧拉函数 $$ \varphi (N) $$

我们回顾一下整数域上欧拉函数的定义: 它指小于 $$ N $$并且与 $$ N $$互素的正整数的个数

在这里对于一个不可约的多项式 $$ p(x) $$有, $$ \varphi (p(x)) = 2^n - 1 $$(除0之外, 都与其互素

对于本题

使用`n.factor()`得到不可再约分的两项, 有 $$ \varphi = (2^{821} - 1)\times (2^{1227} - 1) $$, 接下来求逆元得到 $$ d $$， 解密得到`flag`

更详细的基于多项式的RSA, 请见

[Polynomial based RSA](http://www.diva-portal.se/smash/get/diva2:823505/FULLTEXT01.pdf)或者[有限域上的不可约多项式RSA体制](https://image.hanspub.org/Html/7-2620771_27629.htm)

更多有限域的基础知识可以见我的另一篇博客

[有限域基础](https://gmcnqaq.github.io/abstract-algebra-basis/)(博客滞销, 救救孩子(*･ω-q) 

### exp
**莽夫代码**

**护眼警告**
```python
from pubkey import P, e, n

R.<a> = GF(2^2049)
p, q = n.factor()
# print p, q
phi = (2^(p[0].degree())- 1) * (2^(q[0].degree()) - 1)
d = inverse_mod(e, phi)

f = open('./flag.enc', 'rb')
cipher = f.read()
f.close()
cipher_int = int(cipher.encode('hex'), 16)
cipher_poly = P(R.fetch_int(cipher_int))
msg_poly = pow(cipher_poly, d, n)
msg_int = R(msg_poly).integer_representation()
flag = format(msg_int, '0256x').decode('hex')
```

---

## baby sponge

当时没做出来, 赛后补题

参考了p4大手子的wp

[链接](https://github.com/p4-team/ctf/tree/master/2019-03-23-0ctf-quals/crypto_keccak)

### 题目描述
```python
def proof_of_work(self):
    proof = ''.join([random.choice(string.ascii_letters+string.digits) for _ in xrange(20)])
    digest = sha256(proof).hexdigest()
    self.request.send("sha256(XXXX+%s) == %s\n" % (proof[4:],digest))
    self.request.send('Give me XXXX:')
    x = self.request.recv(10)
    x = x.strip()
    if len(x) != 4 or sha256(x+proof[4:]).hexdigest() != digest: 
        return False
    return True
```
```python
def dohash(self, msg):
    return CompactFIPS202.Keccak(1552, 48, bytearray(msg), 0x06, 32)

def handle(self):
    if not self.proof_of_work():
        return
    self.request.settimeout(3)
    self.dosend("first message(hex): ")
    msg0 = self.recvhex(8000)
    self.dosend("second message(hex): ")
    msg1 = self.recvhex(8000)
    if msg0!=msg1 and self.dohash(msg0) == self.dohash(msg1):
        self.dosend("%s\n" % FLAG)
    else:
        self.dosend(">.<\n")
    self.request.close()
```
```python
def Keccak(rate, capacity, inputBytes, delimitedSuffix, outputByteLen):
    outputBytes = bytearray()
    state = bytearray([0 for i in range(200)])
    rateInBytes = rate//8
    blockSize = 0
    if (((rate + capacity) != 1600) or ((rate % 8) != 0)):
        return
    inputOffset = 0
    # === Absorb all the input blocks ===
    while(inputOffset < len(inputBytes)):
        blockSize = min(len(inputBytes)-inputOffset, rateInBytes)
        for i in range(blockSize):
            state[i] = state[i] ^ inputBytes[i+inputOffset]
        inputOffset = inputOffset + blockSize
        if (blockSize == rateInBytes):
            state = KeccakF1600(state)
            blockSize = 0
    # === Do the padding and switch to the squeezing phase ===
    state[blockSize] = state[blockSize] ^ delimitedSuffix
    if (((delimitedSuffix & 0x80) != 0) and (blockSize == (rateInBytes-1))):
        state = KeccakF1600(state)
    state[rateInBytes-1] = state[rateInBytes-1] ^ 0x80
    state = KeccakF1600(state)
    # === Squeeze out all the output blocks ===
    while(outputByteLen > 0):
        blockSize = min(outputByteLen, rateInBytes)
        outputBytes = outputBytes + state[0:blockSize]
        outputByteLen = outputByteLen - blockSize
        if (outputByteLen > 0):
            state = KeccakF1600(state)
    return outputBytes
```

### 分析

第一层就是一个无脑爆破
```python
def break_pow(suffix, expected):
    for x in itertools.product(string.ascii_letters + string.digits, repeat=4):
        prefix = "".join(list(x))
        h = hashlib.sha256(prefix + suffix).hexdigest()
        if h == expected:
            return prefix
```
第二层需要我们提供两个不同的`msg`, 并且他们要有相同的hash值

使用的hash算法是`Keccak(1552, 48, bytearray(msg), 0x06, 32)`

整个加密过程如下图所示

![](https://upload.wikimedia.org/wikipedia/commons/thumb/7/70/SpongeConstruction.svg/450px-SpongeConstruction.svg.png)

其中`r`部分可以受到我们的输入影响, 而`c`部分则只会在进行f操作时改变

因为本题所用的加密函数的参数`capacity`为48, 不是特别的大, 所以我们可以得到两个有相同部分`c`的`message`

对于`r`部分，因为要与我们的输入进行异或, 所以得到相同的`r`部分是比较简单的

对于两个`message`, 第一个输入置`\0`， 第二个输入设置为两个`message`的异或则可以得到相同的部分

### exp

---

## zer0 lfsr

### 题目描述

```python
class lfsr():
    def __init__(self, init, mask, length):
        self.init = init
        self.mask = mask
        self.lengthmask = 2**(length+1)-1

    def next(self):
        nextdata = (self.init << 1) & self.lengthmask
        i = self.init & self.mask & self.lengthmask
        output = 0
        while i != 0:
            output ^= (i & 1)
            i = i >> 1
        nextdata ^= output
        self.init = nextdata
        return output

def combine(x1,x2,x3):
    return (x1*x2)^(x2*x3)^(x1*x3)
```

```python
l1 = lfsr(int.from_bytes(init1,"big"),0b100000000000000000000000010000000000000000000000,48)
l2 = lfsr(int.from_bytes(init2,"big"),0b100000000000000000000000000000000010000000000000,48)
l3 = lfsr(int.from_bytes(init3,"big"),0b100000100000000000000000000000000000000000000000,48)
```

### 分析

对`lfsr`类的分析:

传入 `init`, `mask`和 `length`三个参数

`nextdata`由 `init`左移一位然后和 `lengthmask`进行了与操作得到，相当于把原来 `init`的最高比特位顶掉了，末尾填了一个0

`i`是由 `init`和 `mask`进行了与操作得到, 由于 `mask`的特殊性，实际上 `init`只有几个比特位起作用, 就是 `mask`为 `1`的那些比特位

后面的循环是为了得到 `output`

`output`首先为0, 然后 `output`和 `i & 1`的结果异或, `i`右移一位. 以此类推, 直到 `i`为0, 所以结合整个流程 `output`实际就是由 `i`的每个比特位异或得到的, 也即是 `init`对应 `mask`为 `1`的那些比特位异或得到

在循环结束时 `nextdata`与 `output`进行了异或，由于之前 `nextdata`的最低比特是0, 所以相当于把 `nextdata`的最低位设成了 `output`

总结**就是对于传入的 `init`和 `mask`, `init`左移一位把最高位顶掉, 最低一位由 `inti & mask`结果的各个比特位异或的结果进行填充，然后返回最低位**

对于 `combine`方法, 其真值表如下:

| $$ x_1 $$ | $$ x_2 $$ | $$ x_3 $$ | result |
| :-------: | :-------: | :-------: | :----: |
|     0     |     0     |     0     |   0    |
|     0     |     0     |     1     |   0    |
|     0     |     1     |     0     |   0    |
|     0     |     1     |     1     |   1    |
|     1     |     0     |     0     |   0    |
|     1     |     0     |     1     |   1    |
|     1     |     1     |     0     |   1    |
|     1     |     1     |     1     |   1    |

可以发现 `combine`的结果与 $$ x_1, x_2, x_3 $$相同的概率都为0.75


最终的输出是3个LFSR的输出通过 `combine`这个非线性的组合生成器得到的结果, 每一个LFSR的反馈输出与最终输出都有0.75的相关性

可以采用**[一次通过方法中的M-S算法 A]()**

设反馈多项式 $$ f(x) $$的级数为n, 抽头数为t, 相关概率 $$ p = P(u_n = z_n) $$, 截取序列 $$ z $$的长度为 $$ N $$

算法实现:

1. 计算序列 $$ z $$每个比特所需要的校验方程平均数

$$ M = \log_2(\frac{N}{2n}) \times (t + 1) $$

2. 计算截取序列 $$ z $$的比特模二加(含有t个比特)和原LFSR序列的相应比特的模二加相同的概率 $$ S $$

$$ S(p, 1) = t $$

$$ Q(q, m, h) = \sum_{i = h}^{m}C_m^i(qs^i(1 - s)^{m - i} + (1 - q)(1 - s)^is^{m - i} $$

### exp

---

## zer0 mi

当时没做出来, 赛后补题

参考了的Armia Prezesa大手子的wp

[链接](https://github.com/miszcz2137/ctf-writeups/blob/master/0ctf2019/zer0mi/write-up.md)

### 题目描述


### 分析

### exp

---

## zer0 des

当时没做出来, 赛后补题

### 题目描述
```python
def genkey(self):
	tmp = os.urandom(8)
	key = ''
	for ch in tmp:
		key += chr(ord(ch)&0xfe)
	return key

def handle(self):
	if not self.proof_of_work():
		return
	# We all know that DES can be bruteforced
	# But it should be longer then 20 minutes?
	self.request.settimeout(20*60)
	key = self.genkey()
	while True:
		self.dosend("plaintext(hex): ")
		pt = self.recvhex(20001)
		if pt=='':
			break
		ct = des.encrypt(pt, key)
		self.dosend("%s\n" % ct.encode('hex'))
	self.dosend("key(hex): ")
	guess = self.recvhex(20)
	if guess == key:
		self.dosend("nyao! %s\n" % FLAG)
	else:
		self.dosend("meow?\n")
	self.request.close()

```

### 分析

### exp
