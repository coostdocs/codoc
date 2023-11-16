---
weight: 12
title: "文件系统"
---

include: [co/fs.h](https://github.com/idealvin/coost/blob/master/include/co/fs.h).


**fs** 模块最小限度的实现了常用的文件系统操作，不同平台路径分隔符建议统一使用 `/`。



## 元数据操作


### fs::exists

```cpp
bool exists(const char* path);
bool exists(const fastring& path);
bool exists(const std::string& path);
```

- 判断文件是否存在，参数 path 是文件或目录路径。



### fs::fsize

```cpp
int64 fsize(const char* path);
int64 fsize(const fastring& path);
int64 fsize(const std::string& path);
```

- 获取文件大小，文件不存在或其他错误返回 -1。



### fs::isdir

```cpp
bool isdir(const char* path);
bool isdir(const fastring& path);
bool isdir(const std::string& path);
```

- 判断文件是否是目录，若 path 存在且是目录，则返回 true，否则返回 false。



### fs::mtime

```cpp
int64 mtime(const char* path);
int64 mtime(const fastring& path);
int64 mtime(const std::string& path);
```

- 获取文件的修改时间，文件不存在时返回 -1。



### fs::mkdir

```cpp
bool mkdir(const char* path, bool p=false);
bool mkdir(const fastring& path, bool p=false);
bool mkdir(const std::string& path, bool p=false);
```

- 创建目录，参数 path 是目录路径，参数 p 表示是否创建整个路径。
- 参数 p 默认为 false，仅当父目录存在时，才会创建目录；参数 p 为 true 时，相当于 `mkdir -p` ，父目录不存在时，先创建父目录。



### fs::mv

```cpp
bool mv(const char* from, const char* to);
bool mv(const fastring& from, const fastring& to);
bool mv(const std::string& from, const std::string& to);
```

- v3.0.2 新增，移动或重命名文件、目录，行为与 linux 系统中 `mv` 命令类似。
- `to` 存在且为目录时，将 `from` 移动到目录 `to` 下面。
- 若目标与 `from` 类型相同(都是目录或文件)，且目标不是非空目录，则目标会被覆盖掉。

```cpp
// 假设目录 d 已存在
fs::mv("xx.txt", "xx.log");  // xx.txt -> xx.log
fs::mv("xx.txt", "d");       // xx.txt -> d/xx.txt
```



### fs::remove

```cpp
bool remove(const char* path, bool r=false);
bool remove(const fastring& path, bool r=false);
bool remove(const std::string& path, bool r=false);
```

- 删除文件或目录，参数 path 是路径。
- 参数 r 默认为 false，仅删除文件或空目录；r 为 true 时，相当于 `rm -r`，可删除非空目录。



### fs::rename

```cpp
bool rename(const char* from, const char* to);
bool rename(const fastring& from, const fastring& to);
bool rename(const std::string& from, const std::string& to);
```

- v3.0.2 中标记为 deprecated，可以使用 [fs::mv](#fsmv) 取代之。



### fs::symlink

```cpp
bool symlink(const char* dst, const char* lnk);
bool symlink(const fastring& dst, const fastring& lnk);
bool symlink(const std::string& dst, const std::string& lnk);
```

- 创建软链接，参数 dst 是目标文件或目录的路径，参数 lnk 是软链接的路径。
- 此函数先调用 `fs::remove(lnk)` 删除旧的软链接，再创建新的软链接文件。
- 在 windows 平台，此函数需要 admin 权限。



### 代码示例

```cpp
bool x = fs::exists(path);  // 判断文件是否存在
bool x = fs::isdir(path);   // 判断文件是否为目录
int64 x = fs::mtime(path);  // 获取文件的修改时间
int64 x = fs::fsize(path);  // 获取文件的大小

fs::mkdir("a/b");           // mkdir a/b
fs::mkdir("a/b", true);     // mkdir -p a/b

fs::remove("x/x.txt");      // rm x/x.txt
fs::remove("a/b");          // rmdir a/b
fs::remove("a/b", true);    // rm -r a/b     

fs::rename("a/b", "a/c");   // mv a/b a/c
fs::symlink("/usr", "x");   // ln -s /usr x
```




## fs::file

**fs::file** 类实现了文件的基本读写操作，与 `fread` 与 `fwrite` 不同，它内部**没有缓存**，直接读写文件。


### file::file

```cpp
1. file();
2. file(file&& f)；
3. file(const char* path, char mode);
4. file(const fastring& path, char mode);
5. file(const std::string& path, char mode);
```

- 1, 默认构造函数，创建一个空的 file 对象，不会打开任何文件。
- 2, 移动构造函数，支持将 file 对象放到 STL 容器中。
- 3-5, 打开指定的文件，参数 path 是文件路径，参数 mode 是打开模式。
- mode 是 `'r'`, `'w'`, `'a'`, `'m'` 或 `'+'` 中的一种，r 是只读模式，w 是写模式，a 是追加模式，m 是修改模式，+ 是读写模式。
- mode 为 `'r'` 时，文件必须存在，否则打开失败。
- mode 为 `'w'` 时，文件不存在时自动创建，文件已存在时清空文件数据。
- mode 为 `'a'`, `'m'` 或 `'+'` 时，文件不存在时自动创建，文件已存在时不清空文件数据。
- `'+'` 是 v3.0 新增，此模式下，读与写共享文件指针，因此在读、写操作前，一般需要调用 [seek()](#fileseek) 方法设置偏移位置。



### file::~file

```cpp
~file();
```

- 析构函数，关闭之前打开的文件，释放相关资源。



### file::close

```cpp
void close();
```

- 关闭文件，析构函数中会自动调用此方法。
- 多次调用此方法是安全的。



### file::exists

```cpp
bool exists() const;
```

- 判断文件是否存在。
- 文件可能被其他进程删除，调用此方法可以判断之前打开的文件，是否仍然存在。



### file::open

```cpp
bool open(const char* path, char mode);
bool open(const fastring& path, char mode);
bool open(const std::string& path, char mode);
```

- 此方法打开指定的文件，path 是文件路径，mode 是打开模式，见[构造函数](#filefile)中的说明。
- 此方法在打开文件前，会先关闭之前打开的文件。



### file::operator bool

```cpp
explicit operator bool() const;
```

- 将 fs::file 转换为 bool 类型，文件成功打开时返回 true，否则返回 false。



### file::operator!

```cpp
bool operator!() const;
```

- 文件未打开或打开失败时返回 true，否则返回 false。



### file::path

```cpp
const char* path() const;
```

- 返回文件路径。
- 若 file 对象并未关联任何文件，则返回空字符串 `""`。



### file::read

```cpp
1. size_t read(void* buf, size_t n);
2. fastring read(size_t n);
```

- 1, 读取数据到指定的 buffer 中，n 是要读取的字节数，返回实际读取的字节数。
- 2, 与 1 类似，但以 fastring 的形式返回读取的数据，n 是要读取的字节数。
- 此方法在遇到文件尾或发生错误时，实际读取的字节数可能小于 n。



### file::seek

```cpp
void seek(int64 off, int whence=seek_beg);
```

- 设置文件指针的当前位置，参数 off 是偏移位置，参数 whence 是起始位置，可以是 `file::seek_beg`, `file::seek_cur`, `file::seek_end` 中的一种。
- 此方法对以 `'a'` (append) 模式打开的文件无效。



### file::size

```cpp
int64 size()  const;
```

- 此方法返回文件的大小，文件未打开或打开失败时，调用此方法会返回 -1。



### file::write

```cpp
1. size_t write(const void* s, size_t n);
2. size_t write(const char* s);
3. size_t write(const fastring& s);
4. size_t write(const std::string& s);
5. size_t write(char c);
```

- 1, 写入指定长度的数据。
- 2-4, 写入字符串。
- 5, 写入单个字符。
- 此方法返回实际写入的字节数，在磁盘空间不足或发生其他错误时，返回值可能小于 n。
- 此方法内部已经处理了 `EINTR` 错误，用户无需额外处理。



### 代码示例

```cpp
fs::file f;               // empty file
fs::file f("xx", 'w');    // write mode
f.open("xx", 'm');        // reopen with modify mode

f.open("xx", 'r');        // read mode
if (f) f.read(buf, 512);  // read at most 512 bytes
fastring s = f.read(32);  // read at most 32 bytes and return fastring

f.open("xx", 'a');        // append mode
if(f) f.write(buf, 32);   // write 32 bytes
f.write("hello");         // write a C string
f.write('c');             // write a single character

f.open("xx", '+');        // read/write mode
f.seek(0);                // seek to beginning before write
f.write("hello");
f.seek(0);                // seek to beginning before read 
f.read(buf, 8);
f.close();                // close the file
```




## fs::fstream

[fs::file](#fsfile) 不支持缓存，写小文件性能较差，为此，coost 另外实现了支持缓存的 `fs::fstream` 类，fs::fstream 只用于写文件，不支持读操作。



### fstream::fstream

```cpp
1. fstream();
2. fstream(fstream&& fs);
3. explicit fstream(size_t cap);
4. fstream(const char* path, char mode, size_t cap=8192);
5. fstream(const fastring& path, char mode, size_t cap=8192);
6. fstream(const std::string& path, char mode, size_t cap=8192);
```

- 1, 默认构造函数，内部缓存大小为 8k。
- 2, 移动构造函数，可以将 fstream 对象放到 STL 容器中。
- 3, 用参数 cap 指定缓存的大小。
- 4-6, 打开指定的文件，path 是文件路径，mode 是模式，cap 是缓存大小，默认为 8k。
- 参数 mode 是 `'w'` 或 `'a'` 中的一种，不支持读模式。
- mode 为 `'w'` 时，文件不存在时自动创建，文件已存在时清空文件数据。
- mode 为 `'a'` 时，文件不存在时自动创建，文件已存在时不会清空文件数据，在文件尾追加写。



### fstream::~fstream

```cpp
~fstream();
```

- 析构函数，关闭打开的文件，释放相关的资源。



### fstream::append

```cpp
fstream& append(const void* s, size_t n);
```

- 追加数据，参数 n 是数据的长度。



### fstream::close

```cpp
void close();
```

- 关闭文件，析构函数中会自动调用此方法。
- 多次调用此方法是安全的。



### fstream::flush

```cpp
void flush();
```

- 将缓存中的数据写入文件。



### fstream::open

```cpp
bool open(const char* path, char mode);
bool open(const fastring& path, char mode);
bool open(const std::string& path, char mode);
```

- 打开指定的文件，参数 path 是文件路径，参数 mode 是打开模式，见[构造函数](#fstreamfstream)中的说明。
- 此方法在打开文件前，会关闭之前打开的文件。



### fstream::operator bool

```cpp
explicit operator bool() const;
```

- 将 fs::fstream 转换为 bool 类型，文件成功打开时返回 true，否则返回 false。



### fstream::operator!

```cpp
bool operator!() const;
```

- 文件未打开或打开失败时返回 true，否则返回 false。



### fstream::operator<<

```cpp
1. fstream& operator<<(const char* s);
2. fstream& operator<<(const fastring& s);
3. fstream& operator<<(const std::string& s);
4. fstream& operator<<(const fastream& s);
5. template<typename T> fstream& operator<<(T v);
```

- 1-3, 参数 s 是字符串类型。
- 4, 参数 s 是 fastream 类型。
- 5, 参数 T 可以是任意内置类型，如 bool, char, int, double 等。



### fstream::reserve

```cpp
void reserve(size_t n);
```

- 调整缓存容量，参数 n 是容量大小。若 n 小于之前的容量，则缓存容量保持不变。



### 代码示例

```cpp
fs::fstream s;                     // cache size: 8k
fs::fstream s(4096);               // cache size: 4k
fs::fstream s("path", 'a');        // append mode, cache size: 8k
fs::fstream s("path", 'w', 4096);  // write mode, cache size: 4k
s.reserve(8192);                   // make cache size at least 8k

s.open("path", 'a');               // open with append mode
if (s) s << "hello world" << 23;   // operator<<
s.append("hello", 5);              // append
s.flush();                         // flush data in cache to file
s.close();                         // close the file
```




## fs::dir

v3.0.1 中新增 **fs::dir** 类，用于读目录。


### dir::dir

```cpp
1. dir();
2. dir(dir&& d);
3. explicit dir(const char* path);
4. explicit dir(const fastring& path);
5. explicit dir(const std::string& path);
```

- 1, 默认构造函数，创建空的 dir 对象，不打开任何目录。
- 2, 移动构造函数。
- 3-5, 打开 path 指定的目录。



### dir::~dir

```cpp
~dir();
```

- 析构函数，关闭打开的目录，释放相关资源。



### dir::all

```cpp
co::vector<fastring> all() const;
```

- 读取目录下所有子项，`.` 与 `..` 会被忽略。



### dir::begin

```cpp
iterator begin() const;
```

- 返回 begin iterator。



### dir::close

```cpp
void close();
```

- 关闭目录，析构函数中会自动调用此方法。
- 多次调用此方法是安全的。



### dir::end

```cpp
iterator end() const;
```

- 返回 end iterator。



### dir::iterator

`dir::iterator` 类用于遍历 dir 对象中打开的目录。


#### operator*

```cpp
fastring operator*() const;
```

- 返回 iterator 对应的文件或目录名。


#### operator++

```cpp
iterator& operator++();
```

- 重载前缀 `++` 操作。


#### operator==

```cpp
bool operator==(const iterator& it) const;
```

- 判断两个 iterator 是否相等。


#### operator!=

```cpp
bool operator!=(const iterator& it) const;
```

- 判断两个 iterator 是否不相等。



### dir::open

```cpp
bool open(const char* path);
bool open(const fastring& path);
bool open(const std::string& path);
```

- 打开 path 指定的目录，打开目录前会先关闭之前打开的目录。



### dir::path

```cpp
const char* path() const;
```

- 返回目录的路径，若 dir 对象未关联任何目录，则返回空字符串 `""`。



### 代码示例

```cpp
fs::dir d("xx");
auto v = d.all(); // 读取所有子项

d.open("abc");
for (auto it = d.begin(); it != d.end(); ++it) {
    cout << *it << endl;
}
```
