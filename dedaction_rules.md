## Template Argument Deduction and Deduction Guides

When invoking a function template or instantiating a class template (C++17), the compiler automatically deduces template arguments based on the types of the arguments passed.

---

### Pass-by-Value Deduction (`template <typename T> void f(T x)`)

When function template parameters are passed **by value**, the compiler applies decay rules during argument deduction:
- **Reference qualifiers are discarded:** Passing an `int&` or `const int&` results in `T = int`.
- **Top-level `const` and `volatile` qualifiers are stripped:** The parameter is a local copy, so the `const` qualifier on the caller's variable does not affect `T`.
- **Array and function types decay to pointers.**

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
void f(T x) {
    std::cout << std::boolalpha;
    std::cout << "is_reference: " << std::is_reference_v<T> << std::endl;
    std::cout << "is_const:     " << std::is_const_v<T> << std::endl;
}

int main() {
    int x = 0;
    const int& y = x;

    f(y); // T is deduced as int (reference and const are stripped)
    return 0;
}
```

Output:
```ascii
is_reference: false
is_const:     false
```

---

### Pass-by-Reference Deduction (`template <typename T> void f(T& x)`)

When function template parameters are passed **by reference**:
- **Reference qualifiers are stripped** to determine the type `T`.
- **`const` qualifiers are preserved:** Passing a `const int&` or `const int` binds `T` to `const int`, producing the parameter type `const int&`.

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
void f(T& x) {
    std::cout << std::boolalpha;
    std::cout << "T is_const: " << std::is_const_v<T> << std::endl;
}

int main() {
    int x = 0;
    const int& y = x;

    f(y); // T is deduced as const int; parameter type is const int&
    return 0;
}
```

Output:
```ascii
T is_const: true
```

---

### Passing References to Pass-by-Value Templates

If a function template accepts `T` by value (`void f(T x)`), but you want `f` to modify the original object:

#### 1. Explicit Template Arguments
You can explicitly supply a reference type as the template argument:

```cpp
#include <iostream>

template <typename T>
void f(T x) {
    x = 42;
}

int main() {
    int x = 0;
    f<int&>(x); // T = int&, so parameter 'x' is int&
    std::cout << x << std::endl; // Prints: 42
    return 0;
}
```

#### 2. `std::ref` and `std::cref` (`<functional>`)
In generic contexts (such as `std::bind`, `std::thread`, or standard algorithms), you can wrap variables in `std::reference_wrapper` using `std::ref` or `std::cref`:

```cpp
#include <functional>
#include <iostream>

template <typename T>
void f(T x) {
    x.get() = 42; // Access underlying reference via .get()
}

int main() {
    int x = 0;
    f(std::ref(x)); // Passes std::reference_wrapper<int> by value
    std::cout << x << std::endl; // Prints: 42
    return 0;
}
```

---

### Class Template Argument Deduction (CTAD) and Deduction Guides (C++17)

Prior to C++17, class templates required explicit template arguments (e.g., `std::pair<int, double> p(1, 2.0);`). In C++17, **CTAD** allows class template arguments to be deduced automatically from constructor arguments:

```cpp
std::pair p(1, 2.0); // Deduced as std::pair<int, double>
```

#### User-Defined Deduction Guides

You can define custom **deduction guides** to instruct the compiler how to deduce template arguments for class templates:

```cpp
#include <iostream>
#include <string>
#include <type_traits>

template <typename T>
struct S {
    T val;
    S(T x) : val(x) {}
};

// User-defined deduction guide:
// When initialized with const char*, deduce S<std::string> instead of S<const char*>
S(const char*) -> S<std::string>;

int main() {
    S s("hello"); // Deduced as S<std::string>

    static_assert(std::is_same_v<decltype(s.val), std::string>);
    std::cout << s.val << std::endl;
    return 0;
}
```

Output:
```ascii
hello
```
