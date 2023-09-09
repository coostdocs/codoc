---
weight: 9
title: "时间"
---

include: [co/time.h](https://github.com/idealvin/coost/blob/master/include/co/time.h).


## epoch time

epoch 是一个特定的时刻 `1970-01-01 00:00:00 UTC`，epoch time 是从 epoch 时刻开始的时间，它受系统时间影响。



### epoch::ms

```cpp
int64 ms();
```

- 返回自 epoch 到当前时刻的时间，单位为毫秒。



### epoch::us

```cpp
int64 us();
```

- 返回自 epoch 到当前时刻的时间，单位为微秒。




## monotonic time

monotonic time 是单调递增时间，大多数平台实现为自系统启动开始的时间，一般用于计时，比系统时间稳定，不受系统时间的影响。



### now::ms

```cpp
int64 ms();
```

- 返回一个单调递增的时间戳，单位为毫秒。
- 在 mac 平台，如果系统不支持 `CLOCK_MONOTONIC`，则使用 `epoch::ms()`。



### now::us

```cpp
int64 us();
```

- 返回一个单调递增的时间戳，单位为微秒。
- 在 mac 平台，如果系统不支持 `CLOCK_MONOTONIC`，则使用 `epoch::us()`。

- 示例

```cpp
int64 beg = now::us();
int64 end = now::us();
LOG << "time used: " << (end - beg) << " us";
```




## 时间字符串(now::str)

```cpp
// fm: 时间输出格式
fastring str(const char* fm="%Y-%m-%d %H:%M:%S");
```

- 此函数以指定格式返回当前系统时间的字符串形式，它基于 `strftime` 实现。

- 示例

```cpp
fastring s = now::str();     // "2021-07-07 17:07:07"
fastring s = now::str("%Y"); // "2021"
```




## sleep

Linux 平台支持微秒级的 sleep，但 Windows 平台难以实现。因此，coost 仅提供毫秒、秒级的 sleep。



### sleep::ms

```cpp
void ms(uint32 n);
```

- sleep 参数 n 指定的一段时间，单位是毫秒。



### sleep::sec

```cpp
void sec(uint32 n);
```

- sleep 参数 n 指定的一段时间，单位是秒。

- 示例

```cpp
sleep::ms(10); // sleep for 10 milliseconds
sleep::sec(1); // sleep for 1 second
```




## co::Timer

**Timer** 类是一个简单的计时器，基于 monotonic time 实现。

{{< hint warning >}}
v3.0.1 将 Timer 类加入 namespace `co` 中。
{{< /hint >}}



### constructor

```cpp
Timer();
```

- 设置计时器的起始时间，对象创建完，即开始计时。



### ms

```cpp
int64 ms() const;
```

- 返回从计时开始到当前的时间，单位为毫秒。



### us

```cpp
int64 us() const;
```

- 返回从计时开始到当前的时间，单位为微秒。



### restart

```cpp
void restart();
```

- 重置计时器起始时间，即重新开始计时。



### 代码示例

```cpp
co::Timer t;
sleep::ms(10);
int64 us = t.us();

t.restart();
sleep::ms(20);
int64 ms = t.ms();
```
