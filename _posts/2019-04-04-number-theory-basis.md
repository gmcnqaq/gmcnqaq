---
layout: post
title: "数论基础"
date: 2019-04-04
excerpt: "介绍一些基础概念"
tag:
- Number Theory
- Crypto
comments: true
---

---

## 整除性和带余除法

### 整除性

设a,b, m均为整数，其中b ≠ 0, 若存在某个m使得$$a = mb$$成立, 则称**b整除a**

也就是说如果b除a没有余数, 则认为b整除a

b除a通常用`b | a`表示, 同时若b \| a, 我们就说b是a的一个**因子**

关于整数整除我们有如下的一些简单的性质:

* 若a \| 1, 则a = ±1

* 若a \| b, 且b \| a, 则a = ±b

* 0是任何非零整数的倍数, ±1是任何非零整数的因子

* 若a \| b且b \| c, 则a \| c

* 对于任意整数m, n, 若b \| g且b \| h, 则可以得出b \| (mg + nh)

### 带余除法

对于a, b两个整数, 其中b ≠ 0, 则 `a = qb + r`, 0 ≤ r ≤ \|b\|. 则称r为a被b除得到的**余数**

显然当 r = 0时, b \| a

---

## 欧几里得算法

```python
def gcd(a, b):
    if a % b == 0:
        return b
    else:
        return gcd(b, a % b)
```

>例:
> $$ a = 888, b = 264 $$
>
> $$ 888 = 2 \times 312 + 264 $$
>
> $$ 312 = 1 \times 264 + 48 $$
>
> $$ 264 = 5 \times 48 + 24 $$
>
> $$ 48 = 2 \times 24 $$
>
> $$ gcd(264, 888) = 24 $$


---

## 模运算

### 模

如果a是一个整数, n是正整数, 则我们定义a除以n所得的余数为**a模n**，整数n称为模数

### 同余


`a ≡ b (mod n)`

对同余的理解: 从采用的`≡`, 即可理解其含义，即: *在mod n运算时a与b是**等价**的*

### 模运算

性质

* $$ [(a\  mod\  n) + (b\  mod\  n)]\ mod\ n = (a + b)\ mod\ n $$ ①
* $$ [(a\  mod\  n) \times (b\  mod\  n)]\ mod\ n = (a \times b)\ mod\ n $$ ②
* 若a与n互素: 若 $$ (a \times b) ≡ (a \times c)\ mod\ n $$, 则 $$ b ≡ c\ mod\ n $$ ③


定义比n小的非负整数集合为 $$ Z_n $$

$$ Z_n = \{0, 1, \dots, (n - 1)\} $$

这个集合称为**剩余类集**, 或模n的剩余类，更确切的说, $$Z_n$$中的每个整数都代表一个剩余类, 我们可以将模n的剩余类表示为[0], [1], [2], ..., [n - 1]

在剩余类的所有整数中，我们通常用最小非负整数来代表这个剩余类

### 拓展欧几里得算法

```python
def egcd(a, b):
    if a == 0:
        return (b, 0, 1)
    else:
        g, y, x = egcd(b % a, a)
        return (g, x - (b // a) * y, y)
```
> 例: $$ a = 264, b = 888 $$
>
> $$ 264 = 888 - 2 \times 312 $$
>
> $$ 48 = 312 - 264 = 312 - (888 - 2 \times 312) = -888 + 3 \times 312 $$
>
> $$ 24 = 264 - 5 \times 48 = (888 - 2 \times 312) - 5 \times (-888 + 3 \times 312) = 6 \times 888 - 17 \times 312 $$

求逆元
```python
def egcd(a, b):
    if a == 0:
        return (b, 0, 1)
    else:
        g, y, x = egcd(b % a, a)
        return (g, x - (b // a) * y, y)

def modinv(a, m):
    g, x, y = egcd(a, m)
    if g != 1:
        raise Exception('modular inverse does not exist')
    else:
        return x % m
```
---

## 素数

**整数p > 1是素数, 当且仅当它只有因子±1和±p**

**算数基本定理**: 任何整数a > 1都可以唯一地因式分解为

$$ a = p_1^{a_1} \times p_2^{a_2} \times \dots \times p_t^{a_t} $$

式中 $$ p_1, p_2, \dots, p_t $$ 均是素数, $$ p_1 < p_2 < \dots < p_t $$, 且所有的 $$ a_i $$ 都是正整数

另一种表示方法:

设P是所有素数的集合,则任意正整数a可以唯一地表示为:

$$ a = \prod_{p\in P} {p^{a_p}}, a_p \ge 0 $$

上式右边是所有可能素数p之积

---

## 费马定理和欧拉定理

### 费马定理

若p是素数, a是正整数且不能被p整除, 则

$$ a^{p - 1} \equiv 1\ mod\ p $$

另一种有用的形式:

若p是素数且a是任意正整数,则

$$ a^p \equiv a\ mod\ p $$

两种形式的差别第一种要求a与p互素, 第二种则没有这个要求

### 欧拉函数

**欧拉函数$$\varphi(n)$$**: 指小于n且与n互素的正整数的个数。习惯上, $$ \varphi(1) = 1 $$

对于素数p, 有

$$ \varphi(p) = p - 1 $$


假设有两个素数p和q, **$$p \ne q$$**, 那么对于$$n = pq$$, 有

$$ \varphi(n) = \varphi(p \times q) = \varphi(p) \times \varphi(q) = (p - 1) \times (q - 1) $$

### 欧拉定理

对任意**互素**的a和n, 有

$$ a^{\varphi(n)} \equiv 1\ mod\ n $$

类似费马定理, 欧拉定理得另一种表达也很有用

$$ a^{\varphi(n) + 1} \equiv a\ mod\ n $$

---

## 中国剩余定理(CRT)

中国剩余定理是数论中最有用的定理之一。CRT说明，某一范围内的整数课通过它的一组剩余类数来重构, 这组剩余类数是对该整数用一组两两互素的整数取模得到的

CRT的一种表现形式, 令

$$ M = \prod_{i = 1}^{k} {m_i} $$

其中 $$ m_i $$ 是两两互素的, 即对 $$ 1 \le i, j \le k, i \ne j $$ 有 $$ gcd(m_i, m_j) = 1 $$。我们可以将 $$ Z_M $$ 中的任意整数对应一个k元组, 该k元组的元素均在 $$ Z_{m_i} $$ 中, 这种对应关系即为

$$ A \Leftrightarrow (a_1, a_2, \dots, a_k) $$

其中 $$ A \in Z_M $$ , 对 $$ 1 \le i \le k, a_i \in Z_{m_i} $$, 且$$ a_i = A\ mod\ m_i $$

CRT说明下面两个断言成立

1. 上式中的映射是$$Z_M$$到笛卡尔积 $$ Z_{m_1} \times Z_{m_2} \times \dots \times Z_{m_k} $$ 的一一对应, 也就是说, 对于任何A, $$ 0 \le A < M $$有唯一的k元组 $$ (a_1, a_2, \dots, a_k) $$与之对应, 其中 $$ 0 \le a_i \le m_i $$, 并且对任何这样的k元组 $$ (a_1, a_2, \dots, a_k) $$, $$ Z_M $$中有唯一的A与之对应
2. $$ Z_M $$中的元素上的运算可以等价于对应的k元组的运算, 即在笛卡尔积的每个分量上独立地执行运算

> 例:
> 将973 mod 1813表示为模37和49的两个数。我们定义
>
> $$ m_1 = 37, m_2 = 49, M = 1813, A = 973 $$
>
> 则 $$M_1 = \frac{M}{m_1} = 49$$且$$M_2 = \frac{M}{m_2} = 37 $$。利用拓展欧几里得算法有 $$ M_1^{-1} = 34\ mod\ m_1 = modinv(M_1, m_1) $$, 且 $$ M_2^{-1} =  4\ mod\ m_2 = modinv(M_2. m_2) $$(注意每个 $$ M_i $$和 $$ M_i^{-1} $$只需计算一次)。对37和49取模, 因为 $$ 973\ mod\ 37 = 11,\ 973\ mod\ 49 = 42 $$, 所以973可以表示为(11, 42)
>
> 假定要处理678 + 973。该如何处理(11, 42)? 首先计算 $$ 678 \Leftrightarrow (678\ mod\ 37,\ 678\ mod\ 48) = (12, 41) $$, 然后将二元组的元素相加并化简 $$ (11 + 12\ mod\ 37,\ 42 + 41\ mod\ 49) = (23, 24) $$。
> 
> $$ (23, 24) \Leftrightarrow a_1M_1M_1^{-1} + a_2M_2M_2^{-1}\ mod\ M $$
> 
> $$ = [23 \times 49 \times 34 + 34 \times 37 \times 4]\ mod\ 1813 $$
> 
> $$ = 43350\ mod\ 1813 $$
> 
> $$ = 1651 $$
>
> 而$$(973 + 678)\ mod\ 1813 = 1651$$

另一种表现形式, 令$$m_1,\ \dots,\ m_k$$是两两互素的整数, 且 $$ 1 \le i,\ j \le k,\ i \ne j $$, 定义M为所有 $$ m_i $$的乘积。设 $$ a_i,\ \dots,\ a_k $$是整数, 则同余方程组

$$ x \equiv a_1\ mod\ m_1 $$

$$ x \equiv a_2\ mod\ m_2 $$

$$ \vdots $$

$$ x \equiv a_k\ mod\ a_k $$

模M有唯一解

> 例:
> 
> $$ x \equiv 2\ mod\ 3;\ x \equiv 3\ mod\ 5;\ x \equiv 2\ mod\ 7 $$
> 
> 求解x
> 
> $$ M = 3 \times 5 \times 7 = 105 $$
>
> $$ M_1 = 35,\ M_1^{-1} = 2 $$
> 
> $$ M_2 = 21,\ M_2^{-1} = 1 $$
>
> $$ M_3 = 15,\ M_3^{-1} = 1 $$
> 
> $$ x = (2 \times 35 \times 2 + 3 \times 21 \times 1 + 2 \times 15 \times 1)\ mod\ 105 $$
> 
>$$ = 233\ mod\ 105 $$
>
> $$ = 23 $$

对CRT的个人理解:

第一种表现形式可以理解为一个向量可以被一个坐标系中的一组坐标表示, 第二种表现形式可以理解为一个确定坐标系中的一组坐标唯一确定一个向量
```python
# via web
from functools import reduce
def egcd(a, b):
    """扩展欧几里得"""
    if 0 == b:
        return 1, 0, a
    x, y, q = egcd(b, a % b)
    x, y = y, (x - a // b * y)
    return x, y, q
def chinese_remainder(pairs):
# 中国剩余定理
    mod_list, remainder_list = [p[0] for p in pairs], [p[1] for p in pairs]
    mod_product = reduce(lambda x, y: x * y, mod_list)
    mi_list = [mod_product//x for x in mod_list]
    mi_inverse = [egcd(mi_list[i], mod_list[i])[0] for i in range(len(mi_list))]
    x = 0
    for i in range(len(remainder_list)):
        x += mi_list[i] * mi_inverse[i] * remainder_list[i]
        x %= mod_product
    return x
if __name__=='__main__':
    print(chinese_remainder([(3, 2), (5, 3), (7, 2)]))
```

---

## 离散对数

### 模n的整数幂

设m是大于1的正整数, 如果a与m互素, 则使同余式

$$ a^d \equiv 1\ mod\ m $$

成立的最小正整数d称为a对模m的**指数**(或阶, 周期长), 记为 $$ ord_m(a) $$

由欧拉定理可以知道至少有一个这样的整数满足条件, 即 $$ \varphi(m) $$

如果a对模m的指数是 $$ \varphi(m) $$, 则称a为模m的一个原根(或者本原根)


<table border = "1" >
    <tr>
    <caption>例: 模19的整数幂</caption>
    </tr>
    <tr>
        <td> $$ a $$ </td>
        <td> $$ a^2 $$ </td>
        <td> $$ a^3 $$ </td>
        <td> $$ a^4 $$ </td>
        <td> $$ a^5 $$ </td>
        <td> $$ a^6 $$ </td>
        <td> $$ a^7 $$ </td>
        <td> $$ a^8 $$ </td>
        <td> $$ a^9 $$ </td>
        <td> $$ a^{10} $$ </td>
        <td> $$ a^{11} $$ </td>
        <td> $$ a^{12} $$ </td>
        <td> $$ a^{13} $$ </td>
        <td> $$ a^{14} $$ </td>
        <td> $$ a^{15} $$ </td>
        <td> $$ a^{16} $$ </td>
        <td> $$ a^{17} $$ </td>
        <td> $$ a^{18} $$ </td>
    </tr>
    <tr>
    <td bgcolor = "LightSlateGray"> 1 </td>
    <td> 1 </td>
    <td> 1 </td>
    <td> 1 </td>
    <td> 1 </td>
    <td> 1 </td>
    <td> 1 </td>
    <td> 1 </td>
    <td> 1 </td>
    <td> 1 </td>
    <td> 1 </td>
    <td> 1 </td>
    <td> 1 </td>
    <td> 1 </td>
    <td> 1 </td>
    <td> 1 </td>
    <td> 1 </td>
    <td> 1 </td>
    </tr>
    <tr>
    <td bgcolor = "LightSlateGray"> 2 </td>
    <td bgcolor = "LightSlateGray"> 4 </td>
    <td bgcolor = "LightSlateGray"> 8 </td>
    <td bgcolor = "LightSlateGray"> 16 </td>
    <td bgcolor = "LightSlateGray"> 13 </td>
    <td bgcolor = "LightSlateGray"> 7 </td>
    <td bgcolor = "LightSlateGray"> 14 </td>
    <td bgcolor = "LightSlateGray"> 9 </td>
    <td bgcolor = "LightSlateGray"> 18 </td>
    <td bgcolor = "LightSlateGray"> 17 </td>
    <td bgcolor = "LightSlateGray"> 15 </td>
    <td bgcolor = "LightSlateGray"> 11 </td>
    <td bgcolor = "LightSlateGray"> 3 </td>
    <td bgcolor = "LightSlateGray"> 6 </td>
    <td bgcolor = "LightSlateGray"> 12 </td>
    <td bgcolor = "LightSlateGray"> 5 </td>
    <td bgcolor = "LightSlateGray"> 10 </td>
    <td bgcolor = "LightSlateGray"> 1 </td>
    </tr>
    <tr>
    <td bgcolor = "LightSlateGray"> 3 </td>
    <td bgcolor = "LightSlateGray"> 9 </td>
    <td bgcolor = "LightSlateGray"> 8 </td>
    <td bgcolor = "LightSlateGray"> 5 </td>
    <td bgcolor = "LightSlateGray"> 15 </td>
    <td bgcolor = "LightSlateGray"> 7 </td>
    <td bgcolor = "LightSlateGray"> 2 </td>
    <td bgcolor = "LightSlateGray"> 6 </td>
    <td bgcolor = "LightSlateGray"> 18 </td>
    <td bgcolor = "LightSlateGray"> 16 </td>
    <td bgcolor = "LightSlateGray"> 10 </td>
    <td bgcolor = "LightSlateGray"> 11 </td>
    <td bgcolor = "LightSlateGray"> 14 </td>
    <td bgcolor = "LightSlateGray"> 4 </td>
    <td bgcolor = "LightSlateGray"> 12 </td>
    <td bgcolor = "LightSlateGray"> 17 </td>
    <td bgcolor = "LightSlateGray"> 13 </td>
    <td bgcolor = "LightSlateGray"> 1 </td>
    </tr>
    <tr>
    <td bgcolor = "LightSlateGray"> 4 </td>
    <td bgcolor = "LightSlateGray"> 16 </td>
    <td bgcolor = "LightSlateGray"> 7 </td>
    <td bgcolor = "LightSlateGray"> 9 </td>
    <td bgcolor = "LightSlateGray"> 17 </td>
    <td bgcolor = "LightSlateGray"> 11 </td>
    <td bgcolor = "LightSlateGray"> 6 </td>
    <td bgcolor = "LightSlateGray"> 5 </td>
    <td bgcolor = "LightSlateGray"> 1 </td>
    <td> 4 </td>
    <td> 16 </td>
    <td> 7 </td>
    <td> 9 </td>
    <td> 17 </td>
    <td> 11 </td>
    <td> 6 </td>
    <td> 5 </td>
    <td> 1 </td>
    </tr>
    <tr>
    <td bgcolor = "LightSlateGray"> 5 </td>
    <td bgcolor = "LightSlateGray"> 6 </td>
    <td bgcolor = "LightSlateGray"> 11 </td>
    <td bgcolor = "LightSlateGray"> 17 </td>
    <td bgcolor = "LightSlateGray"> 9 </td>
    <td bgcolor = "LightSlateGray"> 7 </td>
    <td bgcolor = "LightSlateGray"> 16 </td>
    <td bgcolor = "LightSlateGray"> 4 </td>
    <td bgcolor = "LightSlateGray"> 1 </td>
    <td> 5 </td>
    <td> 6 </td>
    <td> 11 </td>
    <td> 17 </td>
    <td> 9 </td>
    <td> 7 </td>
    <td> 16 </td>
    <td> 4 </td>
    <td> 1 </td>
    </tr>
    <tr>
    <td colspan = "18"> $$ \dots $$ </td>
    </tr>
    <tr>
    <td bgcolor = "LightSlateGray"> 18 </td>
    <td bgcolor = "LightSlateGray"> 1 </td>
    <td> 18 </td>
    <td> 1 </td>
    <td> 18 </td>
    <td> 1 </td>
    <td> 18 </td>
    <td> 1 </td>
    <td> 18 </td>
    <td> 1 </td>
    <td> 18 </td>
    <td> 1 </td>
    <td> 18 </td>
    <td> 1 </td>
    <td> 18 </td>
    <td> 1 </td>
    <td> 18 </td>
    <td> 1 </td>
    </tr>

</table>




性质:

* 如果 $$ a \equiv b \ mod\ m $$, 则  $$ord_m(a) = ord_m(b)$$ ;

* $$ a^d \equiv 1\ mod\ m $$ 的充分必要条件是:  $$ord_m(a) \mid d$$ ;

* $$ ord_m(a) \mid \varphi(m) $$ ;

* 设 $$ a^{-1} $$是a模m的逆元, 则 $$ ord_m(a^{-1}) = ord_m(a) $$

* $$ a^d \equiv a^k\ mod\ m $$, 则 $$ d \equiv k\ mod\ ord_m(a) $$

* 设k为非负整数, 则 $$ ord_m(a^k) = \frac{ord_m(a)}{gcd(ord_m(a), k)}$$

而且在模m的简化剩余系中, 至少有 $$ \varphi(ord_m(a)) $$个数对模m的指数等于 $$ ord_m(a) $$

特别的, 如果a是一个模m的原根, 则 $$ a^k $$也是模m的原根的充分必要条件是 $$ gcd(\varphi(m), k) = 1 $$

* 如果模m存在一个原根, 则模m共有 $$ \varphi(\varphi(m)) $$个不同的原根

* $$ ord_m(ab) = ord_m(a) \times ord_m(b) $$的充分必要条件是 $$ gcd(ord_m(a), ord_m(b)) = 1 $$


原根的重要之处在于, 若a是n的原根, 则其幂 $$ a, a^2, \dots, a^{\varphi(n)} $$是模n各不相同的, 且均与n互素

并非所有整数都有原根

事实上, 只有形如**$$ 2, 4, p^a和p^{2a} $$**的整数才有原根, 其中p是奇素数, a是正整数




> 参考文献: 《密码编码学与网络安全--原理与实践(第七版)》