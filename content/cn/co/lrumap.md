---
weight: 19
title: "LruMap"
---


include: [co/lru_map.h](https://github.com/idealvin/co/blob/master/include/co/lru_map.h).


## LruMap
```cpp
template <typename K, typename V> class LruMap;
```
`LruMap` 是基于 LRU (least recently used) 策略实现的 map，当 map 中元素数量达到上限时，优先替换掉最近最少使用的数据。它基于 std::list、std::unordered_map 实现，内部元素是无序的。




### LruMap::LruMap
```cpp
LruMap();
explicit LruMap(size_t capacity);
```

- 默认构造函数使用 1024 作为最大容量。
- 第二个构造函数以参数 capacity 作为最大容量。





### LruMap::begin
```cpp
iterator begin() const;
```

- 返回指向第一个元素的 iterator，当 LruMap 为空时，返回值与 LruMap::end() 相等。





### LruMap::clear
```cpp
void clear();
```

- 此方法清空 LruMap 内的元素，size() 会变成 0，容量保持不变。





### LruMap::empty
```cpp
bool empty() const;
```

- 此方法判断 LruMap 是否为空。





### LruMap::end
```cpp
iterator end() const;
```

- 返回指向最后一个元素的下一个位置的 iterator，它本身并不指向任何元素。当 LruMap 为空时，begin() 与 end() 相等。





### LruMap::erase
```cpp
void erase(iterator it);
void erase(const key_type& key);
```

- 通过 iterator 或 key 删除元素。





### LruMap::find
```cpp
iterator find(const key_type& key)
```

- 此方法通过 key 查找元素，若找到则返回指向该元素的 iterator，否则返回值等于 end()。





### LruMap::insert
```cpp
template <typename Key, typename Val>
void insert(Key&& key, Val&& value);
```

- 插入元素，仅当 key 不存在时，才会插入新元素。若 key 已经存在，则不会进行任何操作。
- 插入元素时，若元素数量已经达到最大容量，则会删除最近最少访问的元素。





### LruMap::size
```cpp
size_t size() const;
```

- 此方法返回 LruMap 中的元素个数。






### LruMap::swap
```cpp
void swap(LruMap& x) noexcept;
void swap(LruMap&& x) noexcept;
```

- 交换两个 LruMap 的内容，此操作仅交换内部指针、大小、容量等信息。



### 代码示例
```cpp
LruMap<int, int> m(128); // capacity: 128

auto it = m.find(1);
if (it == m.end()) {
    m.insert(1, 23);
} else {
    it->second = 23;
}

m.erase(it);         // erase by iterator
m.erase(it->first);  // erase by key
m.clear();           // clear the map
```


