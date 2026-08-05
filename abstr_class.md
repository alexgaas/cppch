## Abstract Classes and Pure Virtual Functions

The concept of virtual functions leads naturally to the design pattern of an **interface**—a class that defines function declarations without implementations, leaving derived classes to provide the concrete logic. This establishes a contract between the base interface and its derived classes.

### Pure Virtual Functions

A virtual function declared with `= 0` syntax is a **pure virtual function**:

```cpp
#include <iostream>

struct TBase {
    // Pure virtual function
    virtual void f() = 0;
};

struct TDerived : TBase {
};

int main() {
    // TBase b; // Error: cannot instantiate abstract class
    return 0;
}
```

### Abstract Classes

A class containing at least one pure virtual function is an **abstract class**. 
- You **cannot instantiate** an abstract class.
- Any derived class must implement all pure virtual functions from the base class; otherwise, the derived class is also considered abstract and cannot be instantiated.

Attempting to instantiate an abstract class results in a compilation error (**CE**):
`error: variable type 'TBase' is an abstract class (unimplemented pure virtual method 'f')`.

### Implementing Pure Virtual Functions in Derived Classes

When a derived class implements all pure virtual functions, it becomes a concrete class and can be instantiated:

```cpp
#include <iostream>

struct TBase {
    virtual void f() = 0; // Pure virtual function
};

struct TDerived : TBase {
    void f() override {
        std::cout << "derived" << std::endl;
    }
};

int main() {
    TDerived d;
    d.f();

    TBase& b = d;
    b.f();
    return 0;
}
```

Output:
```ascii
derived
derived
```

### Polymorphism with Abstract Classes

Abstract classes allow creating polymorphic collections of pointers or references to manage diverse objects through a unified interface:

```cpp
#include <iostream>
#include <vector>

struct TBase {
    virtual void f() = 0;
    virtual ~TBase() = default; // Virtual destructor for polymorphic cleanup
};

struct TDerived1 : TBase {
    void f() override {
        std::cout << "derived 1" << std::endl;
    }
};

struct TDerived2 : TBase {
    void f() override {
        std::cout << "derived 2" << std::endl;
    }
};

int main() {
    std::vector<TBase*> vec;
    vec.push_back(new TDerived1());
    vec.push_back(new TDerived2());

    for (size_t i = 0; i < vec.size(); ++i) {
        vec[i]->f();
    }

    // Clean up allocated memory
    for (TBase* ptr : vec) {
        delete ptr;
    }

    return 0;
}
```

Output:
```ascii
derived 1
derived 2
```
