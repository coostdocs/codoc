---
weight: 17
title: "定时任务"
---


include: [co/tasked.h](https://github.com/idealvin/co/blob/master/include/co/tasked.h).


## Tasked


`Tasked` 类是一个简单的定时任务调度器，内部由单线程调度所有任务，但可以从任意线程添加任务。Tasked 中的任务阻塞时，会影响后面的所有任务，因此不推荐用 Tasked 调度可能会长时间阻塞的任务。




### Tasked::Tasked
```cpp
Tasked();
Tasked(Tasked&& t);
```

- 第 1 个版本是默认构造函数，对象创建完，调度线程即开始运行。
- 第 2 个版本是 move 构造函数，支持将 Tasked 对象放到 STL 容器中。





### Tasked::~Tasked
```cpp
~Tasked();
```

- 析构函数，退出任务调度线程。





### Tasked::F
```cpp
typedef std::function<void()> F;
```

- 任务类型，固定为 `std::function<void()>` 类型的函数。





### Tasked::run_at
```cpp
void run_at(F&& f, int hour, int minute=0, int second=0);
void run_at(const F& f, int hour, int minute=0, int second=0);
```

- 添加指定时刻运行的任务，f 将在 `hour:minute:second` 时刻运行一次。
- hour 必须是 0-23 之间的整数，minute 与 second 必须是 0-59 之间的整数，默认为 0。





### Tasked::run_daily
```cpp
void run_daily(F&& f, int hour=0, int minute=0, int second=0);
void run_daily(const F& f, int hour=0, int minute=0, int second=0);
```

- 添加每天指定时刻运行的周期性任务，f 将在每天的 `hour:minute:second` 时刻运行一次。
- hour 必须是 0-23 之间的整数，默认为 0，minute 与 second 是 0-59 之间的整数，默认为 0。





### Tasked::run_every
```cpp
void run_every(F&& f, int n);
void run_every(const F& f, int n);
```

- 添加每 n 秒运行一次的周期性任务。





### Tasked::run_in
```cpp
void run_in(F&& f, int n);
void run_in(const F& f, int n);
```

- 添加 n 秒后运行一次的任务。





### Tasked::stop
```cpp
void stop();
```

- 退出任务调度线程，析构函数中会自动调用此方法。
- 多次调用此方法是安全的。





### 代码示例
```cpp
Tasked s;                         // create and start the scheduler
s.run_in(f, 3);                   // run f 3 seconds later
s.run_every(std::bind(f, 0), 3);  // run f every 3 seconds
s.run_at(f, 23);                  // run f once at 23:00:00
s.run_daily(f);                   // run f at 00:00:00 every day
s.run_daily(f, 23);               // run f at 23:00:00 every day
s.run_daily(f, 23, 30);           // run f at 23:30:00 every day
s.stop();                         // stop the scheduler
```


