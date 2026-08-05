## Polymorphism and Virtual Functions

**Polymorphism** ("many forms") allows objects of different derived classes to be treated through a common base class interface while executing behavior specific to their actual derived type.

**Virtual functions** are the mechanism for dynamic (runtime) polymorphism in C++. When a member function is declared `virtual` in a base class, calls to that method through a base class pointer or reference are resolved dynamically based on the actual type of the object at runtime.

### Static vs Dynamic Binding

Without the `virtual` keyword, function calls are resolved at compile time based on the static type of the reference or pointer:

```cpp
#include <iostream>

struct TBase {
    void f() { std::cout << "Base" << std::endl; }
};

struct TDerived : TBase {
    void f() { std::cout << "Derived" << std::endl; }
};

int main() {
    TDerived d;
    TBase& b = d;
    d.f(); // Calls TDerived::f()
    b.f(); // Calls TBase::f() (static binding)
    return 0;
}
```

Output:
```ascii
Derived
Base
```

Now, let's declare `f()` as `virtual` in `TBase`:

```cpp
#include <iostream>

struct TBase {
    virtual void f() { std::cout << "Base" << std::endl; }
};

struct TDerived : TBase {
    void f() { std::cout << "Derived" << std::endl; }
};

int main() {
    TDerived d;
    TBase& b = d;
    d.f(); // Calls TDerived::f()
    b.f(); // Calls TDerived::f() (dynamic binding)
    return 0;
}
```

Output:
```ascii
Derived
Derived
```

Because `f()` is virtual, calling `b.f()` dispatches to `TDerived::f()` because `b` refers to a `TDerived` object. The same applies to pointers:

```cpp
#include <iostream>

struct TBase {
    virtual void f() { std::cout << "Base" << std::endl; }
};

struct TDerived : TBase {
    void f() { std::cout << "Derived" << std::endl; }
};

int main() {
    TDerived* d = new TDerived();
    TBase* b = d;
    d->f(); // Calls TDerived::f()
    b->f(); // Calls TDerived::f()
    delete d;
    return 0;
}
```

Output:
```ascii
Derived
Derived
```

### Slicing and Virtual Functions

Dynamic binding only works with pointers or references. If a derived object is copied by value into a base class variable, object slicing occurs and dynamic polymorphism is lost:

```cpp
#include <iostream>

struct TBase {
    virtual void f() { std::cout << "Base" << std::endl; }
};

struct TDerived : TBase {
    void f() { std::cout << "Derived" << std::endl; }
};

int main() {
    TDerived d;
    TBase b = d; // Sliced into a TBase instance
    d.f();       // Calls TDerived::f()
    b.f();       // Calls TBase::f()
    return 0;
}
```

Output:
```ascii
Derived
Base
```

To explicitly call a base class implementation of a virtual function on a derived object, use qualified scope lookup:

```cpp
b.TBase::f(); // Forces call to TBase::f()
```

### Multi-Level Inheritance and Overriding

When multiple levels of inheritance exist, dynamic dispatch invokes the most overriden version of the virtual function in the inheritance tree:

```cpp
#include <iostream>

struct TBase {
    virtual void f() { std::cout << "Base" << std::endl; }
};

struct TDerived : TBase {
    void f() { std::cout << "Derived" << std::endl; }
};

struct TSubDerived : TDerived {
    // Inherits f() from TDerived
};

int main() {
    TSubDerived d;
    TBase& b = d;
    b.f(); // Calls TDerived::f()
    return 0;
}
```

Output:
```ascii
Derived
```

### The `override` Specifier

A common C++ bug occurs when a derived class intends to override a virtual function but accidentally changes its signature (for example, omitting `const` or changing a parameter type). Without `override`, the compiler silently treats it as a new function hiding the base method:

```cpp
#include <iostream>

struct TBase {
    virtual void f() const { std::cout << "Base" << std::endl; }
};

struct TDerived : TBase {
    void f() { std::cout << "Derived" << std::endl; } // Note missing const!
};

int main() {
    TDerived d;
    TBase& b = d;
    b.f(); // Calls Base because TDerived::f() did not override TBase::f() const
    return 0;
}
```

Output:
```ascii
Base
```

To prevent this error, use the `override` specifier. It instructs the compiler to verify that a base class function with the exact same signature exists:

```cpp
#include <iostream>

struct TBase {
    virtual void f() const { std::cout << "Base" << std::endl; }
};

struct TDerived : TBase {
    void f() override { std::cout << "Derived" << std::endl; } // Error!
};

int main() {
    TDerived d;
    return 0;
}
```

This generates a compilation error (**CE**):
`error: non-virtual member function marked 'override' hides virtual member function (different qualifiers: 'const' vs unqualified)`.

To fix the error, match the exact function signature:
```cpp
void f() const override { std::cout << "Derived" << std::endl; }
```

### The `final` Specifier

The `final` specifier prevents a virtual function from being overridden further down in derived classes (or prevents a class from being inherited when placed on a class definition):

```cpp
#include <iostream>

struct TBase {
    virtual void f() const { std::cout << "Base" << std::endl; }
};

struct TDerived : TBase {
    void f() const override final { std::cout << "Derived" << std::endl; }
};

// struct TSubDerived : TDerived {
//     void f() const override; // Error: cannot override final function TDerived::f
// };
```

### Dynamic Dispatch and Access Control

Access control checks (`public`, `private`, `protected`) are performed at **compile time** using the static type of the reference or pointer. Dynamic dispatch occurs at **runtime**.

```cpp
#include <iostream>

struct TBase {
    virtual void f() const { std::cout << "Base" << std::endl; }
};

struct TDerived : TBase {
private:
    void f() const override final { std::cout << "Derived" << std::endl; }
};

int main() {
    TDerived d;
    TBase& b = d;
    b.f(); // Valid at compile time (Base::f is public); dispatches to TDerived::f at runtime!
    return 0;
}
```

Output:
```ascii
Derived
```

`b.f()` is valid at compile time because `TBase::f()` is `public`. At runtime, dynamic dispatch calls `TDerived::f()`, ignoring `TDerived`'s `private` specifier.
