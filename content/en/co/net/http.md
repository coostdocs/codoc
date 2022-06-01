---
weight: 15
title: "HTTP"
---

include: [co/http.h](https://github.com/idealvin/co/blob/master/include/co/http.h).


## http::Client

`http::Client` is a coroutine-based http client, which is implemented based on [libcurl](https://curl.se/libcurl/). 



### Client::Client

```cpp
explicit Client(const char* serv_url);
```

- Constructor, the parameter serv_url is the url address of the server, and its form is `protocol://host:port`, the following server urls are all ok:
   - "github.com"
   - "https://github.com"
   - "http://127.0.0.1:7788"
   - "http://[::1]:8888"
- Like the tcp::Client, connection is not established in the constructor.



### Client::~Client

```cpp
Client::~Client();
```

- Destructor, close the connection, and release libcurl related resources.



### Client::add_header

```cpp
void add_header(const char* key, const char* val);
void add_header(const char* key, int val);
```

- Add a HTTP header.
- Users can use this method to add headers before performing HTTP requests, and the added headers will be present in all subsequent requests.
- In the second version, the parameter val is an integer, which is automatically converted into a string internally.



### Client::body

```cpp
const fastring& body() const;
```

- Get the response body of the current HTTP request.



### Client::close

```cpp
void close();
```

- Close the HTTP connection, must be called in the coroutine.
- Once this method is called, the http::Client object can no longer be used until you reset the server url by calling [reset()](#clientreset).



### Client::del

```cpp
void del(const char* url, const char* s, size_t n);
void del(const char* url, const char* s);
void del(const char* url);
```

- HTTP **DELETE** request, must be called in the coroutine.
- The parameter url must be a string beginning with `'/'`.
- The first two versions are suitable for DELETE requests with a body, and parameter s is a pointer to the body data.
- The third version is for DELETE requests without body.



### Client::easy_handle

```cpp
void* easy_handle() const;
```

- Return the easy handle of libcurl.



### Client::get

```cpp
void get(const char* url);
```

- HTTP **GET** request, must be called in the coroutine.
- The parameter url must be a string beginning with `'/'`.



### Client::head

```cpp
void head(const char* url);
```

- HTTP **HEAD** request, must be called in the coroutine.
- The parameter url must be a string beginning with `'/'`.



### Client::header

```cpp
const char* header(const char* key);
const fastring& header() const;
```

- The first version gets value of a HTTP header. If the header does not exist, an empty string is returned.
- The second version gets the entire HTTP header part.


- Example

```cpp
http::Client c("xx.com");
c.get("/");
auto s = c.header("Content-Length");
```



### Client::perform

```cpp
void perform();
```

- Perform a HTTP request, get, post and other methods are actually implemented based on this method.
- Users generally don't need to call this method. Only when the get, post and other methods provided by http::Client can't meet their needs, should they consider using this method to customize HTTP requests.


- Example

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

- HTTP **POST** request, must be called in the coroutine.
- The parameter url must be a string beginning with `'/'`.



### Client::put

```cpp
void put(const char* url, const char* path);
```

- HTTP **PUT** request, used to upload a file, must be called in the coroutine.
- The parameter url must be a string beginning with `'/'`.
- The parameter path is path of the file to be uploaded.



### Client::remove_header

```cpp
void remove_header(const char* key);
```

- The headers added by [add_header()](#clientadd_header) method will apply to all subsequent HTTP requests. If the user does not want a header to appear in a subsequent request, this method can be used to remove the header.



### Client::response_code

```cpp
int response_code() const;
```

- Get the response code of the current HTTP request.
- Normally, the return value is a value between 100 and 511.
- If the HTTP request is not sent due to network error or other reasons, or no response from the server was received within the timeout period, this method returns 0.



### Client::status

```cpp
int status() const;
```

- Equal to [response_code()](#clientresponse_code).



### Client::strerror

```cpp
const char* strerror() const;
```

- Get the error information of the current HTTP request. 



### Example


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

`http::Req` is an encapsulation of HTTP request, it is used in http::Server. 


### Req::Req

```cpp
Req() = default;
```

- Default constructor.



### Req::body

```cpp
const char* body() const;
```


- Get the body data in the HTTP request.
- It returns a pointer, **not null-terminated**. Users need call [body_size()](#reqbody_size) to get its length.



### Req::body_size

```cpp
size_t body_size() const;
```

- Returns the length of the HTTP request body.



### Req::header

```cpp
const char* header(const char* key) const;
```

- Get value of a HTTP header. If the header does not exist, an empty string is returned.



### Req::is_method_xxx

```cpp
bool is_method_get() const;
bool is_method_head() const;
bool is_method_post() const;
bool is_method_put() const;
bool is_method_delete() const;
bool is_method_options() const;
```

- Determine the method type of the HTTP request.



### Req::method

```cpp
Method method() const;
```

- Returns the HTTP request method, the return value is one of `kGet, kHead, kPost, kPut, kDelete, kOptions`.



### Req::url

```cpp
const fastring& url() const;
```

- Returns a reference to the url in the HTTP request. This value is part of the start line of the HTTP request.



### Req::version

```cpp
Version version() const;
```

- Returns the HTTP version in the HTTP request. The return value is one of `http::kHTTP10` or `http::kHTTP11`. Currently, HTTP/2.0 is not supported.




## http::Res

`http::Res` class is the encapsulation of HTTP response, it is used in http::Server. 


### Res::Res

```cpp
Res();
```

- Default constructor.



### Res::add_header

```cpp
void add_header(const char* key, const char* val);
void add_header(const char* key, int val);
```

- Add a HTTP header.



### Res::set_body

```cpp
void set_body(const void* s, size_t n);
void set_body(const char* s);
void set_body(const fastring& s);
```

- Set the body part of the HTTP response.
- The parameter s is the body data, and the parameter n is the length of s. In the second version, s ends with `'\0'`.



### Res::set_status

```cpp
void set_status(int status);
```

- Set the HTTP response code, this value is generally between 100 and 511.




## http::Server

`http::Server` is a coroutine-based HTTP server. It supports HTTPS. To use HTTPS, you need to install openssl first. 


### Server::Server

```cpp
Server();
```

- The default constructor, users don't need to care.



### Server::on_req

```cpp
void on_req(std::function<void(const Req&, Res&)>&& f);
template<typename T> void on_req(void (T::*f)(const Req&, Res&), T* o);
```

- Set a callback for processing a HTTP request.
- In the second version, the parameter f is a method in class T, and the parameter o is a pointer to type T.
- When the server receives an HTTP request, it will call the callback set by this method to process the request.



### Server::start

```cpp
void start(const char* ip="0.0.0.0", int port=80);
void start(const char* ip, int port, const char* key, const char* ca);
```

- Start the HTTP server, this method will not block the current thread.
- The parameter ip is the server ip, which can be an IPv4 or IPv6 address, and the parameter port is the server port. 
- The parameter **key** is path of a **PEM** file which stores the SSL private key, and the parameter **ca** is path of a PEM file which stores the SSL certificate. If key or ca is NULL or empty string, SSL will be disabled.



### Server::exit

```cpp
void exit();
```

- Added since v2.0.2.
- Exit the HTTP server, close the listening socket, and no longer receive new connections.
- This method will not close the connections that has been established before.



### Example

```cpp
http::Server s

s.on_req(
    [](const http::Req& req, http::Res& res) {
        if (req.is_method_get()) {
            if (req.url() == "/hello") {
                res.set_status(200);
                res.set_body("hello world");
            } else {
                res.set_status(404);
            }
        } else {
            res.set_status(405);
        }
    }
);

// start as a http server
s.start("0.0.0.0", 7777);

// start as a https server
s.start("0.0.0.0", 7777, "privkey.pem", "certificate.pem");
```


There is a simple http::Server demo in co/test, the user can build and run it like this:

```sh
xmake -b http_serv
xmake r http_serv
```

After starting the http server, you can enter `127.0.0.1/hello` in the address bar of the browser to see the result. 




## Static web server (so::easy)

```cpp
void easy(const char* root_dir=".", const char* ip="0.0.0.0", int port=80);
void easy(const char* root_dir, const char* ip, int port, const char* key, const char* ca);
```

- Start a static web server, the parameter root_dir is the root directory of the web server.
- The parameter ip can be an IPv4 or IPv6 address.
- The second version supports HTTPS, the parameter key is the SSL private key, the parameter ca is the SSL certificate, and both key and ca are files in pem format.
- When key or ca is NULL or an empty string, HTTPS is not used.
- This method will block the current thread.


- Example

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




## Config items

co/http uses [co/flag](../../flag/) to define config items. The flags defined in co/http are listed below.


### http_conn_timeout

```cpp
DEF_uint32(http_conn_timeout, 3000, "#2 connect timeout in ms for http client");
```

- Connect timeout in milliseconds for http::Client.



### http_timeout

```cpp
DEF_uint32(http_timeout, 3000, "#2 send or recv timeout in ms for http client");
```

- Receive and send timeout (libcurl does not distinguish between receive and send timeout) in milliseconds for http::Client.



### http_recv_timeout

```cpp
DEF_uint32(http_recv_timeout, 3000, "#2 recv timeout in ms for http server");
```

- Recv timeout in milliseconds for http::Server.



### http_send_timeout

```cpp
DEF_uint32(http_send_timeout, 3000, "#2 send timeout in ms for http server");
```

- Send timeout in milliseconds for http::Server.



### http_conn_idle_sec

```cpp
DEF_uint32(http_conn_idle_sec, 180, "#2 http server may close the con...");
```

- Timeout in seconds for http::Server to keep an idle connection. If the server does not receive data from the client within this time, it may close the connection.



### http_log

```cpp
DEF_bool(http_log, true, "#2 enable http server log if true");
```

- For http::Server, whether to print logs, the default is true.
- The log in http::Server will print the header part of HTTP request and response.



### http_max_idle_conn

```cpp
DEF_uint32(http_max_idle_conn, 128, "#2 max idle connections for http server");
```

- For http::Server, maximum number of idle connections. When this number is exceeded, some idle connections will be closed.



### http_max_body_size

```cpp
DEF_uint32(http_max_body_size, 8 << 20, "#2 max size of http body, default: 8M");
```

- The maximum body length supported by http::Server, the default is 8M.



### http_max_header_size

```cpp
DEF_uint32(http_max_header_size, 4096, "#2 max size of http header");
```

- The maximum header (the entire HTTP header) length supported by http::Server, the default is 4k.
