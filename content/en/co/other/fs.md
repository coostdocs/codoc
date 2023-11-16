---
weight: 12
title: "File System"
---

include: [co/fs.h](https://github.com/idealvin/coost/blob/master/include/co/fs.h).


The **fs** module implements common file system operations. It is recommended to use `/` uniformly as the path separator on different platforms. 



## Metadata operations


### fs::exists

```cpp
bool exists(const char* path);
bool exists(const fastring& path);
bool exists(const std::string& path);
```

- Check whether the file exists, the parameter `path` is the path of a file or directory.



### fs::fsize

```cpp
int64 fsize(const char* path);
int64 fsize(const fastring& path);
int64 fsize(const std::string& path);
```

- Get the file size. -1 will be returned if the file does not exist.



### fs::isdir

```cpp
bool isdir(const char* path);
bool isdir(const fastring& path);
bool isdir(const std::string& path);
```

- Check whether the file is a directory, if `path` exists and is a directory, it returns true, otherwise it returns false.



### fs::mtime

```cpp
int64 mtime(const char* path);
int64 mtime(const fastring& path);
int64 mtime(const std::string& path);
```

- Get the modification time of the file, return -1 when the file does not exist.



### fs::mkdir

```cpp
bool mkdir(const char* path, bool p=false);
bool mkdir(const fastring& path, bool p=false);
bool mkdir(const std::string& path, bool p=false);
```

- Create a directory, the parameter `p` indicates whether to create the entire path.
- The parameter `p` is false by default, and the directory will be created only when the parent directory exists. When `p` is true, it is equivalent to `mkdir -p`, and the parent directory will be created first if it does not exist.



### fs::mv

```cpp
bool mv(const char* from, const char* to);
bool mv(const fastring& from, const fastring& to);
bool mv(const std::string& from, const std::string& to);
```

- Added in v3.0.2, move or rename a file or directory, the behavior is similar to the `mv` command in Linux.
- When `to` exists and is a directory, `from` will be moved into directory `to`.
- If the target is of the same type as `from` (both are directories or files), and the target is not a non-empty directory, the target will be overwritten.

```cpp
// Assume directory d already exists
fs::mv("xx.txt", "xx.log"); // xx.txt -> xx.log
fs::mv("xx.txt", "d");      // xx.txt -> d/xx.txt
```



### fs::remove

```cpp
bool remove(const char* path, bool r=false);
bool remove(const fastring& path, bool r=false);
bool remove(const std::string& path, bool r=false);
```

- Delete a file or directory.
- The parameter `r` is false by default, which only deletes file or empty directory; when `r` is true, it is equivalent to `rm -r`, which can delete non-empty directory.



### fs::rename

```cpp
bool rename(const char* from, const char* to);
bool rename(const fastring& from, const fastring& to);
bool rename(const std::string& from, const std::string& to);
```

- Deprecated in v3.0.2. Use [fs::mv](#fsmv) instead.



### fs::symlink

```cpp
bool symlink(const char* dst, const char* lnk);
bool symlink(const fastring& dst, const fastring& lnk);
bool symlink(const std::string& dst, const std::string& lnk);
```

- Create a soft link, the parameter `dst` is the path of the target file or directory, and the parameter `lnk` is the path of the soft link.
- This function first calls `fs::remove(lnk)` to delete the old soft link, and then create a new one.
- On windows, this function requires admin permission.



### Code example

```cpp
bool x = fs::exists(path); // Determine whether the file exists
bool x = fs::isdir(path);  // Determine whether the file is a directory
int64 x = fs::mtime(path); // Get the modification time of the file
int64 x = fs::fsize(path); // Get the size of the file

fs::mkdir("a/b");          // mkdir a/b
fs::mkdir("a/b", true);    // mkdir -p a/b

fs::remove("x/x.txt");     // rm x/x.txt
fs::remove("a/b");         // rmdir a/b
fs::remove("a/b", true);   // rm -rf a/b

fs::rename("a/b", "a/c");  // mv a/b a/c
fs::symlink("/usr", "x");  // ln -s /usr x
```




## fs::file

The **fs::file** class implements the basic read and write operations of files. Unlike `fread` and `fwrite`, it has no cache, but reads and writes files directly. 



### file::file

```cpp
1. file();
2. file(file&& f);
3. file(const char* path, char mode);
4. file(const fastring& path, char mode);
5. file(const std::string& path, char mode);
```

- 1, the default constructor, which creates an empty file object without opening a file.
- 2, the move constructor.
- 3-5, open the file specified by `path`, and the parameter `mode` is the open mode.
- `mode` is one of `'r'`, `'w'`, `'a'`, `'m'` or `'+'`, r for read, w for write, a for append, m for modify, + for both read and write.
- When mode is `'r'`, the file must exist, otherwise the file is not opened.
- When mode is `'w'`, a new file will be created if it does not exist, and the file will be truncated if it already exists.
- When mode is `'a'`, `'m'` or `'+'`, a new file will be created when it does not exist, and the file will not be truncated if it already exists.
- `'+'` was added since v3.0. In this mode, read and write share the file pointer, therefore, you may need call the [seek()](#fileseek) method to set the offset before the read or write operation.



### file::~file

```cpp
~file();
```

- Destructor, close the previously opened file and release related resources.



### file::close

```cpp
void close();
```

- Close the file, this method will be automatically called in the destructor.
- It is safe to call this method multiple times.



### file::exists

```cpp
bool exists() const;
```

- Determine whether the file exists.
- The file may be deleted by other processes. This method can be used to determine whether the previously opened file still exists.



### file::open

```cpp
bool open(const char* path, char mode);
bool open(const fastring& path, char mode);
bool open(const std::string& path, char mode);
```


- This method opens the specified file, `mode` is the open mode, see the description in [Constructor](#filefile).
- This method will close the previously opened file before opening a new file.



### file::operator bool

```cpp
explicit operator bool() const;
```

- Convert `fs::file` to bool type, return true when the file is successfully opened, otherwise return false.



### file::operator!

```cpp
bool operator!() const;
```

- It returns true when the file is not opened, otherwise it returns false.



### file::path

```cpp
const char* path() const;
```

- Return path of the file.
- If the file object is not associated with any file, return empty string `""`.



### file::read

```cpp
1. size_t read(void* buf, size_t n);
2. fastring read(size_t n);
```

- 1, reads data into the specified buffer, `n` is the number of bytes to be read, and returns the number of bytes actually read.
- 2, similar to the first one, but returns a fastring, where `n` is the number of bytes to be read.
- The actual bytes read may be less than `n`, when it reaches the end of the file or an error occurs.



### file::seek

```cpp
void seek(int64 off, int whence=seek_beg);
```

- Set the current position of the file pointer, the parameter `off` is the offset position, and the parameter `whence` is the starting position, which can be one of `file::seek_beg`, `file::seek_cur`, `file::seek_end`.
- This method is not valid for files opened with mode `'a'`.



### file::size

```cpp
int64 size() const;
```

- Returns size of the file. When the file is not opened, return -1.



### file::write

```cpp
1. size_t write(const void* s, size_t n);
2. size_t write(const char* s);
3. size_t write(const fastring& s);
4. size_t write(const std::string& s);
5. size_t write(char c);
```

- 1, writes `n` bytes.
- 2-4, write a string.
- 5, writes a single character.
- This method returns the number of bytes actually written. When the disk space is insufficient or other error occurs, the return value may be less than `n`.
- This method has already handled `EINTR` error internally.



### Example

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

[fs::file](#fsfile) does not support caching, and the performance of writing small files is poor. So coost implements the **fs::fstream** class which supports caching, it is designed for writing files and does not support read operations. 



### fstream::fstream

```cpp
1. fstream();
2. fstream(fstream&& fs);
3. explicit fstream(size_t cap);
4. fstream(const char* path, char mode, size_t cap=8192);
5. fstream(const fastring& path, char mode, size_t cap=8192);
6. fstream(const std::string& path, char mode, size_t cap=8192);
```

- 1, the default constructor, and the internal cache size is 8k.
- 2, the move constructor.
- 3, specify the size of the cache to `cap`.
- 4-6, open the specified file, `cap` is the cache size, and the default is 8k.
- The parameter `mode` is one of `'w'` or `'a'`, and the read mode is not supported.
- When mode is `'w'`, the file is automatically created when it does not exist, and the file data is cleared if it already exists.
- When mode is `'a'`, the file will be created automatically if it does not exist, and the file data will not be cleared if it already exists.



### fstream::~fstream

```cpp
~fstream();
```

- Destructor, close the file, release related resources.



### fstream::append

```cpp
fstream& append(const void* s, size_t n);
```

- Append `n` bytes to the file.



### fstream::close

```cpp
void close();
```

- Close the file, this method will be automatically called in the destructor.
- It is safe to call this method multiple times.



### fstream::flush

```cpp
void flush();
```

- Write data in the cache to the file.



### fstream::open

```cpp
bool open(const char* path, char mode);
bool open(const fastring& path, char mode);
bool open(const std::string& path, char mode);
```

- Open the specified file.
- This method will close the previously opened file before opening a new file.



### fstream::operator bool

```cpp
explicit operator bool() const;
```

- Convert `fs::fstream` to bool type, return true when the file is successfully opened, otherwise return false.



### fstream::operator!

```cpp
bool operator!() const;
```

- It returns true when the file is not opened, otherwise it returns false.



### fstream::operator<<

```cpp
1. fstream& operator<<(const char* s);
2. fstream& operator<<(const fastring& s);
3. fstream& operator<<(const std::string& s);
4. fstream& operator<<(const fastream& s);
5. template<typename T> fstream& operator<<(T v);
```

- 1-3, the parameter `s` is a string type.
- 4, the parameter `s` is a fastream.
- 5, `T` can be any built-in type, such as bool, char, int, double, etc.



### fstream::reserve

```cpp
void reserve(size_t n);
```

- Adjust the cache capacity, the parameter `n` is the capacity. If `n` is less than the previous capacity, the capacity remains unchanged.



### Code example

```cpp
fs::fstream s;                   // cache size: 8k
fs::fstream s(4096);             // cache size: 4k
fs::fstream s("path",'a');       // append mode, cache size: 8k
fs::fstream s("path",'w', 4096); // write mode, cache size: 4k
s.reserve(8192);                 // make cache size at least 8k

s.open("path",'a');              // open with append mode
if (s) s << "hello world" << 23; // operator<<
s.append("hello", 5);            // append
s.flush();                       // flush data in cache to file
s.close();                       // close the file
```




## fs::dir

The **fs::dir** class is added in v3.0.1, which is used to read directories.


### dir::dir

```cpp
1. dir();
2. dir(dir&& d);
3. explicit dir(const char* path);
4. explicit dir(const fasting& path);
5. explicit dir(const std::string& path);
```

- 1, the default constructor, creates an empty dir object and does not open any directory.
- 2, the move constructor.
- 3-5, open the directory specified by `path`.



### dir::~dir

```cpp
~dir();
```

- Destructor, close the opened directory and release related resources.



### dir::all

```cpp
co::vector<fastring> all() const;
```

- Read all items in the directory, `.` and `..` will be ignored.



### dir::begin

```cpp
iterator begin() const;
```

- Returns the begin iterator.



### dir::close

```cpp
void close();
```

- Close the directory, it is automatically called in the destructor.
- It is safe to call this method multiple times.



### dir::end

```cpp
iterator end() const;
```

- Returns the end iterator.



### dir::iterator

The `dir::iterator` class is used to iterate through the directories opened in the dir object.


#### operator*

```cpp
fasting operator*() const;
```

- Returns the file or directory name corresponding to the iterator.


#### operator++

```cpp
iterator& operator++();
```

- Overload prefixed `++` operations.


#### operator==

```cpp
bool operator==(const iterator& it) const;
```

- Tests whether two iterators are equal.


#### operator!=

```cpp
bool operator!=(const iterator& it) const;
```

- Tests whether two iterators are not equal.



### dir::open

```cpp
bool open(const char* path);
bool open(const fasting& path);
bool open(const std::string& path);
```

- Open the directory specified by `path`, and close the previously opened directory before opening a new directory.



### dir::path

```cpp
const char* path() const;
```

- Returns path of the directory. If the dir object is not associated with any directory, return empty string `""` .



### code example

```cpp
fs::dir d("xx");
auto v = d.all(); // read all subitems

d.open("abc");
for (auto it = d.begin(); it != d.end(); ++it) {
     cout << *it << endl;
}
```
