## Non-Type Template Parameters

A **non-type template parameter** (NTTP) is a template parameter whose argument is a value rather than a type.

A familiar standard library example is `std::array`, where the array size is specified via a non-type template parameter:

```cpp
#include <array>

std::array<int, 10> a; // '10' is a non-type template parameter
```

In C++, non-type template parameters can be:
- Integral primitive types (`int`, `long long`, `char`, `bool`, `size_t`, etc.)
- Pointers and references (pointing to objects or functions with static storage duration)
- Enumeration types (`enum`, `enum class`)
- *(Since C++20)* Floating-point types and structural class types

---

### Basic Example: Fixed-Size Array

Consider a custom fixed-size `Array` implementation using a size parameter `N`:

```cpp
#include <cstddef>
#include <iostream>

template <typename T, std::size_t N>
struct Array {
    T data[N];
};

int main() {
    Array<int, 10> a1;
    Array<int, 10> a2 = a1; // OK: same type
    return 0;
}
```

However, if we attempt to assign or copy between arrays of different dimensions, the compiler generates a compilation error (**CE**):

```cpp
Array<int, 10> a1;
Array<int, 15> a2 = a1; // Compilation Error!
```

Compiler Output:
```ascii
main.cpp:9:20: error: no viable conversion from 'Array<[...], 10>' to 'Array<[...], 15>'
    9 |     Array<int, 15> a2 = a1;
      |                    ^    ~~
```

**Reason**: `Array<int, 10>` and `Array<int, 15>` are distinct types to the compiler. Because their non-type template arguments differ, they are instantiated as completely separate classes.

---

### Compile-Time Constant Requirement

Arguments passed to non-type template parameters must be **constant expressions** (evaluable at compile time):

```cpp
// OK: 'a' is a compile-time constant
const int a = 10;
Array<int, a> b; 

// Compilation Error! 'a' is initialized at runtime via a function call
int f(int x) { return x; }
const int a = f(10);
Array<int, a> b; // Error: 'a' is not a constant expression
```

---

### Pointer and Reference Parameters

Pointers and references can also serve as non-type template parameters:

```cpp
template <typename T, T* Ptr>
struct PointerWrapper {};

int global_var = 42;
PointerWrapper<int, &global_var> p; // OK: &global_var has static storage duration
```

#### Example: Matrix Dimension Enforcing

Non-type template parameters can enforce matrix multiplication dimension matching at compile time:

```cpp
#include <cstddef>

template <typename T, std::size_t Rows, std::size_t Cols>
struct Matrix {};

// Matrix multiplication: (N x M) * (M x K) -> (N x K)
template <typename T, std::size_t N, std::size_t M, std::size_t K>
Matrix<T, N, K> operator*(const Matrix<T, N, M>& a, const Matrix<T, M, K>& b) {
    return Matrix<T, N, K>{};
}

int main() {
    Matrix<double, 2, 3> A;
    Matrix<double, 3, 4> B;
    Matrix<double, 2, 4> C = A * B; // OK: inner dimensions match (3 == 3)

    // Matrix<double, 3, 5> Invalid = A * B; // Compilation Error: mismatched dimensions
}
```

---

### Compile-Time Computation Metaprogramming

Using non-type template parameters and template specialization, calculations can be performed at compile time.

#### Example 1: Compile-Time Fibonacci Numbers

Without base cases, template instantiation recursively expands infinitely until the compiler reaches its maximum recursion limit:

```cpp
#include <iostream>

template <int N>
struct Fib {
    static const long long val = Fib<N - 1>::val + Fib<N - 2>::val;
};

int main() {
    std::cout << Fib<30>::val << std::endl; // Compilation Error! Exceeds template instantiation depth
    return 0;
}
```

This fails with a compilation error (**CE**):
```ascii
error: recursive template instantiation exceeded maximum depth of 1024
```

Adding base-case specializations for `0` and `1` terminates recursion at compile time:

```cpp
#include <iostream>

template <int N>
struct Fib {
    static const long long val = Fib<N - 1>::val + Fib<N - 2>::val;
};

// Base case for N = 0
template <>
struct Fib<0> {
    static const long long val = 0;
};

// Base case for N = 1
template <>
struct Fib<1> {
    static const long long val = 1;
};

int main() {
    std::cout << Fib<30>::val << std::endl; // Output: 832040 (computed at compile time)
    return 0;
}
```

Output:
```ascii
832040
```

#### Example 2: Compile-Time Primality Test

We can also implement a compile-time primality test using recursive non-type template parameters:

```cpp
#include <cstddef>
#include <iostream>

template <std::size_t N, std::size_t K>
struct IsPrimeHelper {
    static const bool value = (N % K == 0) ? false : IsPrimeHelper<N, K - 1>::value;
};

// Base case: K reaches 1 without finding any divisors
template <std::size_t N>
struct IsPrimeHelper<N, 1> {
    static const bool value = true;
};

template <std::size_t N>
struct IsPrime {
    static const bool value = IsPrimeHelper<N, N - 1>::value;
};

int main() {
    std::cout << std::boolalpha;
    std::cout << "Is 7 prime? " << IsPrime<7>::value << std::endl;  // true
    std::cout << "Is 10 prime? " << IsPrime<10>::value << std::endl; // false
    return 0;
}
```

Output:
```ascii
Is 7 prime? true
Is 10 prime? false
```

> [!NOTE]
> Recursive template metaprogramming works up to the compiler's maximum template instantiation depth (typically 1024 by default). In modern C++ (C++14 onwards), `constexpr` functions are often preferred for compile-time calculations.
