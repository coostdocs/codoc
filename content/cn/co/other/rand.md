---
weight: 7
title: "随机值"
---

include: [co/rand.h](https://github.com/idealvin/coost/blob/master/include/co/rand.h).


### 随机数(co::rand)

```cpp
1. uint32 rand();
2. uint32 rand(uint32& seed);
```

- 1, 返回一个 1 到 2G-2 之间的随机数，线程安全。
- 2, 返回一个 1 到 2G-2 之间的随机数，需要用户指定种子数 `seed`，**种子数必须在 1 到 2G-2 之间**，此函数会更新 `seed` 的值，非线程安全。

{{< hint info >}}
可以用 1 中的返回值初始化 2 中的种子数。
{{< /hint >}}

- 示例

```cpp
uint32 x = co::rand();
uint32 y = co::rand();

uint32 seed = co::rand();
uint32 u = co::rand(seed);
uint32 v = co::rand(seed);
```



### 随机字符串(co::randstr)

```cpp
1. fastring randstr(int n=15);
2. fastring randstr(const char* s, int n);
```

- 1, 返回一个长度为 n(默认为15) 的随机字符串，线程安全。
- 2, 返回一个长度为 n、由字符串 s 中的字符构成的随机字符串，s 中可以使用类似 `0-9`, `a-z` 的缩写，s 展开后的长度不能超过 255，线程安全。

{{< hint info >}}
randstr 基于 [nanoid](https://github.com/ai/nanoid) 算法实现，返回的随机字符串足够长时，一般也可以用作唯一 id。
{{< /hint >}}

- 示例

```cpp
fastring s = co::randstr();
s = co::randstr(8);
s = co::randstr("0123456789", 6);
s = co::randstr("0-9a-f", 8); // 长度为8的16进制字符串
```
