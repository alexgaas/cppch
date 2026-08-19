## Basic Type Traits

**Type traits** are compile-time template utilities that allow querying, inspecting, and transforming the properties of types.

In template metaprogramming, a **metafunction** is a class template evaluated at compile time that accepts types (or values) as input and returns a result as either:
1. **A compile-time value** via a member constant (typically `::value`).
2. **A transformed type** via a member type alias (typically `::type`).

---

### Querying Type Properties: `std::is_same`

Introduced in C++11 under the `<type_traits>` header, `std::is_same` checks whether two types are identical at compile time:

```cpp
#include <iostream>
#include <type_traits>

template <typename T, typename U>
void compareTypes(T, U) {
    std::cout << std::boolalpha << std::is_same<T, U>::value << std::endl;
}

int main() {
    compareTypes(1, 1);       // T = int, U = int -> true
    compareTypes(1, "test");  // T = int, U = const char* -> false
    return 0;
}
```

Output:
```ascii
true
false
```

---

### Implementing a Custom Type Trait: `is_same`

A value-yielding type trait can be implemented using a primary template and partial template specialization:

```cpp
#include <iostream>

// Primary template: types T and U differ
template <typename T, typename U>
struct is_same {
    static constexpr bool value = false;
};

// Partial specialization: both types are identical (T and T)
template <typename T>
struct is_same<T, T> {
    static constexpr bool value = true;
};

int main() {
    std::cout << std::boolalpha;
    std::cout << is_same<int, int>::value << std::endl;       // true
    std::cout << is_same<int, double>::value << std::endl;    // false
    std::cout << is_same<int, const int>::value << std::endl; // false (const int != int)
    return 0;
}
```

Output:
```ascii
true
false
false
```

---

### Type-Transforming Traits: `remove_const` and `remove_reference`

Type traits can also modify types (e.g., stripping `const`, references, or pointers) by defining a member type alias `using type = ...`.

#### Example 1: `remove_const`

```cpp
#include <iostream>
#include <type_traits>

// Primary template: returns T unchanged
template <typename T>
struct remove_const {
    using type = T;
};

// Partial specialization for const types: strips const qualifier
template <typename T>
struct remove_const<const T> {
    using type = T;
};

// C++14 helper alias template
template <typename T>
using remove_const_t = typename remove_const<T>::type;

int main() {
    remove_const<const int>::type a = 10; // 'a' is of type int
    a = 20; // OK: non-const

    std::cout << std::boolalpha;
    std::cout << std::is_same<remove_const_t<const int>, int>::value << std::endl; // true
    return 0;
}
```

Output:
```ascii
true
```

#### Example 2: `remove_reference`

Similarly, references can be stripped using partial specializations for lvalue (`T&`) and rvalue (`T&&`) references:

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
struct remove_reference {
    using type = T;
};

template <typename T>
struct remove_reference<T&> {
    using type = T;
};

template <typename T>
struct remove_reference<T&&> {
    using type = T;
};

template <typename T>
using remove_reference_t = typename remove_reference<T>::type;

int main() {
    std::cout << std::boolalpha;
    std::cout << std::is_same<remove_reference_t<int&>, int>::value << std::endl;  // true
    std::cout << std::is_same<remove_reference_t<int&&>, int>::value << std::endl; // true
    return 0;
}
```

Output:
```ascii
true
true
```

---

### Helper Utilities in Modern C++ (`_t` and `_v`)

To reduce verbosity when accessing `::type` and `::value`:

- **C++14 (`_t` alias templates)**: Standard library type-transforming traits provide helper aliases ending in `_t`:
  ```cpp
  // Instead of: typename std::remove_const<T>::type
  std::remove_const_t<T>
  ```
- **C++17 (`_v` variable templates)**: Standard library value-yielding traits provide variable template helpers ending in `_v`:
  ```cpp
  // Custom variable template helper:
  template <typename T, typename U>
  inline constexpr bool is_same_v = is_same<T, U>::value;

  // Standard library:
  // Instead of: std::is_same<T, U>::value
  std::is_same_v<T, U>
  ```
