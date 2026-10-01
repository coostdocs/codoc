---
weight: 6
title: "单元测试"
---

## 头文件

```cpp
#include "co/unitest.h"
```


## API

公开函数只有一个，在 `co` 命名空间：

```cpp
int co::run_unitests();
```

- 运行单元测试，返回失败用例数；
- `main` 一般是固定写法：

```cpp
#include "co/unitest.h"

int main(int argc, char** argv) {
    flag::parse(argc, argv);
    co::run_unitests();
    return 0;
}
```


## 定义测试单元

```cpp
DEF_test(name) {
    // 测试代码
    // DEF_case(xxx) {}
}
```

- `DEF_test` 宏定义一个测试单元，实际是一个函数，用户可以在其中自由添加代码，如各测试用例共用的初始化代码；
- `name` 必须是合法变量名；
- 有多个 `DEF_test` 时，`name` 不能重复。


## 定义测试用例

```cpp
DEF_case(name) {
    // 测试用例代码
}
```

- `DEF_case` 宏定义一个测试用例，实际是 `DEF_test` 所定义函数中的代码块；
- `name` 不要求是合法变量名；
- `DEF_test` 中不含任何 `DEF_case` 时，会创建一个默认测试用例。


## EXPECT 宏

```cpp
EXPECT(x)
EXPECT_EQ(x, y)   // ==
EXPECT_NE(x, y)   // !=
EXPECT_GE(x, y)   // >=
EXPECT_LE(x, y)   // <=
EXPECT_GT(x, y)   // >
EXPECT_LT(x, y)   // <
```

- 失败信息在最后统一汇总。
- `EXPECT_XX` 中参数需要支持相应比较运算符，且可打印，即支持：
  ```cpp
  operator<<(co::string&, const T&);
  ```  


## 运行逻辑

- 每个 `DEF_test` 内部定义一个 bool flag，默认值 `false`。
- 若所有测试单元 flag 都是默认值，则运行所有测试单元。
- 若有 flag 为 `true`，则只运行 flag 为 `true` 的测试单元。
- 测试单元按注册顺序执行。


## 示例

```cpp
#include "co/unitest.h"

DEF_test(math) {
    DEF_case(add) {
        EXPECT_EQ(1 + 1, 2);
        EXPECT_NE(1 + 1, 3);
        EXPECT_GT(2, 1);
        EXPECT_LT(1, 2);
        EXPECT_GE(2, 2);
        EXPECT_LE(2, 2);
        EXPECT(1 + 1 == 2);
    }

    DEF_case(mul) {
        EXPECT_EQ(2 * 3, 6);
        EXPECT_NE(2 * 3, 5);
    }
}

DEF_test(string) {
    co::string s;

    // 不在 DEF_case 中，默认测试用例
    EXPECT(s.empty());

    DEF_case(empty) {
        EXPECT(s.empty());
    }

    DEF_case(size) {
        s = "hello";
        EXPECT_EQ(s.size(), 5);
    }
}

int main(int argc, char** argv) {
    flag::parse(argc, argv);
    co::run_unitests();
    return 0;
}
```

运行：

```bash
./xx            # 运行 math 和 string 两个单元
./xx -math      # 只运行 math
./xx -string    # 只运行 string
```


## 构建及运行 coost 内部单元测试

[unitest](https://github.com/idealvin/coost/tree/master/unitest) 目录下是 coost 内部单元测试代码，在 coost 根目录执行下述命令构建及运行：

```bash
# 构建
xmake b unitest

# 默认运行所有单元测试代码
xmake r unitest

# 仅运行指定的测试单元
xmake r unitest -os -json
```


测试结果示例：

- 测试全部通过

![unitest_passed.png](/images/unitest_passed.png)

- 测试用例未通过

![unitest_failed.png](/images/unitest_failed.png)
