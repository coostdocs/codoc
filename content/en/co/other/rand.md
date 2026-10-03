---
weight: 7
title: "Random Values"
---

## Header

```cpp
#include "co/rand.h"
```

The API is in the `co` namespace; `flag::parse` is not required.

## Random Numbers

```cpp
// Thread-safe, returns 0 < x < 2^31-1
uint32 co::rand();

// seed must be in (0, 2^31-1); after the call it is updated to the return value
uint32 co::rand(uint32& seed);

// Thread-safe, returns a 64-bit random number
uint64 co::rand64();

// splitmix64, seed can be 0; after the call it is updated
uint64 co::rand64(uint64& seed);
```

- `rand()` / `rand64()` are thread-safe and use thread_local state internally.
- The seed for `rand(seed)` cannot be 0, otherwise subsequent return values will always be 0; it also cannot be `2^31-1`.
- `rand64(seed)` has no requirement on the initial value of seed.
- The thread safety of the versions with seed depends on how the caller uses the seed.

## Random Strings

```cpp
// Write random characters to buf; no '\0' is appended at the end
void co::randchars(void* buf, size_t bufsize);

// Return a random string of length n, default 15, thread-safe
co::string co::randstr(uint32 n=15);

// Use the specified character set; supports ranges such as "0-9", "a-f"
co::string co::randstr(const char* charset, uint32 n);
```

- `randstr()` is thread-safe and uses thread_local state internally.
- The default character set for `randchars`: `a-z`, `A-Z`, `0-9`, `_`, `-`, 64 characters in total.
- `randstr(charset, n)` supports range abbreviations, such as `"0-9"`, `"a-f"`, `"0-9A-Za-z"`.
- `randchars` does not write a terminating `'\0'`.

## Example

```cpp
#include "co/rand.h"
#include "co/print.h"

int main() {
    // Random numbers
    co::println("rand()   = ", co::rand());
    co::println("rand64() = ", co::rand64());

    uint32 seed = 12345;
    co::println("rand(seed) = ", co::rand(seed));
    co::println("seed = ", seed);

    uint64 seed64 = 0;
    co::println("rand64(seed) = ", co::rand64(seed64));
    co::println("seed64 = ", seed64);

    // Random strings
    co::println("randstr()        = ", co::randstr());
    co::println("randstr(8)       = ", co::randstr(8));
    co::println("randstr(0-9, 8)  = ", co::randstr("0-9", 8));
    co::println("randstr(0-9a-f)  = ", co::randstr("0-9a-f", 8));

    char buf[9];
    co::randchars(buf, 8);
    buf[8] = '\0';
    co::println("randchars = ", buf);
    return 0;
}
```
