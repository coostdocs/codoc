---
weight: 5
title: "STL"
---

## Header

```cpp
#include "co/stl.h"
```

The API is in the `co` namespace.

## Overview

- Provides aliases for standard containers, with the allocator replaced by `co::stl_allocator`.
- Provides formatted output for containers.
- `co::vector` is already defined in `co/string.h`; `stl.h` does not redefine it.

## Comparators and Hash

```cpp
template<class T> struct co::less;
template<class T> struct co::greater;
template<class T> struct co::hash;
namespace co::xx { template<class T> struct eq; }
```

- Specialized for `const char*`, comparing / hashing by string content.
- `co::less<const char*>` and `co::greater<const char*>` use `strcmp`.
- `co::hash<const char*>` uses `co::murmur_hash`.
- `co::xx::eq<const char*>` uses `strcmp` for equality.
- `const char*` cannot be `nullptr`.
- Containers internally store only pointers; users must ensure that the key's lifetime is longer than that of the container.

## Container Aliases

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

- The default comparator for `map` / `set` is `co::less`; `const char*` keys are sorted by content.
- The default hash for `hash_map` / `hash_set` is `co::hash`, and equality is `co::xx::eq`.

## Formatted Output

Users can directly use:

```cpp
co::string s;
s << container;

co::print(container);
log::info(container);
```

Output format:
- Strings: `const char*`, `co::string`, `std::string`, enclosed in double quotes and escaped.
- `std::pair<K,V>`: format `first:second`.
- Sequence containers: `[a,b,c]`; supports `vector`, `deque`, `priority_queue`, `list`.
- Set containers: `{a,b,c}`; supports `set`, `hash_set`.
- Map containers: `{k1:v1,k2:v2}`; supports `map`, `hash_map`.
- Empty containers: sequences output `[]`, sets/maps output `{}`.
- Nested containers are supported.

## Example

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
