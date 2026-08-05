## Derived Class Casting

### Upcasting (Derived to Base)

A key feature of public inheritance is the ability to treat a derived class object as a base class object. Converting a derived class reference or pointer to a base class reference or pointer is called **upcasting**, and it happens implicitly.

#### Upcasting References

```cpp
#include <iostream>

struct TBase {
    int a = 0;
};

struct TDerived : TBase {
    int a = 1; // Shadows TBase::a
};

void f(TBase& b) {
    std::cout << b.a << std::endl;
}

int main() {
    TDerived td;
    f(td); // Implicit upcast from TDerived& to TBase&
    return 0;
}
```

Output:
```ascii
0
```

`f` receives a reference to the `TBase` subobject inside `td`, so `b.a` accesses `TBase::a`.

#### Upcasting Pointers

```cpp
#include <iostream>

struct TBase {
    int a = 0;
};

struct TDerived : TBase {
    int a = 1;
};

void f(TBase* b) {
    std::cout << b->a << std::endl;
}

int main() {
    TDerived td;
    f(&td); // Implicit upcast from TDerived* to TBase*
    return 0;
}
```

#### Upcasting by Value and Object Slicing

Passing a derived object by value to a function accepting a base class object copies only the base portion of the derived object. This phenomenon is known as **object slicing**:

```cpp
#include <iostream>

struct TBase {
    int a = 0;
    
    TBase() = default;
    TBase(const TBase&) {
        std::cout << "Base copy constructor" << std::endl;
    }
};

struct TDerived : TBase {
    int a = 1;
    TDerived() = default;
};

void f(TBase b) {
    std::cout << b.a << std::endl;
}

int main() {
    TDerived td;
    f(td); // Slices td down to a TBase copy
    return 0;
}
```

Output:
```ascii
Base copy constructor
0
```

### Memory Layout and Object Sizes

In memory, a derived object is laid out with its base subobject at the beginning, followed by derived class members: `[TBase subobject][TDerived members]`.

```cpp
#include <iostream>

struct TBase {
    int a = 0; // 4 bytes
};

struct TDerived : TBase {
    int a = 1; // 4 bytes (shadows TBase::a)
    TDerived() = default;
};

int main() {
    TDerived td;
    TBase& tb = td;
    std::cout << sizeof(td) << " " << sizeof(tb) << std::endl;
    return 0;
}
```

Output:
```ascii
8 4
```

`sizeof(td)` is 8 bytes because `TDerived` contains both `TBase::a` (4 bytes) and `TDerived::a` (4 bytes).

### Empty Base Optimization (EBO)

In C++, an empty struct or class must have a size of at least 1 byte so that distinct objects have unique addresses. However, when an empty class is used as a base class, the compiler optimizes away its storage space so it takes 0 bytes inside the derived object:

```cpp
#include <iostream>

struct TZero {}; // Size is 1 byte standalone

struct TBase : TZero {
    int a = 0;   // 4 bytes
};

struct TDerived : TBase {
    int a = 1;   // 4 bytes
};

int main() {
    TDerived td;
    TBase& tb = td;
    std::cout << sizeof(td) << " " << sizeof(tb) << std::endl;
    return 0;
}
```

Output:
```ascii
8 4
```

Thanks to Empty Base Optimization (EBO), `TZero` adds no extra padding bytes to `TBase` or `TDerived`.

Comparing addresses shows that the `TBase` subobject starts at offset 0 of `TDerived`:

```cpp
#include <iostream>

struct TBase {
    int a = 0;
};

struct TDerived : TBase {
    int a = 1;
};

int main() {
    TDerived td;
    TBase& tb = td;
    std::cout << &tb.a << std::endl; // Address of TBase::a
    std::cout << &td.a << std::endl; // Address of TDerived::a
    return 0;
}
```

Output (example addresses 4 bytes apart):
```ascii
0x16b47adc0
0x16b47adc4
```

### Downcasting (Base to Derived)

#### Implicit Downcasting is Prohibited

Implicitly converting a base class object or reference to a derived class type is prohibited because a base object does not contain derived class members:

```cpp
#include <iostream>

struct TBase {
    int a = 0;
};

struct TDerived : TBase {
    int a = 1;
};

int main() {
    TDerived d;
    TBase& b = d;
    // TDerived& dd = b; // Error: cannot bind lvalue reference of type TDerived to TBase
    return 0;
}
```

This will produce a compilation error (**CE**):
`error: non-const lvalue reference to type 'TDerived' cannot bind to a value of unrelated type 'TBase'`.

Similarly, trying to copy-construct a `TDerived` from a `TBase` without an explicit constructor fails:

```cpp
TDerived dd = b; // Error: no viable conversion from 'TBase' to 'TDerived'
```

If we define an explicit constructor `TDerived(const TBase&)`, conversion becomes possible:

```cpp
struct TDerived : TBase {
    int a = 1;
    TDerived() = default;
    TDerived(const TBase& b) : TBase(b) {}
};
```

#### Explicit Downcasting with `static_cast`

If we know that a `TBase&` actually refers to a `TDerived` object, we can downcast explicitly using `static_cast`:

```cpp
#include <iostream>

struct TBase {
    int a = 0;
};

struct TDerived : TBase {
    int a = 1;
};

int main() {
    TDerived d;
    TBase& b = d;
    TDerived& dd = static_cast<TDerived&>(b); // Explicit downcast
    std::cout << dd.a << std::endl; // Prints 1
    return 0;
}
```

`static_cast` is checked at compile time. It can only be used between types in the same inheritance hierarchy; casting between unrelated classes results in a **CE**.

### Private Inheritance and Casting

Under private inheritance, the IS-A relationship is hidden from outside code. Outside functions cannot implicitly upcast or use `static_cast` between `Derived` and `Base`:

```cpp
#include <iostream>

struct TBase {
    int a = 0;
};

struct TDerived : private TBase {
    int a = 1;
};

int main() {
    TDerived d;
    // TBase& b = d; // Error: cannot cast 'TDerived' to its private base class 'TBase'
    return 0;
}
```

This will produce a **CE**:
`error: cannot cast 'TDerived' to its private base class 'TBase'`.

However, `reinterpret_cast` or a C-style cast can bypass access control checks and force the reference conversion:

```cpp
#include <iostream>

struct TBase {
    int a = 0;
};

struct TDerived : private TBase {
    int a = 1;
};

int main() {
    TDerived d;
    TBase& b = reinterpret_cast<TBase&>(d); // Bypasses access rules
    std::cout << b.a << std::endl; // Prints 0
    return 0;
}
```

A C-style cast `(TBase&)d` also works because C-style casting falls back to `reinterpret_cast` if `static_cast` is blocked by access rules.
