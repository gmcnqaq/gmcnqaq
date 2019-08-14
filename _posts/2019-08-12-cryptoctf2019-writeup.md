---
layout: post
title: Crypto CTF 2019 writeup
date: 2019-08-12
excerpt: "我好菜啊.jpg"
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



# Crypto CTF 2019

[平台链接](https://cryp.toc.tf/)

---

## Decode me!

### 题目描述

Decode me!

```
D: mb xwhvxw mlnX 4X6AhPLAR4eupSRJ6FLt8AgE6JsLdBRxq57L8IeMyBRHp6IGsmgFIB5E :ztey xam lb lbaH
```

### 分析

题目名字是 **Decode** 猜测应该是某种编码，题目中出现了 **:** （心中缓缓的打出一个 ? 

猜测是某种编码表拓展(卒

(签到题不会，好气

赛后看了一下其他大哥的 wp，发现跟 **:** 一点关系都没有( :P?

[writeup by ITA_Sec](https://ctftime.org/team/59193)

### exp

[link](https://gist.github.com/gmcnqaq/798bffb3b8903729440e2a88b2af3087#file-exp_decode_me-py)

---

## Time Capsule

### 题目描述

You neither need 35 years nor even 20 years to solve this **problem**!

[题目链接](https://gist.github.com/gmcnqaq/798bffb3b8903729440e2a88b2af3087#file-challenge_time_capsule-md)

### 分析

很显然要想得到 flag 就必须要计算出`l = pow(2, pow(2, t), n)`

给出的 t 很大，直接计算显然不是十分现实

实际上这道题利用了 **Time-lock puzzles**，参见[Time-lock puzzles and timed-release Crypto](https://people.csail.mit.edu/rivest/pubs/RSW96.pdf)

其实不看这篇论文，根据 RSA 的推导过程我们也很容易对这个式子进行化简

回顾一下 RSA 的过程



$$
c = m^e \bmod n \\
ed \equiv 1 \bmod \varphi(n) \\
m = c^d \bmod n
$$



在这个过程中有 $$ m = c^d \bmod n = (m^e)^d \bmod n = m $$

可见我们在证明的过程中由欧拉定理对 $$ (m^e)^d $$ 的运算进行了一次化简

那么这里我们同样可以采用相同的方法( 本题中`gcd(2, n) == 1`

即


$$
l = 2^{2^t} \bmod n = 2^{2^t \bmod \varphi(n)} \bmod n
$$


而题目中给出的 n 我们可以用 `yafu` 进行分解，或者从 `factordb` 直接查(看了下应该是有大哥赛后贡献了数据

从而就可以得出 flag

### exp

[link](https://gist.github.com/gmcnqaq/798bffb3b8903729440e2a88b2af3087#file-exp_time_capsule-py)

---

## Clever Girl

### 题目描述

There is no barrier to stop a **clever girl**!

[题目链接](https://gist.github.com/gmcnqaq/798bffb3b8903729440e2a88b2af3087#file-challenge_clever_girl-md)

### 分析

我们得到了一个关于 `p`，`q` 的等式
$$
\frac{p}{p+1} + \frac{q+1}{q} = \frac{2s-X}{s+Y}
$$
对等式化简我们可以得到


$$
\frac{p}{p+1} + \frac{q+1}{q} = \frac{2s-X}{s+Y} \\
1 - \frac{1}{p+1} + 1 + \frac{1}{q} = \frac{2s-X}{s+Y} \\
\frac{q-p-1}{pq+q} = \frac{X+2Y}{S+Y} \\
$$


后面尝试利用 $$ p \times q = n $$ 将 p/q 消去得到一个关于 q/q 的含 未知量S 的二元一次方程组，利用求根公式，对 S 进行遍历，满足 $$ \Delta $$ 可以被开方，因为数据较大，觉得不可行

后来对等式进行大胆假设得到 $$ p - q - 1 = X + 2Y $$ 任意消去 p/q 可以得到关于 p/q 的二元一次方程组，进而得到 p/q

(可能有什么奇妙的黑魔法可以从等式中得到更强的约束条件从而不靠猜测解出 p/q 让我去学学大哥们的 wp

### exp

[link](https://gist.github.com/gmcnqaq/798bffb3b8903729440e2a88b2af3087#file-exp_clever_girl-py)

---

## roXen

### 题目描述

**Relationship with a cryptographer!**

**The Girlfriend:** All you ever care about is crypto! I am sick of it! It's me or crypto!

**The Cryptographer boyfriend:** You meant to say it's you **XOR** cryptography.

**The Girlfriend:** I am leaving you.

[题目链接](https://gist.github.com/gmcnqaq/798bffb3b8903729440e2a88b2af3087#file-challenge_roxen-md)

~~(???我要富萝莉.jpg~~

### 分析

对于 `adlit()`函数

```python
def adlit(x):
    l = len(bin(x)[2:])
    return (2 ** l - 1) ^ x
```

即对 `x` 的每一位取反

又 `q = adlit(p) + 31337` 那么显然有 $$ p + q = 2^{\mbox{nbit}} - 1 + 31337 $$

而我们又已知 $$ n = p \times q $$

所以我们只需求得 $$ p - q $$ ，即可求得 p，q
$$
(p - q)^2 = (p + q)^2 - 4 \times n
$$
所以遍历 nbit 满足上述条件，即可得到得到 p，q 的值

e 满足条件

```python
exp & (exp + 1) == 0
```

即有 e 为全 1 序列

brute force 即可

### exp

[link](https://gist.github.com/gmcnqaq/798bffb3b8903729440e2a88b2af3087#file-exp_roxen-py)

---

## Fast Speedy!

### 题目描述

You can’t **connect the dots** looking forward; you can only connect them looking backwards. So you have to trust that the dots will somehow connect in your future.

[题目链接](https://gist.github.com/gmcnqaq/798bffb3b8903729440e2a88b2af3087#file-challenge_fast_speedy-py)

### 分析

其实就是一个简化的 LFSR

但本题我们只知道密文流序列，而不知道密钥流序列，所以得到 LFSR 的反馈表达式似乎显得不可行

但由于导入的是一张 `png` 格式的图片，而 `png` 格式文件头是固定的 `89504E470D0A1A0A0000000D49484452` 所以我们就可以由此得到一个 128 bit 的密钥流序列

对于这 128 bit 的密钥流序列，显然对于满足条件的初始状态和状态转移矩阵，它生成的密钥流肯定是跟我们的得到的密钥流序列是一样的

所以我们只需 brute force 得到它的初始状态和状态转移矩阵即可(不需要遍历初始状态长度在100之后的，因为我们一直的密钥流数据只有 128 bit，过长的初始状态将导致剩下的校验位不够

### exp

[link](https://gist.github.com/gmcnqaq/798bffb3b8903729440e2a88b2af3087#file-exp_fast_speedy-py)

---

## Good one!!

### 题目描述

So you **work**, and you pay

And you break your back for another day

[题目链接](https://gist.github.com/gmcnqaq/798bffb3b8903729440e2a88b2af3087#file-challenge_good_one-md)

### 分析

当时没做出来，赛后参考了[巨巨的wp](https://github.com/Ariana1729/CTF-Writeups/blob/master/2019/CryptoCTF/Good one!!/README.md)

```python
return 'enc = %s%s' % (x - y, y + x)
```

由于给出的数值是连在一起的，所以第一步是得到 $$ x - y $$ 和 $$ x + y $$ 的值

```python
x, y = y + x * t ** 2, x
```

因为 `x`, `y` 是由上述表达式生成的，所以最后一轮 `x` 肯定是比 `y` 高阶的一个数值。在次数比较高的情况下(`enc` 的长度较大)，对于 `x` 而言，加上或者减去一个比它低阶的数值肯定对它的次数是没有太大影响的，所以对得到的数值中间分开就可以得到 $$ x - y $$ 和 $$ x + y $$ 的值，进而就可以得到 `x` 和 `y` 的值

对于参与运算的每一个 `t` 而言，它肯定是使得 $$ x \gt y \times t^2 $$ 最大的整数(1~9)

我们只需爆破 `t` 的值，然后依次回推即可得到全部的 `t` 序列

对于得到的序列，将其和出现频率最高的子串替换为 `"0"` 就能得到 `flag`

### exp

[link](https://gist.github.com/gmcnqaq/798bffb3b8903729440e2a88b2af3087#file-exp_good_one-py)

---

## Complex RSA

### 题目描述

Things in this world sometimes are different than they appear!

```
nc 167.71.62.250 14559
```

### 分析

给了我们如下的选项

```python
|-------------------------------------|
| Options: 	  	              |
|	[E]ncrypted message           |
|	[K]ey generation function     |
|	[S]end the decrypted message  |
|	[T]ry encryption              |
|	[Q]uit                        |
|-------------------------------------|
```

`Key generation function`

```python
def gen_key(e, nbit):
	p = getPrime(nbit << 2)
	q = getPrime(nbit >> 2)
	print 'p =', p
	print 'q =', q
	n = p * q
	return (e, n)
```

尝试 `Try encryption` 

``` 
Send your Options:
T
Send your input as a pair (a, b):
(2, 0)
((a + b √-1) ** e) (mod n) = (69710083973662420960650316480925438068288166371975472016386747271294652526980981866212196092716023859920994374205287770319975737617103721637267800363157864822650526229855739501578572297094991663430712840764090955204594143999083839187589277191689556497281645985795220084487777862044956053223719905L, 0L)
Send your Options:
T
Send your input as a pair (a, b):
(1, 0)
((a + b √-1) ** e) (mod n) = (1L, 0L)
Send your Options:
T
Send your input as a pair (a, b):
(-1, 0)
((a + b √-1) ** e) (mod n) = (199291381515984130967052617856948856222979657260967765121842969270785960048466405484485957149489556450947292482011232428747212536351984662701567361540013057935566787433691398084436880597717720416784039020432250426599141886931473627893196620881451640377505831257646839522405236305394434155717850150L, 0L)

```

因为表达式中出现了 $$ (a + b\sqrt-1)^e \bmod n $$ ，有虚数 $$ i $$ 的引入。实际上就是一个基于[高斯整数](https://zh.wikipedia.org/wiki/高斯整數)实现的 RSA

简单的总结一下与正整数域上的 RSA 的不同之处在于

- $$ \varphi(n) = \mathrm{lcm}(p^2-1, q^2-1) $$
- 乘法拓展到复数的乘法($$ i^2 = -1 $$

因为本题能够处理负数，所以我们可以很容易得到 n 的值，使用 `yafu` 可以直接分解得到 `p` 和 `q`

而 e 也是一个未知数，但我们拥有加密功能，所以求 e，就转换成了一个 DLP 问题。常规做法`discrete_log()` ，考虑到本题的 e 不会太大并且 n 的因子较少且其中一个很大，所以直接爆破反而会来的更加快

### exp

```python
def break_pow(method, expected):
    func_dict = {"md5": hashlib.md5, "sha1": hashlib.sha1, "sha224": hashlib.sha224, "sha256": hashlib.sha256,
                 "sha384": hashlib.sha384, "sha512": hashlib.sha512}
    func = func_dict[method]
    for x in itertools.product(string.printable, repeat=4):
        suffix = "".join(list(x)).encode()
        if func(suffix).hexdigest()[-6:] == expected:
            return suffix
```

其中一组数据

```python
from Crypto.Util.number import *
import sys

sys.setrecursionlimit(100000)

n = 318477448635658150021906178952578390266041495384711783024345990439365373291533156759606225335822226362064436700219569885177606542442821809853191914838173038099447112485976551457395438236801084466315980380391878030458619209661033008475882637490575714860717415866367886000747401514987300899747685403
p = 170109438196037861
q = 1872191525720270311735307142089233049802005560508759892746824719406153353802895187050764562864498106469014347513650209227608824521893521897690580885102040731782067733905189168548690266286973273710405930786958251647796736947230613592181306690119251508812518181135482745803538618623

assert isPrime(p) & isPrime(q) == 1
assert p * q == n

c1 = 164363563816984865835340627437970391637520217508599536406741782361310116173248369990053665639360029551417918169293236932361163307454213192197782670840386716647085218796343730799374554977249172164928966335942037556954755721015775993256593782179687919417618869463071184013292222417846273135630276550
m1 = 2
e = 0
for i in range(n):
    tmp = pow(2, i, n)
    if tmp == c1:
        e = i
        print(e)
        break
assert pow(m1, e, n) == c1


def cadd(a, b, n):
    return (a[0] + b[0] % n, a[1] + b[1] % n)


def cmul(a, b, n):
    return ((a[0] * b[0] - a[1] * b[1]) % n, (a[0] * b[1] + a[1] * b[0]) % n)


def cpow(a, k, n):
    if (k == 0):
        return (1, 0)
    if (k == 1):
        return a
    if (k % 2 == 0):
        a = cmul(a, a, n)
        return cpow(a, k // 2, n)
    else:
        return cmul(a, cpow(cmul(a, a, n), (k - 1) // 2, n), n)


phi_p = pow(p, 2) - 1
phi_q = pow(q, 2) - 1
g = GCD(phi_p, phi_q)
phi_n = phi_p * phi_q // g
d = inverse(e, phi_n)

cipher = (
    131451699456472540437632770504002717412607377092303950991457449423332162775505100909335006832924283767100678802512632641925105293611592671465089577021037758893592124320645981516595777848523540270688550299886661771137250218354416525142981768788834957118235314195149467477420981776666278769417258772,
    309938185758309100891615134281024769784418072855683396677679875691224697551294223593859790399889562365151235024609148176541382307760904813052107921464506505862928946375233067738544501786037626498583407917176961160944073996508326445444376755174355470972649148404434193320688640901918361023099658416)
msg = cpow(cipher, d, n)
print(msg)

```

---

## Alone in in the dark

### 题目描述

We are alone in the dark with a **single line**!

### 分析

---

## Starving Parrot

### 题目描述

What do you call a starving parrot?

```
nc 167.71.62.250 43139
```

### 分析

```python
|-------------------------------------|
| Options:                            |
|	[E]ncryption function         |
|	[K]eygen function             |
|	[O]racle function             |
|	[F]lag (encrypted)!           |
|	[P]ublic key!                 |
|	[T]est poracle                |
|	[Q]uit                        |
|-------------------------------------|
```

`Encryption function`

```python
def encrypt(n, msg):
    msg = bytes_to_long(msg)
    assert 8 * msg < n
    msg *= 2
    msg += 1
    enc = msg
    enc %= n
    return ((jacobi(msg, n) + 3) * enc) ** 2 % n
```

`Keygen function`

```python
def keygen(nbit): # non-performant and dirty function
    while True:
        r, s = random.getrandbits(nbit), random.getrandbits(nbit)
        p, q = poracle(r), poracle(s)
        if isPrime(p):
            if isPrime(q):
                if p % 8 == 3:
                    n = p * q
                    if n % 8 == 5:
                        return n
```

`Oracle function`

```python
def poracle(r):
    if r >= 2 ** 40:
        return poly(r)  # secret polynomial
    else:
        u = random.randint(1, 2 ** 40)
        return poly(u * r)
```

`Test poracle`

```
T
| send an integer: 
0
| poracle(0) = 2019
T
| send an integer: 
1
| poracle(1) = 1230329455499110414641309792238687987173616277399254348958767935688802487729809367919531862270445683542700283803859953875321704830712216921137158616546785
T
| send an integer: 
10000000000000
| poracle(10000000000000) = 10000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000370000000002019

```

通过上面几轮的测试我们不难发现