## Virtual Destructors

### Why Virtual Destructors are Necessary

Deleting a derived class instance through a pointer to a base class that lacks a `virtual` destructor causes **undefined behavior (UB)**. In practice, only the base class destructor executes, skipping the derived class destructor and leaking resources:

```cpp
#include <iostream>

struct TBase {
    int* bp = new int();

    ~TBase() {
        std::cout << "~Base" << std::endl;
        delete bp;
    }
};

struct TDerived : TBase {
    int* dp = new int();

    ~TDerived() {
        std::cout << "~Derived" << std::endl;
        delete dp;
    }
};

int main() {
    TBase* b = new TDerived();
    delete b; // Bug: Non-virtual destructor! Only ~TBase() is called, dp is leaked!

    return 0;
}
```

Output:
```ascii
~Base
```

### Fixing Resource Leaks with Virtual Destructors

When the base class destructor is marked `virtual`, `delete b` uses dynamic dispatch to call `~TDerived()`, which automatically calls `~TBase()` afterwards:

```cpp
#include <iostream>

struct TBase {
    int* bp = new int();

    virtual ~TBase() {
        std::cout << "~Base" << std::endl;
        delete bp;
    }
};

struct TDerived : TBase {
    int* dp = new int();

    ~TDerived() override {
        std::cout << "~Derived" << std::endl;
        delete dp;
    }
};

int main() {
    TBase* b = new TDerived();
    delete b; // Correct: calls ~TDerived() then ~TBase()

    return 0;
}
```

Output:
```ascii
~Derived
~Base
```

### Rule for Polymorphic Base Classes

If a class is intended to serve as a base class in a polymorphic hierarchy (where objects are deleted via base pointers), its destructor **must be declared `virtual`**. Even if the base class has no resources to free, define `virtual ~TBase() = default;`.

### Pure Virtual Destructors

A class can be made abstract by declaring its destructor pure virtual (`virtual ~TBase() = 0;`). 

Unlike regular pure virtual functions, **a pure virtual destructor must still have an implementation defined outside the class**, because derived class destructors always call their base class destructor during cleanup:

```cpp
#include <iostream>

struct TBase {
    virtual ~TBase() = 0; // Pure virtual destructor makes TBase abstract
};

// Definition required outside the class definition
TBase::~TBase() {}

struct TDerived : TBase {
    int* dp = new int();

    ~TDerived() override {
        std::cout << "~Derived" << std::endl;
        delete dp;
    }
};

int main() {
    TBase* b = new TDerived();
    delete b;

    return 0;
}
```

Output:
```ascii
~Derived
```

### Virtual Constructors Do Not Exist

C++ does not support virtual constructors. Object construction requires knowing the exact concrete type at compile time to allocate memory and initialize the object. To instantiate objects polymorphically, use factory functions or the virtual copy idiom (e.g., a `virtual TBase* clone() const = 0;` method).
