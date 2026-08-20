## Exception Handling Basics

In C++, **exceptions** provide a structured, type-safe mechanism to signal and handle anomalous runtime conditions.

---

### The `throw` Expression

The `throw` expression is used to signal an exception. The operand of `throw` is copied into a temporary exception storage object managed by the C++ runtime (outside the call stack):

```cpp
#include <string>

void f() {
    throw std::string("An unhandled error occurred");
}

int main() {
    f();
    return 0;
}
```

If an exception is not caught, it propagates up the call stack to `main()`. When no matching handler is found:
1. `std::terminate()` is automatically invoked.
2. By default, `std::terminate()` calls `std::abort()` to abnormally terminate the program with a non-zero exit code:

```ascii
libc++abi: terminating due to uncaught exception of type std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char>>
[1]    30970 abort      ./a.out
```

---

### Stack Unwinding and RAII

When an exception is thrown, the C++ runtime searches up the call stack for an enclosing `try/catch` block.

During this search, the stack is **unwound**:
- Each active stack frame is popped in reverse call order.
- All automatic (local) objects constructed within those stack frames are properly destroyed in reverse order of their construction.
- This automatic cleanup is the basis for the **Resource Acquisition Is Initialization (RAII)** idiom in C++.

---

### The `try` and `catch` Blocks

Exceptions are handled using `try` and `catch` blocks:

```cpp
#include <iostream>
#include <string>

void f() {
    throw std::string("Error: resource unavailable");
}

int main() {
    try {
        f();
    } catch (const std::string& s) { // Catch by const reference to avoid slicing and copying
        std::cout << "Caught exception: " << s << std::endl;
    }

    return 0;
}
```

Output:
```ascii
Caught exception: Error: resource unavailable
```

#### Key Rules for `try/catch`:
- **Separate Scopes:** `try` and `catch` blocks introduce separate local block scopes. Variables declared inside a `try` block are not accessible inside the `catch` block.
- **Catch by Const Reference:** Always prefer catching exceptions by `const &` (`catch (const std::exception& e)`) to avoid unnecessary copies and prevent **object slicing** when catching polymorphic exception hierarchies.
- **Multiple Catch Blocks:** Handlers are evaluated in order from top to bottom. More derived exception types must be listed before base types.
- **Catch-All Handler (`catch (...)`):** Catches any thrown exception regardless of its type.

```cpp
try {
    // Risky operations
} catch (const std::out_of_range& e) {
    // Specific standard library exception
} catch (const std::exception& e) {
    // Base class for standard exceptions
} catch (...) {
    // Catches all other types
}
```

---

### Exceptions vs. Runtime Errors & Hardware Faults

> [!IMPORTANT]
> In C++, general runtime errors and hardware faults (such as segmentation faults, null pointer dereferences, or integer division by zero) are **not** C++ exceptions.

Hardware faults generate operating system signals (e.g., `SIGSEGV`, `SIGFPE`), which lead to undefined behavior (**UB**) or immediate process termination and **cannot** be caught by standard C++ `catch (...)` blocks.

```cpp
#include <vector>

int main() {
    try {
        std::vector<int> v;
        v[100000] = 1; // Undefined Behavior! Unchecked access causes a segmentation fault
    } catch (...) {
        // Will NEVER catch a segmentation fault!
    }
    return 0;
}
```

Output:
```ascii
[1]    31132 segmentation fault  ./a.out
```

#### Bounds Checking with `.at()`
To perform safe, bounds-checked container access that throws a catchable C++ exception (`std::out_of_range`), use `.at()` instead of `operator[]`:

```cpp
#include <iostream>
#include <stdexcept>
#include <vector>

int main() {
    try {
        std::vector<int> v;
        v.at(100000) = 1; // Throws std::out_of_range
    } catch (const std::out_of_range& e) {
        std::cout << "Caught out_of_range exception: " << e.what() << std::endl;
    }

    return 0;
}
```

Output:
```ascii
Caught out_of_range exception: vector
```

---

### Standard Operators and Built-in Exceptions

Several built-in C++ operators throw specific standard library exceptions upon failure:

- **`dynamic_cast<T&>`:** Throws `std::bad_cast` when a reference cast to a derived polymorphic type fails (pointer casts return `nullptr` instead).
- **`new` Operator:** Throws `std::bad_alloc` when heap memory allocation fails (unless using `new (std::nothrow)`).
- **`typeid(*ptr)`:** Throws `std::bad_typeid` when dereferencing a null pointer to a polymorphic type.

