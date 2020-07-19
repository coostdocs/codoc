---
title: "解决 namespace log 与 math 库 log() 函数的冲突"
date: 2018-10-09T23:07:10+08:00
author: Alvin
keywords: C++,namespace,namespace log
---

C++ 中，在全局范围内定义 namespace log 时，编译器报错，排查后发现原因是与 math 库中的 log() 函数有冲突。将 log 换成其他的名字，可以解决冲突，但仅仅为了一个几乎用不到的函数，就做出如此巨大的让步，多少有点不甘心。

经过研究，最终找到一个比较完美的解决方案：

```cpp
namespace ___ {
namespace log {

void init() {
    std::cout << "log::init()" << std::endl;
}

} // namespace log
} // namespace ___

using namespace ___;
```

上面的代码耍了一个小技巧，将 `namespace log` 嵌套定义在 `namespace ___` 内，然后用 `using namespace ___;` 将 log 引入全局范围内。

下面是一段测试代码：

```cpp
#include <iostream>
#include <math.h>

namespace ___ {
namespace log {

void init() {
    std::cout << "log::init()" << std::endl;
}

} // namespace log
} // namespace ___

using namespace ___;

int main() {
    log::init();
    std::cout << ::log(3.2) << std::endl;
    return 0;
}
```

编译执行结果如下：
```cpp
log::init()
1.16315
```

可以看到，前面不加 `::` 时，`log` 表示 `namespace ___::log`，而加 `::` 时，`::log` 则表示全局的 log() 函数。解决了冲突，还不影响标准库函数的使用，算是完美。