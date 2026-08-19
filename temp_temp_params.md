## Template Template Parameters

A **template template parameter** is a template parameter that is itself a template rather than a concrete type or non-type value.

This feature allows passing uninstantiated class or alias templates (such as `std::vector` or custom container templates) directly as arguments into other templates.

---

### Basic Syntax and Usage

A common use case is designing a generic container adapter or data structure (e.g., `Stack`) where the container template itself is parameterized:

```cpp
#include <iostream>
#include <vector>

template <template <typename> typename Container>
struct Stack {
    Container<int> c;
};

int main() {
    Stack<std::vector> s;
    s.c.push_back(42);
    std::cout << s.c.front() << std::endl;
    return 0;
}
```

Output:
```ascii
42
```

> [!NOTE]
> **`typename` vs. `class` in Template Template Parameter Declarations:**
> Prior to C++17, template template parameters were required to use the `class` keyword (e.g., `template <template <typename> class Container>`). Starting in C++17, the `typename` keyword is also permitted in this position (`template <template <typename> typename Container>`). Both keywords are semantically equivalent.

---

### Non-Type Template Parameters in Template Template Parameters

Template template parameters can also accept templates that take non-type template parameters (such as `std::size_t`):

```cpp
#include <array>
#include <cstddef>
#include <iostream>

template <template <typename, std::size_t> typename Container>
struct Stack {
    Container<int, 5> c;
};

int main() {
    Stack<std::array> s;
    s.c[0] = 42;
    std::cout << s.c[0] << std::endl;
    return 0;
}
```

Output:
```ascii
42
```

---

### Default Template Arguments and Generic Element Types

Template template parameters can be combined with type parameters and default arguments to allow flexible data types and container selections:

```cpp
#include <deque>
#include <iostream>
#include <string>
#include <vector>

template <typename T, template <typename...> typename Container = std::vector>
struct Stack {
    Container<T> c;

    void push(const T& val) {
        c.push_back(val);
    }

    T top() const {
        return c.back();
    }
};

int main() {
    // Uses default container (std::vector)
    Stack<int> s1;
    s1.push(10);
    s1.push(20);
    std::cout << "Stack with std::vector: " << s1.top() << std::endl;

    // Uses explicit container (std::deque)
    Stack<std::string, std::deque> s2;
    s2.push("hello");
    s2.push("world");
    std::cout << "Stack with std::deque: " << s2.top() << std::endl;

    return 0;
}
```

Output:
```ascii
Stack with std::vector: 20
Stack with std::deque: world
```

> [!TIP]
> Standard library containers such as `std::vector` often have additional template parameters with default values (such as custom allocators). Using a variadic template template parameter (`template <typename...> typename Container`) ensures compatibility across standard library implementations and C++ standards.
