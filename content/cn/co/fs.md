---
weight: 22
title: "文件系统"
---


include: [co/fs.h](https://github.com/idealvin/co/blob/master/include/co/fs.h).


`co/fs.h` 最小限度的实现了常用的文件系统操作，不同平台路径分隔符建议统一使用 `'/'`。




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





### fs::remove
```cpp
bool remove(const char* path, bool rf=false);
bool remove(const fastring& path, bool rf=false);
bool remove(const std::string& path, bool rf=false);
```

- 删除文件或目录，参数 path 是路径。
- path 是目录时，参数 rf 表示是否强制删除，默认为 false，仅删除空目录。若 rf 为 true，则相当于 `rm -rf`，非空目录也会被删除。
- path 是文件时，参数 rf 会被忽略。





### fs::rename
```cpp
bool rename(const char* from, const char* to);
bool rename(const fastring& from, const fastring& to);
bool rename(const std::string& from, const std::string& to);
```

- 重命名文件或目录，参数 from 是原路径，参数 to 是新路径。
- 参数 to 是目录时，windows 要求 to 和 from 在同一个盘符(drive) 下面。
- 一般建议在参数 to 指定的路径不存在时使用，详情可以参考 [win32/MoveFile](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-movefilea), [linux/rename](https://www.man7.org/linux/man-pages/man2/rename.2.html)。





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
fs::remove("a/b", true);    // rm -rf a/b     

fs::rename("a/b", "a/c");   // mv a/b a/c
fs::symlink("/usr", "x");   // ln -s /usr x
```






## 读写文件(fs::file)


`fs::file` 类实现了文件的基本读写操作，与 `fread` 与 `fwrite` 不同，它内部**没有缓存**，直接读写文件。




### file::file
```cpp
file();
file(file&& f)；
file(const char* path, char mode);
file(const fastring& path, char mode);
file(const std::string& path, char mode);
```

- 第 1 个版本是默认构造函数，创建一个空的 file 对象，不会打开任何文件。
- 第 2 个版本是 move 构造函数，支持将 file 对象放到 STL 容器中。
- 第 3-5 个版本，打开指定的文件，参数 path 是文件路径，参数 mode 是打开模式。
- mode 是 `'r'`, `'w'`, `'a'`, `'m'` 中的一种，r 是只读模式，w 是写模式，a 是追加模式，m 是修改模式，与 w 类似，但不会清空已存在文件的数据。
- mode 为 `'r'` 时，文件必须存在，否则打开失败。
- mode 为 `'w'` 时，文件不存在时自动创建，文件已存在时清空文件数据。
- mode 为 `'a'` 时，文件不存在时自动创建，文件已存在时不清空文件数据。
- mode 为 `'m'` 时，文件不存在时自动创建，文件已存在时不清空文件数据。





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
const fastring& path() const;
```

- 此方法返回 file 内部文件路径的引用。
- 若 file 对象并未关联任何文件，则返回值会引用一个空字符串。





### file::read
```cpp
size_t read(void* buf, size_t n);
fastring read(size_t n);
```

- 第 1 个版本读取数据到指定的 buffer 中，n 是要读取的字节数，返回实际读取的字节数。
- 第 2 个版本与第 1 个版本类似，但以 fastring 的形式返回读取的数据，n 是要读取的字节数。
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
size_t write(const void* s, size_t n);
size_t write(const char* s);
size_t write(const fastring& s);
size_t write(const std::string& s);
size_t write(char s);
```

- 第 1 个版本写入指定长度的数据。
- 第 2-4 个版本写入字符串。
- 第 5 个版本写入单个字符。
- 此方法返回实际写入的字节数，在磁盘空间不足或发生其他错误时，返回值可能小于 n。
- 此方法内部已经处理了 `EINTR` 错误，用户无需担心。





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
f.close();                // close the file
```






## 文件流(fs::fstream)


`fs::file` 不支持缓存，写小文件性能较差，为此，`co/fs.h` 另外实现了支持缓存的 `fs::fstream` 类，fs::fstream 只用于写文件，不支持读操作。




### fstream::fstream
```cpp
fstream();
fstream(fstream&& fs);
explicit fstream(size_t cap);
fstream(const char* path, char mode, size_t cap=8192);
fstream(const fastring& path, char mode, size_t cap=8192);
fstream(const std::string& path, char mode, size_t cap=8192);
```

- 第 1 个版本是默认构造函数，内部缓存大小为 8k。
- 第 2 个版本是 move 构造函数，可以将 fstream 对象放到 STL 容器中。
- 第 3 个版本用参数 cap 指定缓存的大小。
- 第 4-6 个版本打开指定的文件，path 是文件路径，mode 是模式，cap 是缓存大小，默认为 8k。
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
fstream& operator<<(const char* s);
fstream& operator<<(const fastring& s);
fstream& operator<<(const std::string& s);
fstream& operator<<(const fastream& s);
template<typename T> fstream& operator<<(T v);
```

- 第 1-3 个版本中，参数 s 是字符串类型。
- 第 4 个版本中，参数 s 是 fastream 类型。
- 第 5 个版本中，T 可以是任意内置类型，如 bool, char, int, double 等。





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


