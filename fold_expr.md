## Fold Expressions

Introduced in C++17, **fold expressions** simplify operations over variadic template parameter packs. They allow reducing (folding) a parameter pack over a binary operator directly, eliminating the need for recursive template instantiations and base-case overloads.

---

### The Four Forms of Fold Expressions

Let `pack` be an unexpanded parameter pack, `init` be an initial non-pack expression, and `op` be any valid binary operator (such as `+`, `*`, `<<`, `&&`, `,`):

| Fold Form | Syntax | Expansion |
| :--- | :--- | :--- |
| **Unary Right Fold** | `(pack op ...)` | `(E1 op (... op (E_N-1 op E_N)))` |
| **Unary Left Fold** | `(... op pack)` | `(((E1 op E2) op ...) op E_N)` |
| **Binary Right Fold** | `(pack op ... op init)` | `(E1 op (... op (E_N op init)))` |
| **Binary Left Fold** | `(init op ... op pack)` | `(((init op E1) op E2) op ... op E_N)` |

---

### Basic Example: Summing Values

```cpp
#include <iostream>

// Unary left fold: (((1 + 2) + 3) + 4)
template <typename... Args>
auto sum(Args... args) {
    return (... + args);
}

// Binary left fold with 0 as init: (((0 + 1) + 2) + ...)
// Safely supports empty parameter packs: sumWithInit() -> 0
template <typename... Args>
auto sumWithInit(Args... args) {
    return (0 + ... + args);
}

int main() {
    std::cout << "Sum: " << sum(1, 2, 3, 4) << std::endl;
    std::cout << "Sum with init (empty): " << sumWithInit() << std::endl;
    return 0;
}
```

Output:
```ascii
Sum: 10
Sum with init (empty): 0
```

---

### Empty Parameter Packs

> [!IMPORTANT]
> A **unary fold** over an empty parameter pack is ill-formed (causes a compilation error) for almost all operators.

The only exceptions allowed for unary folds over empty packs are:
- `&&` $\rightarrow$ evaluates to `true`
- `||` $\rightarrow$ evaluates to `false`
- `,` (comma) $\rightarrow$ evaluates to `void()`

For all other operators (such as `+`, `*`, `<<`), use a **binary fold** with an explicit initial value to support empty packs.

---

### Common Practical Patterns

#### 1. Stream Output Fold
Using a binary left fold where `std::cout` is the initial expression:

```cpp
#include <iostream>

template <typename... Args>
void printCompact(const Args&... args) {
    (std::cout << ... << args) << std::endl; // (((std::cout << arg1) << arg2) << ...)
}

int main() {
    printCompact(1, 2, "abc"); // Prints: 12abc
    return 0;
}
```

Output:
```ascii
12abc
```

#### 2. Comma Operator Fold with Delimiters
Using a unary left fold over the comma operator `,` allows executing an expression for each element in sequence:

```cpp
#include <iostream>

template <typename... Args>
void printWithSpaces(const Args&... args) {
    ((std::cout << args << " "), ...) << std::endl;
}

int main() {
    printWithSpaces(1, 2.5, "abc");
    return 0;
}
```

Output:
```ascii
1 2.5 abc 
```

#### 3. Logical Predicate Checking (`all` / `any`)

```cpp
#include <iostream>

template <typename... Args>
bool allTrue(Args... args) {
    return (... && args); // Unary left fold: short-circuits on first false
}

int main() {
    std::cout << std::boolalpha;
    std::cout << allTrue(true, 1 == 1, 5 > 2) << std::endl; // true
    std::cout << allTrue(true, false, true) << std::endl;    // false
    std::cout << allTrue() << std::endl;                     // true (empty pack rule)
    return 0;
}
```

Output:
```ascii
true
false
true
```

#### 4. Batch Operations on Containers

```cpp
#include <iostream>
#include <vector>
#include <utility>

template <typename T, typename... Args>
void pushAll(std::vector<T>& vec, Args&&... args) {
    (vec.push_back(std::forward<Args>(args)), ...);
}

int main() {
    std::vector<int> v;
    pushAll(v, 10, 20, 30);
    std::cout << "Vector size: " << v.size() << std::endl;
    return 0;
}
```

Output:
```ascii
Vector size: 3
```
