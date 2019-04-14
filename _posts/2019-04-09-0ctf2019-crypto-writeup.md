---
layout: post
title: 0ctf2019 Crypto writeup
date: 2019-04-09
excerpt: "弱鸡的补题之旅  毕业要紧 先咕咕咕"
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

rsa.sage
```python
#!/usr/bin/env sage
# coding=utf-8

from pubkey import P, n, e
from secret import flag
from os import urandom

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

if __name__ == '__main__':
    ptext = flag + os.urandom(256-len(flag))
    ctext = encrypt(ptext)
    with open('flag.enc', 'wb') as f:
        f.write(ctext)
```

### 分析

### exp

---

## baby sponge

### 题目描述

### 分析

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


最终的输出是3个LFSR的输出通过 `combine`这个非线性的组合生成器得到的结果, 每一个LFSR的反馈输出与最终输出都有0.75的相关性, 可以采用快速相关攻击

目前快速相关攻击主要分为三个流派:

|     流派     |                                                                                                       思想                                                                                                       |
| :----------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| 一次通过方法 |                                            对密钥流序列的每个比特构建校验等式, 按照广义择多原理直接从密钥流序列中选择满足等式最多的比特作为正确估计, 恢复LFSR初始状态                                            |
| 迭代译码方法 |                                    密钥流序列的每个比特都有一个先验概率 $$ p $$, 通过几轮迭代更新$$ p $$, 修正BSC信道带来的错误, 恢复除LFSR输出序列, 然后恢复出LFSR的初始状态                                    |
|    多步法    | 将初始状态分为几个部分, 分步进行恢复, 每一步根据抽头数选择不同的计算校验等式的方法. 每一步进行假设检验排除错误猜测, 通过检测的候选状态进入下一步, 最后一步之后, 剩余的比特用一个小规模的穷举搜索和相关检测来恢复 |

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

### 题目描述


### 分析

### exp

---

## zer0 des

### 题目描述

### 分析

### exp