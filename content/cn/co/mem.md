---
weight: 3
title: "内存分配"
---

include: [co/mem.h](https://github.com/idealvin/coost/blob/master/include/co/mem.h).



### co::alloc

```cpp
1. void* alloc(size_t size);
2. void* alloc(size_t size, size_t align);
```

- 1, 分配 `size` 字节的内存。
- 2, 分配 `size` 字节的内存，内存边界是 `align` 字节对齐的(align <= 1024)。

{{< hint warning >}}
第 2 个版本中，`align` 必须是 2 的幂，且不能超过 1024。
{{< /hint >}}



### co::free

```cpp
void free(void* p, size_t size);
```

- 释放 [co::alloc](#coalloc) 或 [co::realloc](#corealloc) 分配的内存，`size` 是所分配内存的大小。

{{< hint warning >}}
`co::free` 不同于系统提供的 `::free`，需要额外带上一个 `size` 参数。
{{< /hint >}}



### co::realloc

```cpp
void* realloc(void* p, size_t old_size, size_t new_size);
```

- 重新分配内存，`old_size` 是之前的内存大小，`new_size` 是新的大小，**后者必须大于前者**。



### co::zalloc

```cpp
void* zalloc(size_t size);
```

- 分配 `size` 字节的内存，并将内存清零。



### ———————————
### co::make

```cpp
template<typename T, typename... Args>
inline T* make(Args&&... args);
```

- 调用 [co::alloc](#coalloc) 分配内存，并用参数 `args` 在所分配的内存上构建 `T` 类型的对象。

- 示例

```cpp
int* p = co::make<int>(7);
std::string* s = co::make<std::string>(3, 'x');
```



### co::del

```cpp
template<typename T>
inline void del(T* p, size_t n=sizeof(T));
```

- 销毁 [co::make](#comake) 创建的对象，并释放内存。
- 参数 `n` 是 `p` 所指向对象的内存大小，默认为 `sizeof(T)`。

- 示例

```cpp
int* p = co::make<int>(7);
co::del(p);
```



### co::make_rootic

```cpp
template<typename T, typename... Args>
inline T* make_rootic(Args&&... args);
```

- 与 [co::make_static](#comake_static) 类似，只是该函数创建的静态对象比 `co::make_static` 创建的静态对象后析构。



### co::make_static

```cpp
template<typename T, typename... Args>
inline T* make_static(Args&&... args);
```

- 用参数 `args` 创建 `T` 类型的静态对象，所创建的静态对象在程序退出时会自动销毁。

- 示例

```cpp
fastring& s = *co::make_static<fastring>(32, 'x');
```



### ———————————
### co::shared

```cpp
template<typename T>
class shared;
```

与 `std::shared_ptr` 类似，但有些细微区别。


#### constructor

```cpp
1. constexpr shared() noexcept;
2. constexpr shared(std::nullptr_t) noexcept;
3. shared(const shared& x) noexcept;
4. shared(shared&& x) noexcept;

5. shared(const shared<X>& x) noexcept;
6. shared(shared<X>&& x) noexcept;
```

- 1-2, 创建空的 `shared` 对象。
- 3, 拷贝构造函数，若 `x` 不是空对象，则将内部引用计数加 1。
- 4, 移动构造函数，对象构造完成后，`x` 变为空对象。
- 5-6, 从 `shared<X>` 对象构建 `shared<T>` 对象，`T` 是 `X` 的基类，且 `T` 的析构函数是 virtual 的。

{{< hint warning >}}
`co::shared` 对象不能直接从 `T*` 指针构建，coost 提供 [co::make_shared](#comake_shared) 用于构建 `co::shared` 对象。
{{< /hint >}}


#### destructor

```cpp
~shared();
```

- 若对象非空，则将内部引用计数减 1，引用计数减至 0 时会销毁内部的对象，并释放内存。


#### operator=

```cpp
1. shared& operator=(const shared& x);
2. shared& operator=(shared&& x);
3. shared& operator=(const shared<X>& x);
4. shared& operator=(shared<X>&& x);
```

- 赋值操作。
- 3-4, 用 `shared<X>` 类型的值对 `shared<T>` 类型的对象进行赋值，`T` 是 `X` 的基类，且 `T` 的析构函数是 virtual 的。


#### get

```cpp
T* get() const noexcept;
```

- 获取内部对象的指针。


#### operator->

```cpp
T* operator->() const;
```

- 重载 `operator->`，返回内部对象的指针。


#### operator*

```cpp
T& operator*() const;
```

- 重载 `operator*`，返回内部对象的引用。


#### operator==

```cpp
bool operator==(T* p) const noexcept;
```

- 判断内部对象的指针值是否与 `p` 相等。


#### operator!=

```cpp
bool operator!=(T* p) const noexcept;
```

- 判断内部对象的指针值是否与 `p` 不相等。


#### operator bool

```cpp
explicit operator bool() const noexcept;
```

- 内部指针为NULL时返回 false，否则返回 true。


#### ref_count

```cpp
size_t ref_count() const noexcept;
```

- 获取内部对象的引用计数。


#### reset

```cpp
void reset();
```

- 若内部指针不为 NULL，则将引用计数减 1(减至 0 时销毁内部对象)，再将内部指针设置为 NULL。


#### swap

```cpp
void swap(shared& x);
void swap(shared&& x);
```

- 交换 `co::shared` 对象的内部指针。


#### use_count

```cpp
size_t use_count() const noexcept;
```

- 与 [ref_count](#ref_count) 等价。



### co::make_shared

```cpp
template<typename T, typename... Args>
inline shared<T> make_shared(Args&&... args);
```

- 用参数 `args` 创建 `shared<T>` 类型的对象。

- 示例

```cpp
co::shared<int> i = co::make_shared<int>(23);
co::shared<fastring> s = co::make_shared<fastring>(32, 'x');
```



### co::unique

```cpp
template<typename T>
class unique;
```

与 `std::unique_ptr` 类似，但有些细微区别。


#### constructor

```cpp
1. constexpr unique() noexcept;
2. constexpr unique(std::nullptr_t) noexcept;
3. unique(unique& x) noexcept;
4. unique(unique&& x) noexcept;

5. unique(unique<X>& x) noexcept;
6. unique(unique<X>&& x) noexcept;
```

- 1-2, 创建空的 `unique` 对象。
- 3-4, 将 `x` 内部的指针转移到所构建的 `unique` 对象中，`x` 内部指针变为 NULL。
- 5-6, 从 `unique<X>` 对象构建 `unique<T>` 对象，`T` 是 `X` 的基类，且 `T` 的析构函数是 virtual 的。

{{< hint warning >}}
`co::unique` 对象不能直接从 `T*` 指针构建，coost 提供 [co::make_unique](#comake_unique) 用于构建 `co::unique` 对象。
{{< /hint >}}


#### destructor

```cpp
~unique();
```

- 销毁内部对象，并释放内存。


#### operator=

```cpp
1. unique& operator=(unique& x);
2. unique& operator=(unique&& x);
3. unique& operator=(unique<X>& x);
4. unique& operator=(unique<X>&& x);
```

- 赋值操作。
- 3-4, 用 `unique<X>` 类型的值对 `unique<T>` 类型的对象进行赋值，`T` 是 `X` 的基类，且 `T` 的析构函数是 virtual 的。


#### get

```cpp
T* get() const noexcept;
```

- 获取内部对象的指针。


#### operator->

```cpp
T* operator->() const;
```

- 重载 `operator->`，返回内部对象的指针。


#### operator*

```cpp
T& operator*() const;
```

- 重载 `operator*`，返回内部对象的引用。


#### operator==

```cpp
bool operator==(T* p) const noexcept;
```

- 判断内部对象的指针值是否与 `p` 相等。


#### operator!=

```cpp
bool operator!=(T* p) const noexcept;
```

- 判断内部对象的指针值是否与 `p` 不相等。


#### operator bool

```cpp
explicit operator bool() const noexcept;
```

- 内部指针为NULL时返回 false，否则返回 true。


#### reset

```cpp
void reset();
```

- 销毁内部对象，并释放内存。


#### swap

```cpp
void swap(unique& x);
void swap(unique&& x);
```

- 交换 `co::unique` 对象的内部指针。



### co::make_unique

```cpp
template<typename T, typename... Args>
inline unique<T> make_unique(Args&&... args);
```

- 用参数 `args` 创建 `unique<T>` 类型的对象。

- 示例

```cpp
co::unique<int> i = co::make_unique<int>(23);
co::unique<fastring> s = co::make_unique<fastring>(32, 'x');
```
