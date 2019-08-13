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
|---------------------------------|
| Options: 	  	                  |
|	[E]ncrypted message           |
|	[K]ey generation function     |
|	[S]end the decrypted message  |
|	[T]ry encryption              |
|	[Q]uit                        |
|---------------------------------|
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

### exp

---

## Alone in in the dark

### 题目描述

We are alone in the dark with a **single line**!

### 分析

