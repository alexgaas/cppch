## Variadic Templates

Introduced in C++11, **variadic templates** allow templates (functions, classes, and aliases) to accept an arbitrary number of template arguments of varying types.

---

### Key Concepts and Terminology

- **Template Parameter Pack (`typename... Args`):** A template parameter that accepts zero or more template type arguments.
- **Function Parameter Pack (`const Args&... args`):** A function parameter that accepts zero or more function arguments.
- **Pack Expansion (`args...`):** An expression followed by an ellipsis (`...`) that expands into a comma-separated list of elements.
- **`sizeof...` Operator:** A compile-time operator that returns the number of elements contained in a parameter pack.

---

### Variadic Function Templates: Recursive Unpacking

Before C++17 fold expressions, the standard technique to process a parameter pack was recursive function template instantiation with a non-variadic base case:

```cpp
#include <iostream>

// Base case: terminates recursion when no arguments remain
void print() {}

// Recursive step: processes the first argument ('head') and recurses on the rest ('tail')
template <typename Head, typename... Tail>
void print(const Head& head, const Tail&... tail) {
    std::cout << head << std::endl;
    print(tail...); // Pack expansion: unpacks remaining arguments into a comma-separated list
}

int main() {
    print(1, 2.5, "abc");
    return 0;
}
```

Output:
```ascii
1
2.5
abc
```

---

### The `sizeof...` Operator

The `sizeof...` operator computes the number of elements in a parameter pack at compile time:

```cpp
#include <iostream>

void print() {}

template <typename Head, typename... Tail>
void print(const Head& head, const Tail&... tail) {
    std::cout << head << " (remaining arguments: " << sizeof...(tail) << ")" << std::endl;
    print(tail...);
}

int main() {
    print(1, 2.5, "abc");
    return 0;
}
```

Output:
```ascii
1 (remaining arguments: 2)
2.5 (remaining arguments: 1)
abc (remaining arguments: 0)
```

---

### Modern Simplification: `if constexpr` (C++17)

In C++17, `if constexpr` allows handling the base case directly inside the template function without declaring an empty `print()` overload:

```cpp
#include <iostream>

template <typename Head, typename... Tail>
void print(const Head& head, const Tail&... tail) {
    std::cout << head << std::endl;
    if constexpr (sizeof...(tail) > 0) {
        print(tail...); // Instantiated only if tail is not empty
    }
}

int main() {
    print(1, 2.5, "abc");
    return 0;
}
```

---

### Variadic Class Templates

Variadic parameter packs can also be used in class templates (the foundational mechanism behind `std::tuple`):

```cpp
#include <cstddef>
#include <iostream>

// Primary template declaration
template <typename... Ts>
struct Tuple;

// Base case: empty tuple
template <>
struct Tuple<> {};

// Recursive specialization: stores head and inherits from Tuple<Tail...>
template <typename Head, typename... Tail>
struct Tuple<Head, Tail...> : Tuple<Tail...> {
    Head value;
    Tuple(Head head, Tail... tail) : Tuple<Tail...>(tail...), value(head) {}
};

int main() {
    Tuple<int, double, const char*> t(10, 3.14, "hello");
    std::cout << t.value << std::endl; // Accesses head element: 10
    return 0;
}
```

Output:
```ascii
10
```

> [!NOTE]
> In C++17, **fold expressions** provide a concise, non-recursive syntax to perform binary operations (such as `+`, `*`, `<<`, `,`) across all elements of a parameter pack.
