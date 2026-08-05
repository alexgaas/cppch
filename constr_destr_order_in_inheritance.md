## Order of Constructors and Destructors in Inheritance

### Basic Construction and Destruction Order

When a derived class object is created, its base class subobject must be constructed first. When the object is destroyed, destructors are called in the exact reverse order:

```cpp
#include <iostream>

struct TBase {
    TBase() {
        std::cout << "TBase" << std::endl;
    }

    ~TBase() {
        std::cout << "~TBase" << std::endl;
    }
};

struct TDerived : TBase {
    TDerived() {
        std::cout << "TDerived" << std::endl;
    }

    ~TDerived() {
        std::cout << "~TDerived" << std::endl;
    }
};

int main() {
    TDerived td;
    return 0;
}
```

Output:
```ascii
TBase
TDerived
~TDerived
~TBase
```

The construction and destruction order follows a strict hierarchy:
1. Base class subobject is constructed (`TBase`).
2. Derived class body is constructed (`TDerived`).
3. Derived class body is destroyed (`~TDerived`).
4. Base class subobject is destroyed (`~TBase`).

### Member Initialization and Inheritance

When base and derived classes contain member variables, the full sequence of construction and destruction is:

```cpp
#include <iostream>

struct MemberA {
    MemberA() { std::cout << "MemberA" << std::endl; }
    ~MemberA() { std::cout << "~MemberA" << std::endl; }
};

struct MemberB {
    MemberB() { std::cout << "MemberB" << std::endl; }
    ~MemberB() { std::cout << "~MemberB" << std::endl; }
};

struct TBase {
    MemberA a_;

    TBase() {
        std::cout << "TBase" << std::endl;
    }

    ~TBase() {
        std::cout << "~TBase" << std::endl;
    }
};

struct TDerived : TBase {
    MemberB b_;

    TDerived() {
        std::cout << "TDerived" << std::endl;
    }

    ~TDerived() {
        std::cout << "~TDerived" << std::endl;
    }
};

int main() {
    TDerived td;
    return 0;
}
```

Output:
```ascii
MemberA
TBase
MemberB
TDerived
~TDerived
~MemberB
~TBase
~MemberA
```

**Construction sequence:**
1. Base class data members (in order of declaration).
2. Base class constructor body.
3. Derived class data members (in order of declaration).
4. Derived class constructor body.

**Destruction sequence:**
Destructors execute in the **exact reverse order** of construction.

### Explicit Base Constructor Initialization

A derived constructor can pass parameters to a base class constructor using a member initializer list:

```cpp
#include <iostream>

struct TBase {
    TBase() {
        std::cout << "TBase" << std::endl;
    }

    TBase(int i) : i_(i) {
        std::cout << "TBase i:" << i << std::endl;
    }

    ~TBase() {
        std::cout << "~TBase" << std::endl;
    }

private:
    int i_;
};

struct TDerived : TBase {
    TDerived() {
        std::cout << "TDerived" << std::endl;
    }

    TDerived(int i) : TBase(i), y_(i) {
        std::cout << "TDerived i:" << i << std::endl;
    }

    ~TDerived() {
        std::cout << "~TDerived" << std::endl;
    }

private:
    int y_;
};

int main() {
    TDerived td(1);
    return 0;
}
```

Output:
```ascii
TBase i:1
TDerived i:1
~TDerived
~TBase
```

Even if `y_(i)` is written before `TBase(i)` in the initializer list, the base class constructor `TBase(i)` is **always executed first**.

### Direct Base Initialization Rule

A derived class can only explicitly call the constructor of a **direct** base class (or a virtual base class). It cannot directly invoke the constructor of an indirect base class:

```cpp
#include <iostream>

struct TBase {
    TBase(int i) {}
};

struct TDerived : TBase {
    TDerived(int i) : TBase(i) {}
};

struct TSubDerived : TDerived {
    // Error: TBase is not a direct base of TSubDerived
    // TSubDerived(int i) : TBase(i) {}

    // Correct: Call direct base constructor
    TSubDerived(int i) : TDerived(i) {}
};

int main() {
    TSubDerived sd(1);
    return 0;
}
```

Attempting to call `TBase(i)` from `TSubDerived` results in a compilation error (**CE**):
`error: type 'TBase' is not a direct or virtual base of 'TSubDerived'`.

### Inheriting Base Constructors

Starting in C++11, a derived class can inherit all constructors of its direct base class using `using TBase::TBase;`:

```cpp
#include <iostream>

struct TBase {
    TBase() {
        std::cout << "TBase" << std::endl;
    }

    TBase(int i) : i_(i) {
        std::cout << "TBase i:" << i << std::endl;
    }

    ~TBase() {
        std::cout << "~TBase" << std::endl;
    }

private:
    int i_;
};

struct TDerived : TBase {
    using TBase::TBase; // Inherits default and int constructors from TBase
};

int main() {
    TDerived td(1);
    return 0;
}
```

Output:
```ascii
TBase i:1
~TBase
```
