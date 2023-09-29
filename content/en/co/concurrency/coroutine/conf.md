---
weight: 9
title: "Config"
---


## Config

Coost uses `co.flag` to define config items for the coroutine module. Please refer to [co.flag documents](../../../flag/) for detailed usage.



### co_hook_log

```cpp
DEF_bool(co_hook_log, false, ">>#1 print log for API hooks");
```

- Print API hook related logs, default is false.

{{< hint warning >}}
Coost v3.0.1 renamed `hook_log` to `co_hook_log`.
{{< /hint >}}



### co_sched_log

```cpp
DEF_bool(co_sched_log, false, ">>#1 print logs for coroutine schedulers");
```

- Print debug logs for coroutine scheduling, default is false.

{{< hint warning >}}
Coost v3.0.1 renamed `co_debug_log` to `co_sched_log`.
{{< /hint >}}



### co_sched_num

```cpp
DEF_uint32(co_sched_num, os::cpunum(), ">>#1 number of coroutine schedulers");
```

- The number of coroutine scheduling threads, which defaults to the number of system CPU cores. In the current implementation, its maximum value is also the number of system CPU cores.



### co_stack_num

```cpp
DEF_uint32(co_stack_num, 8, ">>#1 number of stacks per scheduler, must be power of 2");
```

- Added in v3.0.1, the number of shared coroutine stacks for each scheduler, must be power of 2, and is 8 by default.



### co_stack_size

```cpp
DEF_uint32(co_stack_size, 1024 * 1024, ">>#1 size of the stack shared by coroutines");
```

- Coroutine stack size, default is 1M.
