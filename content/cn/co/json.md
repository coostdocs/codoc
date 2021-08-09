---
weight: 8
title: "JSON"
---

include: [co/json.h](https://github.com/idealvin/co/blob/master/include/co/json.h).


`co/json` 是一个类似 [rapidjson](https://github.com/Tencent/rapidjson) 的 json 库，与 rapidjson 相比，它既有一定的性能优势，同时又更简单易用。


最新版本的实现中，将 json 对象构建到一块连续的内存上，大大减少了内存分配操作，json 的 parsing 速度也有了显著提升，实测可以超过 [rapidjson](https://github.com/Tencent/rapidjson) 的两倍。由于一个 json 对象就是一块连续的内存，拷贝起来很容易，因此移除了早期版本中使用的引用计数。






## 基本概念




### JSON 数据格式


[JSON](https://www.json.org/json-en.html) 是一种简单的数据格式，它支持两种数据结构：

- 由一系列 **key/value** 键值对构成的集合，这类结构称为 **object**，对应编程语言中的 struct, map 等等。
- 由一系列 **value** 构成的列表，这类结构称为 **array**，对应编程语言中的 vector, list 等等。



上述 key 是 string，value 一般也称为 **JSON value**，可以是 object, array, number, string, bool(false, true), null 中的任意一种。number 是整数或浮点数，大部分实现会将整数与浮点数区分开。


object 由一对大括号括起来，array 由一对中括号括起来，它们看起来像下面这样：
```cpp
{"a":1, "b":false, "s":"xxx"}
[1, 3.14, true, "xxx"]
```
由定义 object 与 array 可以嵌套，从而可以表示树等复杂数据结构。




### json::Json


co/json 用 `json::Json` 类表示 JSON，为了方便，还给出了下面的 typedef 定义：
```cpp
typedef json::Json Json;
```
在 co/json 的实现中，Json 类的数据成员仅有一个指针，它指向一块连续的内存。实际应用中，Json 一般是 object 或 array 类型。




### json::Value


co/json 用 `json::Value` 类表示 JSON value，它可以是 object, array, string, int, double, bool, null 中的任意一种。


json::Value 的结构如下：
```cpp
class Value {
    Json* _root;
    uint32 _index;
};
```
它包含一个 Json 类的指针，以及一个 index，表示它在 Json 内存块中的位置。用户无法直接创建 json::Value 对象，只能由 Json 类中的一些方法创建。


**json::Value 几乎支持 Json 类中的所有操作，因此，下面关于 Json 类的文档，一般也适用于 json::Value 类。**






## json::Json (Json)




### Json::Json
```cpp
Json();
Json(Json&& r);
Json(const Json& r);
```

- 第 1 个版本是默认构造函数，构建一个 null 对象。
- 第 2 个版本是 move 构造函数，参数 r 的内容转移到所构建的 Json 对象中，r 自身失效。
- 第 3 个版本是拷贝构造函数，通过内存拷贝构建一个 Json 对象。





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
Json& operator=(const Json& r);
Json& operator=(Json&& r) noexcept;
```

- 第 1 个版本进行内存拷贝，用 r 的内容覆盖原来的内容。
- 第 2 个版本直接转移 r 的内容，r 自身失效。





### ———————————
### Json::add_member
```cpp
void add_member(const char* key, bool x);
void add_member(const char* key, int64 x);
void add_member(const char* key, int x);
void add_member(const char* key, uint32 x);
void add_member(const char* key, uint64 x);
void add_member(const char* key, double x);
void add_member(const char* key, const char* x, size_t n);
void add_member(const char* key, const char* x);
void add_member(const char* key, const std::string& x);
void add_member(const char* key, const fastring& x);
```

- 向 object 类型的 Json 中添加 key-value 键值对 (null 调用此方法后自动变成 object)。
- 参数 key 是 `'\0'` 结尾的 C 字符串，参数 x 是所添加的值。
- 第 1 个版本添加 bool 类型的值。
- 第 2-5 个版本添加整数类型的值。
- 第 6 个版本添加 double 类型的值。
- 第 7-10 个版本添加 string 类型的值。
- **温馨提示：** co/json 出于性能上的考虑，要求 key 中不能包含双引号，最好不要包含任何转义字符。我从未见有人这样用过，如果有的话，请告诉我，我再考虑支持一下^o^。



- 示例
```cpp
Json r;
r.add_member("a", 1);     // r -> {"a":1}
r.add_member("d", 3.3);   // r -> {"a":1, "d":3.3}
r.add_member("s", "xx");  // r -> {"a":1, "d":3.3, "s":"xx"}
```




### Json::add_null
```cpp
void add_null(const char* key);
```

- 向 object 类型的 Json 中添加 null。



- 示例
```cpp
Json r;
r.add_null("x");  // r -> {"x": null}
```




### Json::add_array
```cpp
Value add_array(const char* key, uint32 cap=0);
```

- 向 object 类型的 Json 中添加 array。
- 参数 cap 指定所添加 array 的初始容量，默认为 0。如果预先知道 array 的大小(size) 为 n，则可以直接将 cap 设为 n，这样可以减少不必要的内存重分配及消耗。
- 此方法向 Json 中添加一个空的 array，并返回表示该 array 的 `json::Value` 对象，用户可以通过返回值继续往该 array 里面添加元素。



- 示例
```cpp
Json r;
auto a = r.add_array("xxx", 3);  // r -> {"xxx": []}
a.push_back(1, 2, 3);            // r -> {"xxx": [1,2,3]}
```




### Json::add_object
```cpp
Value add_object(const char* key, uint32 cap=0);
```

- 向 object 类型的 Json 中添加 object。
- 参数 cap 指定所添加 object 的初始容量，默认为 0。如果预先知道 object 的大小(size) 为 n，则可以直接将 cap 设为 n，这样可以减少不必要的内存重分配及消耗。
- 此方法向 Json 中添加一个空的 object，并返回表示该 object 的 `json::Value` 对象，用户可以通过返回值继续往该 object 里面添加元素。



- 示例
```cpp
Json r;
auto o = r.add_object("xxx", 3);  // r -> {"xxx": {}}
o.add_member("a", 1);             // r -> {"xxx": {"a":1}}
o.add_member("b", 2);             // r -> {"xxx": {"a":1, "b":2}}
o.add_member("c", 3);             // r -> {"xxx": {"a":1, "b":2, "c":3}}
```




### Json::push_back
```cpp
void push_back(bool x);
void push_back(int64 x);
void push_back(int x);
void push_back(uint32 x);
void push_back(uint64 x);
void push_back(double x);
void push_back(const char* x, size_t n);
void push_back(const char* x);
void push_back(const std::string& x);
void push_back(const fastring& x);
template <typename X, typename ...V> void push_back(X x, V... v);
```

- 向 array 类型的 Json 中添加元素(null 调用此方法后自动变成 array)。
- 第 1 个版本添加 bool 类型的值。
- 第 2-5 个版本添加整数类型的值。
- 第 6 个版本添加 double 类型的值。
- 第 7-10 个版本添加 string 类型的值。
- 第 11 个版本接受任意数量的参数，参数必须是版本 1-10 中的类型。




- 示例
```cpp
Json r;
r.push_back(1);             // r -> [1]
r.push_back(3.3);           // r -> [1, 3.3]
r.push_back("xx");          // r -> [1, 3.3, "xx"]

Json s;
s.push_back(1, 3.3, "xx");  // s -> [1, 3.3, "xx"]
```




### Json::push_null
```cpp
void push_null();
```

- 向 array 类型的 Json 中添加 null 对象。



- 示例
```cpp
Json r;
r.push_null();  // r -> [null]
```




### Json::push_array
```cpp
Value push_array(uint32 cap=0);
```

- 向 array 类型的 Json 中添加 array。
- 参数 cap 指定所添加 array 的初始容量，默认为 0。如果预先知道 array 的大小(size) 为 n，则可以直接将 cap 设为 n，这样可以减少不必要的内存重分配及消耗。
- 此方法向 Json 中添加一个空的 array，并返回表示该 array 的 `json::Value` 对象，用户可以通过返回值继续往该 array 里面添加元素。



- 示例
```cpp
Json r;
auto a = r.push_array(3);  // r -> [[]]
a.push_back(1, 2, 3.3);    // r -> [[1, 2, 3.3]]
```




### Json::push_object
```cpp
Value push_object(uint32 cap=0);
```

- 向 array 类型的 Json 中添加 object。
- 参数 cap 指定所添加 object 的初始容量，默认为 0。如果预先知道 object 的大小(size) 为 n，则可以直接将 cap 设为 n，这样可以减少不必要的内存重分配及消耗。
- 此方法向 Json 中添加一个空的 object，并返回表示该 object 的 `json::Value` 对象，用户可以通过返回值继续往该 object 里面添加元素。



- 示例
```cpp
Json r;
auto o = r.push_object(3);  // r -> [{}]
o.add_member("a", 1);       // r -> [{"a":1}]
o.add_member("b", 2);       // r -> [{"a":1, "b":2}]
o.add_member("c", 3);       // r -> [{"a":1, "b":2, "c":3}]
```


### ———————————
### Json::str
```cpp
fastream& str(fastream& fs) const;
fastring str(uint32 cap=256) const;
```

- 将 Json 对象转换成字符串。
- 第 1 个版本将 JSON 字符串写入 fastream 中，返回值与参数 fs 相同。
- 第 2 个版本直接返回 JSON 字符串，参数 cap 是字符串初始容量，默认为 256，设置为合适的值可以减少内存重分配。





### Json::pretty
```cpp
fastream& pretty(fastream& fs) const;
fastring pretty(uint32 cap=256) const;
```

- 将 Json 对象转换成更漂亮的 JSON 字符串，除了结果好看点，其他与 `Json::str()` 一样。





### Json::dbg
```cpp
fastream& dbg(fastream& fs) const;
fastring dbg(uint32 cap=256) const;
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
- 解析失败时，Json 内部状态是不确定的，如果用户没有判断返回值，就直接使用 Json 对象，可能导致未定义的行为。
- 一般**推荐使用**更安全的 `json::parse()` 函数，它在解析失败时始终返回 null 对象。





### json::parse
```cpp
Json parse(const char* s, size_t n);
Json parse(const char* s);
Json parse(const fastring& s);
Json parse(const std::string& s);
```

- 从 JSON 字符串解析 Json 对象。
- 此函数不是 Json 类中的方法，而是定义于 `namespace json` 下的函数。
- 此函数返回一个 Json 对象，解析失败时，返回 null。





### 代码示例
```cpp
Json r;
auto a = r.add_array("a");
a.push_back(1, 2, 3);

fastring s = r.str();     // s -> {"a":[1,2,3]}
fastring p = r.pretty(); 
LOG << r.dbg();           // print json debug string
LOG << r;                 // the same as above, may be more efficient

Json x;
x.parse_from(s);
x.parse_from(p);

s = "{\"a\":[1,false,3.14]}";
Json v = json::parse(s);  // v -> {"a":[1,false,3.14]}
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
### Json::get_bool
```cpp
bool get_bool() const;
```

- 获取 bool 类型的值，若调用此方法的 Json 对象不是 bool 类型，返回值为 false。





### Json::get_int
```cpp
int get_int() const;
int32 get_int32() const;
int64 get_int64() const;
uint32 get_uint32() const;
uint64 get_uint64() const;
```

- 获取整数类型的值，若调用这些方法的 Json 对象不是整数类型，返回值为 0。





### Json::get_double
```cpp
double get_double() const;
```

- 获取 double 类型的值，若调用此方法的 Json 对象不是 double 类型，返回值是 0.0。





### Json::get_string
```cpp
const char* get_string() const;
```

- 获取字符串类型的值，若调用此方法的 Json 对象不是字符串类型，返回值是空字符串。
- 此方法返回 `'\0'` 结尾的 C 字符串，用户还可以调用 `string_size()` 方法获取字符串的长度。





### 代码示例
```cpp
Json r;
r.add_member("a", 1);
r.add_member("d", 3.3);
r.add_member("s", "xx");

r["a"].get_int();     // return 1
r["d"].get_double();  // return 3.3
r["s"].get_string();  // return "xx"
```




### ———————————
### Json::set_null
```cpp
void set_null();
```

- 将 Json 对象设置为 null。





### Json::set_array
```cpp
void set_array();
```

- 将 Json 对象设置为一个空的 array。





### Json::set_object
```cpp
void set_object();
```

- 将 Json 对象设置为一个空的 object。





### 代码示例
```cpp
Json r;
r.set_array();   // r -> []
r.push_back(1);  // r -> [1]
r.set_object();  // r -> {}
r.set_null();    // r -> null
```




### ———————————
### Json::operator[]
```cpp
Value operator[](int i) const;
Value operator[](uint32 i) const;
Value operator[](const char* key) const
```

- 重载 `operator[]`，获取 Json 对象中的元素，返回一个 `json::Value` 对象。
- 前两个版本适用于 array 类型，参数 i 是元素在 array 中的偏移位置，i 必须在 array 大小范围内。
- 第 3 个版本适用于 object 类型，若 key 对应的元素不存在，返回 null。



- 示例
```cpp
Json r;
r.add_member("x", 1);
auto a = r.add_array("a");
a.push_back(1, 2, 3);       // r -> {"x":1, "a":[1,2,3]}

auto x = r["x"];
x.get_int();      // 1

a = r["a"];
a.array_size();   // 3
a[0].get_int();   // 1
a[1].get_int();   // 2
```




### Json::has_member
```cpp
bool has_member(const char* key) const;
```

- 判断 Json 对象中是否存在 key 对应的元素。
- 调用此方法的 Json 对象必须是 object 或 null。



- 示例
```cpp
Json r;
r.add_member("a", 1);
r.has_member("a");  // true
r.has_member("x");  // false
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
Json r;
r.add_member("x", 1);
auto a = r.add_array("a");
a.push_back(1, 2, 3);       // r -> {"x":1, "a":[1,2,3]}

r.size();       // 2
r["x"].size();  // 0
r["a"].size();  // 3
```




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
### Json::begin
```cpp
iterator begin() const;
```

- 返回指向 Json 对象的 beginning iterator。
- 调用此方法的 Json 对象必须是 array, object 或 null。
- 当 Json 对象为空时，此方法返回值等于 `Json::end()`。





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
Value operator*() const;
```

- 重载 `operator*`，此方法**仅适用于 array 类型的 iterator**。
- Json 为 array 时，iterator 指向 array 中的元素。此方法返回一个 json::Value 对象，表示 iterator 指向的元素。





#### iterator::key
```cpp
const char* key();
```

- 此方法**仅适用于 object 类型的 iterator**。
- Json 为 object 时，iterator 指向 object 中的 key-value 键值对，此方法返回该键值对中的 key。





#### iterator::value
```cpp
Value value();
```

- 此方法**仅适用于 object 类型的 iterator**。
- Json 为 object 时，iterator 指向 object 中的 key-value 键值对，此方法返回该键值对中的 value，即一个 json::Value 对象。



### 遍历 array 或 object


co/json 支持用 iterator 遍历 array 或 object 类型的 Json 对象。
```cpp
// {"i":7, "s":"xx", "a":[123, true, "nice"]}
Json r;
r.add_member("i", 7);
r.add_member("s", "xx");
auto a = r.add_array("a");
a.push_back(123, true, "nice");

// object
for (auto it = r.begin(); it != r.end(); ++it) {
    LOG << it.key() << ": " << it.value();
}

// array
a = r["a"];
for (auto it = a.begin(); it != a.end(); ++it) {
    LOG << (*it);
}
```






## json::Value


`json::Value` 的构造函数是私有的，用户不能直接创建 json::Value 对象。


json::Value 一般用作中间临时对象，**它支持 Json 类中除 parse_from() 外的所有方法**，其用法也完全一样，因此这里就不再赘述了。另外，它还支持一些 Json 类不支持的 `operator=` 赋值操作，用于对 Json 类中的元素赋值。




### json::Value::operator=
```cpp
void operator=(bool x);
void operator=(int64 x);
void operator=(int32 x);
void operator=(uint32 x);
void operator=(uint64 x);
void operator=(double x);
void operator=(const char* x);
void operator=(const fastring& x);
void operator=(const std::string& x);
```

- 赋值操作，将 json::Value 的值设置为参数 x 指定的值，json::Value 的类型也会变成相应的类型。
- 第 1 个版本用 bool 类型的值赋值。
- 第 2-5 个版本用整数类型的值赋值。
- 第 6 个版本用 double 类型的值赋值。
- 第 7-9 个版本用字符串类型赋值。



- 示例
```cpp
Json r;
r.add_member("x", 1);

auto a = r["x"];
a = 3;         // r -> {"x": 3}
a = false;     // r -> {"x": false}
a = "nice";    // r -> {"x": "nice"}
r["x"] = 3.3;  // r -> {"x": 3.3}
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


object 类型内部实际上是以数组的形式存储的，`operator[]` 会与数组中的 key 一一比较，可能较慢。在特别注重性能的场合，建议将 operator[] 的结果缓存起来，不要对同一个 key 多次调用 operator[] 操作。
```cpp
Json r;
r.add_member("a", 1);

auto a = r["a"];
if (a.is_int()) LOG << a.get_int();
```


