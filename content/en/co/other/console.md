---
weight: 1
title: "Console"
---

include: [co/cout.h](https://github.com/idealvin/coost/blob/master/include/co/cout.h).


## Colored output

See below:

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

- Accept any number of arguments and output to `stdout` with a newline at the end.
- A mutex lock is used internally, and multiple threads can call `co::print` at the same time.

```cpp
co::print("hello ", 23);
co::print(text::red("hello"));

co::vector<int> v = { 1, 2, 3 };
co::print(v);
```
