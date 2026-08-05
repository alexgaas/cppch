## Visibility and Accessibility

In C++, member **visibility** (name lookup) is distinct from member **accessibility** (access control). 

During compilation:
1. **Name Lookup**: The compiler searches for the identifier starting from the local scope and moving outward to base classes. Once a name is found, lookup stops—even if a function with that name has incompatible parameter types or is marked `private`.
2. **Access Control**: Only *after* name lookup completes and overload resolution chooses a candidate does the compiler check access specifiers (`public`, `protected`, `private`).

### Name Shadowing and Scope

When a derived class defines a member with the same name as a member in the base class, the derived class member **shadows** (hides) the base class member.

```cpp
#include <iostream>

struct TBase {
    int a = 0;
    void f() {
        std::cout << "base\n";
    }
};

struct TDerived : TBase {
    int b = 0;
    void f() {
        std::cout << "derived\n";
    }
};

int main() {
    TDerived d;
    d.f(); // Calls TDerived::f()
    return 0;
}
```

Output:
```ascii
derived
```

`TDerived::f()` hides `TBase::f()`. To call the base class implementation, use qualified scope lookup (`TBase::`):

```cpp
#include <iostream>

struct TBase {
    int a = 0;
    void f() {
        std::cout << "base\n";
    }
};

struct TDerived : TBase {
    int b = 0;
    void f() {
        std::cout << "derived\n";
    }
};

int main() {
    TDerived d;
    d.TBase::f(); // Explicitly calls TBase::f()
    return 0;
}
```

Output:
```ascii
base
```

The same shadowing behavior applies to data fields:

```cpp
#include <iostream>

struct TBase {
    int a = 0;
};

struct TDerived : TBase {
    int a = 1; // Shadows TBase::a
};

int main() {
    TDerived d;
    std::cout << d.a << std::endl;        // Prints 1 (TDerived::a)
    std::cout << d.TBase::a << std::endl;  // Prints 0 (TBase::a)
    return 0;
}
```

### Visibility vs Access Control

What happens if `a` in `TDerived` is `private`?

```cpp
#include <iostream>

struct TBase {
    int a = 0;
};

struct TDerived : TBase {
private:
    int a = 1; // Private member shadows public TBase::a
};

int main() {
    TDerived d;
    std::cout << d.a << std::endl; // Error: 'a' is private in TDerived
    return 0;
}
```

This will produce a compilation error (**CE**):
`error: 'a' is a private member of 'TDerived'`.

Name lookup finds `TDerived::a` and stops searching, so `TBase::a` is never considered. Then, access checking fails because `TDerived::a` is `private`.

However, `TBase::a` still exists in memory and can be accessed explicitly with qualified scope lookup: `d.TBase::a`.

### Overloading Across Base and Derived Classes

Functions in a derived class hide **all** functions with the same name in the base class, regardless of parameter signatures:

```cpp
#include <iostream>

struct TBase {
    void f(double) {
        std::cout << "base\n";
    }
};

struct TDerived : TBase {
    void f(int) {
        std::cout << "derived\n";
    }
};

int main() {
    TDerived d;
    d.f(0.0); // Calls TDerived::f(int) after converting 0.0 to 0!
    return 0;
}
```

Output:
```ascii
derived
```

`TDerived::f(int)` hides `TBase::f(double)`. When calling `d.f(0.0)`, name lookup finds only `TDerived::f(int)`. The compiler converts `0.0` to `int` and calls `TDerived::f(int)`. If `TDerived::f` took a `std::string`, `d.f(0.0)` would fail to compile because `0.0` cannot be converted to `std::string`, and `TBase::f` is hidden.

### Unhiding Base Methods with `using`

To bring hidden base class methods into the derived class scope so they can participate in overload resolution, use a `using` declaration:

```cpp
#include <iostream>
#include <string>

struct TBase {
    void f(double) {
        std::cout << "base\n";
    }
};

struct TDerived : TBase {
    using TBase::f; // Unhides TBase::f in TDerived scope

    void f(std::string) {
        std::cout << "derived\n";
    }
};

int main() {
    TDerived d;
    d.f(0.0);       // Calls TBase::f(double)
    d.f("hello");   // Calls TDerived::f(std::string)
    return 0;
}
```

Output:
```ascii
base
derived
```

Note: A `using` declaration cannot bring `private` base class members into derived scope; attempting to do so will result in a **CE**.

### Inheriting Constructors (`using Base::Base`)

Starting in C++11, `using` declarations can also inherit constructors from a base class:

```cpp
#include <iostream>

struct TBase {
    TBase() = default;
    TBase(int a) : a(a) {}

    int a = 0;
};

struct TDerived : TBase {
    using TBase::TBase; // Inherits TBase(int) constructor

    void f(int) {
        std::cout << "derived\n";
    }
};

int main() {
    TDerived d(10); // Uses inherited TBase(int) constructor
    std::cout << d.a << std::endl;
    return 0;
}
```

Output:
```ascii
10
```
