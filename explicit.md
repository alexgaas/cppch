## User-Defined Conversion Operators and the `explicit` Keyword

C++ allows overloading conversion operators for custom classes. Let's illustrate with an example: a `UserId` class constructed from an `int` that can also be converted back to an `int` on demand:

```cpp
#include <iostream>

class UserId {
private:
    int id_ = 0;

public:
    // Converting constructor: int -> UserId
    UserId(int id) : id_(id) {}
    
    // User-defined conversion operator: UserId -> int
    operator int() const {
        return id_;
    }
};

int main() {
    UserId id = 1;
    // Implicit conversion to int applied automatically!
    std::cout << id + 10 << std::endl;
    return 0;
}
```
Output: `11`

#### Ambiguity with Implicit Conversions

What happens if we define a non-member binary `operator+` for `UserId`?

```cpp
#include <iostream>

class UserId {
    friend int operator+(UserId a, UserId b);
private:
    int id_ = 0;

public:
    UserId(int id) : id_(id) {}
    
    operator int() const {
        return id_;
    }
};

int operator+(UserId a, UserId b) {
    return a.id_ + b.id_; 
}

int main() {
    UserId id = 1;
    // std::cout << id + 10 << std::endl; // Error!
    return 0;
}
```
Compiling `id + 10` results in a **compilation error** due to ambiguity:
```ascii
main.cpp:25:21: error: use of overloaded operator '+' is ambiguous (with operand types 'UserId' and 'int')
```
The compiler cannot decide between two viable user-defined conversion sequences:
1. Convert `id` to `int` via `operator int()`, then call built-in `int + int`.
2. Convert `10` to `UserId` via `UserId(int)`, then call `operator+(UserId, UserId)`.

#### The `explicit` Keyword

Even without ambiguity, implicit conversions can be dangerous because they break strong typing. For example, if we defined another class `ViewId` with an implicit conversion to `int`, the compiler would silently permit `UserId + ViewId` by converting both to `int`, even though adding a user ID to a view ID is logically invalid.

To prevent accidental implicit conversions, use the `explicit` keyword:

```cpp
#include <iostream>

class UserId {
private:
    int id_ = 0;

public:
    UserId(int id) : id_(id) {}
    
    // Disallow implicit conversion to int
    explicit operator int() const {
        return id_;
    }
};

int main() {
    UserId id = 1;
    // std::cout << id + 10 << std::endl; // Error!
    return 0;
}
```
Compiling `id + 10` fails with a **compilation error**:
```ascii
main.cpp:20:21: error: invalid operands to binary expression ('UserId' and 'int')
```

To convert an `explicit`-marked object, perform an explicit conversion using `static_cast`:

```cpp
#include <iostream>

class UserId {
private:
    int id_ = 0;

public:
    UserId(int id) : id_(id) {}
    
    explicit operator int() const {
        return id_;
    }
};

int main() {
    UserId id = 1;
    std::cout << static_cast<int>(id) + 10 << std::endl; // OK!
    return 0;
}
```
Output: `11`

#### Explicit Constructors

We can also mark single-argument constructors (converting constructors) as `explicit` to prevent implicit conversions during copy initialization:

```cpp
#include <iostream>

class UserId {
private:
    int id_ = 0;

public:
    explicit UserId(int id) : id_(id) {}
    
    explicit operator int() const {
        return id_;
    }
};

int main() {
    // UserId id = 1; // Error: no viable conversion from 'int' to 'UserId'
    
    // Explicit static_cast:
    UserId id1 = static_cast<UserId>(1);
    
    // Direct initialization:
    UserId id2(1);
    UserId id3{1};

    std::cout << static_cast<int>(id1) + 10 << std::endl;
    return 0;
}
```

> **Best Practice**: Single-argument constructors should almost always be declared `explicit` to avoid accidental, unintended implicit type conversions.

#### User-Defined Literals

C++ allows overloading literal suffixes so custom objects can be constructed directly using custom literal syntax (e.g., `10_uid`):

```cpp
#include <iostream>

class UserId {
    friend int operator+(UserId a, UserId b);
private:
    int id_ = 0;

public:
    explicit UserId(int id) : id_(id) {}
    
    explicit operator int() const {
        return id_;
    }
};

// User-defined literal operator for _uid suffix
UserId operator""_uid(unsigned long long x) {
    return UserId(static_cast<int>(x));
}

int operator+(UserId a, UserId b) {
    return a.id_ + b.id_;
}

int main() {
    UserId id(1);
    std::cout << id + 10_uid << std::endl; // Equivalent to id + UserId(10)
    return 0;
}
```
Output: `11`

Note that integer user-defined literal operators must accept `unsigned long long` as their parameter type.

#### Contextual Conversion

Contextual conversion occurs when an expression is evaluated in a boolean context (such as inside `if`, `while`, or `for` conditions, or with logical operators `&&`, `||`, `!`).

An `explicit operator bool()` is permitted in contextual boolean conversions without requiring an explicit `static_cast`:

```cpp
#include <iostream>

struct T {
    explicit operator bool() const {
        return true;
    }
};

int main() {
    T x;
    if (x) { // Contextual conversion to bool - allowed even with 'explicit'!
        std::cout << "Condition evaluated to true!" << std::endl;
    }
    return 0;
}
```
Output: `Condition evaluated to true!`
