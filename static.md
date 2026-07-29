## Static Members (Fields and Methods)

Classes and structs in C++ can have `static` fields and methods. The `static` keyword indicates that a member belongs to the class itself rather than to any specific object instance.

For the compiler, `static` variables are stored in static memory. They are initialized once and exist for the entire duration of the program.

```cpp
#include <iostream>

struct TType {
    static int x; // Declaration of a static member variable
};

// Out-of-line definition required for non-inline static data members
int TType::x = 0;

int main() {
    TType::x = 1; // Access static field via scope resolution operator ::
    std::cout << TType::x << std::endl;
    return 0;
}
```

Static fields are useful for maintaining shared state across all instances of a class, such as object counters or shared configuration settings.

#### Static Member Functions

Static member functions can be called without creating an instance of the class (using `ClassName::FunctionName()`). Static methods do not have access to a `this` pointer and can only directly access other static members of the class.

A classic use case for static methods is the **Singleton** pattern:

```cpp
class TType {
private:
    static TType* obj = nullptr; // In-class initialization of non-const static field

    TType() {}

public:
    static TType& Get() {
        if (obj) return *obj;
        obj = new TType();
        return *obj;
    }
};

int main() {
    return 0;
}
```

Compiling this in pre-C++17 mode results in a **compilation error**:
```ascii
main.cpp:3:23: error: non-const static data member must be initialized out of line
    3 |         static TType* obj = nullptr;
      |                       ^     ~~~~~~~
1 error generated.
```

To resolve this, define the static data member out-of-line outside the class definition:

```cpp
#include <iostream>

class TType {
private:
    static TType* obj; // Declaration

    TType() {}

public:
    static TType& Get() {
        if (obj) return *obj;
        obj = new TType();
        return *obj;
    }
};

// Out-of-line definition and initialization
TType* TType::obj = nullptr;

int main() {
    TType& instance = TType::Get();
    return 0;
}
```

> **C++17 `inline static`**: Starting in C++17, you can use `inline static TType* obj = nullptr;` inside the class body to initialize static data members directly in-class without an out-of-line definition.

#### `const static` and `constexpr` Members

You can initialize `const static` members directly inside the class definition for integral types:
```cpp
struct TType {
    static const int x = 1; // Allowed for const integral types
};
```
However, in pre-C++17 C++, attempting in-class initialization for non-integral types like `double` (`static const double x = 1.0;`) causes a **compilation error**. To initialize floating-point static constants in-class, use `constexpr` (C++11+) or `inline static const` (C++17+):

```cpp
struct TType {
    static constexpr double pi = 3.14159; // Allowed for floating-point types in C++11+
};
```
