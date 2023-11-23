---
weight: 8
title: "JSON"
---

include: [co/json.h](https://github.com/idealvin/coost/blob/master/include/co/json.h).


**co.json** is a JSON library similar to [rapidjson](https://github.com/Tencent/rapidjson). Compared with rapidjson, it has better performance and is easier to use. 


## Basic concepts

[JSON](https://www.json.org/json-en.html) is a simple data format that supports two data structures:

- A collection consisting of a series of **key/value** pairs. This type of structure is called **object**, which corresponds to struct, map, etc, in programming languages.
- A list composed of a series of **value**, this kind of structure is called **array**, which corresponds to vector, list, etc, in programming languages.

In the above, the key is a string, and the value is generally called **JSON value**, which can be any of object, array, number, string, bool(false, true), or null. number is an integer or a floating-point number, and most implementations will distinguish integers from floating-point numbers. 

Object is enclosed by a pair of braces, array is enclosed by a pair of square brackets, they look like this:

```cpp
{"a":1, "b":false, "s":"xxx"}
[1, 2, 3]
```

By definition, object and array can be nested, which can represent complex data structures such as trees. 




## global

### json::array

```cpp
Json array();
```

- This function is in namespace json, it returns an empty array.



### json::object

```cpp
Json object();
```

- This function is in namespace json, it returns an empty object.



### json::parse

```cpp
Json parse(const char* s, size_t n);
Json parse(const char* s);
Json parse(const fastring& s);
Json parse(const std::string& s);
```

- Parse Json from a JSON string.
- This function is not a method in the Json class, but a function defined in `namespace json`.
- This function returns a Json object, when the parsing failed, it returns null.




## co::Json

### constructor

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

- 1-2, construct a null object.
- 3-4, move constructor and copy constructor, **both implement a move semantic**, the parameter `v` will become a null object after the construction.
- 5, construct a JSON object of bool type.
- 6, construct a JSON object of double type.
- 7-10, construct a JSON object of integer type.
- 11-14, construct a JSON object of string type.
- 15, construct a JSON object of object or array type from a initialization list.


- Example

```cpp
co::Json a;          // null
co::Json b(nullptr); // null
co::Json c = false;  // bool
co::Json d = 3.14;   // double
co::Json e = 23;     // integer
co::Json f = "xx";   // string

co::Json g = {1, 2, 3};  // g -> [1, 2, 3]
co::Json h = {"a", "b"}; // h -> ["a", "b"]

co::Json i = {           // i -> { "a": "b" }
    {"a", "b"}
};

co::Json j = {           // j -> {"a": 1, "b": [1,2,3]}
    {"a", 1},
    {"b", {1, 2, 3}},
};

co::Json x(i);            // i -> null
co::Json y(std::move(j)); // j -> null
```



### operator=

```cpp
Json& operator=(Json&& v);
Json& operator=(Json& v);
void operator=(const Json&) = delete;
```

- Assignment, the 2 methods above are equal, `v` is moved to the calling Json object, and **becomes a null object after the operation**.



### dup

```cpp
Json dup() const;
```

- Return a deep copy of a JSON object.


- 示例

```cpp
co::Json x = {1, 2, 3}; // x -> [1,2,3]
co::Json y, z;
y = x;       // x -> null, y -> [1,2,3]
z = y.dup(); // y:[1,2,3], z -> [1,2,3]
```




### ———————————
### is_null

```cpp
bool is_null() const;
```

- Determine whether the Json is null.



### is_bool

```cpp
bool is_bool() const;
```

- Determine whether the Json is bool type.



### is_int

```cpp
bool is_int() const;
```

- Determine whether the Json is integer type.



### is_double

```cpp
bool is_double() const;
```

- Determine whether the Json is double type.



### is_string

```cpp
bool is_string() const;
```

- Determine whether the Json is string type.



### is_array

```cpp
bool is_array() const;
```

- Determine whether the Json is array type.



### is_object

```cpp
bool is_object() const;
```

- Determine whether the Json is object type.




### ———————————
### as_bool

```cpp
bool as_bool() const;
```

- Get value of bool type.
- For int or double types, returns false if the value is 0, otherwise returns true.
- For string type, returns true if the value is `"true"` or `"1"`, otherwise returns false.
- For other non-bool types, return false.



### as_int

```cpp
int as_int() const;
int32 as_int32() const;
int64 as_int64() const;
```

- Get value of integer type.
- For bool, double or string types, the result is automatically converted to an integer.
- For other non-integer types, 0 is returned.



### as_double

```cpp
double as_double() const;
```

- Get value of double type. Return 0 if the Json object calling this method is not double type.
- Get value of double type.
- For bool, int or string types, the result is automatically converted to double type.
- For other non-double types, 0 is returned.



### as_string

```cpp
fastring as_string() const;
```

- Get value of string type, return fastring.
- For non-string types, this method is equal to [str()](#str), and the result will be automatically converted to string type.



### as_c_str

```cpp
const char* as_c_str() const;
```

- Returns a null-terminated C-style string, [string_size()](#string_size) can be called to get its length.
- For non-string types, return an empty string.



### get

```cpp
1. Json& get(uint32 i) const;
2. Json& get(int i) const;
3. Json& get(const char* key) const;

4. template <class T,  class ...X>
   inline Json& get(T&& v, X&& ... x) const;
```

- Get JSON object according to index or key. **This method is a read-only operation and will not modify the JSON object that calls this method**.
- 1-2, get the i-th element of the array object. If the JSON object that calls this method is not of type array, or `i` exceeds the range of the array, the returned result will refer to a null object.
- 3, get the JSON value corresponding to `key`, if the JSON object called this method is not of type object, or `key` does not exist, the returned result will refer to a null object.
- 4, can take any number of parameters, each parameter is an index or a key. When it encounters the first invalid index or non-existing key, it returns immediately, and the return result will refer to a null object.



### set

```cpp
template <class T>
inline Json& set(T&& v) {
    return *this = Json(std::forward<T>(v));
}

template <class A, class B,  class ...X>
inline Json& set(A&& a, B&& b, X&& ... x);
```

- Set the value of the JSON object.
- The last parameter of `set` is the value to be set, other parameters are index or key.



### Example

```cpp
co::Json r = {
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
co::Json x;
x.set("a", 1);
x.set("b", co::Json({0,1,2}));
x.set("c", "d", 0, "oo");
```




### ———————————
### operator==

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

- Check if the value of the Json object is equal to `v`.
- If the type of the Json object is different from `v`, return false directly.



### operator!=

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

- Check if the value of the Json object is not equal to `v`.
- If the type of the Json object is different from `v`, return true directly.



### Example

```cpp
co::Json x = {
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
### add_member

```cpp
Json& add_member(const char* key, Json&& v);
Json& add_member(const char* key, Json& v);
```

- Add a key-value pair to a Json of object type (non-object Json automatically becomes object after calling this method).
- This method will reserve the order in which keys were added, and keys may appear repeatedly.
- The parameter key is a C string ending in `'\0'`, and the parameter v is the value.
- **`v` is moved and becomes null after this operation**.
- **NOTE**: for performance reasons, it is required that the key cannot contain double quotes.


- Example

```cpp
co::Json r;
r.add_member("a", 1);    // r -> {"a":1}
r.add_member("d", 3.3);  // r -> {"a":1, "d":3.3}
r.add_member("s", "xx"); // r -> {"a":1, "d":3.3, "s":"xx"}

co::Json x;
x.add_member("xx", r);                            // r -> null
r.add_member("o", co::Json().add_member("x", 3)); // r -> {"o":{"x":3}}
co::Json().add_member("o", 1).add_member("k", 2); // -> {"o":1,"k":2}
```



### erase

```cpp
void erase(uint32 i);
void erase(int i);
void erase(const char* key);
```

- The first two, erase the ith element from an array.
- The third, erase the element by `key` from an object.



### push_back

```cpp
Json& push_back(Json&& v);
Json& push_back(Json& v);
```

- Add elements to an array (non-array Json automatically becomes an array after calling this method).
- **`v` is moved and becomes null after this operation**.


- Example

```cpp
co::Json r;
r.push_back(1);    // r -> [1]
r.push_back(3.3);  // r -> [1, 3.3]
r.push_back("xx"); // r -> [1, 3.3, "xx"]

co::Json x;
x.push_back(r);  // r -> null, x -> [[1, 3.3, "xx"]]
r.push_back(co::Json().push_back(1).push_back(2)); // r -> [[1,2]]
```



### remove

```cpp
void remove(uint32 i);
void remove(int i);
void remove(const char* key);
```

- The first two, remove the ith element from an array.
- The third, remove the element by `key` from an object.
- The last element will be moved to the position where the element was removed.




### reset

````cpp
void reset();
````

- Reset the Json object to null.



### swap

````cpp
void swap(Json& v) noexcept;
void swap(Json&& v) noexcept;
````

- Swap the contents of two Json objects.




### ———————————
### operator[]

```cpp
Json& operator[](uint32 i) const;
Json& operator[](int i) const;
Json& operator[](const char* key) const;
```

- Overload `operator[]`, get the elements in the Json by index or key.
- 1-2, for array type, get the i-th element of the array object, i must be within the size range of the array.
- 3, for object type, when the key does not exist, a null object will be inserted into the Json.
- **In general, it is recommended to replace this operation with the read-only `get()` method whenever possible.**


- Example

```cpp
co::Json r = {
    { "a", 7 },
    { "x", { 1, 2, 3 } },
};

r["a"].as_int();    // 7
r["x"][0].as_int(); // 1
```



### has_member

```cpp
bool has_member(const char* key) const;
```

- Determine whether there is an element corresponding to key in Json.
- Return false if the Json calling this method is not object type.


- Example

```cpp
co::Json r = {{"a", 1}};
r.has_member("a"); // true
r.has_member("x"); // false
```



### size

```cpp
uint32 size() const;
```

- If Json is object or array, this method returns the number of elements.
- If Json is string type, this method returns the length of the string.
- For all other types, this method returns 0.


- Example

```cpp
co::Json r = {
    {"x", 1},
    {"s", "hello"},
    {"a", {1, 2, 3}},
};

r.size();      // 3
r["x"].size(); // 0
r["s"].size(); // 5
r["a"].size(); // 3
```



### empty

```cpp
bool empty() const;
```

- Check whether the Json is empty, which is equal to `size() == 0`.



### string_size

```cpp
uint32 string_size() const;
```

- Return the length of the string type. Return 0 if the Json calling this method is not string type.



### array_size

```cpp
uint32 array_size() const;
```

- Return the number of elements of array type. Return 0 if the Json calling this method is not array type.



### object_size

```cpp
uint32 object_size() const;
```

- Return the number of elements of object type. Return 0 if the Json calling this method is not object type.




### ———————————
### str

```cpp
fastream& str(fastream& s, int mdp=16) const;
fastring& str(fastring& s, int mdp=16) const;
fastring str(int mdp=16) const;
```

- Convert Json to a string.
- The 1st version appends the JSON string to a fastream, and the return value is the same as the parameter s.
- The 2nd version appends the JSON string to a fastring, and the return value is the same as the parameter s.
- The 3rd version returns a JSON string.
- The parameter `mdp` is short for `max decimal places`, which means the maximum number of decimal places for float point numbers.



### pretty

```cpp
fastream& pretty(fastream& s, int mdp=16) const;
fastring& pretty(fastring& s, int mdp=16) const;
fastring pretty(int mdp=16) const;
```

- Like the str(), but convert Json to a more beautiful JSON string.



### dbg

```cpp
fastream& dbg(fastream& s, int mdp=16) const;
fastring& dbg(fastring& s, int mdp=16) const;
fastring dbg(int mdp=16) const;
```

- Convert Json to a debug string, like `str()`, but will truncate string type to the first 32 bytes if its length exceeds 512 bytes.
- This method is generally used to print logs. In some cases, the Json object may contain a long string, such as the base64 encoding of a picture. At such cases, use `dbg()` instead of `str()` to avoid printing too many significant logs.



### parse_from

```cpp
bool parse_from(const char* s, size_t n);
bool parse_from(const char* s);
bool parse_from(const fastring& s);
bool parse_from(const std::string& s);
```

- Parse Json from a JSON string.
- In the first version, s is not required to end with `'\0'`.
- When the parsing is successful, it returns true, otherwise it returns false.
- When the parsing fails, the calling Json becomes null.



### Example

```cpp
co::Json r = {
    { "a", {1,2,3} }
};

fastring s = r.str();    // s -> {"a":[1,2,3]}
fastring p = r.pretty(); 
LOG << r.dbg();          // print json debug string
LOG << r;                // the same as above, but is  more efficient

co::Json x;
x.parse_from(s);
x.parse_from(p);

co::Json v = json::parse(s);
```




### ———————————
### begin

```cpp
iterator begin() const;
```

- Returns the beginning iterator.
- The Json calling this method must be array, object or null.
- When the Json is empty, the return value is equal to `end()`.
- If the Json calling this method is not array or object type, the return value is equal to [end()](#end).



### end

```cpp
const iterator::End& end() const;
```

- Returns a fake end iterator.
- The return value is actually not an iterator object, but a iterator can be compared with it. If an iterator is equal to `end()`, it means that there is no more element.



### iterator

#### operator==

```cpp
bool operator==(const End&) const;
```

- Determine whether a iterator is equal to `End`, End is the fake end iterator.


#### operator!=

```cpp
bool operator!=(const End&) const;
```

- Determine if a iterator is not equal to `End`, End is the fake end iterator.


#### operator++

```cpp
iterator& operator++();
```

- The prefix operator++.


#### operator*

```cpp
Json& operator*() const;
```

- Overload `operator*`, this method **only applies to iterator of array type**.
- When Json is an array, the iterator points to the elements in the array.


#### key

```cpp
const char* key() const;
```

- This method **only applies to iterator of object type**.
- When Json is an object, the iterator points to the key-value pair in the object, and this method returns the key.


#### value

```cpp
Json& value() const;
```

- This method **only applies to iterator of object type**.
- When Json is an object, the iterator points to the key-value pair in the object, and this method returns the value.



### Traversing the Json

co.json supports traversing a Json of type array or object by iterator:

```cpp
// {"i":7, "s":"xx", "a":[123, true, "nice"]}
co::Json r = {
    {"i", 7},
    {"s", "xx"},
    {"a", {1, 2, 3}},
}

// object
for (auto it = r.begin(); it != r.end(); ++it) {
    LOG << it.key() << ": " << it.value();
}

// array
co::Json& a = r["a"];
for (auto it = a.begin(); it != a.end(); ++it) {
    LOG << (*it);
}
```




## Performance tips

Some users may add members in the following way:

```cpp
co::Json r;
r["a"] = 1;
r["s"] = "hello world";
```

Although the above code works, the efficiency may be not so good. The `operator[]` will first look up the key, which may be slow. It is generally recommended to use **add_member()** instead:

```cpp
co::Json r;
r.add_member("a", 1);
r.add_member("s", "hello world");
```

Or construct a Json like this:

```cpp
co::Json r = {
    {"a", 1},
    {"s", "hello world"},
};
```

For read-only operations, it is recommended to replace `operator[]` with [get()](#get), which has no side effects.

```cpp
co::Json r = {{"a", 1}};
r.get("a").as_int(); // 1
```
