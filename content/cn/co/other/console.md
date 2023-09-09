---
weight: 1
title: "终端输出"
---

include: [co/cout.h](https://github.com/idealvin/coost/blob/master/include/co/cout.h).


## 颜色

如下：

```cpp
cout << text::red("hello\n");
cout << text::green("hello\n");
cout << text::blue("hello\n");
cout << text::yellow("hello\n");
cout << text::magenta("hello\n");
cout << text::cyan("hello\n");
cout << "hello\n";
cout << text::bold("hello\n");
cout << text::bold("hello\n").red();
cout << text::bold("hello\n").green();
cout << text::bold("hello\n").blue();
cout << text::bold("hello\n").yellow();
cout << text::bold("hello\n").magenta();
cout << text::bold("hello\n").cyan();
```



## co::print

```cpp
template<typename ...X>
void print(X&& ... x);
```

- 接受任意数量的参数，输出到 `stdout`，末尾会添加换行符。
- 内部有加锁，支持多线程同时调用 `co::print`。

```cpp
co::print("hello ", 23);
co::print(text::red("hello"));

co::vector<int> v = { 1, 2, 3 };
co::print(v);
```
