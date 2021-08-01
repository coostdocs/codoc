---
weight: 18
title: "随机数"
---

include: [co/random.h](https://github.com/idealvin/co/blob/master/include/co/random.h).


## Random


`Random` 类是一个速度极快的伪随机数生成器，可以连续无重复的生成 1 ~ 2G-2 之间的整数。

### Random::Random
```cpp
Random();
explicit Random(uint32_t seed);
```

- 默认构造函数使用 1 作为种子数，第 2 个构造函数使用参数 seed 作为种子数。





### Random::next
```cpp
uint32_t next();
```

- 返回下一个伪随机数，**非线程安全**。一旦种子数确定了，调用 next() 产生的随机序列就被完全决定了。



- 示例
```cpp
Random r(7);
int n = r.next();
```


