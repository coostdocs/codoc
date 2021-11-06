---
weight: 16
title: "RPC"
---


include: [co/so/rpc.h](https://github.com/idealvin/co/blob/master/include/co/so/rpc.h).


co/rpc is a RPC framework similar to [grpc](https://github.com/grpc/grpc) and [thrift](https://github.com/apache/thrift), it uses JSON format for transmission data, which is different from binary protocols such as protobuf, thrift, etc.




## rpc::Service


`rpc::Service` is an interface class, which represents a service. A RPC server can contain multiple services. 




### Service::name


```cpp
virtual const char* name() const = 0;
```


- Returns the service name, which contains the package name, such as "xx.yy.HelloWorld".
- In the RPC request, a **"service" ** field must be carried, and its value is the service name.





### Service::process


```cpp
virtual void process(const Json& req, Json& res) = 0;
```


- This method is used to process a RPC request, and the result is filled in res.
- When the server receives an RPC request, it **finds the corresponding service according to the "service" field in req**, and then calls the process() method to process the request.







## rpc::Server


`rpc::Server` is implemented based on `tcp::Server`, which supports SSL and user name and password authentication. Test results a few years ago showed that a single-threaded QPS can reach 120k+. 




### Server::Server


```cpp
Server();
```


- The default constructor, users do not need to care.





### Server::add_service


```cpp
void add_service(Service* s);
```


- Add a service, the parameter s must be dynamically created with operator new.
- The user can call this method multiple times to add multiple services, and **different services must have different names**.





### Server::add_userpass


```cpp
void add_userpass(const char* user, const char* pass);
void add_userpass(const char* s);
```


- The first version adds a pair of username and password. The user can call this method multiple times to add multiple user names and passwords.
- In the second version, s is a JSON string containing usernames and passwords, and its value is like `{"user1":"pass1", "user2":"pass2"}`. Users can call this method to add multiple pairs of usernames and passwords at once.
- rpc::Server does not store the password plaintext in memory, but saves the md5 of the password.



- Example



```cpp
DEF_string(password, "", "password");
DEF_string(userpass, "", "usernames and passwords");

rpc::Server s;
s.add_userpass("alice", FLG_password.c_str());
FLG_password.safe_clear();

s.add_userpass(FLG_userpass.c_str());
FLG_userpass.safe_clear();
```


- The above example, defines a flag to save the password. The default value is empty. The password must be passed in from the command line or configuration file.
- After calling the add_userpass() method, immediately call the safe_clear() method to clear the password content.





### Server::start


```cpp
void start(const char* ip, int port, const char* key=NULL, const char* ca=NULL);
```


- Start the RPC server, this method will not block the current thread.
- The parameter ip is the server ip, which can be an IPv4 or IPv6 address, and the parameter port is the server port. 
- The parameter **key** is path of a **PEM **file which stores the SSL private key, and the parameter **ca** is path of a PEM file which stores the SSL certificate. They are NULL by default, and SSL is disabled.




### Server::exit
```cpp
void exit();
```

- Added since v2.0.2.
- Exit the RPC server, close the listening socket, and no longer receive new connections.
- This method will not close the connections that has been established before.






## RPC server example


### Define a proto file


```cpp
// hello_world.proto

package xx // namespace xx

service HelloWorld {
    hello,
    world,
}
```


- The above is a simple proto file, # Or // are for comments.

- **package xx** specifies the package name. In C++, it means that the code is generated into a namespace. You can use package xx.yy.zz to generate nested namespaces.
- **service HelloWorld** defines a service class that inherits from **rpc::Service**, and its name is **"xx.HelloWorld"**. The service class provides two RPC methods, hello and world.
- **A proto file can define only one service. **In the proto file, the contents after the service definition will be ignored. Generally, you can add the JSON parameter of the RPC methods after the service definition, as shown below:



```cpp
package xx  // namespace xx

// class HelloWorld : public rpc::Service
service HelloWorld {  
    hello,  // void HelloWorld::hello(const Json& req, Json& res);
    world,  // void HelloWorld::world(const Json& req, Json& res);
}

// All the following are ignored by the code generator.
// param
hello.req {
    "service": "xx.HelloWorld",
    "method": "hello"
}

hello.res {
    "err": 200,
    "errmsg": "ok"
}

world.req {
    "service": "xx.HelloWorld",
    "method": "world"
}

world.res {
    "err": 200,
    "errmsg": "ok"
}
```




### Generate RPC framework code


[gen](https://github.com/idealvin/co/tree/master/gen) is the RPC code generator provided by co, which can be used to generate code for RPC service and client.


```bash
xmake -b gen             # build gen
cp gen /usr/local/bin    # put gen in the /usr/local/bin directory
gen hello_world.proto    # Generate code
gen *.proto              # Batch generation
```




The file generated for hello_world.proto is hello_world.h, and code of the service class is as follows:


```cpp
// Autogenerated. DO NOT EDIT. All changes will be undone.

#pragma once

#include "co/so/rpc.h"

namespace xx {

class HelloWorld : public rpc::Service {
  public:
    typedef void (HelloWorld::*Fun)(const Json&, Json&);

    HelloWorld() : _name("xx.HelloWorld") {
        _methods[hash64("hello")] = &HelloWorld::hello;
        _methods[hash64("world")] = &HelloWorld::world;
    }

    virtual ~HelloWorld() {}

    virtual const char* name() const {
        return _name.c_str();
    }

    virtual void process(const Json& req, Json& res) {
        json::Value method = req["method"];
        if (!method.is_string()) {
            res.add_member("err", 400);
            res.add_member("errmsg", "req has no method");
            return;
        }

        auto it = _methods.find(hash64(method.get_string(), method.string_size()));
        if (it == _methods.end()) {
            res.add_member("err", 404);
            res.add_member("errmsg", "method not found");
            return;
        }

        (this->*(it->second))(req, res);
    }

    virtual void hello(const Json& req, Json& res) = 0;

    virtual void world(const Json& req, Json& res) = 0;

  private:
    std::unordered_map<uint64, Fun> _methods;
    fastring _name;
};

} // xx
```


- As you can see, the HelloWorld class inherits from rpc::Service, and it has already implemented the name() and process() methods in rpc::Service.
- In the process() method, the corresponding RPC method will be called according to the "method" field in req.
- Users only need to inherit the HelloWorld class and implement the hello and world methods.





### Implement the RPC service


```cpp
#include "hello_world.h"

namespace xx {

class HelloWorldImpl: public HelloWorld {
  public:
    HelloWorldImpl() = default;
    virtual ~HelloWorldImpl() = default;

    virtual void hello(const Json& req, Json& res) {
        res.add_member("method", "hello");
        res.add_member("err", 200);
        res.add_member("errmsg", "ok");
    }

    virtual void world(const Json& req, Json& res) {
        res.add_member("method", "world");
        res.add_member("err", 200);
        res.add_member("errmsg", "ok");
    }
};

} // xx
```


- The above is just a very simple example. In actual applications, it is generally necessary to perform corresponding business processing according to the parameters in req, and then fill in res.





### Start RPC server


```cpp
rpc::Server s;
s.add_userpass("alice", "nice");
s.add_service(new xx::HelloWorldImpl);

// without ssl
s.start("127.0.0.1", 7788);

// with ssl
s.start("127.0.0.1", 7788, "privkey.pem", "certificate.pem");
```


- Line 2 calls add_userpass() to add a pair of username and password, which is optional.
- Line 3 calls add_service() to add the implementation of the HelloWorld service. 
- Lines 6 and 9 start the RPC server, and line 9 adds two more parameters to specify SSL transmission.







## rpc::Client




### Client::Client


```cpp
Client(const char* ip, int port, bool use_ssl=false);
```


- Constructor. The parameter ip is the ip of the server, which can be a domain name, IPv4 or IPv6 address; the parameter port is the server port; the parameter use_ssl indicates whether to enable SSL transmission, the default is false, and SSL is not enabled.
- When rpc::Client was constructed, no connection was established.





### Client::~Client


```cpp
Client::~Client();
```


- Destructor, close the internal connection.





### Client::call


```cpp
void call(const Json& req, Json& res);
```


- Perform a RPC request, it must be called in the coroutine.
- The parameter `req` must contain at least two fields, "service" and "method", which are used to call the method in the specified service.
- The parameter `res` is the response result of the RPC request.
- If the RPC request is not sent, or no response from the server is received, res will not be filled.
- This method checks the connection status before sending the RPC request, and establishes the connection first if it is not connected.





### Client::close


```cpp
void close();
```


- Close the connection, it is safe to call this function multiple times.





### Client::ping


```cpp
void ping();
```


- Send a heartbeat to rpc::Server.





### Client::set_userpass


```cpp
void set_userpass(const char* user, const char* pass);
```


- Set a pair of username and password.
- This method only needs to be called once, if it is called multiple times, the later value will overwrite the previous value.
- This method must be called before initiating an RPC request.





## RPC client example




### Use the rpc::Client


```cpp
DEF_bool(use_ssl, false, "use ssl if true");
DEF_string(password, "", "password");
DEF_int32(n, 3, "request num");

void client_fun() {
    rpc::Client c("127.0.0.1", 7788, FLG_use_ssl);
    c.set_userpass("alice", FLG_password.c_str());
    FLG_password.safe_clear(); // clear password in the memory

    for (int i = 0; i < FLG_n; ++i) {
        Json req, res;
        req.add_member("service", "xx.HelloWorld");
        req.add_member("method", "hello");
        c.call(req, res);
        co::sleep(1000);
    }

    c.close();
}

go(client_fun);
```


- In RPC request, **"service" and "method"** are required fields.
- To use rpc::Client, users need to manually set the "service" and "method" fields in the request.





### Use the automatically generated HelloWorldClient


The code generated by hello_world.proto above also contains a client code:


```cpp
class HelloWorldClient {
  public:
    HelloWorldClient(const char* ip, int port, bool use_ssl=false)
        : _rpc_cli(ip, port, use_ssl), _serv_name("xx.HelloWorld") {
    }

    HelloWorldClient(const HelloWorldClient& c)
        : _rpc_cli(c._rpc_cli), _serv_name(c._serv_name) {
    }

    ~HelloWorldClient() {}

    void set_userpass(const char* user, const char* pass) {
        _rpc_cli.set_userpass(user, pass);
    }

    void close() {
        _rpc_cli.close();
    }

    Json make_req_hello() {
        Json req;
        req.add_member("service", _serv_name);
        req.add_member("method", "hello");
        return req;
    }

    Json make_req_world() {
        Json req;
        req.add_member("service", _serv_name);
        req.add_member("method", "world");
        return req;
    }

    Json perform(const Json& req) {
        Json res;
        _rpc_cli.call(req, res);
        return res;
    }

    void ping() {
        _rpc_cli.ping();
    }

  private:
    rpc::Client _rpc_cli;
    fastring _serv_name;
};
```


- HelloWorldClient just simply wraps rpc::Client, which is a little more convenient than rpc::Client, and there is no need to manually set the "service" and "method" fields.



```cpp
#include "hello_world.h"

std::unique_ptr<xx:HelloWorldClient> proto;

co::Pool p(
    []() { return (void*) new xx::HelloWorldClient(*proto); },
    [](void* p) { delete (xx::HelloWorldClient*) p; }
);

void client_fun() {
    co::PoolGuard<xx::HelloWorldClient> c(p);

    for (int i = 0; i < 10; ++i) {
        Json req = c->make_req_hello();
        req.add_member("xx", "123");
        Json res = c->perform(req);
        co::sleep(1000);
    }
}

proto.reset(new xx::HelloWorldClient("127.0.0.1", 7788));
proto->set_userpass("alice", "nice");

for (int i = 0; i < 8; ++i) {
    go(client_fun);
}
```


- In the above example, co::Pool is used to store the clients, and multiple coroutines can share these clients.
- co::PoolGuard automatically pops an idle client from co::Pool when it is created, and automatically puts the client back into co::Pool when it is destructed.
- The ccb of co::Pool uses copy construction to copy a client from proto, so that ip, port, username and password need to be set only once in the proto client.
- The make_req_xxx() method provided by HelloWorldClient returns a Json object filled with the "service" and "method" fields.
- The perform() method provided by HelloWorldClient performs the RPC request and returns the RPC response result.







## Configuration items




### rpc_conn_idle_sec


```cpp
DEF_int32(rpc_conn_idle_sec, 180, "#2 connection may be closed if no data...");
```


- Timeout in **seconds **for idle connections in rpc::Server. If a connection does not receive any data within this time, the server may close the connection.





### rpc_conn_timeout


```cpp
DEF_int32(rpc_conn_timeout, 3000, "#2 connect timeout in ms");
```


- Connect timeout in milliseconds for rpc::Client.





### rpc_log


```cpp
DEF_bool(rpc_log, true, "#2 enable rpc log if true");
```


- Whether to print RPC logs, the default is true, rpc::Server and rpc::Client will print RPC requests and responses.





### rpc_max_idle_conn


```cpp
DEF_int32(rpc_max_idle_conn, 128, "#2 max idle connections");
```


- Maximum number of idle connections for rpc::Server. The default is 128. When this number is exceeded, the server will close some idle connections.





### rpc_max_msg_size


```cpp
DEF_int32(rpc_max_msg_size, 8 << 20, "#2 max size of rpc message, default: 8M");
```


- The maximum length of RPC messages, the default is 8M.





### rpc_recv_timeout


```cpp
DEF_int32(rpc_recv_timeout, 1024, "#2 recv timeout in ms");
```


- RPC recv timeout in milliseconds.





### rpc_send_timeout


```cpp
DEF_int32(rpc_send_timeout, 1024, "#2 send timeout in ms");
```


- RPC send timeout in milliseconds.
