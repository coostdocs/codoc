---
weight: 8
title: "JSON"
---

include: [co/json.h](https://github.com/idealvin/co/blob/master/include/co/json.h).




`co/json` is a JSON library similar to [rapidjson](https://github.com/Tencent/rapidjson). Compared with rapidjson, it has certain performance advantages and is easier to use. 


In the latest version, a JSON is built into a contiguous memory, which greatly reduces memory allocations. The parsing speed has also been significantly improved, which can be twice faster as rapidjson. Since a JSON object is a contiguous piece of memory, it is easy to copy, so reference counting used in earlier versions is removed. 




## Basic concepts


### JSON data format


[JSON](https://www.json.org/json-en.html) is a simple data format that supports two data structures:


- A collection consisting of a series of **key/value** pairs. This type of structure is called **object**, which corresponds to struct, map, etc, in programming languages.
- A list composed of a series of **value**, this kind of structure is called **array**, which corresponds to vector, list, etc, in programming languages.



In the above, the key is a string, and the value is generally called **JSON value**, which can be any of object, array, number, string, bool(false, true), or null. number is an integer or floating-point number, and most implementations will distinguish integers from floating-point numbers. 


object is enclosed by a pair of braces, array is enclosed by a pair of square brackets, they look like this:


```cpp
{"a":1, "b":false, "s":"xxx"}
[1, 3.14, true, "xxx"]
```


By definition, object and array can be nested, which can represent complex data structures such as trees. 




### json::Json


co/json uses the `json::Json` class to represent JSON. For convenience, the following typedef definition is also given:


```cpp
typedef json::Json Json;
```


The Json class has only one data member, a pointer, which points to a contiguous memory. In practical applications, Json is generally of object or array type. 




### json::Value


co/json uses the `json::Value` class to represent JSON value, which can be any of object, array, string, int, double, bool, null. 


The structure of json::Value is as follows:


```cpp
class Value {
    Json* _root;
    uint32 _index;
};
```


It contains a pointer of the Json class, and an index, which indicates its position in the Json memory block. Users cannot create json::Value objects directly, they can only be created by methods in the Json class. 




**json::Value supports almost all operations in the Json class. Therefore, the following documents about the Json class generally also apply to the json::Value class.**



## json::Json (Json)


### Json::Json


```cpp
Json();
Json(Json&& r);
Json(const Json& r);
```


- The first version is the default constructor, which constructs a null object.
- The second version is the move constructor. The content of the parameter r is transferred to the constructed Json object, and r itself is invalid.
- The third version is the copy constructor, which constructs a Json object through memory copy.





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





### Json::operator=


```cpp
Json& operator=(const Json& r);
Json& operator=(Json&& r) noexcept;
```


- The first version performs a memory copy and overwrites the original content with the content of r.
- The second version directly transfers the content of r, and r itself becomes invalid.





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


- Add key-value pairs to Json of object type (null automatically becomes object after calling this method).
- The parameter key is a C string ending in `'\0'`, and the parameter x is the value.
- The first version adds a value of bool type.
- Versions 2-5 add a value of integer type.
- The sixth version adds a value of double type.
- Versions 7-10 add a value of string type.
- **NOTE**: for performance reasons, it is required that the key cannot contain double quotes, and it is best not to contain any escape characters. I have never seen anyone use it in this way. If so, please let me know and I will consider supporting it.



- Example



```cpp
Json r;
r.add_member("a", 1);    // r -> {"a":1}
r.add_member("d", 3.3);  // r -> {"a":1, "d":3.3}
r.add_member("s", "xx"); // r -> {"a":1, "d":3.3, "s":"xx"}
```




### Json::add_null


```cpp
void add_null(const char* key);
```


- Add null to an object.
- Example



```cpp
Json r;
r.add_null("x"); // r -> {"x": null}
```




### Json::add_array


```cpp
Value add_array(const char* key, uint32 cap=0);
```


- Add an empty array to an object.
- The parameter cap specifies the initial capacity of the added array, and the default is 0. If you know in advance that the size of the array is n, you can directly set cap to n, which can reduce unnecessary memory reallocation and consumption.
- This method returns a `json::Value` representing the array added. The user can continue to add elements to the array through the return value.



- Example



```cpp
Json r;
auto a = r.add_array("xxx", 3); // r -> {"xxx": []}
a.push_back(1, 2, 3);           // r -> {"xxx": [1,2,3]}
```




### Json::add_object


```cpp
Value add_object(const char* key, uint32 cap=0);
```


- Add an empty object to an object.
- The parameter cap specifies the initial capacity of the added object, and the default is 0. If you know the size of the object as n in advance, you can directly set the cap to n, which can reduce unnecessary memory reallocation and consumption.
- This method returns a `json::Value` object representing the object added. The user can continue to add elements to the object through the return value.



- Example



```cpp
Json r;
auto o = r.add_object("xxx", 3); // r -> {"xxx": {}}
o.add_member("a", 1);            // r -> {"xxx": {"a":1}}
o.add_member("b", 2);            // r -> {"xxx": {"a":1, "b":2}}
o.add_member("c", 3);            // r -> {"xxx": {"a":1, "b":2, "c":3}}
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


- Add elements to an array (null automatically becomes an array after calling this method).
- The first version adds a value of bool type.
- Versions 2-5 add a value of integer type.
- The sixth version adds a value of double type.
- Versions 7-10 add a value of string type.
- The 11th version accepts any number of parameters, and the parameters must be of the type in version 1-10.



- Example



```cpp
Json r;
r.push_back(1);            // r -> [1]
r.push_back(3.3);          // r -> [1, 3.3]
r.push_back("xx");         // r -> [1, 3.3, "xx"]

Json s;
s.push_back(1, 3.3, "xx"); // s -> [1, 3.3, "xx"]
```




### Json::push_null


```cpp
void push_null();
```


- Add null to an array.



- Example



```cpp
Json r;
r.push_null(); // r -> [null]
```




### Json::push_array


```cpp
Value push_array(uint32 cap=0);
```


- Add an empty array to an array.
- The parameter cap specifies the initial capacity of the added array, and the default is 0. If you know in advance that the size of the array is n, you can directly set cap to n, which can reduce unnecessary memory reallocation and consumption.
- This method returns a `json::Value` object representing the array added. The user can continue to add elements to the array through the return value.
- Example



```cpp
Json r;
auto a = r.push_array(3); // r -> [[]]
a.push_back(1, 2, 3.3);   // r -> [[1, 2, 3.3]]
```




### Json::push_object


```cpp
Value push_object(uint32 cap=0);
```


- Add an empty object to an array.
- The parameter cap specifies the initial capacity of the added object, and the default is 0. If you know the size of the object as n in advance, you can directly set the cap to n, which can reduce unnecessary memory reallocation and consumption.
- This method returns a `json::Value` object representing the object added. The user can continue to add elements to the object through the return value.



- Example



```cpp
Json r;
auto o = r.push_object(3); // r -> [{}]
o.add_member("a", 1);      // r -> [{"a":1}]
o.add_member("b", 2);      // r -> [{"a":1, "b":2}]
o.add_member("c", 3);      // r -> [{"a":1, "b":2, "c":3}]
```




### Json::str


```cpp
fastream& str(fastream& fs) const;
fastring str(uint32 cap=256) const;
```


- Convert Json to a string.
- The first version writes the JSON string into a fastream, and the return value is the same as the parameter fs.
- The second version directly returns a JSON string. The parameter cap is the initial capacity of the string, and the default is 256. Setting it to an appropriate value can reduce memory reallocation.





### Json::pretty


```cpp
fastream& pretty(fastream& fs) const;
fastring pretty(uint32 cap=256) const;
```


- Like the str(), but convert Json to a more beautiful JSON string.





### Json::dbg


```cpp
fastream& dbg(fastream& fs) const;
fastring dbg(uint32 cap=256) const;
```


- Convert Json to a debug string, like `Json::str()`, but will truncate string type to the first 32 bytes if its length exceeds 512 bytes.
- This method is generally used to print logs. In some application scenarios, the Json object may contain a long string, such as the base64 encoding of a picture file. At this time, use `Json::dbg()` instead of `Json::str()` to avoid printing too many significant logs.





### Json::parse_from


```cpp
bool parse_from(const char* s, size_t n);
bool parse_from(const char* s);
bool parse_from(const fastring& s);
bool parse_from(const std::string& s);
```


- Parse Json from a JSON string.
- In the first version, s is not required to end with `'\0'`.
- When the parsing is successful, it returns true, otherwise it returns false.
- When the parsing fails, the internal state of Json is uncertain. If the user does not check the return value, and use the Json object directly, it may cause undefined behavior.
- It is **recommended to use** `json::parse()` instead of this method, which always returns a null when parsing failed.





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





### Code example


```cpp
Json r;
auto a = r.add_array("a");
a.push_back(1, 2, 3);

fastring s = r.str();          // s -> "{\"a\":[1,2,3]}"
fastring p = r.pretty();
LOG << r.dbg();                // print json debug string
LOG << r;                      // the same as above, but may be more efficient

Json x;
x.parse_from(s);
x.parse_from(p);

s = "{\"a\":[1,false,3.14]}";
Json v = json::parse(s);       // v -> {"a":[1,false,3.14]}
```




### ———————————




### Json::is_null


```cpp
bool is_null() const;
```


- Determine whether the Json is null.





### Json::is_bool


```cpp
bool is_bool() const;
```


- Determine whether the Json is of type bool.





### Json::is_int


```cpp
bool is_int() const;
```


- Determine whether the Json is an integer type.





### Json::is_double


```cpp
bool is_double() const;
```


- Determine whether the Json is of type double.





### Json::is_string


```cpp
bool is_string() const;
```


- Determine whether the Json is a string type.





### Json::is_array


```cpp
bool is_array() const;
```


- Determine whether the Json is an array type.





### Json::is_object


```cpp
bool is_object() const;
```


- Determine whether the Json is an object type.





### ———————————




### Json::get_bool


```cpp
bool get_bool() const;
```


- Get value of bool type. Return false if the Json calling this method is not bool type.





### Json::get_int


```cpp
int get_int() const;
int32 get_int32() const;
int64 get_int64() const;
uint32 get_uint32() const;
uint64 get_uint64() const;
```


- Get value of integer type. Return 0 if the Json calling these methods is not integer type.





### Json::get_double


```cpp
double get_double() const;
```


- Get value of double type. Return 0.0 if the Json object calling this method is not double type.





### Json::get_string


```cpp
const char* get_string() const;
```


- Get value of string type. Return empty string if the Json calling this method is not string type.
- This method returns the C string ending with `'\0'`, and the user can also call the `string_size()` method to get the length of the string.





### Code example


```cpp
Json r;
r.add_member("a", 1);
r.add_member("d", 3.3);
r.add_member("s", "xx");

r["a"].get_int();    // 1
r["d"].get_double(); // 3.3
r["s"].get_string(); // "xx"
```




### ———————————




### Json::set_null


```cpp
void set_null();
```


- Set the Json to null.





### Json::set_array


```cpp
void set_array();
```


- Set the Json to an empty array.





### Json::set_object


```cpp
void set_object();
```


- Set the Json to an empty object.





### Code example


```cpp
Json r;
r.set_array();  // r -> []
r.push_back(1); // r -> [1]
r.set_object(); // r -> {}
r.set_null();   // r -> null
```




### ———————————




### Json::operator[]


```cpp
Value operator[](int i) const;
Value operator[](uint32 i) const;
Value operator[](const char* key) const
```


- Overload `operator[]`, get the elements in the Json, and return a `json::Value`.
- The first two versions are applicable to array types, the parameter i is the offset of the element in the array, and i must be within the range of the array.
- The third version is applicable to object types. If the element corresponding to the key does not exist, null is returned.



- Example



```cpp
Json r;
r.add_member("x", 1);
auto a = r.add_array("a");
a.push_back(1, 2, 3);  // r -> {"x":1, "a":[1,2,3]}

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


- Determine whether there is an element corresponding to key in Json.
- The Json calling this method must be object or null.



- Example



```cpp
Json r;
r.add_member("a", 1);
r.has_member("a"); // true
r.has_member("x"); // false
```




### Json::size


```cpp
uint32 size() const;
```


- If Json is object or array, this method returns the number of elements.
- If Json is a string type, this method returns the length of the string.
- For all other types, this method returns 0.



- Example



```cpp
Json r;
r.add_member("x", 1);
auto a = r.add_array("a");
a.push_back(1, 2, 3); // r -> {"x":1, "a":[1,2,3]}

r.size();       // 2
r["x"].size();  // 0
r["a"].size();  // 3
```




### Json::string_size


```cpp
uint32 string_size() const;
```


- Returns the length of the string type. Return 0 if the Json calling this method is not string type.





### Json::array_size


```cpp
uint32 array_size() const;
```


- Returns the number of elements of array type. Return 0 if the Json calling this method is not array type.





### Json::object_size


```cpp
uint32 object_size() const;
```


- Returns the number of elements of object type. Return 0 if the Json calling this method is not object type.





### ———————————




### Json::begin


```cpp
iterator begin() const;
```


- Returns the beginning iterator.
- The Json calling this method must be array, object or null.
- When the Json is empty, the return value is equal to `Json::end()`.





### Json::end


```cpp
const iterator::End& end() const;
```


- Returns a fake end iterator.
- The return value is actually not an iterator object, but a iterator can be compared with it. If iterator is equal to `Json::end()`, it means that there is no more element.





### Json::iterator


#### iterator::operator==


```cpp
bool operator==(const End&) const;
```


- Determine whether a iterator is equal to `End`, End is the fake end iterator.





#### iterator::operator!=


```cpp
bool operator!=(const End&) const;
```


- Determine if a iterator is not equal to `End`, End is the fake end iterator.





#### iterator::operator++


```cpp
iterator& operator++();
```


- The prefix operator++.





#### iterator::operator*


```cpp
Value operator*() const;
```


- Overload `operator*`, this method **only applies to iterator of array type**.
- When Json is an array, the iterator points to the elements in the array. This method returns a json::Value that represents the element pointed to by the iterator.





#### iterator::key


```cpp
const char* key();
```


- This method **only applies to iterator of object type**.
- When Json is an object, the iterator points to the key-value pair in the object, and this method returns the key.





#### iterator::value


```cpp
Value value();
```


- This method **only applies to iterator of object type**.
- When Json is an object, the iterator points to the key-value pair in the object, and this method returns the value.





### Traversing the Json


co/json supports traversing a Json of type array or object by iterator:


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


The constructor of `json::Value` is private, and users cannot create json::Value objects directly. 


json::Value is generally used as an intermediate temporary object, **it supports all methods in the Json class except parse_from()** and **safe_clear()**, and its usage is exactly the same, so I won't repeat it here. In addition, it also supports some assignment operations that the Json does not support. 




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


- Assignment, set value of json::Value to the value specified by the parameter x, and the type of json::Value will also become the corresponding type.



- Example



```cpp
Json r;
r.add_member("x", 1);

auto a = r["x"];
a = 3;          // r -> {"x": 3}
a = false;      // r -> {"x": false}
a = "nice";     // r -> {"x": "nice"}
r["x"] = 3.3;   // r -> {"x": 3.3}
```




## Performance optimization suggestions


Some users may add members in the following way:


```cpp
Json r;
r["a"] = 1;
r["s"] = "hello world";
```


Although the above code works, the efficiency is not very high. The `operator[]` will first look up the key which may be slow. It is generally recommended to use **add_member()** instead:


```cpp
Json r;
r.add_member("a", 1);
r.add_member("s", "hello world");
```


The object type is actually stored in the form of an array, and `operator[]` will compare with the keys in the array one by one. In occasions where performance is particularly important, it is recommended to cache the result of operator[] instead of calling operator[] multiple times for the same key.


```cpp
Json r;
r.add_member("a", 1);

auto a = r["a"];
if (a.is_int()) LOG << a.get_int();
```
