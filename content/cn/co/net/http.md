---
weight: 15
title: "HTTP"
---


include: [co/http.h](https://github.com/idealvin/coost/blob/master/include/co/http.h).


## http::Client

`http::Client` 是基于协程的 http 客户端，它基于 [libcurl](https://curl.se/libcurl/) 实现。



### Client::Client

```cpp
explicit Client(const char* serv_url);
```

- 构造函数，参数 serv_url 是服务器的 url 地址，它的形式是 `protocol://host:port`，下面的 server url 都是合理的：
   - "github.com"
   - "https://github.com"
   - "http://127.0.0.1:7788"
   - "http://[::1]:8888"
- http::Client 对象创建时，并不会立即建立连接。



### Client::~Client

```cpp
Client::~Client();
```

- 析构函数，关闭连接，释放 libcurl 相关资源。



### Client::add_header

```cpp
void add_header(const char* key, const char* val);
void add_header(const char* key, int val);
```

- 添加 HTTP 头部，用户在进行 HTTP 请求前，可以用此方法添加头部，这些头部会自动添加到后续所有请求中。
- 第 2 个版本中，参数 val 是整数，内部自动转换成字符串。



### Client::body

```cpp
const fastring& body() const;
```

- 获取当前 HTTP 请求的响应体。



### Client::close

```cpp
void close();
```

- 关闭 HTTP 连接，一般需要在协程中调用此方法。
- 调用此方法后，http::Client 对象就不能再用了，直到用户调用 [reset()](#clientreset) 方法重置 server url。



### Client::del

```cpp
void del(const char* url, const char* s, size_t n);
void del(const char* url, const char* s);
void del(const char* url);
```

- HTTP **DELETE** 请求，必须在协程中调用。
- 参数 url 必须是 `'/'` 开头的字符串。
- 前两个版本，适用于带 body 部分的 DELETE 请求，参数 s 是 body，n 是 s 的长度，第 2 个版本 s 以 '\0' 结尾。
- 第 3 个版本适用于不带 body 的 DELETE 请求。



### Client::easy_handle

```cpp
void* easy_handle() const;
```

- 返回 libcurl 的 easy handle。



### Client::get

```cpp
void get(const char* url);
```

- HTTP **GET** 请求，必须在协程中调用。
- 参数 url 必须是 `'/'` 开头的字符串。



### Client::head

```cpp
void head(const char* url);
```

- HTTP **HEAD** 请求，必须在协程中调用。
- 参数 url 必须是 `'/'` 开头的字符串。



### Client::header

```cpp
const char* header(const char* key);
const fastring& header() const;
```

- 第 1 个版本获取当前 HTTP 响应中 header 的值，header 不存在时，返回一个空字符串。
- 第 2 个版本获取当前 HTTP 响应的整个 header 部分(包括起始行)。


- 示例

```cpp
http::Client c("xx.com");
c.get("/");
auto s = c.header("Content-Length");
```



### Client::perform

```cpp
void perform();
```

- 执行 HTTP 请求，get, post 等方法实际上都是基于此方法实现。
- 用户一般不需要调用此方法，只有当 http::Client 提供的 get, post 等方法满足不了需要时，才考虑用此方法自定义 HTTP 请求。


- 示例

```cpp
void Client::post(const char* url, const char* data, size_t size) {
    curl_easy_setopt(_ctx->easy, CURLOPT_POST, 1L);
    curl_easy_setopt(_ctx->easy, CURLOPT_URL, make_url(url));
    curl_easy_setopt(_ctx->easy, CURLOPT_POSTFIELDS, data);
    curl_easy_setopt(_ctx->easy, CURLOPT_POSTFIELDSIZE, (long)size);
    this->perform();
}
```



### Client::post

```cpp
void post(const char* url, const char* s, size_t n);
void post(const char* url, const char* s);
```

- HTTP **POST** 请求，必须在协程中调用。
- 参数 url 必须是 `'/'` 开头的字符串。
- 参数 s 是 HTTP 请求的 body 数据，n 是 s 的长度，第 2 个版本 s 以 '\0' 结尾。



### Client::put

```cpp
void put(const char* url, const char* path);
```

- HTTP **PUT** 请求，用于上传文件到服务器，必须在协程中调用。
- 参数 url 必须是 `'/'` 开头的字符串。
- 参数 path 是需要上传的文件路径。



### Client::remove_header

```cpp
void remove_header(const char* key);
```

- 删除之前添加的 HTTP header。调用 add_header() 方法添加的头部会存在于后续所有 HTTP 请求中，如果用户不想在后续请求中带上某些 header，可以用此方法显示删除。



### Client::reset

```cpp
void reset(const char* serv_url);
```

- 重置 server url，关于 url 的规范详见 [Client::Client](#clientclient)。



### Client::response_code

```cpp
int response_code() const;
```

- 获取当前 HTTP 请求的响应码。
- 正常情况下返回值是 100 到 511 之间的值。
- 若 HTTP 请求没有发送出去，或者没有收到服务端的响应，此方法返回 0，用户可以用 strerror() 方法获取错误信息。



### Client::status

```cpp
int status() const;
```

- 与 [response_code()](#clientresponse_code) 等价，返回当前 HTTP 请求的响应吗。



### Client::strerror

```cpp
const char* strerror() const;
```

- 获取当前 HTTP 请求的错误信息。



### 代码示例

```cpp
void f() {
    http::Client c("https://github.com");
    int r;
    
    c.get("/");
    r = c.status();
    LOG << "staus: " << r;
    LOG_IF(r == 0) << "error: " << c.strerror();
    LOG << "body size: " << c.body().size();
    LOG << c.header();

    c.get("/idealvin/co");
    LOG << "body size: " << c.body().size();
    LOG << "Content-Length: " << c.header("Content-Length");
    LOG << c.header();

    c.close();
}

go(f);
```




## http::Req

`http::Req` 类是对 HTTP 请求的封装，用于实现 http::Server。


### Req::Req

```cpp
Req() = default;
```

- 默认构造函数。



### Req::body

```cpp
const char* body() const;
```

- 获取 HTTP 请求中的 body 数据，返回一个指针，用户需要调用 [body_size()](#reqbody_size) 方法获取它的长度。



### Req::body_size

```cpp
size_t body_size() const;
```

- 返回 HTTP 请求 body 的长度。



### Req::header

```cpp
const char* header(const char* key) const;
```

- 获取 HTTP header 的值，header 不存在时，返回一个空字符串。



### Req::is_method_xxx

```cpp
bool is_method_get() const;
bool is_method_head() const;
bool is_method_post() const;
bool is_method_put() const;
bool is_method_delete() const;
bool is_method_options() const;
```

- 判断 HTTP 请求的方法类型。



### Req::method

```cpp
Method method() const;
```

- 返回 HTTP 请求方法，返回值是 `kGet, kHead, kPost, kPut, kDelete, kOptions` 中的一种。



### Req::url

```cpp
const fastring& url() const;
```

- 返回 HTTP 请求中的 url 的引用，这个值是 HTTP 请求 start line 中的一部分。



### Req::version

```cpp
Version version() const;
```

- 返回 HTTP 请求中的 HTTP 版本，返回值是 http::kHTTP10 或 http::kHTTP11 中的一种，目前不支持 HTTP/2.0。



## http::Res

`http::Res` 类是对 HTTP 响应的封装，用于实现 http::Server。



### Res::Res

```cpp
Res();
```

- 默认构造函数。



### Res::add_header

```cpp
void add_header(const char* key, const char* val);
void add_header(const char* key, int val);
```

- 添加 HTTP header。



### Res::set_body

```cpp
void set_body(const void* s, size_t n);
void set_body(const char* s);
void set_body(const fastring& s);
```

- 设置 HTTP 响应的 body 部分。
- 参数 s 是 body 数据，参数 n 是 s 的长度，第 2 个版本中 s 以 `'\0'` 结尾。



### Res::set_status

```cpp
void set_status(int status);
```

- 设置 HTTP 响应码，这个值一般是 100 到 511 之间的值。




## http::Server

`http::Server` 是基于协程的 HTTP 服务器，它支持 HTTPS，使用 HTTPS 需要安装 openssl。


### Server::Server

```cpp
Server();
```

- 默认构造函数，用户无需关心。



### Server::on_req

```cpp
Server& on_req(std::function<void(const Req&, Res&)>&& f);
Server& on_req(const std::function<void(const Req&, Res&)>& f)

template<typename T>
Server& on_req(void (T::*f)(const Req&, Res&), T* o);
```

- 设置处理客户端 HTTP 请求的 callback。
- 第 3 个版本中，参数 f 是类中的方法，参数 o 是 T 类型的指针。
- 服务端接收到 HTTP 请求时，会调用此方法设置的 callback，处理该请求。



### Server::start

```cpp
void start(const char* ip="0.0.0.0", int port=80);
void start(const char* ip, int port, const char* key, const char* ca);
```

- 启动 HTTP server，此方法不会阻塞当前线程。
- 参数 key 是存放 SSL private key 的 PEM 文件路径，参数 ca 是存放 SSL 证书的 PEM 文件路径，默认 key 和 ca 是 NULL，不启用 SSL。
- 从 v3.0 开始，server 启动后就不再依赖于 `http::Server` 对象。



### Server::exit

```cpp
void exit();
```

- v2.0.3 新增。
- 退出 HTTP server，关闭 listening socket，不再接收新的连接。
- 从 v3.0 开始，HTTP server 退出后，之前已经建立的连接将在未来被重置。



### 代码示例

```cpp
void cb(const http::Req& req, http::Res& res) {
    if (req.is_method_get()) {
        if (req.url() == "/hello") {
            res.set_status(200);
            res.set_body("hello world");
        } else {
            res.set_status(404);
        }
    } else {
        res.set_status(405); // method not allowed
    }
}

// http
http::Server().on_req(cb).start("0.0.0.0", 80);

// https
http::Server().on_req(cb).start(
    "0.0.0.0", 443, "privkey.pem", "certificate.pem"
);
```

`co/test` 提供了一个简单的 [demo](https://github.com/idealvin/coost/blob/master/test/so/http_serv.cc)，用户可以按下述方式编译运行:

```bash
xmake -b http_serv
xmake r http_serv
```

启动 http server 后，可以在浏览器的地址栏中输入 `127.0.0.1/hello` 看结果。




## 静态 web server (so::easy)

```cpp
void easy(const char* root_dir=".", const char* ip="0.0.0.0", int port=80);
void easy(const char* root_dir, const char* ip, int port, const char* key, const char* ca);
```

- 启动一个静态 web server，参数 root_dir 是 web server 的根目录。
- 参数 ip 可以是 IPv4 或 IPv6 地址。
- 参数 key 是存放 SSL private key 的 PEM 文件路径，参数 ca 是存放 SSL 证书的 PEM 文件路径，默认 key 和 ca 是 NULL，不启用 SSL。
- 此方法会阻塞当前线程。


- 示例

```cpp
#include "co/flag.h"
#include "co/http.h"

DEF_string(d, ".", "root dir"); // root dir of web server

int main(int argc, char** argv) {
    flag::init(argc, argv);
    so::easy(FLG_d.c_str()); // mum never have to worry again
    return 0;
}
```




## 配置项

co/http 使用 [co/flag](../../flag/) 定义配置项，下面列出的是 co/http 内部定义的 flag。


### http_conn_timeout

```cpp
DEF_uint32(http_conn_timeout, 3000, "#2 connect timeout in ms for http client");
```

- http::Client 的连接超时时间，单位为毫秒。



### http_timeout

```cpp
DEF_uint32(http_timeout, 3000, "#2 send or recv timeout in ms for http client");
```

- http::Client 接收、发送数据的超时时间 (libcurl 内部并不区分接收与发送超时时间)，单位为毫秒。



### http_recv_timeout

```cpp
DEF_uint32(http_recv_timeout, 3000, "#2 recv timeout in ms for http server");
```

- http::Server 接收超时时间，单位为毫秒。



### http_send_timeout

```cpp
DEF_uint32(http_send_timeout, 3000, "#2 send timeout in ms for http server");
```

- http::Server 发送超时时间，单位为毫秒。



### http_conn_idle_sec

```cpp
DEF_uint32(http_conn_idle_sec, 180, "#2 http server may close the con...");
```

- http::Server 保持空闲连接的超时时间，单位为秒，server 在此时间内，若未收到客户端的数据，则可能会关闭该连接。



### http_log

```cpp
DEF_bool(http_log, true, "#2 enable http server log if true");
```

- http::Server 是否打印日志，默认为 true。
- http::Server 中的日志会打印 HTTP 请求或响应的头部。



### http_max_idle_conn

```cpp
DEF_uint32(http_max_idle_conn, 128, "#2 max idle connections for http server");
```

- http::Server 最大空闲连接数，超过此数量时，会关闭部分空闲连接。



### http_max_body_size

```cpp
DEF_uint32(http_max_body_size, 8 << 20, "#2 max size of http body, default: 8M");
```

- http::Server 支持的最大 body 长度，默认为 8M。



### http_max_header_size

```cpp
DEF_uint32(http_max_header_size, 4096, "#2 max size of http header");
```

- http::Server 支持的最大 HTTP 头部(整个头部)长度，默认为 4k。
