---
weight: 8
title: "JSON"
---

## Header

```cpp
#include "co/json.h"
```

The API is in the `json` namespace.

## json::any

`json::any` represents any JSON value. Its type can be null, bool, int, double, string, array, or object.

### Construction and Copying

```cpp
any();                    // null
any(decltype(nullptr));   // null
any(any&& v);             // move
any(any& v);              // move (non-const)

any(const any&) = delete;
void operator=(const any&) = delete;

any& operator=(any&& v);  // move
any& operator=(any& v);   // move

any dup() const;          // explicit deep copy

any(bool v);
any(double v);
any(int64 v);
any(int32 v);
any(uint32 v);
any(uint64 v);
any(const void* p, size_t n); // initialize as string type
any(const char* s);
any(const co::string& s);
any(const std::string& s);
any(std::initializer_list<any> v);
```

- Copy construction and copy assignment are disabled.
- Move is supported; after moving, the source object becomes null.
- `dup()` recursively performs a deep copy.
- Integers are uniformly stored as `int64`.

Example:

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

### Type Query and Value Retrieval

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

If the JSON is not of the corresponding type, `as_xxx()` will attempt type conversion:

- `as_bool()`: non-zero is true; the strings `"true"` or `"1"` are true; everything else is false.
- `as_int64()`: strings use [co::stoi64](../string/#string-to-number).
- `as_double()`: strings use [co::stod](../string/#string-to-number).
- `as_c_str()`: returns `""` for non-string types.
- `as_string()`: returns `""` for null, and returns `str()` for non-string types.

### Access and Modification

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

- `get` is read-only; if out of bounds or the key does not exist, it returns a reference to an internal null object.
- `set` creates the value if it does not exist; the last parameter is the value, and the remaining parameters are indices or keys.
- `operator[]`:
  - The const version is read-only and is equivalent to `get(i)` / `get(key)`;
  - The non-const version creates the value if it does not exist; if the current type does not match, it first resets it to an array or object.

Example:

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

### Addition and Deletion

```cpp
any& add_member(const char* key, any&& v);
any& add_member(const char* key, any& v);
any& push_back(any&& v);
any& push_back(any& v);

void remove(uint32 i); // move the last element to i, O(1), reorders
void remove(int i);
void remove(const char* key);
void erase(uint32 i);  // shift forward, O(n), preserves order
void erase(int i);
void erase(const char* key);
```

- `add_member` adds a key-value pair to an object (if it is not an object, it is first reset to an object); duplicate keys are allowed.
- `push_back` adds an element to the end of an array (if it is not an array, it is first reset to an array).
- The parameter `v` uses move semantics; after the call, `v` becomes null.
- `remove` and `erase` delete elements from an array or object.
- `remove` is `O(1)` and may reorder elements; `erase` is `O(n)` and moves elements to preserve order.

Example:

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

### Serialization

```cpp
co::string str(int mdp=16)    const;  // compact
co::string dbg(int mdp=16)    const;  // truncates long strings (>512 bytes)
co::string pretty(int mdp=16) const;  // indents by 4 spaces
```

- `mdp`: maximum number of significant decimal places for floating-point numbers; default is 16.

Example:

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

### Size

```cpp
uint32 size() const;
bool   empty() const;
uint32 array_size() const;
uint32 object_size() const;
uint32 string_size() const;
```

- `size()`: number of array elements, number of object key-value pairs, string length, otherwise 0.
- `empty()` is equivalent to `size() == 0`.
- `array_size()` / `object_size()` / `string_size()` return the length only for the corresponding type, otherwise 0.

### has_member

```cpp
bool has_member(const char* key) const;
```

- Determines whether `key` exists; returns false for non-object types.

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

- array: `operator*` returns the element.
- object: `key()` returns the key, and `value()` returns the value.
- Postfix `++` is not supported.

{{< hint warning >}}
For an object iterator, `operator*` is not allowed.
{{< /hint >}}

Example:

```cpp
#include "co/json.h"
#include "co/print.h"

int main() {
    // Use iterator to traverse object
    json::any x;
    x.add_member("a", 1);
    x.add_member("b", 2);
    for (auto it = x.begin(); it != x.end(); ++it) {
        co::println(it.key(), " = ", it.value().as_int());
    }

    // Use iterator to traverse array
    json::any y;
    y.push_back(1).push_back(2);
    for (auto it = y.bengin(); it != y.end(); ++it) {
        co::println(*it);
    }

    // Use index to traverse array
    for (uint32 i = 0; i < y.array_size(); ++i) {
        co::println(y[i]);
    }

    return 0;
}
```

## Parsing

```cpp
json::any json::parse(const char* s, size_t n);
json::any json::parse(const char* s);
json::any json::parse(const co::string& s);
json::any json::parse(const std::string& s);
```

- Parses a string into a JSON object; returns null on failure.

Example:

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

## Quickly Create array or object

```cpp
json::any json::array();  // returns an empty array
json::any json::object(); // returns an empty object
json::any json::array(std::initializer_list<any> v);
json::any json::object(std::initializer_list<any> v);
```

Example:

```cpp
json::any a = json::array({1,2,3});
json::any o = json::object({
    {"a", 1},
    {"s", "hello world"},
});
```

## Print JSON to Terminal or Logs

coost overloads the following operator, so `json::any` can be printed directly in `co::print` or logs.

```cpp
co::string& operator<<(co::string& s, const json::any& x) noexcept;
```

Example:

```cpp
json::any o = json::object({
    {"a", 1},
    {"s", "hello world"},
});

co::println("o: ", o);
log::info("o: ", o);
```

## Performance Optimization Suggestions

Some users like to add elements in the following way:

```cpp
json::any r;
r["a"] = 1;
r["s"] = "hello world";
```

This works, but it is **not efficient**. `operator[]` first searches for the key; if found, it updates the value; if not found, it inserts a new element. It is recommended to use the **add_member()** method instead:

```cpp
json::any r;
r.add_member("a", 1);
r.add_member("s", "hello world");
```

Or construct it like this:

```cpp
json::any r = {
    {"a", 1},
    {"s", "hello world"},
};
```
