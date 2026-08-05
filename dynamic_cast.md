## Dynamic Cast and RTTI

### Static Type vs. Dynamic Type

Consider a simple example:

```cpp
#include <iostream>

struct TBase {
    virtual void f() { std::cout << "base\n"; }
};

struct TDerived : TBase {
    void f() override { std::cout << "derived\n"; }
};

int main() {
    TDerived d;
    TBase& b = d;
    b.f();
    return 0;
}
```

How does the compiler determine which function implementation to execute when calling a virtual method through a base pointer or reference? The compiler cannot resolve this call purely at compile time; instead, it generates dynamic dispatch code to resolve the call at runtime.

In C++, **every expression has a static type determined at compile time**. However, the static type of a reference or pointer may differ from the actual type of the object it refers to at runtime:

```cpp
TDerived d;
TBase& b = d;
```

Here, the variable `b` has a static type of `TBase&` (known at compile time), but the underlying object it refers to is of type `TDerived`. The actual runtime type of the referenced object is called its **dynamic type**.

The compiler cannot determine dynamic types at compile time because the concrete object assigned to a pointer or reference often depends on runtime execution paths and conditions:

```cpp
#include <iostream>

struct TBase {
    virtual void f() { std::cout << "base\n"; }
};

struct TDerived : TBase {
    void f() override { std::cout << "derived\n"; }
};

int main() {
    TDerived d;
    TBase b;
    int x;
    std::cin >> x;
    TBase& bb = x % 2 == 0 ? d : b;
    bb.f();
    return 0;
}
```

Sample output:
- Input `4`:
  ```ascii
  derived
  ```
- Input `3`:
  ```ascii
  base
  ```

### Polymorphic Types and `typeid`

A class containing at least one virtual function is a **polymorphic type**.

Polymorphic types support **Run-Time Type Information (RTTI)**. In C++, polymorphic objects store a pointer to a virtual method table (vtable), which includes RTTI metadata. We can inspect this metadata at runtime using the `typeid` operator.

The `typeid` operator returns a reference to a `const std::type_info` object (defined in `<typeinfo>`):

```cpp
#include <iostream>
#include <typeinfo>

struct TBase {
    virtual void f() { std::cout << "base\n"; }
};

struct TDerived : TBase {
    void f() override { std::cout << "derived\n"; }
};

int main() {
    TDerived d;
    TBase b;
    int x;
    std::cin >> x;
    TBase& bb = x % 2 == 0 ? d : b;
    bb.f();
    std::cout << typeid(bb).name() << std::endl;
    return 0;
}
```

Sample output for input `3`:
```ascii
base
5TBase
```

*(Note: `name()` returns an implementation-defined string. For example, compilers like GCC and Clang mangle `TBase` as `5TBase`, where `5` indicates the length of the symbol name. These names are compiler-dependent.)*

We can compare dynamic types using `typeid`:

```cpp
#include <iostream>
#include <typeinfo>

struct TBase {
    virtual void f() { std::cout << "base\n"; }
};

struct TDerived : TBase {
    void f() override { std::cout << "derived\n"; }
};

int main() {
    TDerived d;
    TBase b;

    TBase& bb1 = d;
    TBase& bb2 = b;

    std::cout << std::boolalpha;
    std::cout << (typeid(bb1) == typeid(TDerived)) << std::endl; // true
    std::cout << (typeid(bb2) == typeid(TDerived)) << std::endl; // false
    std::cout << (typeid(bb1) == typeid(bb2)) << std::endl;      // false
    return 0;
}
```

Output:
```ascii
true
false
false
```

`typeid` is rarely used directly in production software, but it provides a low-level mechanism to inspect dynamic types at runtime.

### Downcasting and Cross-Casting with `dynamic_cast`

Consider an inheritance hierarchy with a common polymorphic base class:

```cpp
#include <iostream>

struct TBase {
    int b;
    virtual void f() {}
};

struct TDerived1 : TBase {
    int d1;
};

struct TDerived2 : TBase {
    int d2;
};

struct TSubDerived : TDerived1, TDerived2 {
    int s;
};
```

Suppose we have a pointer of type `TDerived1*` that actually points to a `TSubDerived` instance, and we attempt to cast it to `TDerived2*` using `static_cast`:

```cpp
int main() {
    TSubDerived s;
    TDerived1* d1 = &s;
    // TDerived2* d2 = static_cast<TDerived2*>(d1); // Compilation Error!
    return 0;
}
```

Attempting to use `static_cast` between sibling derived classes causes a compilation error (**CE**):
`error: static_cast from 'TDerived1 *' to 'TDerived2 *', which are not related by inheritance, is not allowed`.

Since `TDerived1` and `TDerived2` are separate branches of the inheritance hierarchy, `static_cast` cannot convert between them directly at compile time.

However, because `TBase` is a polymorphic type and `d1` actually points to a `TSubDerived` object, we can use `dynamic_cast` to perform a **cross-cast** at runtime:

```cpp
#include <iostream>

struct TBase {
    int b;
    virtual void f() {}
};

struct TDerived1 : TBase {
    int d1;
};

struct TDerived2 : TBase {
    int d2;
};

struct TSubDerived : TDerived1, TDerived2 {
    int s;
};

int main() {
    TSubDerived s;
    TDerived1* d1 = &s;
    TDerived2* d2 = dynamic_cast<TDerived2*>(d1); // Valid cross-cast at runtime!
    std::cout << (d2 != nullptr ? "Cast succeeded" : "Cast failed") << std::endl;
    return 0;
}
```

Output:
```ascii
Cast succeeded
```

### Pointer vs. Reference Behavior in `dynamic_cast`

`dynamic_cast` safely verifies dynamic types at runtime, but handles failed casts differently depending on whether pointers or references are used:

1. **Pointers**: If the cast fails, `dynamic_cast` returns `nullptr`:

   ```cpp
   TBase b;
   TBase* bp = &b;
   TDerived1* dp = dynamic_cast<TDerived1*>(bp); // Cast fails, dp is nullptr
   if (!dp) {
       std::cout << "Pointer cast failed!" << std::endl;
   }
   ```

2. **References**: Because C++ references cannot be null, a failed `dynamic_cast` throws a `std::bad_cast` exception (defined in `<typeinfo>`):

   ```cpp
   #include <iostream>
   #include <typeinfo>

   TBase b;
   TBase& br = b;
   try {
       TDerived1& dr = dynamic_cast<TDerived1&>(br); // Throws std::bad_cast
   } catch (const std::bad_cast& e) {
       std::cout << "Reference cast failed: " << e.what() << std::endl;
   }
   ```
