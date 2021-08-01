---
weight: 19
title: "LruMap"
---

include: [co/lru_map.h](https://github.com/idealvin/co/blob/master/include/co/lru_map.h).


## LruMap


```cpp
template <typename K, typename V> class LruMap;
```


`LruMap` is a map implemented based on the LRU (least recently used) strategy. When the number of elements in the map reaches the upper limit, the least recently used data will be replaced first. It is implemented based on std::list and std::unordered_map, and the internal elements are unordered. 


### LruMap::LruMap


```cpp
LruMap();
explicit LruMap(size_t capacity);
```


- The default constructor uses 1024 as the maximum capacity.
- The second constructor takes the parameter capacity as the maximum capacity.



### LruMap::begin


```cpp
iterator begin() const;
```


- Returns an iterator pointing to the first element. When LruMap is empty, the return value is equal to LruMap::end().



### LruMap::clear


```cpp
void clear();
```


- This method clears the elements in LruMap, size() will become 0, and the capacity will remain unchanged.



### LruMap::empty


```cpp
bool empty() const;
```


- This method determines whether LruMap is empty.



### LruMap::end


```cpp
iterator end() const;
```


- Returns an iterator pointing to the next position of the last element, it does not point to any element itself. When LruMap is empty, begin() and end() are equal.



### LruMap::erase


```cpp
void erase(iterator it);
void erase(const key_type& key);
```


- Erase an element through iterator or key.



### LruMap::find


```cpp
iterator find(const key_type& key)
```


- This method finds an element by key, and returns an iterator refer to the element if it is found, otherwise the return value is equal to end().



### LruMap::insert


```cpp
template <typename Key, typename Val>
void insert(Key&& key, Val&& value);
```


- Insert an element, only when the key does not exist, it will insert a new element. If the key already exists, no operation will be performed.
- During the insert operation, if the number of elements has reached the maximum capacity, the least recently accessed element will be removed.



### LruMap::size


```cpp
size_t size() const;
```


- Returns the number of elements in LruMap. 




### LruMap::swap


```cpp
void swap(LruMap& x) noexcept;
void swap(LruMap&& x) noexcept;
```


- Exchange the contents of two LruMaps. This operation only exchanges internal pointers, size, and capacity.



### Code example


```cpp
LruMap<int, int> m(128); // capacity: 128

auto it = m.find(1);
if (it == m.end()) {
    m.insert(1, 23);
} else {
    it->second = 23;
}

m.erase(it);        // erase by iterator
m.erase(it->first); // erase by key
m.clear();          // clear the map
```
