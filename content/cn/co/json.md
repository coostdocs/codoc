---
weight: 8
title: "JSON"
---

## 头文件

```cpp
#include "co/json.h"
```

API 在 `json` 命名空间。

## json::any

`json::any` 表示任意 JSON 值，类型可以是 null、bool、int、double、string、array 或 object。

### 构造与拷贝

```cpp
any();                    // null
any(decltype(nullptr));   // null
any(any&& v);             // 移动
any(any& v);              // 移动（非 const）

any(const any&) = delete;
void operator=(const any&) = delete;

any& operator=(any&& v);  // 移动
any& operator=(any& v);   // 移动

any dup() const;          // 显式深拷贝

any(bool v);
any(double v);
any(int64 v);
any(int32 v);
any(uint32 v);
any(uint64 v);
any(const void* p, size_t n); // 初始化字符串类型
any(const char* s);
any(const co::string& s);
any(const std::string& s);
any(std::initializer_list<any> v);
```

- 禁用拷贝构造、拷贝赋值。
- 支持移动，移动后源对象变成 null。
- `dup()` 递归深拷贝。
- 整数统一存 `int64`。

示例:

```cpp
json::any a;               // null
json::any b(nullptr);      // null
json::any c(false);        // bool
json::any d(3.14);         // double
json::any e(23);           // integer
json::any f("xx");         // string
json::any g = {1, 2, 3};   // [1, 2, 3]
json::any h = {"a", "b"};  // ["a", "b"]
json::any i = {            // { "a": "b" }
    {"a", "b"}
}; 
json::any j = {            // {"a": 1, "b": [1,2,3]}
    {"a", 1},
    {"b", {1, 2, 3}},
};

json::any x(h);            // h -> null
json::any y(std::move(i)); // i -> null
y = j;                     // j -> null
```


### 类型查询与取值

```cpp
int  type()      const;
bool is_null()   const;
bool is_bool()   const;
bool is_int()    const;
bool is_double() const;
bool is_string() const;
bool is_array()  const;
bool is_object() const;

bool        as_bool()   const;
int64       as_int64()  const;
int32       as_int32()  const;
int         as_int()    const;
double      as_double() const;
const char* as_c_str()  const;
co::string  as_string() const;
```

若 JSON 不是相应类型，`as_xxx()` 会尝试类型转换：

- `as_bool()`：非 0 为 true，字符串 `"true"` 或 `"1"` 为 true，其余 false。
- `as_int64()`：字符串用 [co::stoi64](../string/#字符串转数值)。
- `as_double()`：字符串用 [co::stod](../string/#字符串转数值)。
- `as_c_str()`：非 string 返回 `""`。
- `as_string()`：null 返回 `""`，非 string 返回 `str()`。


### 访问与修改

```cpp
any& get() const;
any& get(uint32 i) const;
any& get(int i) const;
any& get(const char* key) const;

template<typename T, typename ...X>
any& get(T&& v, X&& ... x) const;

template<typename T>
any& set(T&& v);

template<typename A, typename B, typename ...X>
any& set(A&& a, B&& b, X&& ... x);

any& operator[](uint32 i) noexcept;
any& operator[](int i) noexcept;
const any& operator[](uint32 i) const noexcept;
const any& operator[](int i) const noexcept;
any& operator[](const char* key) noexcept;
const any& operator[](const char* key) const noexcept;
```

- `get` 只读，越界或 key 不存在返回内部 null 对象的引用。
- `set` 不存在时创建，最后一个参数是值，其余参数是索引或 key。
- `operator[]`:
  - const 版本只读，等价于 `get(i)` / `get(key)`;
  - 非 const 版本不存在时创建对应元素或成员，并返回可写引用；若当前类型不匹配，会先重置为 array 或 object。

示例:

```cpp
json::any r = {
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
json::any x;
x.set("a", 1);
x.set("b", json::any({0,1,2}));
x.set("c", "d", 0, "oo");
```


### 添加与删除

```cpp
any& add_member(const char* key, any&& v);
any& add_member(const char* key, any& v);
any& push_back(any&& v);
any& push_back(any& v);

void remove(uint32 i); // 末尾移到 i，O(1)，打乱顺序
void remove(int i);
void remove(const char* key);
void erase(uint32 i);  // 前移，O(n)，保持顺序
void erase(int i);
void erase(const char* key);
```

- `add_member` 添加 key-value 到 object 中(非 object 先重置为 object)，允许重复 key。
- `push_back` 添加元素到 array 末尾(非 array 先重置为 array)。
- 参数 `v` 采用 move 语义；调用后 `v` 变为 null。
- `remove`、`erase` 删除 array 或 object 中元素。
- `remove` 是 `O(1)`，可能打乱元素顺序；`erase` 是 `O(n)`，移动元素保持顺序。

示例:

```cpp
json::any r;
r.add_member("i", 1);    // r -> {"i":1}
r.add_member("d", 3.3);  // r -> {"i":1, "d":3.3}
r.add_member("s", "xx"); // r -> {"i":1, "d":3.3, "s":"xx"}

json::any x;
x.add_member("xx", r);                             // r -> null
r.add_member("o", json::any().add_member("x", 3)); // r -> {"o":{"x":3}}

json::any c;
c.push_back(1).push_back(2);  // c -> [1,2]

json::any d;
d.push_back(c);  // c -> null, d -> [[1, 2]]
```


### 序列化

```cpp
co::string str(int mdp=16)    const;  // 紧凑
co::string dbg(int mdp=16)    const;  // 截断长字符串（>512 字节）
co::string pretty(int mdp=16) const;  // 缩进 4 空格
```

- `mdp`：浮点数最大有效小数位数，默认 16。

示例:

```cpp
#include "co/json.h"
#include "co/print.h"

int main() {
    json::any x;
    x.add_member("name", "coost");
    x.add_member("version", 4);
    co::println("str:    ", x.str());
    co::println("pretty:\n", x.pretty());
    return 0;
}
```


### 大小

```cpp
uint32 size() const;
bool   empty() const;
uint32 array_size() const;
uint32 object_size() const;
uint32 string_size() const;
```

- `size()`：array 元素数，object 键值对数，string 长度，其它 0。
- `empty()` 等价于 `size() == 0`。
- `array_size()` / `object_size()` / `string_size()` 仅对应类型返回长度，其它 0。


### has_member

```cpp
bool has_member(const char* key) const;
```

- 判断 `key` 是否存在，非 object 类型返回 false。


### iterator

```cpp
struct iterator {
    bool operator!=(_End) const;
    bool operator==(_End) const;
    iterator& operator++();
    const char* key() const;
    any& value() const;
    any& operator*() const;
};

iterator begin() const;
const iterator::_End end() const;
```

- array：`operator*` 返回元素。
- object：`key()` 返回键，`value()` 返回值。
- 不支持后缀 `++`。

{{< hint warning >}}
object 类型的 iterator，不允许使用 `operator*`。
{{< /hint >}}

示例:

```cpp
#include "co/json.h"
#include "co/print.h"

int main() {
    // 用 iterator 遍历 object
    json::any x;
    x.add_member("a", 1);
    x.add_member("b", 2);
    for (auto it = x.begin(); it != x.end(); ++it) {
        co::println(it.key(), " = ", it.value().as_int());
    }

    // 用 iterator 遍历 array
    json::any y;
    y.push_back(1).push_back(2);
    for (auto it = y.bengin(); it != y.end(); ++it) {
        co::println(*it);
    }

    // 用索引遍历 array
    for (uint32 i = 0; i < y.array_size(); ++i) {
        co::println(y[i]);
    }

    return 0;
}
```


## 解析

```cpp
json::any json::parse(const char* s, size_t n);
json::any json::parse(const char* s);
json::any json::parse(const co::string& s);
json::any json::parse(const std::string& s);
```

- 将字符串解析成 JSON 对象，失败返回 null。

示例:

```cpp
#include "co/json.h"
#include "co/print.h"

int main() {
    const char* s = R"({"name":"coost","version":4,"tags":["cpp","coroutine"]})";
    json::any x = json::parse(s);

    co::println("name:    ", x["name"].as_c_str());
    co::println("version: ", x["version"].as_int());

    auto& tags = x.get("tags");
    for (uint32 i = 0; i < tags.array_size(); ++i) {
        co::println("tag[", i, "] = ", tags[i].as_c_str());
    }
    return 0;
}
```


## 快速创建 array 或 object

```cpp
json::any json::array();  // 返回空 array
json::any json::object(); // 返回空 object
json::any json::array(std::initializer_list<any> v);
json::any json::object(std::initializer_list<any> v);
```

示例:

```cpp
json::any a = json::array({1,2,3});
json::any o = json::object({
    {"a", 1},
    {"s", "hello world"},
});
```


## 终端或日志中打印 JSON

coost 重载了如下操作符，可在 `co::print` 或日志中直接打印 `json::any`。

```cpp
co::string& operator<<(co::string& s, const json::any& x) noexcept;
```

示例:

```cpp
json::any o = json::object({
    {"a", 1},
    {"s", "hello world"},
});

co::println("o: ", o);
log::info("o: ", o);
```


## 性能优化建议

有些用户喜欢用如下方式添加元素：

```cpp
json::any r;
r["a"] = 1;
r["s"] = "hello world";
```

可行，但**效率不高**。`operator[]` 先查找 key，找到就更新值，没找到插入新元素。建议用 **add_member()** 方法取代:

```cpp
json::any r;
r.add_member("a", 1);
r.add_member("s", "hello world");
```

或者像下面这样构造:

```cpp
json::any r = {
    {"a", 1},
    {"s", "hello world"},
};
```
