---
weight: 7
title: "Random Value"
---

include: [co/rand.h](https://github.com/idealvin/coost/blob/master/include/co/rand.h).


### Random number (co::rand)

```cpp
1. uint32 rand();
2. uint32 rand(uint32& seed);
```

- 1, returns a random number between 1 and 2G-2, thread-safe.
- 2, returns a random number between 1 and 2G-2 with the specified seed number, **which must be between 1 and 2G-2**. This function will update the value of `seed`, it is not thread-safe.

{{<hint info>}}
The value of `seed` in 2 can be initialized with the return value in 1.
{{< /hint >}}

- example

```cpp
uint32 x = co::rand();
uint32 y = co::rand();

uint32 seed = co::rand();
uint32 u = co::rand(seed);
uint32 v = co::rand(seed);
```



### Random string (co::randstr)

```cpp
1. fasting randstr(int n=15);
2. fasting randstr(const char* s, int n);
```

- 1, returns a random string of length `n` (15 by default), thread-safe.
- 2, returns a random string of length `n` composed of characters in string `s`. Abbreviations like `0-9`, `a-z` can be used in `s`. The expanded length of `s` cannot exceed 255. Thread-safe.

{{<hint info>}}
`randstr` is implemented based on the algorithm of [nanoid](https://github.com/ai/nanoid). When the returned random string is long enough, it can be used as a unique id.
{{< /hint >}}

- example

```cpp
fasting s = co::randstr();
s = co::randstr(8);
s = co::randstr("0123456789", 6);
s = co::randstr("0-9a-f", 8); // 8-byte hex string
```
