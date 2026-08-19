## Two-Phase Translation and Template Instantiation

Template processing in C++ is divided into two distinct compilation phases, accompanied by **lazy (on-demand) instantiation** rules for member functions.

---

### Two-Phase Translation

1. **Phase 1 (Template Definition Time):**
   - The compiler verifies basic syntax.
   - Non-dependent names (identifiers that do not depend on template parameters) are resolved and checked immediately.
   - Non-dependent static checks are enforced.

2. **Phase 2 (Template Instantiation Time):**
   - Occurs when the template is instantiated with concrete template arguments.
   - Dependent names and expressions are checked and substituted.

---

### Implicit (Lazy) Instantiation

In C++, class template member functions and static data members are instantiated **lazily (on-demand)**. If a member function is never invoked or referenced, the compiler never instantiates it.

Consequently, invalid code inside an uncalled member function does not generate a compilation error:

```cpp
#include <iostream>

template <typename T>
struct S {
    void validMethod() {
        std::cout << "Valid method called" << std::endl;
    }

    void invalidMethod() {
        T::non_existent_method(); // Would fail during Phase 2 if instantiated
    }
};

int main() {
    S<int> s;
    s.validMethod(); // OK: invalidMethod() is never called, so it is never instantiated
    return 0;
}
```

Output:
```ascii
Valid method called
```

---

### Explicit Template Instantiation

You can force the compiler to instantiate a template and **all** of its members using **explicit instantiation**:

```cpp
template <typename T, int N>
struct ArrayWrapper {
    int data[N];
};

// Explicitly instantiate ArrayWrapper with N = -1
template struct ArrayWrapper<int, -1>;

int main() {
    return 0;
}
```

This immediately triggers a compilation error (**CE**):

```ascii
error: 'data' declared as an array with a negative size
    int data[N];
        ^
note: in instantiation of template class 'ArrayWrapper<int, -1>' requested here
template struct ArrayWrapper<int, -1>;
                ^
```

Similarly, explicitly instantiating `S<int>` from the previous example forces `invalidMethod()` to be instantiated:

```cpp
template struct S<int>; // Compilation Error: 'int' has no member named 'non_existent_method'
```

> [!TIP]
> In large codebases, explicit instantiation definitions (`template struct S<int>;`) combined with explicit instantiation declarations (`extern template struct S<int>;`) are often used to reduce compile times and prevent duplicate template code across translation units.

---

### Virtual Member Functions Exception

Virtual member functions are an important exception to lazy instantiation:

1. **Virtual functions are always instantiated:** Whenever a class template is instantiated, all of its virtual member functions are instantiated immediately—even if they are never called. This is required because the compiler must construct the class's virtual method table (vtable) and populate it with valid function pointers.

```cpp
template <typename T>
struct Base {
    virtual void vfunc() {
        T::invalid(); // Always instantiated when Base<T> is instantiated!
    }
};

int main() {
    Base<int> b; // Compilation Error: vfunc() is instantiated to build the vtable
    return 0;
}
```

2. **Virtual methods cannot be member function templates:** Member function templates cannot be declared `virtual` because the size of the vtable must be fixed when the class is defined:

```cpp
template <typename T>
struct S {
    template <typename U>
    virtual void f(U); // Compilation Error!
};
```

Compiler Error:
```ascii
error: 'virtual' cannot be specified on member function templates
    virtual void f(U);
    ^~~~~~~
```
