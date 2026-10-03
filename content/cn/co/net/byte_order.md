---
weight: 1
title: "字节序"
---


## 头文件

```cpp
#include "co/byte_order.h"
```

API 在 `co` 命名空间。


## 概述

- 数据在内存中以字节（8 bit）为基本单位存储。
- 大端机：高位字节在低地址，低位字节在高地址。
- 小端机：低位字节在低地址，高位字节在高地址。
- 单个字节在大、小端机器上完全相同。
- 多字节基本类型（如 `int`、`double`）在大、小端机器上字节序不同。
- 字符串(co::string / std::string)由单字节构成，不受字节序影响。
- 网络传输采用大端字节序（网络字节序）。

发送数据到网络前，需将多字节基本类型转成网络字节序；接收后需转回主机字节序。


## API

```cpp
uint16 hton16(uint16 v);
uint32 hton32(uint32 v);
uint64 hton64(uint64 v);
uint16 ntoh16(uint16 v);
uint32 ntoh32(uint32 v);
uint64 ntoh64(uint64 v);
```

- 分别适用于 2、4、8 字节整数。
- `hton` 系列：主机字节序 → 网络字节序。
- `ntoh` 系列：网络字节序 → 主机字节序。

示例:

```cpp
uint32 h = 777;
uint32 n = hton32(h);
```
