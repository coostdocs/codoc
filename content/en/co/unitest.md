---
weight: 7
title: "Unitest"
---

include: [co/unitest.h](https://github.com/idealvin/co/blob/master/include/co/unitest.h).


## Basic concepts


`co/unitest` is a unit testing framework, similar to [google gtest](https://github.com/google/googletest), but easier to use. 




### Test Units and Test Cases


A test program can be divided into multiple test units according to functions or modules, and there can be multiple test cases under each test unit. For example, a test unit can be defined for a class (or module) in C++, and a test case can be defined for each method in the class (or module). 


In co/unitest, when a test unit is defined, a class is generated, and there is a `run()` method in the class, and code of all test cases are in the run() method. co/unitest almost hides all the details. Users can't see the class, or even the run() method, so they can concentrate on writing test code:


```cpp
DEF_test(test_name) {
    DEF_case(a) {
        // write test code here
    }
    
    DEF_case(b) {
        // write test code here
    }
}
```


The above `DEF_test` actually implements the run() method in the class, while `DEF_case` defines a test case. A test case is actually a code block in the run() method, not even a function. 




### DEF_test


```cpp
#define DEF_test(_name_) \
    DEF_bool(_name_, false, "enable this test if true."); \
    ... \
    void _UTest_##_name_::run()
```


- The `DEF_test` macro is used to define a test unit, and the parameter `_name_` is the name of the test unit.
- The first line of the macro defines a bool type flag variable, which is the switch of the test unit. For example, `DEF_test(os)` defines a test unit os, and we can use `-os` in the command line to enable test cases in this unit.
- The codes omitted in the middle of the macro actually defines a class, and the last line defines the method `run()` in the class, which requires the user to complete the function body.





### DEF_case


```cpp
#define DEF_case(name) _current_case.reset(new unitest::Case(#name));
```


- The `DEF_case` macro is used to define a test case in the test unit. The parameter name is the name of the test case. It must be used inside the run() method defined by `DEF_test`.
- The name of a test unit must be albe to use as part of the class name or variable name. The test case name does not have this restriction. For example, `DEF_case(sched.Copool)` is also reasonable.
- The code after DEF_case is all test code of this test case until the next DEF_case appears.
- The code of the test case is generally enclosed by a pair of curly braces to isolate it from other test cases.
- DEF_test may not contain any DEF_case. In this case, co/unitest will create a default test case.





### EXPECT assertion


```cpp
#define EXPECT(x) ...
#define EXPECT_EQ(x, y) EXPECT_OP(x, y, ==, "EQ")
#define EXPECT_NE(x, y) EXPECT_OP(x, y, !=, "NE")
#define EXPECT_GE(x, y) EXPECT_OP(x, y, >=, "GE")
#define EXPECT_LE(x, y) EXPECT_OP(x, y, <=, "LE")
#define EXPECT_GT(x, y) EXPECT_OP(x, y, >, "GT")
#define EXPECT_LT(x, y) EXPECT_OP(x, y, <, "LT")
```


- `EXPECT` asserts that `x` is true, and x can be any expression with a value of type bool.
- `EXPECT_EQ` asserts `x == y`.
- `EXPECT_NE` asserts `x != y`.
- `EXPECT_GE` asserts that `x >= y`.
- `EXPECT_LE` asserts that `x <= y`.
- `EXPECT_GT` asserts that `x > y`.
- `EXPECT_LT` asserts that `x < y`.
- When defining a test case with `DEF_case`, you can use these macro assertions. If an assertion fails, it means that the test case fails. The terminal will print related error messages in red color.





## Write test code


### Test code example


```cpp
// os.cc
#include "co/unitest.h"
#include "co/os.h"

namespace test {
    
DEF_test(os) {
    DEF_case(homedir) {
        EXPECT_NE(os::homedir(), "");
    }

    DEF_case(pid) {
        EXPECT_GE(os::pid(), 0);
    }

    DEF_case(cpunum) {
        EXPECT_GT(os::cpunum(), 0);
    }
}
    
} // namespace test
```


- The above code defines a test unit named os, and os has 3 test cases.
- When running the test program, you can use `-os` in the command line to enable this unit test.
- The code of different test units are generally put in different .cc source files.
- The user can put test code in a namespace, if necessary.





### Default test case


```cpp
DEF_test(os) {
    EXPECT_NE(os::homedir(), "");
    EXPECT_GE(os::pid(), 0);
    EXPECT_GT(os::cpunum(), 0);
}
```


- The above code does not contain any DEF_case, co/unitest will create a default test case named "default".
- For more complex test codes, it is generally not recommended to use the default test cases. It is better to divide them into different cases so that the code looks clearer.





### main() function


```cpp
#include "co/unitest.h"

int main(int argc, char** argv) {
    flag::init(argc, argv);
    unitest::run_all_tests();
    return 0;
}
```


- Call the `run_all_tests()` method provided by `co/unitest` in the main function to start running the test code.
- Before calling `run_all_tests()`, the `flag::init()` method must be called to parse the command line parameters.





## Build and run the test program


### Build the unitest code


```bash
xmake -b unitest
```


- Execute the above command in the co root directory to compile the unit test code in the [co/unitest](https://github.com/idealvin/co/tree/master/unitest) directory and generate the `unitest` binary program .





### Run all test cases

```bash
# Run all test cases
xmake r unitest
```




### Run test cases in specified test units


```bash
# Run only test cases in the os test unit
xmake r unitest -os

# Run test cases in the os or json test units
xmake r unitest -os -json
```


- By default, all test units are disabled, you need to enable the test code with the corresponding switch.





### Test result example


- All tests passed

![unitest_passed.png](/images/unitest_passed.png)


- Test case failed

![unitest_failed.png](/images/unitest_failed.png)
