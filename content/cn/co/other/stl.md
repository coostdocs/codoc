---
weight: 5
title: "STL"
---


include: [co/stl.h](https://github.com/idealvin/coost/blob/master/include/co/stl.h).


## 常用容器

下表中，左边 coost 中的容器与右边相应的 std 版本是等价的，仅内部的内存分配器不同。

| coost | std |
|------|------|
| co::deque | std::deque |
| co::list | std::list |
| co::map | std::map |
| co::set | std::set |
| co::multimap | std::multimap |
| co::multiset | std::multiset |
| co::hash_map | std::unordered_map |
| co::hash_set | std::unordered_set |

{{< hint warning >}}
当 key 为 `const char*` 类型(即 C 风格字符串)时，`co::map`, `co::set`, `co::multimap`, `co::multiset`, `co::hash_map`, `co::hash_set` 会根据字符串内容比较 key 及计算 key 的 hash 值。
{{< /hint >}}




## co::lru_map

```cpp
template<typename K, typename V>
class lru_map;
```

`co::lru_map` 是基于 LRU (least recently used) 策略实现的 map，当 map 中元素数量达到上限时，优先替换掉最近最少使用的数据。它基于 `co::hash_map` 与 `co::list` 实现，内部元素是无序的。



### constructor

```cpp
1. lru_map();
2. explicit lru_map(size_t capacity);
```

- 1, 默认构造函数，使用 1024 作为最大容量。
- 2, 使用参数 `capacity` 作为最大容量。



### begin

```cpp
iterator begin() const;
```

- 返回指向第一个元素的 iterator，当 lru_map 为空时，返回值与 [end()](#lru_mapend) 相等。



### clear

```cpp
void clear();
```

- 清空 lru_map 内的元素，size 会变成 0，容量保持不变。



### empty

```cpp
bool empty() const;
```

- 判断 lru_map 是否为空。



### end

```cpp
iterator end() const;
```

- 返回指向最后一个元素的下一个位置的 iterator，它本身并不指向任何元素。



### erase

```cpp
void erase(iterator it);
void erase(const key_type& key);
```

- 通过 iterator 或 key 删除元素。



### find

```cpp
iterator find(const key_type& key)
```

- 查找 `key` 对应的元素。



### insert

```cpp
template<typename Key, typename Val>
void insert(Key&& key, Val&& value);
```

- 插入元素，仅当 key 不存在时，才会插入新元素。若 key 已经存在，则不会进行任何操作。
- 插入元素时，若元素数量已经达到最大容量，则会删除最近最少访问的元素。



### size

```cpp
size_t size() const;
```

- 返回 lru_map 中的元素个数。



### swap

```cpp
void swap(lru_map& x) noexcept;
void swap(lru_map&& x) noexcept;
```

- 交换两个 lru_map 的内容，此操作仅交换内部指针、大小、容量等信息。



### 代码示例

```cpp
co::lru_map<int, int> m(128); // capacity: 128

auto it = m.find(1);
if (it == m.end()) {
    m.insert(1, 23);
} else {
    it->second = 23;
}

m.erase(it);  // erase by iterator
m.erase(1);   // erase by key
m.clear();    // clear the map
```




## co::vector

C++ 标准库中 `std::vector<bool>` 是个不太明智的设计，为此，coost 单独实现了 `co::vector`。


### constructor

```cpp
1. constexpr vector() noexcept;
2. explicit vector(size_t cap);
3. vector(const vector& x);
4. vector(vector&& x) noexcept;
5. vector(std::initializer_list<T> x);
6. vector(T* p, size_t n);

7. template<typename It>
   vector(It beg, It end);

8. template<typename X>
   vector(size_t n, X&& x);
```

- 1, 默认构造函数，构造一个空的 vector，size 与 capacity 均为 0。
- 2, **构造一个空的 vector**，capacity 为 `cap`。
- 3, 拷贝构造函数。
- 4, 移动构造函数，构造完后，`x` 变成空对象。
- 5, 用初始化列表构造 vector 对象，`T` 是 vector 中元素的类型。
- 6, 用数组构造 vector 对象，`p` 指向数组第一个元素，`n` 是数组长度。
- 7, 用 `[beg, end)` 范围内的元素构造 vector 对象。
- 8, 有两种情况：`X` 不是 int 或元素类型 `T` 是 int 时，vector 初始化为 `n` 个 `x`；`X` 是 int 且元素类型 `T` 不是 int 时，vector 初始化为 `n` 个 `T` 类型的默认值。

{{< hint warning >}}
构造函数 2 与 `std::vector` 中相应版本的行为是不同的，若要构建 `n` 个默认值构成的 vector，可以使用构造函数 8(第 2 个参数传 0)：
```cpp
co::vector<fastring> v(32, 0);
```
{{< /hint >}}

- 示例

```cpp
co::vector<int> a(32);         // size: 0, capacity: 32
co::vector<int> b = { 1, 2 };  // [1,2]
co::vector<int> v(8, 0);       // 包含 8 个 0
co::vector<fastring> s(8, 0);  // 包含 8 个空的 fastring 对象
```



### destructor

```cpp
~vector();
```

- 释放内存，析构后 vector 变为空对象。



### operator=

```cpp
1. vector& operator=(const vector& x);
2. vector& operator=(vector&& x);
3. vector& operator=(std::initializer_list<T> x);
```

- 赋值操作。

- 示例

```cpp
co::vector<int> v;
v = { 1, 2, 3 };

co::vector<int> x;
x = v;
x = std::move(v);
```



### ———————————
### back

```cpp
T& back();
const T& back() const;
```

- 返回 vector 中最后一个元素的引用。

{{< hint warning >}}
若 vector 为空，调用此方法会导致未定义的行为。
{{< /hint >}}



### front

```cpp
T& front();
const T& front() const;
```

- 返回 vector 第一个元素的引用。

{{< hint warning >}}
若 vector 为空，调用此方法会导致未定义的行为。
{{< /hint >}}



### operator[]

```cpp
T& operator[](size_t n);
const T& operator[](size_t n) const;
```

- 返回 vector 中第 `n` 个元素的引用。

{{< hint warning >}}
若 `n` 超出合理的范围，调用此方法会导致未定义的行为。
{{< /hint >}}



### ———————————
### begin

```cpp
iterator begin() const noexcept;
```

- 返回指向第一个元素的 iterator。



### end

```cpp
iterator end() const noexcept;
```

- 返回 end iterator。



### ———————————
### capacity

```cpp
size_t capacity() const noexcept;
```

- 返回 vector 的容量。



### data

```cpp
T* data() const noexcept;
```

- 返回指向 vector 内部元素的指针。



### empty

```cpp
bool empty() const noexcept;
```

- 判断 vector 是否为空。



### size

```cpp
size_t size() const noexcept;
```

- 返回 vector 中元素个数。



### ———————————
### clear

```cpp
void clear();
```

- 清空 vector，size 变为 0，capacity 不变。



### reserve

```cpp
void reserve(size_t n);
```

- 调整 vector 的容量，确保容量至少是 `n`。
- 当 `n` 小于原来的容量时，则保持容量不变。



### reset

```cpp
void reset();
```

- 销毁 vector 中所有元素，并释放内存。



### resize

```cpp
void resize(size_t n);
```

- 将 vector 的 size 调整为 `n`。
- 当 `n` 大于原来的 size 时，用默认元素值填充扩展的部分。



### swap

```cpp
void swap(vector& x) noexcept;
void swap(vector&& x) noexcept;
```

- 交换两个 vector，仅交换内部指针、容量、大小。



### ———————————
### append

```cpp
1. void append(const T& x);
2. void append(T&& x);
3. void append(size_t n, const T& x);
4. void append(const T* p, size_t n);
5. void append(const vector& x);
6. void append(vector&& x);

7. template<typename It>
   void append(It beg, It end);
```

- 添加元素到 vector 尾部。
- 1-2, 追加单个元素。
- 3, 追加 `n` 个元素 `x`。
- 4, 追加数组，`n` 是数组长度。
- 5-6, 追加 vector `x` 中的所有元素。
- 7, 追加 `[beg, end)` 范围内的元素。

- 示例

```cpp
int a[4] = { 1, 2, 3, 4 };
co::vector<int> v;
v.append(7);
v.append(v);
v.append(a, 4);
```



### emplace_back

```cpp
template<typename ... X>
void emplace_back(X&& ... x);
```

- 在 vector 尾部插入新元素，该元素由参数 `x...` 构建。

- 示例

```cpp
co::vector<fastring> v;
v.emplace_back(4, 'x'); // append("xxxx")
```



### push_back

```cpp
void push_back(const T& x);
void push_back(T&& x);
```

- 添加元素 `x` 到 vector 尾部。



### pop_back

```cpp
T pop_back();
```

- 取出并返回 vector 尾部的元素。

{{< hint warning >}}
若 vector 为空，调用此方法会导致未定义的行为。
{{< /hint >}}



### remove

```cpp
void remove(size_t n);
```

- 删除第 `n` 个元素，并将最后一个元素移动到所删除元素的位置。

- 示例

```cpp
co::vector<int> v = { 1, 2, 3, 4 };
v.remove(1);  // v -> [1, 4, 3]
v.remove(2);  // v -> [1, 4]
```



### remove_back

```cpp
void remove_back();
```

- 删除 vector 尾部的元素，vector 为空时不进行任何操作。


