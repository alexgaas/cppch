## Operator Overloading

C++ allows operators (such as `+`, `-`, `==`, etc.) to be overloaded for user-defined classes. The first and most fundamental operator to understand when overloading is the assignment operator (`=`).

#### Assignment Operator

The assignment operator evaluates right-to-left in chained expressions (e.g., `x = y = z` is evaluated as `x = (y = z)`). The standard signature for a copy assignment operator overload is:
```cpp
Type& operator=(const Type& t);
```

A reference to the target object (`Type&`) is returned because:
- We want to support chained assignment expressions like:
  ```cpp
  (x = y) = z;
  ```
- Returning a reference avoids creating temporary copies and allows operations to apply directly to the object.

Let's examine a naive implementation of a class managing a dynamically allocated array:
```cpp
#include <iostream>
#include <cstring>
#include <initializer_list>
#include <algorithm>

class TType {
public:
    TType() = default;

    TType(std::initializer_list<int> list) {
        sz = list.size();
        t = new int[sz];
        std::copy(list.begin(), list.end(), t);
    }

    TType(const TType& type) {
        sz = type.sz;
        t = new int[sz];
        std::copy(type.t, type.t + sz, t);
    }

    ~TType() {
        delete[] t;
    }

    TType& operator=(const TType& t) {
        delete[] this->t;
        sz = t.sz;
        this->t = new int[sz];
        std::copy(t.t, t.t + sz, this->t);
        return *this;
    }

    void print() const {
        for (size_t i = 0; i < sz; i++) {
            std::cout << t[i] << " ";
        }
        std::cout << std::endl;
    }

private:
    int* t = nullptr;
    size_t sz = 0;
};

int main() {
    TType t = {1, 2, 3};
    t.print();
    t = t; // Self-assignment!
    t.print();
    return 0;
}
```

You can already see an issue with this naive implementation: if a user assigns an object to itself (`t = t`), deleting `this->t` before copying deletes the source data, causing memory corruption or unexpected behavior.

#### Self-Assignment Check

To fix this, we can add a check to verify whether the object is being assigned to itself:
```cpp
if (&t == this) {
    return *this;
}
```
If the address of the passed argument matches `this`, the object returns `*this` immediately without modifying anything:
```cpp
TType& operator=(const TType& t) {
    if (&t == this) return *this;
    delete[] this->t;
    sz = t.sz;
    this->t = new int[sz];
    std::copy(t.t, t.t + sz, this->t);
    return *this;
}
```
Output:
```ascii
1 2 3
1 2 3
```

However, this manual assignment operator still has drawbacks:
1. It duplicates memory allocation and copy logic from the copy constructor and destructor (violating the **DRY** / "Don't Repeat Yourself" principle).
2. It is not exception-safe: if `new int[sz]` throws an exception, `this->t` has already been deleted, leaving the object in an invalid state.

> **Note**: Never explicitly call the destructor (e.g., `this->~TType()`) inside `operator=`. The compiler will call the destructor automatically when the object goes out of scope; calling it manually leads to double-free bugs and **undefined behavior** (UB).

#### Copy-and-Swap Idiom

An idiomatic and exception-safe way to implement the assignment operator is the **copy-and-swap idiom**.

First, define a member `swap` function:
```cpp
void swap(TType& tt) noexcept {
    std::swap(t, tt.t);
    std::swap(sz, tt.sz);
}
```

Then, implement `operator=` using copy-and-swap:
```cpp
TType& operator=(const TType& t) {
    if (&t == this) return *this;
    TType temp = t; // Create a temporary copy
    swap(temp);     // Swap contents with temporary
    return *this;   // Old resource deleted automatically when 'temp' goes out of scope
}
```
Alternatively, by passing the parameter by value, the copy is created automatically:
```cpp
TType& operator=(TType temp) {
    swap(temp);
    return *this;
}
```

Here is the complete updated implementation:
```cpp
#include <iostream>
#include <initializer_list>
#include <algorithm>
#include <utility>

class TType {
public:
    TType() = default;

    TType(std::initializer_list<int> list) {
        sz = list.size();
        t = new int[sz];
        std::copy(list.begin(), list.end(), t);
    }

    TType(const TType& type) {
        sz = type.sz;
        t = new int[sz];
        std::copy(type.t, type.t + sz, t);
    }

    ~TType() {
        delete[] t;
    }

    void swap(TType& tt) noexcept {
        std::swap(t, tt.t);
        std::swap(sz, tt.sz);
    }

    TType& operator=(TType temp) {
        swap(temp);
        return *this;
    }

    void print() const {
        for (size_t i = 0; i < sz; i++) {
            std::cout << t[i] << " ";
        }
        std::cout << std::endl;
    }

private:
    int* t = nullptr;
    size_t sz = 0;
};

int main() {
    TType t = {1, 2, 3};
    t.print();
    t = t;
    t.print();
    return 0;
}
```

By default, the compiler automatically generates a copy assignment operator if one is not user-defined, provided all class fields are copy-assignable. If a class contains `const` or reference members, the compiler cannot generate a default copy assignment operator, resulting in a **compilation error** if copy assignment is attempted.

#### The Rule of Three

The **Rule of Three** states that if a class requires an explicit definition for any of the following three special member functions, it almost certainly requires all three:
1. **Destructor**
2. **Copy Constructor**
3. **Copy Assignment Operator**
