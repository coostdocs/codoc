---
title: "素数的一个判定定理"
date: 2018-08-31T06:26:00+08:00
author: Alvin
keywords: prime number,素数,素数判定,Wilson定理
mathjax: true
---

素数有一个判定定理，被称为 [Wilson 定理](https://en.wikipedia.org/wiki/Wilson%27s_theorem)：

> 当且仅当 $p$ 是素数时，$(p-1)\ ! \equiv -1 \pmod{p}$

这个定理的证明不是很容易想到，记录于此，以供参考。

在《[费马小定理及其欧拉推广](../fermat-theorem/)》一文中，我们已经知道：

> 当 $a$ 与素数 $p$ 互素时，$a,\ 2a,\ \cdots,\ (p-1)a$ 这 $p-1$ 个数除以 $p$ 的余数恰好是 $1,\ 2,\ \cdots,\ p-1$.

我们现在令 $1 \leqslant a \leqslant p-1$，显然 $a$ 与素数 $p$ 是互素的，因此上面的结论成立。这意味着：

> 当 $p$ 是素数时，对任意 $1 \leqslant a \leqslant p-1$，存在一个数 $i$，$1 \leqslant i \leqslant p-1$，使得 $i \cdot a$ 除以 $p$ 的余数是 $1$，即 $i \cdot a \equiv 1 \pmod{p}$.

考虑 $i = a$ 的情况：
$$a \cdot a \equiv 1 \pmod{p}$$

即 $p$ 整除 $(a+1)(a-1)$，只有 $a = p-1$ 与 $a = 1$ 时满足。因此，小于 $p$ 的正整数中，除了 $1$ 与 $p-1$ 之外，其余的数刚好两两配对，使得：

$$
i \cdot j \equiv 1 \pmod{p}
$$

再加上
$$1 \cdot (p-1) \equiv p-1 \pmod{p}$$

由同余乘法不难得到
$$1 \cdot 2 \cdots (p-1) \equiv p-1 \pmod{p}$$

即
$$(p-1)\ ! \equiv -1 \pmod{p}$$

反过来，当 $(p-1)\ ! \equiv -1 \pmod{p}$ 时，即有
$$(p-1)\ ! + 1 = kp$$

若 $p$ 不是素数，则 $p$ 有素因子 $q$，$q$ 必是 $1,\ 2,\ \cdots,\ (p-1)$ 中的一个，因此 $q$ 整除 $(p-1)\ !$，从而得到 $q$ 整除 $1$，矛盾。这样就证明了 $p$ 是素数。
