## Multiple Inheritance

In C++, a class can inherit from more than one base class. This is called **multiple inheritance**.

### Memory Layout in Multiple Inheritance

Base class subobjects are laid out sequentially in memory in the order they are listed in the class inheritance list:

```cpp
#include <iostream>

struct TZero {
    int z = 0;
};

struct TBase {
    int b = 1;
};

struct TDerived : TZero, TBase {
    int d = 2;
};

int main() {
    TDerived d;
    std::cout << sizeof(d) << std::endl;
    std::cout << &d.z << " " << &d.b << " " << &d.d << std::endl;
    return 0;
}
```

Output:
```ascii
12
0x16dbfeda8 0x16dbfedac 0x16dbfedb0
```

Memory layout: `[TZero (4 bytes)][TBase (4 bytes)][TDerived (4 bytes)]`.

If we reverse the base class order (`struct TDerived : TBase, TZero`), `TBase` will be placed first in memory instead.

### Ambiguous Member Lookup

If two base classes declare a function or member variable with the same name, accessing that name on a derived instance results in a compilation error (**CE**):

```cpp
#include <iostream>

struct TZero {
    void f() { std::cout << "zero" << std::endl; }
    int z = 0;
};

struct TBase {
    void f() { std::cout << "base" << std::endl; }
    int b = 1;
};

struct TDerived : TZero, TBase {
    int d = 2;
};

int main() {
    TDerived d;
    // d.f(); // Error: ambiguous name lookup
    return 0;
}
```

Error: `member 'f' found in multiple base classes of different types (ambiguous name lookup)`.

To resolve the ambiguity, specify the intended base class using scope qualification:

```cpp
d.TZero::f(); // Calls TZero::f()
d.TBase::f(); // Calls TBase::f()
```

### Common Issues with Multiple Inheritance

#### 1. The Diamond Problem

When two intermediate base classes inherit from a common base class, and a derived class inherits from both intermediate classes, a "diamond" shape is formed:

```
    TZero
   /     \
TBase1  TBase2
   \     /
   TDerived
```

```cpp
#include <iostream>

struct TZero {
    void f() { std::cout << "zero" << std::endl; }
    int z = 0;
};

struct TBase1 : TZero {
    int b1 = 1;
};

struct TBase2 : TZero {
    int b2 = 2;
};

struct TDerived : TBase1, TBase2 {
    int d = 2;
};

int main() {
    TDerived d;
    // d.f(); // Error: 'f' is ambiguous
    return 0;
}
```

`TDerived` contains two distinct `TZero` subobjects (`TBase1::TZero` and `TBase2::TZero`). Calling `d.f()` causes a **CE** because the compiler cannot determine which `TZero` subobject to use.

#### 2. Inaccessible / Duplicate Base Class

Inheriting the same base class directly and indirectly also creates duplicate base subobjects:

```cpp
#include <iostream>

struct TZero {
    void f() { std::cout << "zero" << std::endl; }
    int z = 0;
};

struct TBase : TZero {
    int b = 1;
};

struct TDerived : TZero, TBase { // Direct and indirect inheritance of TZero
    int d = 2;
};

int main() {
    TDerived d;
    return 0;
}
```

This generates a compiler warning or error regarding inaccessible base classes due to ambiguity.

### Pointer Adjustments in Multiple Inheritance

In single inheritance, the base subobject is located at offset 0 of the derived object (`[Base][Derived members]`). Therefore, casting a `Derived*` to a `Base*` does not change the memory address (`(Base*)ptr == (Derived*)ptr`).

In multiple inheritance (`struct TDerived : TZero, TBase`), `TZero` is placed at offset 0, while `TBase` is placed at offset `sizeof(TZero)`. 

Consequently, casting a `TDerived*` to `TBase*` requires adjusting (shifting) the pointer address by `sizeof(TZero)` bytes. `static_cast` performs this pointer offset adjustment automatically at compile time:

```cpp
TDerived d;
TDerived* pDerived = &d;
TZero* pZero = static_cast<TZero*>(pDerived); // Same address as pDerived
TBase* pBase = static_cast<TBase*>(pDerived); // Address shifted by sizeof(TZero)
```
