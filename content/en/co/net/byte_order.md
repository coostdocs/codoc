---
weight: 1
title: "Byte Order"
---


## Header

```cpp
#include "co/byte_order.h"
```

The API is in the `co` namespace.


## Overview

- Data is stored in memory with bytes (8 bits) as the basic unit.
- Big-endian machine: high-order bytes are at lower addresses, low-order bytes are at higher addresses.
- Little-endian machine: low-order bytes are at lower addresses, high-order bytes are at higher addresses.
- A single byte is identical on big-endian and little-endian machines.
- Multi-byte primitive types (such as `int`, `double`) have different byte orders on big-endian and little-endian machines.
- Strings (co::string / std::string) are composed of single bytes and are not affected by byte order.
- Network transmission uses big-endian byte order (network byte order).

Before sending data to the network, multi-byte primitive types must be converted to network byte order; after receiving, they must be converted back to host byte order.


## API

```cpp
uint16 hton16(uint16 v);
uint32 hton32(uint32 v);
uint64 hton64(uint64 v);
uint16 ntoh16(uint16 v);
uint32 ntoh32(uint32 v);
uint64 ntoh64(uint64 v);
```

- They apply to 2-, 4-, and 8-byte integers respectively.
- The `hton` series: host byte order → network byte order.
- The `ntoh` series: network byte order → host byte order.

Example:

```cpp
uint32 h = 777;
uint32 n = hton32(h);
```
