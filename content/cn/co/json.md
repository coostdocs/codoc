---
weight: 8
title: "JSON"
---

include: [co/json.h](https://github.com/idealvin/co/blob/master/include/co/json.h).


`co/json` 是一个类似 [rapidjson](https://github.com/Tencent/rapidjson) 的 JSON 库，与 rapidjson 相比，它既有性能上的优势，同时又更简单易用。


## 基本概念

[JSON](https://www.json.org/json-en.html) 是一种简单的数据格式，它支持两种数据结构：

- 由一系列 **key/value** 键值对构成的集合，这类结构称为 **object**，对应编程语言中的 struct, map 等等。
- 由一系列 **value** 构成的列表，这类结构称为 **array**，对应编程语言中的 vector, list 等等。

上述 key 是 string，value 一般也称为 **JSON value** (co/json 中用 **Json** 类表示)，可以是 object, array, number, string, bool(false, true), null 中的任意一种。number 是整数或浮点数，大部分实现会将整数与浮点数区分开。

object 由一对大括号括起来，array 由一对中括号括起来，它们看起来像下面这样：

```cpp
{"a":1, "b":false, "s":"xxx"}
[1, 2, 3]
```

由定义 object 与 array 可以嵌套，从而可以表示树等复杂数据结构。




## Json

### Json::Json

```cpp
 1. Json() noexcept;
 2. Json(decltype(nullptr)) noexcept;

 3. Json(Json&& v) noexcept;
 4. Json(Json& v) noexcept;
    Json(const Json& v) = delete;

 5. Json(bool v);
 6. Json(double v);
 7. Json(int64 v);
 8. Json(int32 v);
 9. Json(uint32 v);
10. Json(uint64 v);

11. Json(const void* p, size_t n);
12. Json(const char* s);
13. Json(const fastring& s);
14. Json(const std::string& s);

15. Json(std::initializer_list<Json> v);
```

- 1-2, 构建一个 null 对象。
- 3-4, move 构造与拷贝构造函数，**二者均执行 move 语义**，构造函数执行后，参数 `v` 将变为一个 null 对象。
- 5, 构造 bool 类型的 JSON 对象。
- 6, 构造 double 类型的 JSON 对象。
- 7-10, 构造整数类型的 JSON 对象。
- 11-14, 构造字符串类型的 JSON 对象。
- 15, 根据初始化列表构建 object 或 array 对象。


- 示例

```cpp
Json a;          // null
Json b(nullptr); // null
Json c = false;  // bool
Json d = 3.14;   // double
Json e = 23;     // integer
Json f = "xx";   // string

Json g = {1, 2, 3};  // g -> [1, 2, 3]
Json h = {"a", "b"}; // h -> ["a", "b"]

Json i = {           // i -> { "a": "b" }
    {"a", "b"}
};

Json j = {           // j -> {"a": 1, "b": [1,2,3]}
    {"a", 1},
    {"b", {1, 2, 3}},
};

Json x(i);            // i -> null
Json y(std::move(j)); // j -> null
```



### json::array

```cpp
Json array();
```

- 此函数在 namespace json 下，它返回一个空的 array 对象。



### json::object

```cpp
Json object();
```

- 此函数在 namespace json 下，它返回一个空的 object 对象。



### Json::operator=

```cpp
Json& operator=(Json&& v);
Json& operator=(Json& v);
void operator=(const Json&) = delete;
```

- 赋值操作，上面的两个方法等价，**该操作执行后，参数 `v` 变成 null 对象**。



### Json::dup

```cpp
Json dup() const;
```

- 深度拷贝一个 JSON 对象。


- 示例

```cpp
Json x = {1, 2, 3}; // x -> [1,2,3]
Json y, z;
y = x;       // x -> null, y -> [1,2,3]
z = y.dup(); // y:[1,2,3], z -> [1,2,3]
```




### ———————————
### Json::is_null

```cpp
bool is_null() const;
```

- 判断 Json 对象是否为 null。



### Json::is_bool

```cpp
bool is_bool() const;
```

- 判断 Json 对象是否为 bool 类型。



### Json::is_int

```cpp
bool is_int() const;
```

- 判断 Json 对象是否为整数类型。



### Json::is_double

```cpp
bool is_double() const;
```

- 判断 Json 对象是否为 double 类型。



### Json::is_string

```cpp
bool is_string() const;
```

- 判断 Json 对象是否为字符串类型。



### Json::is_array

```cpp
bool is_array() const;
```

- 判断 Json 对象是否为 array 类型。



### Json::is_object

```cpp
bool is_object() const;
```

- 判断 Json 对象是否为 object 类型。




### ———————————
### Json::as_bool

```cpp
bool as_bool() const;
```

- 获取 bool 类型的值。
- 对于 int 或 double 类型，若值为 0，返回 false，否则返回 true。
- 对于 string 类型，若值为 `"true"` 或 `"1"`，返回 true，否则返回 false。
- 对于其他非 bool 类型，返回 false。



### Json::as_int

```cpp
int as_int() const;
int32 as_int32() const;
int64 as_int64() const;
```

- 获取整数类型的值。
- 对于 bool, double 或 string 类型，结果自动转换为整数类型。
- 对于其他非整数类型，返回 0。



### Json::as_double

```cpp
double as_double() const;
```

- 获取 double 类型的值。
- 对于 bool, int 或 string 类型，结果自动转换为 double 类型。
- 对于其他非 double 类型，返回 0。



### Json::as_string

```cpp
fastring as_string() const;
```

- 获取字符串类型的值，返回 fastring。
- 对于非 string 类型，此方法等价于 [Json::str()](#jsonstr)，结果将自动转换为 string 类型。



### Json::as_c_str

```cpp
const char* as_c_str() const;
```

- 返回 `\0` 结尾的 C 风格字符串，可以用 [string_size()](#jsonstring_size) 获取其长度，一般用于对性能要求较高的地方。
- 对于非 string 类型，返回空字符串。



### Json::get

```cpp
1. Json& get(uint32 i) const;
2. Json& get(int i) const;
3. Json& get(const char* key) const;

4. template <class T,  class ...X>
   inline Json& get(T&& v, X&& ... x) const;
```

- 根据 index 或 key 获取 JSON 对象，**此方法是只读操作，不会修改调用此方法的 JSON 对象**。
- 1-2, 获取 array 对象的第 i 个元素，若调用此方法的 JSON 对象不是 array 类型，或者 `i` 超出了 array 的范围，返回结果将引用一个 null 对象。
- 3, 获取 `key` 对应的 JSON value 对象，若调用此方法的 JSON 对象不是 object 类型，或者 `key` 不存在，返回结果将引用一个 null 对象。
- 4, 可以带任意数量的参数，每个参数是 index 或者 key。该方法遇到第一个不合适的 index 或 不存在的 key 时，立即返回，返回结果将引用一个 null 对象。



### Json::set

```cpp
template <class T>
inline Json& set(T&& v) {
    return *this = Json(std::forward<T>(v));
}

template <class A, class B,  class ...X>
inline Json& set(A&& a, B&& b, X&& ... x);
```

- 设置 JSON 对象的值。
- `set` 方法最后一个参数是所设置的值，其他参数是 index 或者 key。



### 代码示例

```cpp
Json r = {
    { "a", 7 },
    { "b", false },
    { "c", { 1, 2, 3 } },
    { "s", "23" },
};

r.get("a").as_int();    // 7
r.get("b").as_bool();   // false
r.get("s").as_string(); // "23"
r.get("s").as_int();    // 23
r.get("c", 0).as_int(); // 1
r.get("c", 1).as_int(); // 2

// x -> {"a":1,"b":[0,1,2],"c":{"d":["oo"]}}
Json x;
x.set("a", 1);
x.set("b", Json({0,1,2}));
x.set("c", "d", 0, "oo");
```




### ———————————
### Json::operator==

```cpp
bool operator==(bool v) const;
bool operator==(double v) const;
bool operator==(int64 v) const;
bool operator==(int v) const;
bool operator==(uint32 v) const;
bool operator==(uint64 v) const;
bool operator==(const char* v) const;
bool operator==(const fastring& v) const;
bool operator==(const std::string& v) const;
```

- 判断 Json 对象的值是否等于 `v`。
- 若 Json 对象的类型与 `v` 不同，则直接返回 false。



### Json::operator!=

```cpp
bool operator!=(bool v) const;
bool operator!=(double v) const;
bool operator!=(int64 v) const;
bool operator!=(int v) const;
bool operator!=(uint32 v) const;
bool operator!=(uint64 v) const;
bool operator!=(const char* v) const;
bool operator!=(const fastring& v) const;
bool operator!=(const std::string& v) const;
```

- 判断 Json 对象的值是否不等于 `v`。
- 若 Json 对象的类型与 `v` 不同，则直接返回 true。



### 代码示例

```cpp
Json x = {
    {"a", 3},
    {"b", false},
    {"s", "xx"},
};

x == 7;          // false
x["a"] == 3;     // true
x["b"] == false; // true
x["s"] == "xx";  // true
```




### ———————————
### Json::add_member

```cpp
Json& add_member(const char* key, Json&& v);
Json& add_member(const char* key, Json& v);
```

- 向 object 类型的 Json 中添加 key-value 键值对 (非 object 对象调用此方法后自动变成 object)。
- 此方法将保持 key 的添加顺序，key 可能重复出现。
- 参数 key 是 `'\0'` 结尾的 C 字符串，参数 v 是所添加的值。
- 参数 v 执行 move 语义，调用此方法后，v 变为 null 对象。
- **NOTE：** co/json 出于性能上的考虑，要求 key 中不能包含双引号。


- 示例

```cpp
Json r;
r.add_member("a", 1);    // r -> {"a":1}
r.add_member("d", 3.3);  // r -> {"a":1, "d":3.3}
r.add_member("s", "xx"); // r -> {"a":1, "d":3.3, "s":"xx"}

Json x;
x.add_member("xx", r);                        // r -> null
r.add_member("o", Json().add_member("x", 3)); // r -> {"o":{"x":3}}
Json().add_member("o", 1).add_member("k", 2); // -> {"o":1,"k":2}
```



### Json::push_back

```cpp
Json& push_back(Json&& v);
Json& push_back(Json& v);
```

- 向 array 类型的 Json 中添加元素(非 array 对象调用此方法后自动变成 array)。
- 参数 v 执行 move 语义，调用此方法后，v 变为 null 对象。


- 示例

```cpp
Json r;
r.push_back(1);    // r -> [1]
r.push_back(3.3);  // r -> [1, 3.3]
r.push_back("xx"); // r -> [1, 3.3, "xx"]

Json x;
x.push_back(r);  // r -> null, x -> [[1, 3.3, "xx"]]
r.push_back(Json().push_back(1).push_back(2)); // r -> [[1,2]]
```



### Json::reset

```cpp
void reset();
```

- 重置 Json 对象为 null。



### Json::swap

```cpp
void swap(Json& v) noexcept;
void swap(Json&& v) noexcept;
```

- 交换两个 Json 对象的内容。




### ———————————
### Json::operator[]

```cpp
Json& operator[](uint32 i) const;
Json& operator[](int i) const;
Json& operator[](const char* key) const;
```

- 重载 `operator[]`，根据 index 或 key 获取 Json 对象中的元素。
- 1-2, 适用于 array 类型，获取 array 对象的第 i 个元素，i 必须在 array 大小范围内。
- 3, 适用于 object 类型，key 不存在时，会在 Json 中插入一个 null 对象。
- **一般情况下，建议尽量用只读的 `get()` 方法取代此操作。**


- 示例

```cpp
Json r = {
    { "a", 7 },
    { "x", { 1, 2, 3 } },
};

r["a"].as_int();    // 7
r["x"][0].as_int(); // 1
```



### Json::has_member

```cpp
bool has_member(const char* key) const;
```

- 判断 Json 对象中是否存在 key 对应的元素。
- 若调用此方法的 Json 对象不是 object 类型，返回 false。


- 示例

```cpp
Json r = {{"a", 1}};
r.has_member("a"); // true
r.has_member("x"); // false
```



### Json::size

```cpp
uint32 size() const;
```

- 若 Json 是 object 或 array 类型，此方法返回元素个数。
- 若 Json 是 string 类型，此方法返回字符串长度。
- 所有其他类型，此方法返回 0。


- 示例

```cpp
Json r = {
    {"x", 1},
    {"s", "hello"},
    {"a", {1, 2, 3}},
};

r.size();      // 3
r["x"].size(); // 0
r["s"].size(); // 5
r["a"].size(); // 3
```



### Json::empty

```cpp
bool empty() const;
```

- 判断 Json 对象是否为空，等价于 `size() == 0`。



### Json::string_size

```cpp
uint32 string_size() const;
```

- 返回 string 类型的长度，若调用此方法的 Json 对象不是 string 类型，返回 0。



### Json::array_size

```cpp
uint32 array_size() const;
```

- 返回 array 类型的元素个数，若调用此方法的 Json 对象不是 array 类型，返回 0。



### Json::object_size

```cpp
uint32 object_size() const;
```

- 返回 object 类型的元素个数，若调用此方法的 Json 对象不是 object 类型，返回 0。




### ———————————
### Json::str

```cpp
fastream& str(fastream& s, int mdp=16) const;
fastring& str(fastring& s, int mdp=16) const;
fastring str(int mdp=16) const;
```

- 将 Json 对象转换成字符串。
- 第 1 个版本将 JSON 字符串追加到 fastream 中，返回值与参数 s 相同。
- 第 2 个版本将 JSON 字符串追加到 fastring 中，返回值与参数 s 相同。
- 第 3 个版本直接返回 JSON 字符串。
- 参数 `mdp` 是 `max decimal places` 的缩写，表示最多保留多少位小数。



### Json::pretty

```cpp
fastream& pretty(fastream& s, int mdp=16) const;
fastring& pretty(fastring& s, int mdp=16) const;
fastring pretty(int mdp=16) const;
```

- 将 Json 对象转换成更漂亮的 JSON 字符串，除了结果好看点，其他与 `Json::str()` 一样。



### Json::dbg

```cpp
fastream& dbg(fastream& s, int mdp=16) const;
fastring& dbg(fastring& s, int mdp=16) const;
fastring dbg(int mdp=16) const;
```

- 将 Json 对象转换成 debug 字符串，当 string 类型的值长度超过 512 字节时，截断取前 32 字节，其他与 `Json::str()` 一样。
- 此方法一般用于打印日志。有些应用场景中，Json 对象可能包含较长的 string，如一个图片文件的 base64 编码，这个时候用 `Json::dbg()` 取代 `Json::str()` 方法，可以避免打印过多无意义的日志。



### Json::parse_from

```cpp
bool parse_from(const char* s, size_t n);
bool parse_from(const char* s);
bool parse_from(const fastring& s);
bool parse_from(const std::string& s);
```

- 从 JSON 字符串解析 Json 对象。
- 第 1 个版本中，s 不要求以 `'\0'` 结尾。
- 解析成功时，返回 true，否则返回 false。
- 解析失败时，Json 变成 null 对象。



### json::parse

```cpp
Json parse(const char* s, size_t n);
Json parse(const char* s);
Json parse(const fastring& s);
Json parse(const std::string& s);
```

- 从 JSON 字符串解析 Json 对象。
- 此函数不是 Json 类中的方法，而是定义于 `namespace json` 下的函数。
- 此函数返回一个 Json 对象，解析失败时，返回 null 对象。



### 代码示例

```cpp
Json r = {
    { "a", {1,2,3} }
};

fastring s = r.str();    // s -> {"a":[1,2,3]}
fastring p = r.pretty(); 
LOG << r.dbg();          // print json debug string
LOG << r;                // the same as above, but is  more efficient

Json x;
x.parse_from(s);
x.parse_from(p);

Json v = json::parse(s);
```




### ———————————
### Json::begin

```cpp
iterator begin() const;
```

- 返回指向 Json 对象的 beginning iterator。
- 若调用此方法的 Json 对象不是 array 或 object 类型，返回值等于 [Json::end()](#jsonend)。



### Json::end

```cpp
const iterator::End& end() const;
```

- 返回一个假的 end iterator。
- 返回值实际上并不是一个 iterator 对象，但 iterator 可以与它比较，若 iterator 与 `Json::end()` 相等，表示没有更多的元素了。



### Json::iterator

#### iterator::operator==

```cpp
bool operator==(const End&) const;
```

- 判断 iterator 是否等于 `End`，End 是一个假的 end iterator。


#### iterator::operator!=

```cpp
bool operator!=(const End&) const;
```

- 判断 iterator 是否不等于 `End`，End 是一个假的 end iterator。


#### iterator::operator++

```cpp
iterator& operator++();
```

- 重载前缀 `++` 操作，不支持后缀 `++` 操作。


#### iterator::operator*

```cpp
Json& operator*() const;
```

- 重载 `operator*`，此方法**仅适用于 array 类型的 iterator**。
- Json 为 array 时，iterator 指向 array 中的元素。


#### iterator::key

```cpp
const char* key() const;
```

- 此方法**仅适用于 object 类型的 iterator**。
- Json 为 object 时，iterator 指向 object 中的 key-value 键值对，此方法返回该键值对中的 key。


#### iterator::value

```cpp
Json& value() const;
```

- 此方法**仅适用于 object 类型的 iterator**。
- Json 为 object 时，iterator 指向 object 中的 key-value 键值对，此方法返回该键值对中的 value 的引用。



### 遍历 array 或 object

co/json 支持用 iterator 遍历 array 或 object 类型的 Json 对象。

```cpp
// {"i":7, "s":"xx", "a":[123, true, "nice"]}
Json r = {
    {"i", 7},
    {"s", "xx"},
    {"a", {1, 2, 3}},
}

// object
for (auto it = r.begin(); it != r.end(); ++it) {
    LOG << it.key() << ": " << it.value();
}

// array
Json& a = r["a"];
for (auto it = a.begin(); it != a.end(); ++it) {
    LOG << (*it);
}
```




## 性能优化建议

有些用户喜欢用下面的方式添加元素：

```cpp
Json r;
r["a"] = 1;
r["s"] = "hello world";
```

上面的操作虽然可行，但是**效率并不高**。`operator[]` 操作会先查找 key，找到了就更新值，没找到就插入新的元素。一般建议用 **add_member()** 方法取而代之：

```cpp
Json r;
r.add_member("a", 1);
r.add_member("s", "hello world");
```

或者像下面这样构造 Json 对象：

```cpp
Json r = {
    {"a", 1},
    {"s", "hello world"},
};
```

对于只读操作，建议用 [get()](#jsonget) 取代 `operator[]`，前者无副作用。

```cpp
Json r = {{"a", 1}};
r.get("a").as_int(); // 1
```
