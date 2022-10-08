---
weight: 13
title: "Byte order"
---

include: [co/byte_order.h](https://github.com/idealvin/coost/blob/master/include/co/byte_order.h).




Data in the computer is stored in bytes (8 bit). Big-endian computers use big-endian byte order, that is, the high-order byte is at the lower address, and the low-order byte is at the higher address. The little-endian machine uses little-endian byte order, that is, the low-order byte is at the lower address, and the high-order byte is at the higher address. 


A single byte is exactly the same on big endian and little endian machines, while the basic data types of multiple bytes have are different. The basic data types mentioned here refer to built-in types like `int`, `double`. String is not included here, as it is a sequence of single byte and have the same storage format on big or little endian machines. 


The data transmitted on network is in big-endian byte order, which is also called network byte order. When sending data to the network, the basic multi-byte type needs to be converted into network byte order, and when receiving data from the network, it needs to be converted into the byte order of the host. 




`byte_order.h` defines the following methods:


```cpp
ntoh16 ntoh32 ntoh64
hton16 hton32 hton64
```


These methods are applicable to integers with lengths of 2, 4, and 8 bytes. The `ntoh` series converts network byte order to host byte order, and the `hton` series converts host byte order to network byte order. . 


- Code example



```cpp
uint32 h = 777;
uint32 n = hton32(h);
```
