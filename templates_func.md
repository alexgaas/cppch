## Function Template Overloading

Function templates can be overloaded both with other function templates and with non-template functions.

### Overload Resolution Rules

Consider an example with several overloaded function templates and a non-template function:

```cpp
#include <iostream>

template <typename T, typename U>
void f(T i, U j) {
    std::cout << "1" << std::endl;
}

template <typename T>
void f(T i, T j) {
    std::cout << "2" << std::endl;
}

void f(int i, int j) {
    std::cout << "3" << std::endl;
}

int main() {
    f(1, 1.0);   // Calls f<int, double>(int, double) -> prints 1
    f(1.0, 1.0); // Calls f<double>(double, double)   -> prints 2
    f(1, 1);     // Calls non-template f(int, int)   -> prints 3
    return 0;
}
```

Output:
```ascii
1
2
3
```

When resolving calls among overloaded function templates and non-template functions, the compiler follows key principles:
- **Non-template functions are preferred** over template functions if the match is exact.
- **More specialized templates are preferred** over more general templates (e.g., `f(T, T)` is more specialized than `f(T, U)`).
- **Exact type matching is preferred** over implicit type conversions.

---

### Implicit Conversions in Template Argument Deduction

Template argument deduction for a single template parameter `T` shared across multiple parameters does not perform implicit type conversions:

```cpp
#include <iostream>

template <typename T>
void f(T i, T j) {
    std::cout << "2" << std::endl;
}

int main() {
    f(1, 1.0); // Compilation Error!
    f(1.0, 1.0);
    f(1, 1);
    return 0;
}
```

This produces a compilation error (**CE**):
```ascii
main.cpp:9:5: error: no matching function for call to 'f'
    9 |     f(1, 1.0);
      |     ^
```

**Reason**: When deducing `T` for `f(1, 1.0)`, template argument deduction attempts to deduce `T` as `int` from the first argument and `double` from the second. Because `T` cannot be deduced as two conflicting types simultaneously, template deduction fails. Implicit type conversions (such as converting `1` to `double` or `1.0` to `int`) are not attempted during template argument deduction.

---

### Explicit Template Arguments and Empty Angle Brackets (`<>`)

#### Explicit Template Arguments

We can explicitly specify template arguments at the call site to bypass or guide template argument deduction:

```cpp
#include <iostream>

template <typename T, typename U>
void f(T i, U j) {
    std::cout << "1" << std::endl;
}

template <typename T>
void f(T i, T j) {
    std::cout << "2" << std::endl;
}

void f(int i, int j) {
    std::cout << "3" << std::endl;
}

int main() {
    f<double>(1, 1);
    return 0;
}
```

Output:
```ascii
2
```

When providing explicit template arguments, you do not need to specify all of them if trailing parameters can be deduced. For example, `f<double>(1, 1)` sets `T = double`. The arguments `1` and `1` (of type `int`) are then implicitly converted to `double` because `T` was explicitly specified.

#### Forcing Template Overloads (`f<>`)

We can also use an empty template argument list `f<>(1, 1)` to force the compiler to choose only from function template overloads, ignoring non-template functions:

```cpp
#include <iostream>

template <typename T, typename U>
void f(T i, U j) {
    std::cout << "1" << std::endl;
}

template <typename T>
void f(T i, T j) {
    std::cout << "2" << std::endl;
}

void f(int i, int j) {
    std::cout << "3" << std::endl;
}

int main() {
    f<>(1, 1);
    return 0;
}
```

Output:
```ascii
2
```

Even though `f(int, int)` is an exact non-template match, specifying `<>` disables non-template functions from candidate selection, resolving to `f<int>(1, 1)` (which prints `2`).

---

### Overload Ambiguity: Value vs. Reference Parameters

Consider an example with pass-by-value and pass-by-reference overloads:

```cpp
#include <iostream>

template <typename T>
void f(T& t) {
    std::cout << "1" << std::endl;
}

template <typename T>
void f(T t) {
    std::cout << "2" << std::endl;
}

int main() {
    f(1);    // Calls f(T t) -> prints 2 (1 is an rvalue)
    int x = 1;
    f(x);    // Compilation Error! Ambiguous call
    return 0;
}
```

Calling `f(x)` with an lvalue results in a compilation error (**CE**):
```ascii
main.cpp:16:5: error: call to 'f' is ambiguous
   16 |     f(x);
      |     ^
main.cpp:4:6: note: candidate function [with T = int]
    4 | void f(T& t){
      |      ^
main.cpp:9:6: note: candidate function [with T = int]
    9 | void f(T t){
      |      ^
```

For the lvalue `x`, both `f(T&)` (pass-by-reference) and `f(T)` (pass-by-value) are equally valid matches, creating ambiguity. Note that `f(1)` works cleanly because `1` is an rvalue (prvalue) and cannot bind to a non-const lvalue reference `T&`.

A similar ambiguity occurs between `void f(const T& t)` and `void f(T t)` when called with an lvalue.

---

### Const vs. Non-Const Reference Overloading

When overloading between `const T&` and `T&`, the compiler prefers the overload matching the constness of the argument:

```cpp
#include <iostream>

template <typename T>
void f(const T& t) {
    std::cout << "1" << std::endl;
}

template <typename T>
void f(T& t) {
    std::cout << "2" << std::endl;
}

int main() {
    const int x = 1;
    f(x); // Calls f(const T&) -> prints 1

    int y = 1;
    f(y); // Calls f(T&)       -> prints 2

    return 0;
}
```

Output:
```ascii
1
2
```

- Calling `f(x)` with a `const` variable selects `f(const T&)` because `const` lvalues cannot bind to non-const references (`T&`).
- Calling `f(y)` with a non-const variable selects `f(T&)` because a non-const reference is a more specialized match for a non-const lvalue than a const reference.

---

### Non-Deducible Template Parameters and Default Arguments

#### Non-Deducible Template Parameters

If a template parameter is not used in the function's parameter list, the compiler cannot deduce its type during call-site deduction:

```cpp
#include <iostream>

template <typename T, typename U>
void f(T t) {
    U u{};
    std::cout << u << std::endl;
}

int main() {
    f(0); // Compilation Error! Cannot deduce U
    return 0;
}
```

This produces a compilation error (**CE**):
```ascii
main.cpp:10:5: error: no matching function for call to 'f'
   10 |     f(0);
      |     ^
main.cpp:4:6: note: candidate template ignored: couldn't infer template argument 'U'
    4 | void f(T t){
      |      ^
```

#### Default Template Arguments

We can define a **default template argument** for parameters that cannot be deduced:

```cpp
#include <iostream>

template <typename T, typename U = int>
void f(T t) {
    U u{};
    std::cout << u << std::endl;
}

int main() {
    f(0);              // T is deduced as int, U defaults to int -> prints 0
    f<int, double>(0); // T is int, U is explicitly double        -> prints 0
    return 0;
}
```

Output:
```ascii
0
0
```
