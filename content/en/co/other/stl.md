---
weight: 5
title: "STL"
---

include: [co/stl.h](https://github.com/idealvin/coost/blob/master/include/co/stl.h).


## Common containers

In the table below, the containers in coost on the left are equivalent to the corresponding std versions on the right, only the internal memory allocators are different.

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
When the key is of `const char*` type (C-style string), `co::map`, `co::set`, `co::multimap`, `co::multiset`, `co:: hash_map`, `co::hash_set` will compare the key and calculate the hash value of the key according to the content of the string.
{{< /hint >}}




## co::lru_map

```cpp
template<typename K, typename V>
class lru_map;
```

`co::lru_map` is a map implemented based on the LRU (least recently used) strategy. When the number of elements in the map reaches the upper limit, the least recently used data will be replaced first. It is implemented based on `co::hash_map` and `co::list`, and the internal elements are unordered.



### constructor

```cpp
1. lru_map();
2. explicit lru_map(size_t capacity);
```

- 1, the default constructor, uses 1024 as the maximum capacity.
- 2, use the parameter `capacity` as the maximum capacity.



### begin

```cpp
iterator begin() const;
```

- Returns an iterator refer to the first element. When lru_map is empty, the return value is equal to [end()](#lru_mapend).



### clear

```cpp
void clear();
```

- Clear the elements in lru_map, the size will become 0, and the capacity will remain unchanged.



### empty

```cpp
bool empty() const;
```

- Determine whether lru_map is empty.



### end

```cpp
iterator end() const;
```

- Returns an iterator refer to the next position of the last element.



### erase

```cpp
void erase(iterator it);
void erase(const key_type& key);
```

- Remove element by iterator or key.



### find

```cpp
iterator find(const key_type& key)
```

- Find the element corresponding to `key`.



### insert

```cpp
template<typename Key, typename Val>
void insert(Key&& key, Val&& value);
```

- Inserts an element, only if the key does not exist, a new element will be inserted. If the key already exists, nothing will be done.
- If the number of elements has reached the maximum capacity, the least recently used element will be deleted.



### size

```cpp
size_t size() const;
```

- Returns the number of elements in lru_map.



### swap

```cpp
void swap(lru_map& x) noexcept;
void swap(lru_map&& x) noexcept;
```

- Swap the contents of two lru_maps.



### Code example

```cpp
co::lru_map<int, int> m(128); // capacity: 128

auto it = m.find(1);
if (it == m.end()) {
     m.insert(1, 23);
} else {
     it->second = 23;
}

m.erase(it); // erase by iterator
m.erase(1);  // erase by key
m.clear();   // clear the map
```




## co::vector

`std::vector<bool>` in the C++ standard library is an unwise design. For this reason, coost implemented `co::vector` separately.


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

- 1, the default constructor, constructs an empty vector with size and capacity both being 0.
- 2, **Construct an empty vector** with capacity as `cap`.
- 3, copy constructor.
- 4, Move constructor, after construction, `x` becomes an empty object.
- 5, Construct vector with a initialization list, `T` is the type of elements in the vector.
- 6, Construct vector from an array, `p` points to the first element of the array, `n` is the length of the array.
- 7, Construct vector with elements in range `[beg, end)`.
- 8, There are two cases: when `X` is not int or the element type `T` is int, the vector is initialized with `n` values of `x`; when `X` is int and the element type `T` is not int, the vector is initialized to `n` default values ​​of type `T`.

{{< hint warning >}}
The behavior of constructor 2 is different from the corresponding version in `std::vector`. To construct a vector of `n` default values, you can use constructor 8 (the second parameter passes 0):
```cpp
co::vector<fastring> v(32, 0);
```
{{< /hint >}}

- Example

```cpp
co::vector<int> a(32);        // size: 0, capacity: 32
co::vector<int> b = { 1, 2 }; // [1,2]
co::vector<int> v(8, 0);      // contains 8 zeros
co::vector<fastring> s(8, 0); // contains 8 empty fastring objects
```



### destructor

```cpp
~vector();
```

- Release the memory, and the vector becomes an empty object after destruction.



### operator=

```cpp
1. vector& operator=(const vector& x);
2. vector& operator=(vector&& x);
3. vector& operator=(std::initializer_list<T> x);
```

- Assignment operations.

- Example

```cpp
co::vector<int> v;
v = { 1, 2, 3 };

co::vector<int> x;
x = v;
x = std::move(v);
```



### ————————————
### back

```cpp
T& back();
const T& back() const;
```

- Returns a reference to the last element in vector.

{{< hint warning >}}
If vector is empty, calling this method results in undefined behavior.
{{< /hint >}}



### front

```cpp
T& front();
const T& front() const;
```

- Returns a reference to the first element of the vector.

{{< hint warning >}}
If vector is empty, calling this method results in undefined behavior.
{{< /hint >}}



### operator[]

```cpp
T& operator[](size_t n);
const T& operator[](size_t n) const;
```

- Returns a reference to the nth element of vector.

{{< hint warning >}}
If `n` is outside a reasonable range, calling this method results in undefined behavior.
{{< /hint >}}



### ————————————
### begin

```cpp
iterator begin() const noexcept;
```

- Returns an iterator pointing to the first element.



### end

```cpp
iterator end() const noexcept;
```

- Returns the end iterator.



### ————————————
### capacity

```cpp
size_t capacity() const noexcept;
```

- Returns the capacity of the vector.



### data

```cpp
T* data() const noexcept;
```

- Returns a pointer to the internal elements of the vector.



### empty

```cpp
bool empty() const noexcept;
```

- Determine whether the vector is empty.



### size

```cpp
size_t size() const noexcept;
```

- Returns the number of elements in the vector.



### ————————————
### clear

```cpp
void clear();
```

- Empty the vector, the size becomes 0, and the capacity remains unchanged.



### reserve

```cpp
void reserve(size_t n);
```

- Adjust the capacity of the vector to ensure that the capacity is at least `n`.
- When `n` is less than the original capacity, keep the capacity unchanged.



### reset

```cpp
void reset();
```

- Destroy all elements in the vector and release the memory
.



### resize

```cpp
void resize(size_t n);
```

- Resize vector to `n`.
- When `n` is larger than the original size, fill the extended part with the default value.



### swap

```cpp
void swap(vector& x) noexcept;
void swap(vector&& x) noexcept;
```

- Swaps two vectors.



### ————————————
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

- Add elements to the end of the vector.
- 1-2, append a single element.
- 3, append `n` elements `x`.
- 4, append an array, `n` is length of the array.
- 5-6, append all elements in vector `x`.
- 7, append elements in range `[beg, end)`.

- Example

```cpp
int a[4] = { 1, 2, 3, 4 };
co::vector<int> v;
v. append(7);
v. append(v);
v. append(a, 4);
```



### emplace_back

```cpp
template<typename ... X>
void emplace_back(X&& ... x);
```

- Insert a new element constructed with the arguments `x...` at the end of the vector.

- Example

```cpp
co::vector<fastring> v;
v. emplace_back(4, 'x'); // append("xxxx")
```



### push_back

```cpp
void push_back(const T& x);
void push_back(T&& x);
```

- Appends the element `x` to the end of the vector.



### pop_back

```cpp
T pop_back();
```

- Pop and return the last element of the vector.

{{< hint warning >}}
If vector is empty, calling this method results in undefined behavior.
{{< /hint >}}



### remove

```cpp
void remove(size_t n);
```

- Remove the nth element and moves the last element to the position of the deleted element.

- Example

```cpp
co::vector<int> v = { 1, 2, 3, 4 };
v. remove(1); // v -> [1, 4, 3]
v. remove(2); // v -> [1, 4]
```



### remove_back

```cpp
void remove_back();
```

- Remove the last element of the vector, and do nothing if the vector is empty.
