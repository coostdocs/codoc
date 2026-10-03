---
weight: 5
title: "STL"
---


## 头文件

```cpp
#include "co/stl.h"
```

API 在 `co` 命名空间。


## 概述

- 提供标准容器别名，allocator 换成 `co::stl_allocator`。
- 提供容器格式化输出。
- `co::vector` 已在 `co/string.h` 中定义，`stl.h` 不重复定义。


## 比较器与哈希

```cpp
template<class T> struct co::less;
template<class T> struct co::greater;
template<class T> struct co::hash;
namespace co::xx { template<class T> struct eq; }
```

- 对 `const char*` 特化，按字符串内容比较 / 哈希。
- `co::less<const char*>`、`co::greater<const char*>` 用 `strcmp`。
- `co::hash<const char*>` 用 `co::murmur_hash`。
- `co::xx::eq<const char*>` 用 `strcmp` 判等。
- `const char*` 不能为 `nullptr`。
- 容器内部只保存指针，用户需保证 key 生命周期长于容器。


## 容器别名

```cpp
template<class T, class Alloc = co::stl_allocator<T>>
using deque = std::deque<T, Alloc>;

template<class T, class Compare = less<T>>
using priority_queue = std::priority_queue<T, co::vector<T>, Compare>;

template<class T, class Alloc = co::stl_allocator<T>>
using list = std::list<T, Alloc>;

template<class K, class V, class Compare = less<K>,
         class Alloc = co::stl_allocator<std::pair<const K, V>>>
using map = std::map<K, V, Compare, Alloc>;

template<class K, class V, class Compare = less<K>,
         class Alloc = co::stl_allocator<std::pair<const K, V>>>
using multimap = std::multimap<K, V, Compare, Alloc>;

template<class K, class Compare = less<K>, class Alloc = co::stl_allocator<K>>
using set = std::set<K, Compare, Alloc>;

template<class K, class Compare = less<K>, class Alloc = co::stl_allocator<K>>
using multiset = std::multiset<K, Compare, Alloc>;

template<class K, class V, class Hash = hash<K>, class Pred = xx::eq<K>,
         class Alloc = co::stl_allocator<std::pair<const K, V>>>
using hash_map = std::unordered_map<K, V, Hash, Pred, Alloc>;

template<class K, class Hash = hash<K>, class Pred = xx::eq<K>,
         class Alloc = co::stl_allocator<K>>
using hash_set = std::unordered_set<K, Hash, Pred, Alloc>;
```

- `map` / `set` 默认比较器为 `co::less`，`const char*` key 按内容排序。
- `hash_map` / `hash_set` 默认哈希为 `co::hash`，相等为 `co::xx::eq`。


## 格式化输出

用户直接使用：

```cpp
co::string s;
s << container;

co::print(container);
log::info(container);
```

输出格式：
- 字符串：`const char*`、`co::string`、`std::string`，加双引号并转义。
- `std::pair<K,V>`：格式 `first:second`。
- 序列容器：`[a,b,c]`，支持 `vector`、`deque`、`priority_queue`、`list`。
- 集合容器：`{a,b,c}`，支持 `set`、`hash_set`。
- 映射容器：`{k1:v1,k2:v2}`，支持 `map`、`hash_map`。
- 空容器：序列输出 `[]`，集合/映射输出 `{}`。
- 支持嵌套容器。


## 示例

```cpp
#include "co/stl.h"
#include "co/print.h"

int main() {
    co::vector<int> v = {1, 2, 3};
    co::println("v = ", v);

    co::map<co::string, int> m;
    m["a"] = 1;
    m["b"] = 2;
    co::println("m = ", m);

    co::hash_map<co::string, int> hm;
    hm["x"] = 10;
    co::println("hm = ", hm);

    co::set<int> s = {3, 1, 2};
    co::println("s = ", s);

    std::pair<int, co::string> p{1, "one"};
    co::println("p = ", p);

    co::vector<co::vector<int>> vv = {{1, 2}, {3, 4}};
    co::println("vv = ", vv);
    return 0;
}
```
