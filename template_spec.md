## Template Specialization

Template specialization allows defining a custom implementation of a class, struct, or function template for specific template argument types.

While a primary template defines the general implementation for any type `T`, a specialization provides an alternative definition optimized or required for a specific type (e.g., `int` or a custom user-defined type).

---

### Explicit (Full) Specialization

In explicit (or full) specialization, all template parameters are fully specified using `template<>`.

```cpp
#include <iostream>

template <typename T>
struct S {
    T x;
};

// Full specialization for T = int
template <>
struct S<int> {
    int y = 0;
};

int main() {
    S<double> s1; // Uses primary template: has member 'x'
    s1.x = 3.14;

    S<int> s2;    // Uses explicit specialization: has member 'y'
    s2.y = 42;

    return 0;
}
```

From the compiler's perspective, `S<int>` and `S<double>` are completely distinct types with independent definitions. Consequently, there is no implicit conversion between different instantiations of the same template:

```cpp
S<double> s1;
S<float> ss = s1; // Compilation Error: no viable conversion from 'S<double>' to 'S<float>'
```

---

### Primary Template Declarations and Real-World Usage (`std::hash`)

Template specialization allows customizing behavior for specific types. A classic real-world example is `std::hash`.

Before defining an explicit specialization, a **primary template declaration** must be present:

```cpp
#include <iostream>

// Primary template declaration (undefined)
template <typename T>
struct Hash;

// Explicit specialization for int
template <>
struct Hash<int> {
    size_t operator()(int x) const {
        return static_cast<size_t>(x) * x;
    }
};

// Custom user-defined type
struct my_type {};

// Explicit specialization for my_type
template <>
struct Hash<my_type> {
    size_t operator()(my_type) const {
        return 0;
    }
};

int main() {
    Hash<int> h;
    Hash<my_type> m;
    
    // Hash<double> d; // Compilation Error! Primary template is undefined for double
    return 0;
}
```

#### Requirement for Primary Template

Attempting to specialize a template without declaring the primary template first results in a compilation error (**CE**):

```cpp
#include <iostream>

/* Missing primary template declaration:
template <typename T>
struct Hash;
*/

template <>
struct Hash<int> { // Compilation Error: explicit specialization of undeclared template
    size_t operator()(int x) { return x * x; }
};
```

Compiler Output:
```ascii
main.cpp:9:8: error: explicit specialization of undeclared template struct 'Hash'
    9 | struct Hash<int> {
      |        ^   ~~~~~
```

---

### Function Template Specialization vs. Overloading

Function templates can also be explicitly specialized, but function template specializations interact with overload resolution in specific ways.

> [!IMPORTANT]
> Function template specializations **do not participate in overload resolution**. Overload resolution selects only among primary templates and non-template functions.

#### Example 1: Non-Template Functions vs. Function Template Specializations

```cpp
#include <iostream>

template <typename T>
void f(T x) {
    std::cout << "1" << std::endl;
}

// Specialization of f(T) for T = int
template <>
void f(int x) {
    std::cout << "2" << std::endl;
}

// Non-template function
void f(int x) {
    std::cout << "3" << std::endl;
}

int main() {
    f(0); // Calls non-template f(int) -> prints 3
    return 0;
}
```

Output:
```ascii
3
```

**Explanation**: Overload resolution compares the primary template `f(T)` (#1) with the non-template function `f(int)` (#3). Because non-template functions are preferred over template functions for exact matches, `f(int)` (#3) is selected. The specialization `template<> void f(int)` (#2) is ignored during overload resolution.

#### Example 2: Target Selection of Specialization

```cpp
#include <iostream>

template <typename T, typename U>
void f(T x, U y) {
    std::cout << "1" << std::endl;
}

template <typename T>
void f(T x, T y) {
    std::cout << "2" << std::endl;
}

// Specialization for (int, int)
template <>
void f(int x, int y) {
    std::cout << "3" << std::endl;
}

int main() {
    f(0, 0);            // Output: 3
    f<int, int>(0, 0);  // Output: 1
    return 0;
}
```

Output:
```ascii
3
1
```

**Explanation**:
- When defining `template<> void f(int, int)` (#3), the compiler determines which primary template it specializes by finding the most specialized match among existing primary templates. Here, `f(T, T)` (#2) is more specialized than `f(T, U)` (#1), so #3 specializes #2.
- Calling `f(0, 0)` causes overload resolution to select `f(T, T)` (#2). Since `f(T, T)` has an explicit specialization for `(int, int)`, execution dispatches to #3 (printing `3`).
- Calling `f<int, int>(0, 0)` explicitly selects `f(T, U)` (#1), which has no specialization, so it prints `1`.

---

### Partial Specialization

Partial specialization allows specializing a template for a subset of template arguments or for specific type patterns (e.g., pointers, references, or matching pairs of types).

> [!NOTE]
> In C++, partial specialization is supported **only for class/struct templates**, not for function templates.

#### Example 1: Matching Type Parameters

```cpp
#include <iostream>

template <typename T, typename U>
struct S {
    void print() { std::cout << "Primary template" << std::endl; }
};

// Partial specialization when both type parameters are identical
template <typename T>
struct S<T, T> {
    void print() { std::cout << "Partial specialization (T, T)" << std::endl; }
};

int main() {
    S<int, double> s1;
    s1.print(); // Prints: Primary template

    S<int, int> s2;
    s2.print(); // Prints: Partial specialization (T, T)

    return 0;
}
```

#### Example 2: Reference Types

```cpp
#include <iostream>

template <typename T, typename U>
struct S {
    void f() { std::cout << "1" << std::endl; }
};

// Partial specialization for reference types
template <typename T, typename U>
struct S<T&, U&> {
    void f() { std::cout << "2" << std::endl; }
};

int main() {
    S<int&, int&> s;
    s.f(); // Output: 2
    return 0;
}
```

Output:
```ascii
2
```
