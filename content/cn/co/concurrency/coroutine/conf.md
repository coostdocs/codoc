---
weight: 9
title: "配置项"
---


## 配置

Coost 使用 `co.flag` 定义了协程相关的配置项，配置详细用法请参考 [co.flag 文档](../../flag/)。



### co_hook_log

```cpp
DEF_bool(co_hook_log, false, ">>#1 print log for API hooks");
```

- 打印 API hook 相关的日志，默认为 false。

{{< hint warning >}}
v3.0.1 中将配置项 `hook_log` 重命名为 `co_hook_log`。
{{< /hint >}}



### co_sched_log

```cpp
DEF_bool(co_sched_log, false, ">>#1 print logs for coroutine schedulers");
```

- 打印协程调度相关的调试日志，默认为 false。

{{< hint warning >}}
v3.0.1 将配置项 `co_debug_log` 重命名为 `co_sched_log`。
{{< /hint >}}



### co_sched_num

```cpp
DEF_uint32(co_sched_num, os::cpunum(), ">>#1 number of coroutine schedulers");
```

- 协程调度线程的数量，默认为系统 CPU 核数。目前的实现中，这个值最大也是系统 CPU 核数。



### co_stack_num

```cpp
DEF_uint32(co_stack_num, 8, ">>#1 number of stacks per scheduler, must be power of 2");
```

- v3.0.1 新增，每个协程调度器的共享协程栈数量，**该值必须是 2 的幂**，默认为 8。



### co_stack_size

```cpp
DEF_uint32(co_stack_size, 1024 * 1024, ">>#1 size of the stack shared by coroutines");
```

- 协程栈大小，默认为 1M。
