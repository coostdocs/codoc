---
weight: 9
title: "Time"
---

include: [co/time.h](https://github.com/idealvin/coost/blob/master/include/co/time.h).


## epoch time

The EPOCH is a specific time `1970-01-01 00:00:00 UTC`, and the epoch time is the time since the EPOCH. 



### epoch::ms

```cpp
int64 ms();
```

- Return milliseconds since EPOCH.



### epoch::us

```cpp
int64 us();
```

- Return microseconds since EPOCH.




## monotonic time

Monotonic time is a monotonic increasing time, it is implemented as the time since last reboot of system on most platforms. It is generally used for timing and is more stable than system time. 


### now::ms

```cpp
int64 ms();
```

- Returns a monotonically increasing timestamp in milliseconds.
- On mac platform, if the system does not support `CLOCK_MONOTONIC`, `epoch::ms()` will be used instead.



### now::us

```cpp
int64 us();
```

- Returns a monotonically increasing timestamp in microseconds.
- On mac platform, if the system does not support `CLOCK_MONOTONIC`, `epoch::us()` will be used instead.

- Example

```cpp
int64 beg = now::us();
int64 end = now::us();
LOG << "time used: "<< (end-beg) <<" us";
```




## Time string (now::str)

```cpp
// fm: time output format
fastring str(const char* fm="%Y-%m-%d %H:%M:%S");
```

- This function returns the string form of the current system time in the specified format. It is implemented based on `strftime`.

- Example

```cpp
fastring s = now::str();     // "2021-07-07 17:07:07"
fastring s = now::str("%Y"); // "2021"
```




## sleep


### sleep::ms

```cpp
void ms(uint32 n);
```

- Sleep for n milliseconds.



### sleep::sec

```cpp
void sec(uint32 n);
```

- Sleep for n seconds.

- Example

```cpp
sleep::ms(10); // sleep for 10 milliseconds
sleep::sec(1); // sleep for 1 second
```




## co::Timer

**co::Timer** is a simple timer based on monotonic time. 

{{< hint warning >}}
Timer was added to namespace `co` since v3.0.1.
{{< /hint >}}



### constructor

```cpp
Timer();
```

- Set the start time of the timer, and start timing when the object is created.



### ms

```cpp
int64 ms() const;
```

- Return milliseconds since start of the timing.



### us

```cpp
int64 us() const;
```

- Return microseconds since start of the timing.



### restart

```cpp
void restart();
```

- Restart the timer.



### Example

```cpp
co::Timer t;
sleep::ms(10);
int64 us = t.us();

t.restart();
sleep::ms(20);
int64 ms = t.ms();
```
