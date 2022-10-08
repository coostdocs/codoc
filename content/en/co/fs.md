---
weight: 22
title: "File System"
---

include: [co/fs.h](https://github.com/idealvin/coost/blob/master/include/co/fs.h).




`co/fs.h` implements common file system operations. It is recommended to use `'/'` uniformly as the path separator on different platforms. 




## Metadata operations


### fs::exists


```cpp
bool exists(const char* path);
bool exists(const fastring& path);
bool exists(const std::string& path);
```


- Check whether the file exists, the parameter path is path of a file or directory.





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


- Check whether the file is a directory, if path exists and is a directory, it returns true, otherwise it returns false.





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


- Create a directory, the parameter p indicates whether to create the entire path.
- The parameter p is false by default, and the directory will be created only when the parent directory exists; when the parameter p is true, it is equivalent to `mkdir -p`, and the parent directory will be created first if it does not exist, .





### fs::remove


```cpp
bool remove(const char* path, bool rf=false);
bool remove(const fastring& path, bool rf=false);
bool remove(const std::string& path, bool rf=false);
```


- Delete a file or directory.
- When path is a directory, the parameter rf indicates whether to force deletion, the default is false, and only delete an empty directory. If rf is true, it is equivalent to `rm -rf`, non-empty directory will also be deleted.
- When path is a file, the parameter rf will be ignored.





### fs::rename


```cpp
bool rename(const char* from, const char* to);
bool rename(const fastring& from, const fastring& to);
bool rename(const std::string& from, const std::string& to);
```


- Rename the file or directory, the parameter from is the original path, and the parameter to is the new path.
- When the parameter to is a directory, windows requires to and from to be under the same drive.
- It is generally recommended to use this method when the path specified by the parameter to does not exist. For details, please refer to [win32/MoveFile](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-movefilea), [linux/rename](https://www.man7.org/linux/man-pages/man2/rename.2.html).





### fs::symlink


```cpp
bool symlink(const char* dst, const char* lnk);
bool symlink(const fastring& dst, const fastring& lnk);
bool symlink(const std::string& dst, const std::string& lnk);
```


- Create a soft link, the parameter dst is the path of the target file or directory, and the parameter lnk is the path of the soft link.
- This function first calls `fs::remove(lnk)` to delete the old soft link, and then create a new soft link.
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






## Read and write files (fs::file)


The `fs::file` class implements the basic read and write operations of files. Unlike `fread` and `fwrite`, it has no cache, but reads and writes files directly. 




### file::file


```cpp
file();
file(file&& f);
file(const char* path, char mode);
file(const fastring& path, char mode);
file(const std::string& path, char mode);
```


- The first version is the default constructor, which creates an empty file object without opening a file.
- The second version is the move constructor.
- Version 3-5, open the specified file, the parameter path is the file path, and the parameter mode is the open mode.
- mode is one of `'r'`, `'w'`, `'a'`, `'m'` or `'+'`, r for read, w for write, a for append, m for modify, + for both read and write.
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


- This method opens the specified file, mode is the open mode, see the description in [Constructor](#filefile).
- This method will close the previously opened file before opening a new file.





### file::operator bool


```cpp
explicit operator bool() const;
```


- Convert fs::file to bool type, return true when the file is successfully opened, otherwise return false.





### file::operator!


```cpp
bool operator!() const;
```


- It returns true when the file is not opened, otherwise it returns false.





### file::path


```cpp
const fastring& path() const;
```


- This method returns a reference to the file path.
- If the file object is not associated with any file, the return value will refer to an empty string.





### file::read


```cpp
size_t read(void* buf, size_t n);
fastring read(size_t n);
```


- The first version reads data into the specified buffer, n is the number of bytes to be read, and returns the number of bytes actually read.
- The second version is similar to the first version, but returns a fastring, where n is the number of bytes to be read.
- The actual bytes read may be less than n, when it reaches the end of the file or an error occurs.





### file::seek


```cpp
void seek(int64 off, int whence=seek_beg);
```


- Set the current position of the file pointer, the parameter off is the offset position, and the parameter whence is the starting position, which can be one of `file::seek_beg`, `file::seek_cur`, `file::seek_end`.
- This method is not valid for files opened with mode `'a'`.





### file::size


```cpp
int64 size() const;
```


- This method returns the size of the file. When the file is not opened, calling this method will return -1.





### file::write


```cpp
size_t write(const void* s, size_t n);
size_t write(const char* s);
size_t write(const fastring& s);
size_t write(const std::string& s);
size_t write(char c);
```


- The first version writes a n-byte data.
- The 2-4th version writes a string.
- The fifth version writes a single character.
- This method returns the number of bytes actually written. When the disk space is insufficient or other error occurs, the return value may be less than n.
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






## File Stream (fs::fstream)


`fs::file` does not support caching, and the performance of writing small files is poor. For this reason, `co/fs.h` also implements the `fs::fstream` class that supports caching, it is designed for writing files and does not support read operations. 




### fstream::fstream


```cpp
fstream();
fstream(fstream&& fs);
explicit fstream(size_t cap);
fstream(const char* path, char mode, size_t cap=8192);
fstream(const fastring& path, char mode, size_t cap=8192);
fstream(const std::string& path, char mode, size_t cap=8192);
```


- The first version is the default constructor, and the internal cache size is 8k.
- The second version is the move constructor.
- The third version uses the parameter cap to specify the size of the cache.
- Version 4-6 open the specified file, cap is the cache size, and the default is 8k.
- The parameter mode is one of `'w'` or `'a'`, and the read mode is not supported.
- When mode is `'w'`, the file is automatically created when it does not exist, and the file data is cleared if it already exists.
- When the mode is `'a'`, the file will be created automatically if it does not exist, and the file data will not be cleared if it already exists.





### fstream::~fstream


```cpp
~fstream();
```


- Destructor, close the file, release related resources.





### fstream::append


```cpp
fstream& append(const void* s, size_t n);
```


- Append data to the file, the parameter n is the length of the data.





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


- Write the data in the cache to the file.





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


- Convert fs::fstream to bool type, return true when the file is successfully opened, otherwise return false.





### fstream::operator!


```cpp
bool operator!() const;
```


- It returns true when the file is not opened, otherwise it returns false.





### fstream::operator<<


```cpp
fstream& operator<<(const char* s);
fstream& operator<<(const fastring& s);
fstream& operator<<(const std::string& s);
fstream& operator<<(const fastream& s);
template<typename T> fstream& operator<<(T v);
```


- In versions 1-3, the parameter s is a string type.
- In the 4th version, the parameter s is a fastream.
- In the fifth version, T can be any built-in type, such as bool, char, int, double, etc.





### fstream::reserve


```cpp
void reserve(size_t n);
```


- Adjust the cache capacity, the parameter n is the capacity. If n is less than the previous capacity, the capacity remains unchanged.





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
