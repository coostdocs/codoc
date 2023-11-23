---
weight: 3
title: "Memory Allocation"
---

include: [co/mem.h](https://github.com/idealvin/coost/blob/master/include/co/mem.h).



### co::alloc

```cpp
1. void* alloc(size_t size);
2. void* alloc(size_t size, size_t align);
```

- 1, allocate `size` bytes of memory.
- 2, allocate `size` bytes of memory, and the memory boundary is `align` byte-aligned (align <= 1024).

{{< hint warning >}}
In the 2nd, `align` must be power of 2 and cannot exceed 1024.
{{< /hint >}}



### co::free

```cpp
void free(void* p, size_t size);
```

- Free memory allocated by [co::alloc](#coalloc) or [co::realloc](#corealloc), where `size` is the size of the allocated memory.

{{< hint warning >}}
`co::free` is different from the `::free` provided by the system, it needs an additional `size` parameter.
{{< /hint >}}



### co::realloc

```cpp
void* realloc(void* p, size_t old_size, size_t new_size);
```

- Reallocate memory, `old_size` is the previous memory size, `new_size` is the new size, **the latter must be greater than the former**.



### co::zalloc

```cpp
void* zalloc(size_t size);
```

- Allocate `size` bytes of memory and fill the memory with zeros.



### ———————————
### co::make

```cpp
template<typename T, typename... Args>
inline T* make(Args&&... args);
```

- Call [co::alloc](#coalloc) to allocate memory and make an object of type `T` on it with argument `args`.

- Example

```cpp
int* p = co::make<int>(7);
std::string* s = co::make<std::string>(3, 'x');
```



### co::del

```cpp
template<typename T>
inline void del(T* p, size_t n=sizeof(T));
```

- Destroy objects created by [co::make](#comake) and free the memory.
- The parameter `n` is the memory size of the object pointed to by `p`, and the default is `sizeof(T)`.

- Example

```cpp
int* p = co::make<int>(7);
co::del(p);
```



### co::make_rootic

```cpp
template<typename T, typename... Args>
inline T* make_rootic(Args&&... args);
```

- Similar to [co::make_static](#comake_static), except that static objects created by this function are destructed later than static objects created by `co::make_static`.



### co::make_static

```cpp
template<typename T, typename... Args>
inline T* make_static(Args&&... args);
```

- Create a static object of type `T` with the parameter `args`, and it will be automatically destroyed when the program exits.

- Example

```cpp
fasting& s = *co::make_static<fastring>(32, 'x');
```



### ———————————
### co::shared

```cpp
template<typename T>
class shared;
```

Similar to `std::shared_ptr`, but with some minor differences.


#### constructor

```cpp
1. constexpr shared() noexcept;
2. constexpr shared(std::nullptr_t) noexcept;
3. shared(const shared& x) noexcept;
4. shared(shared&& x) noexcept;

5. shared(const shared<X>& x) noexcept;
6. shared(shared<X>&& x) noexcept;
```

- 1-2, create an empty `shared` object.
- 3, Copy constructor, if `x` is not an empty object, increase the internal reference count.
- 4, Move constructor, when the construction is completed, `x` becomes an empty object.
- 5-6, Construct `shared<T>` object from `shared<X>` object, `T` is the base class of `X`, and the destructor of `T` is virtual.

{{< hint warning >}}
`co::shared` objects cannot be constructed directly from `T*` pointer, coost provides [co::make_shared](#comake_shared) for constructing `co::shared` objects.
{{< /hint >}}


#### destructor

```cpp
~shared();
```

- If the object is not empty, the internal reference count will be decremented by 1. When the reference count is reduced to 0, the internal object will be destroyed and the memory will be released.


#### operator=

```cpp
1. shared& operator=(const shared& x);
2. shared& operator=(shared&& x);
3. shared& operator=(const shared<X>& x);
4. shared& operator=(shared<X>&& x);
```

- Assignment operations.
- 3-4, assign value of `shared<X>` to object of `shared<T>`, `T` is the base class of `X`, and the destructor of `T` is virtual .


#### get

```cpp
T* get() const noexcept;
```

- Get a pointer to the internal object.


#### operator->

```cpp
T* operator->() const;
```

- Overload `operator->`, return a pointer to the internal object.


#### operator*

```cpp
T& operator*() const;
```

- Overload `operator*`, return a reference to the internal object.


#### operator==

```cpp
bool operator==(T* p) const noexcept;
```

- Determine whether the internal pointer is equal to `p`.


#### operator!=

```cpp
bool operator!=(T* p) const noexcept;
```

- Determine whether the internal pointer is not equal to `p`.


#### operator bool

```cpp
explicit operator bool() const noexcept;
```

- Returns false if the internal pointer is NULL, otherwise returns true.


#### ref_count

```cpp
size_t ref_count() const noexcept;
```

- Get the reference count of the internal object.


#### reset

```cpp
void reset();
```

- If the internal pointer is not NULL, decrement the reference count by 1 (destroy the internal object when reduced to 0), and then set the internal pointer to NULL.


#### swap

```cpp
void swap(shared& x);
void swap(shared&& x);
```

- Swaps internal pointers of `co::shared` objects.


#### use_count

```cpp
size_t use_count() const noexcept;
```

- Equivalent to [ref_count](#ref_count).



### co::make_shared

```cpp
template<typename T, typename... Args>
inline shared<T> make_shared(Args&&... args);
```

- Creates an object of type `shared<T>` with arguments `args`.

- Example

```cpp
co::shared<int> i = co::make_shared<int>(23);
co::shared<fastring> s = co::make_shared<fastring>(32, 'x');
```



### co::unique

```cpp
template<typename T>
class unique;
```

Similar to `std::unique_ptr`, but with some minor differences.


#### constructor

```cpp
1. constexpr unique() noexcept;
2. constexpr unique(std::nullptr_t) noexcept;
3. unique(unique& x) noexcept;
4. unique(unique&& x) noexcept;

5. unique(unique<X>& x) noexcept;
6. unique(unique<X>&& x) noexcept;
```

- 1-2, create an empty `unique` object.
- 3-4, transfer the internal pointer of `x` to the constructed `unique` object, and the internal pointer of `x` becomes NULL.
- 5-6, Construct `unique<T>` object from `unique<X>` object, `T` is the base class of `X`, and the destructor of `T` is virtual.

{{< hint warning >}}
`co::unique` object cannot be constructed directly from `T*` pointer, coost provides [co::make_unique](#comake_unique) for constructing `co::unique` object.
{{< /hint >}}


#### destructor

```cpp
~unique();
```

- Destroy the internal object, and free the memory.


#### operator=

```cpp
1. unique& operator=(unique& x);
2. unique& operator=(unique&& x);
3. unique& operator=(unique<X>& x);
4. unique& operator=(unique<X>&&x);
```

- Assignment operations.
- 3-4, assign value of `unique<X>` to object of `unique<T>`, `T` is the base class of `X`, and the destructor of `T` is virtual .


#### get

```cpp
T* get() const noexcept;
```

- Get a pointer to the internal object.


#### operator->

```cpp
T* operator->() const;
```

- Overload `operator->`, return a pointer to the internal object.


#### operator*

```cpp
T& operator*() const;
```

- Overload `operator*`, return a reference to the internal object.


#### operator==

```cpp
bool operator==(T* p) const noexcept;
```

- Determine whether the internal pointer is equal to `p`.


#### operator!=

```cpp
bool operator!=(T* p) const noexcept;
```

- Determine whether the internal pointer is not equal to `p`.


#### operator bool

```cpp
explicit operator bool() const noexcept;
```

- Returns false if the internal pointer is NULL, otherwise returns true.


#### reset

```cpp
void reset();
```

- Destroy the internal object, and free the memory.


#### swap

```cpp
void swap(unique& x);
void swap(unique&& x);
```

- Swap the internal pointers of `co::unique` objects.



### co::make_unique

```cpp
template<typename T, typename... Args>
inline unique<T> make_unique(Args&&... args);
```

- Create an object of type `unique<T>` with arguments `args`.

- Example

```cpp
co::unique<int> i = co::make_unique<int>(23);
co::unique<fastring> s = co::make_unique<fastring>(32, 'x');
```
