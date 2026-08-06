## Virtual Method Tables (Vtables) and Polymorphic Object Layout

### Vptr and Vtable Basics

Consider a simple comparison between a non-polymorphic struct and a polymorphic struct:

```cpp
#include <iostream>

struct A {};

struct B {
    virtual void f() {}
};

int main() {
    A a;
    B b;
    std::cout << "size of A: " << sizeof(a) << std::endl;
    std::cout << "size of B: " << sizeof(b) << std::endl;
    return 0;
}
```

Output (on 64-bit platforms):
```ascii
size of A: 1
size of B: 8
```

The size of `A` is `1` byte because standard C++ requires empty objects to have non-zero size so that distinct instances have unique memory addresses.

The size of `B` is `8` bytes (on 64-bit architectures) because `B` contains a virtual function, making it a **polymorphic class**. The compiler automatically inserts a hidden pointer into every object of a polymorphic class, called the **vptr** (virtual table pointer).

The `vptr` points to a static array of pointers maintained by the compiler for each polymorphic class, called the **vtable** (virtual method table). The vtable contains:
1. A pointer to RTTI metadata (`type_info`).
2. Pointers to virtual functions implemented by the class.

```text
B Object:                B Vtable:
+------------+           +-------------------+
|    vptr    | --------> |   type_info ptr   |
+------------+           +-------------------+
                         |      &B::f        |
                         +-------------------+
```

### Memory Layout with Data Members

Consider an inheritance hierarchy with data members:

```cpp
#include <iostream>

struct Base {
    int d;
    virtual void f() {}
};

struct Derived : Base {
    int b;
    void f() override {}
};

int main() {
    Derived d;
    std::cout << "sizeof(Base): " << sizeof(Base) << std::endl;
    std::cout << "sizeof(Derived): " << sizeof(Derived) << std::endl;
    return 0;
}
```

Output:
```ascii
sizeof(Base): 16
sizeof(Derived): 16
```

On 64-bit platforms:
- In `Base`, the `vptr` occupies the first 8 bytes (offset 0 to 7). `int d` (4 bytes) is stored at offset 8. The compiler adds 4 bytes of tail padding to satisfy 8-byte alignment, bringing `sizeof(Base)` to 16 bytes.
- In `Derived`, `int b` (4 bytes) is packed into the 4 bytes of padding right after `Base::d`. Thus, `Derived` also fits within 16 bytes.

```text
Derived Object Layout (16 bytes):
+-----------------------+-------------------+-------------------+
|      vptr (8B)        |   Base::d (4B)    |   Derived::b (4B) |
+-----------------------+-------------------+-------------------+
```

If `Derived` declares an additional virtual function `virtual void g() {}`, its vtable contains pointers for both virtual functions:

```text
Derived Vtable:
+-------------------+
|   type_info ptr   |
+-------------------+
|    &Derived::f    |
+-------------------+
|    &Derived::g    |
+-------------------+
```

### Virtual Function Call Dispatch Mechanism

When calling a virtual function through a base pointer or reference:

```cpp
Derived d;
Base& b = d;
b.f();
```

The runtime dynamic dispatch sequence proceeds as follows:

1. **Compile-time lookup**: The compiler checks if `f()` is a virtual function in `Base` and finds its index slot in the vtable (e.g., slot index 0).
2. **Fetch `vptr`**: At runtime, the program reads the object's `vptr` (located at offset 0 of the object referenced by `b`). Since `b` refers to `d`, `vptr` points to `Derived`'s vtable.
3. **Fetch function pointer**: The program looks up index 0 in `Derived`'s vtable, obtaining the function address `&Derived::f`.
4. **Indirect Call**: The program calls `&Derived::f`, passing `&d` as the implicit `this` pointer argument.

```text
b (Base&) ---> Derived Object:              Derived Vtable:
               +------------+               +-------------------+
               |    vptr    | ------------> |   type_info ptr   |
               +------------+               +-------------------+
               |  Base::d   |               |    &Derived::f    | --- Dispatch target
               +------------+               +-------------------+
               | Derived::b |               
               +------------+               
```

### Non-Polymorphic Base to Polymorphic Derived Class

If a base class does not define any virtual functions, it is non-polymorphic and does not contain a `vptr`. If a derived class introduces a virtual function, the `vptr` is added at `Derived`:

```cpp
struct Base {
    int d; // Non-polymorphic: 4 bytes
};

struct Derived : Base {
    int b;
    virtual void f() {} // Introduces vtable!
};

struct SubDerived : Derived {
    int s;
    void f() override {}
};
```

Memory Layouts:
- `Base`: `[ Base::d (4B) ]` (`sizeof` = 4)
- `Derived`: `[ vptr (8B) | Base::d (4B) | Derived::b (4B) ]` (`sizeof` = 16)
- `SubDerived`: `[ vptr (8B) | Base::d (4B) | Derived::b (4B) | SubDerived::s (4B) | padding (4B) ]` (`sizeof` = 24)

Note: If an instance of `SubDerived` is cast to `Base`, calling `f()` is not possible through `Base` because `Base` has no definition of `f()` (and no `vptr`).

### Virtual Functions with Multiple Inheritance

In non-virtual multiple inheritance, a derived class inherits base subobjects from multiple base classes:

```cpp
#include <iostream>

struct Base {
    int b;
    virtual void fg() {}
};

struct Derived1 : Base {
    int d1;
    virtual void fd1() {}
};

struct Derived2 : Base {
    int d2;
    virtual void fd2() {}
};

struct SubDerived : Derived1, Derived2 {
    int s;
    virtual void fs() {}
};
```

Because `SubDerived` inherits from two separate polymorphic base classes (`Derived1` and `Derived2`), it contains **two distinct `vptr` pointers** pointing to two separate vtables:

```text
SubDerived Object Layout (40 bytes):
+------------------+------------------+------------------+------------------+------------------+------------------+------------------+------------------+
| Derived1::vptr1  | Base(1)::b (4B)  | Derived1::d1(4B) | Derived2::vptr2  | Base(2)::b (4B)  | Derived2::d2(4B) | SubDerived::s(4B)|   padding (4B)   |
|     (8 bytes)    |                  |                  |    (8 bytes)     |                  |                  |                  |                  |
+------------------+------------------+------------------+------------------+------------------+------------------+------------------+------------------+
|<----------------- Derived1 Subobject (16B) ------------>|<----------------- Derived2 Subobject (16B) ------------>|
```

#### Pointer Adjustments and Thunks

When casting a `SubDerived*` to a `Derived2*`:

```cpp
SubDerived sub;
Derived1* pd1 = &sub; // Points to start of sub (offset 0)
Derived2* pd2 = &sub; // Adjusted by compiler (offset +16)
```

1. `pd1` points to the start of the `SubDerived` object (offset 0), using `Derived1::vptr1`.
2. `pd2` is automatically offset by the compiler (+16 bytes) so that it points directly to the `Derived2` subobject and its `Derived2::vptr2`.

When calling a virtual function overridden in `SubDerived` through `pd2` (such as `pd2->fg()`), the compiler uses an **adjustor thunk**—a small piece of code generated by the compiler that subtracts 16 bytes from the `this` pointer to adjust it back to the true start of `SubDerived` before executing `SubDerived::fg()`.

### Vtable Header Layout in Virtual Inheritance (Itanium C++ ABI)

In addition to function pointers and RTTI, standard C++ ABIs (such as the Itanium C++ ABI used by GCC and Clang) store metadata in negative offsets *before* the `type_info` pointer in the vtable header:

```text
Vtable Layout Header (Negative Offsets):
+-----------------------+
|  virtual base offset  |  (Offset to virtual base subobject)
+-----------------------+
|     offset to top     |  (Offset from vptr to start of complete object)
+-----------------------+
|     type_info ptr     |  (RTTI metadata pointer)
+-----------------------+ <--- vptr points here!
|     &Class::fn1       |
+-----------------------+
|     &Class::fn2       |
+-----------------------+
```

- **Offset to Top**: The distance from the `vptr` location to the beginning of the complete object. This offset is essential for `dynamic_cast<void*>` and downcasting in complex inheritance hierarchies.
- **Virtual Base Offset (vbase offset)**: The distance from the `vptr` to the virtual base class subobject. This allows dynamic resolution of virtual base locations at runtime without hardcoding fixed offsets.

### Pitfalls and Edge Cases in Virtual Dispatch

#### 1. Default Arguments Are Bound Statically

Default arguments in C++ function calls are evaluated at **compile time** based on the static type of the pointer or reference:

```cpp
#include <iostream>

struct Base {
    virtual void f(int x = 1) const {
        std::cout << "Base " << x << std::endl;
    }
};

struct Derived : Base {
    void f(int x = 2) const override {
        std::cout << "Derived " << x << std::endl;
    }
};

int main() {
    const Base& b = Derived();
    b.f(); // Outputs: Derived 1
    return 0;
}
```

Output:
```ascii
Derived 1
```

**Explanation**: Virtual dispatch executes `Derived::f` dynamically at runtime. However, default arguments are supplied at the call site at compile time based on the static type (`Base`), inserting `1` as the default argument value.

#### 2. Virtual Dispatch in Constructors and Destructors

Calling a virtual function inside a constructor or destructor invokes the implementation corresponding to the class currently being constructed or destroyed:

```cpp
#include <iostream>

struct Base {
    Base() {
        f(0); // Calls Base::f
    }

    virtual void f(int x) const {
        std::cout << "Base " << x << std::endl;
    }
};

struct Derived : Base {
    Derived() {
        f(1); // Calls Derived::f
    }

    void f(int x) const override {
        std::cout << "Derived " << x << std::endl;
    }
};

int main() {
    const Base& b = Derived();
    b.f(2); 
    return 0;
}
```

Output:
```ascii
Base 0
Derived 1
Derived 2
```

**Explanation**: While `Base::Base()` is running, the `Derived` subobject has not yet been constructed. To prevent accessing uninitialized member variables of `Derived`, the object's `vptr` initially points to `Base`'s vtable. It is updated to point to `Derived`'s vtable only when `Derived::Derived()` begins execution.

##### Calling Pure Virtual Functions in Constructors

Attempting to call a pure virtual function directly inside a constructor results in **undefined behavior** (or a runtime crash):

```cpp
struct Base {
    Base() {
        f(0); // Undefined Behavior / Runtime Crash!
    }

    virtual void f(int x) const = 0;
};
```

Runtime output (GCC/Clang):
```ascii
libc++abi: Pure virtual function called!
```

#### 3. Undefined Virtual Functions Prevent Vtable Generation

If a class declares a non-inline virtual function but does not define it, the compiler cannot emit the vtable for that class:

```cpp
struct Base {
    virtual void f(); // Declared, but not defined!
};

int main() {
    Base b; // Linker Error!
    return 0;
}
```

Linker Output:
```ascii
Undefined symbols for architecture x86_64:
  "vtable for Base", referenced from:
      Base::Base() in main.o
NOTE: a missing vtable usually means the first non-inline virtual member function has no definition.
```

**Key Function Rule**: In the C++ ABI, the compiler emits the vtable for a class in the translation unit where the first non-inline virtual function (known as the *key function*) is defined.
