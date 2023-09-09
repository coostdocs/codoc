---
weight: 10
title: "Timed task scheduler"
---


include: [co/tasked.h](https://github.com/idealvin/coost/blob/master/include/co/tasked.h).


## co::Tasked

The class **Tasked** implements a simple timed task scheduler. All tasks are scheduled internally by a single thread, but tasks can be added from any thread. When a task is blocked, it will affect all subsequent tasks. Therefore, it is not recommended to use it to schedule tasks that may block for a long time. 

{{< hint info >}}
`Tasked` was added to namespace `co` since v3.0.1.
{{< /hint >}}


### Constructor

```cpp
1. Tasked();
2. Tasked(Tasked&& t);
```

- 1, the default constructor. When the object is created, the scheduling thread will start immediately.
- 2, the move constructor.



### Destructor

```cpp
~Tasked();
```

- The destructor, stop the task scheduling thread.



### F

```cpp
typedef std::function<void()> F;
```

- The task type.



### run_at

```cpp
void run_at(F&& f, int hour, int minute=0, int second=0);
void run_at(const F& f, int hour, int minute=0, int second=0);
```

- Add a task to run at the specified time, `f` will run once at time of `hour:minute:second`.
- `hour` must be an integer between 0-23, `minute` and `second` must be an integer between 0-59, and the default is 0.



### run_daily

```cpp
void run_daily(F&& f, int hour=0, int minute=0, int second=0);
void run_daily(const F& f, int hour=0, int minute=0, int second=0);
```

- Add a periodic task that runs at a specified time every day, `f` will run once every day at the time of `hour:minute:second`.
- `hour` must be an integer between 0-23, the default is 0, `minute` and `second` are integers between 0-59, and the default is 0.



### run_every

```cpp
void run_every(F&& f, int n);
void run_every(const F& f, int n);
```

- Add a periodic task that runs every `n` seconds.



### run_in

```cpp
void run_in(F&& f, int n);
void run_in(const F& f, int n);
```

- Add a task that runs once in `n` seconds.



### stop

```cpp
void stop();
```

- Stop the task scheduling thread, this method will be automatically called in the destructor.
- It is safe to call this method multiple times.



### Code example

```cpp
co::Tasked s;                       // create and start the scheduler
s.run_in(f, 3);                     // run f 3 seconds later
s.run_every(std::bind(f, 0), 3);    // run f every 3 seconds
s.run_at(f, 23);                    // run f once at 23:00:00
s.run_daily(f);                     // run f at 00:00:00 every day
s.run_daily(f, 23);                 // run f at 23:00:00 every day
s.run_daily(f, 23, 30);             // run f at 23:30:00 every day
s.stop();                           // stop the scheduler
```
