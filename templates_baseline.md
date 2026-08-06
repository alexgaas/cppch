## Template Basics

### Function Templates

A function template defines a family of functions parameterized by one or more types or non-type parameters. Consider a simple example:

```cpp
#include <iostream>

template <typename T>
T max(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    std::cout << max(1, 2) << std::endl;       // Instantiates max<int>(int, int)
    std::cout << max(1.5, 2.5) << std::endl;   // Instantiates max<double>(double, double)
    return 0;
}
```

Output:
```ascii
2
2.5
```

### Two-Phase Name Lookup and Instantiation

Compilers process template code in two distinct phases:

1. **Phase 1 (Template Definition Time)**: The compiler checks syntax and validates non-dependent names (names that do not depend on template parameters).
2. **Phase 2 (Template Instantiation Time)**: When a template is called with concrete type arguments, the compiler substitutes those types and checks whether the resulting code is valid.

If type substitution produces invalid operations (e.g., invoking a missing operator or copy constructor), the compiler generates a compilation error (**CE**):

```cpp
#include <iostream>

template <typename T>
T m(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    m(std::cout, std::cout); // Compilation Error!
    return 0;
}
```

Compiler Error Output:
```ascii
error: call to deleted constructor of 'std::ostream'
```

`std::ostream` does not support copy construction or operator `>`, causing Phase 2 instantiation to fail.

### Function Template Instantiation in Memory

Each unique set of template arguments generates a separate function definition in binary code. We can inspect the function pointers for different instantiations to confirm this:

```cpp
#include <iostream>

template <typename T>
T m(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    int (*i)(int, int) = &m;
    double (*j)(double, double) = &m;

    std::cout << (void*)i << std::endl;
    std::cout << (void*)j << std::endl;
    return 0;
}
```

Output:
```ascii
0x100ba46cc
0x100ba4790
```

The pointers `i` and `j` point to two distinct function definitions in memory. The compiler generates specialized machine code for every type substitution used in the program.

### Class Templates

Classes, structs, and unions can also be parameterized by type arguments:

```cpp
template <typename T>
struct S {
    T value;
};

int main() {
    S<int> s{42};
    return 0;
}
```

The template parameter `T` is accessible throughout the scope of the class definition.

### Alias Templates (C++11)

Using the `using` syntax, we can create template aliases for existing types:

```cpp
#include <map>
#include <string>

template <typename K, typename V>
using Map = std::map<K, V>;

int main() {
    Map<int, std::string> m;
    m[1] = "one";
    return 0;
}
```

### Variable Templates (C++14)

C++14 introduced **variable templates**, allowing constants or static values to be parameterized by type:

```cpp
#include <iostream>

template <typename T>
constexpr T pi = T(3.1415926535897932385L);

int main() {
    std::cout << pi<double> << std::endl;
    std::cout << pi<float> << std::endl;
    return 0;
}
```

### Template Scoping Rules

Templates **cannot** be defined inside block scope (i.e., inside a function body). All template declarations must appear at namespace scope or class scope.

### Out-of-Line Member Function Templates

When defining a member function template outside of its class template, both the class template parameter list and the member function template parameter list must be specified:

```cpp
#include <iostream>

template <typename T>
struct S {
    template <typename U>
    void f(U a);
};

template <typename T>
template <typename U>
void S<T>::f(U a) {
    std::cout << "Member function called with argument: " << a << std::endl;
}

int main() {
    S<int> s;
    s.f(42); // Calls member function template f<int>
    return 0;
}
```

### Template Argument Deduction

When instantiating a class template, template arguments must generally be specified explicitly (e.g., `S<int> s;`). For function templates, the compiler can automatically deduce template arguments from the types of the arguments passed:

```cpp
S<int> s; // Explicit template argument for class template S
s.f(42);   // Template argument U is automatically deduced as int
```

### `typename` vs. `class` in Template Parameters

In template parameter declarations (`template <typename T>` vs. `template <class T>`), the keywords `typename` and `class` are completely interchangeable. `typename` is widely preferred in modern C++ code to clarify that the parameter accepts any type (including primitives, pointers, and user-defined types), not just classes.
