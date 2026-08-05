## Inheritance

The idea of inheritance is simple: some types can be sub-types of other types. A derived type inherits all functionality of its base type and can extend it with additional fields or methods.

### Public, Private, and Protected Inheritance

Let's start with an example of public inheritance:

```cpp
#include <iostream>

class Base {
public:
    int a = 0;
    void f() {
        std::cout << "base" << std::endl;
    }
};

class Derived : public Base {
public:
    int b = 1;
    void g() {
        std::cout << "derived" << std::endl;
    }
};

int main() {
    Derived d;
    d.g();
    d.f();

    return 0;
}
```

Output:
```ascii
derived
base
```

As expected, `Derived` inherited all fields and methods from `Base`. 

What is the difference between `public` and `private` inheritance?
- With **public inheritance**, all public members of `Base` remain public in `Derived` and can be accessed from outside `Derived`.
- With **private inheritance**, public and protected members of `Base` become `private` in `Derived`. Calling base members from outside `Derived` is forbidden.

Example of private inheritance:

```cpp
#include <iostream>

class Base {
public:
    int a = 0;
    void f() {
        std::cout << "base" << std::endl;
    }
};

class Derived : private Base {
public:
    int b = 1;
    void g() {
        std::cout << "derived" << std::endl;
    }
};

int main() {
    Derived d;
    d.g();
    d.f(); // Error: f is private within this context

    return 0;
}
```

This will result in a compilation error (**CE**):
```ascii
main.cpp:22:7: error: 'f' is a private member of 'Base'
   22 |     d.f();
      |       ^
1 error generated.
```

The method `f()` exists in `Derived`, but it is not accessible outside `Derived`. However, inside member functions of `Derived`, you can still call methods and access fields of `Base`:

```cpp
#include <iostream>

class Base {
public:
    int a = 0;
    void f() {
        std::cout << "base" << std::endl;
    }
};

class Derived : private Base {
public:
    int b = 1;
    void g() {
        std::cout << "derived " << a << std::endl;
    }
};

int main() {
    Derived d;
    d.g();
    // d.f(); // Prohibited outside Derived
    return 0;
}
```

What if we declare `main` as a `friend` of `Derived`? Will that allow `main` to access `Base` members through `Derived` under private inheritance?

```cpp
#include <iostream>

class Base {
public:
    int a = 0;
    void f() {
        std::cout << "base" << std::endl;
    }
};

class Derived : private Base {
    friend int main();

public:
    int b = 1;
    void g() {
        std::cout << "derived " << a << std::endl;
    }
};

int main() {
    Derived d;
    d.g();
    d.f();
    return 0;
}
```

It does! Output:
```ascii
derived 0
base
```

However, if `main` is only declared as a `friend` of `Base` (and not `Derived`), calling `d.f()` will still fail with a **CE**. Friendship is not inherited, and private inheritance prevents outside code from reaching `Base` members via `Derived`.

A derived class can never access `private` members of its base class:

```cpp
#include <iostream>

class Base {
    int a = 0; // private by default in class
public:
    void f() {
        std::cout << "base" << std::endl;
    }
};

class Derived : private Base {
public:
    int b = 1;
    void g() {
        std::cout << "derived " << a << std::endl; // Error: 'a' is private in Base
    }
};

int main() {
    Derived d;
    d.g();
    return 0;
}
```

This will result in a **CE**:
```ascii
main.cpp:15:41: error: 'a' is a private member of 'Base'
   15 |             std::cout << "derived "  << a << std::endl;
      |                                         ^
```

In addition to `public` and `private` inheritance, C++ supports `protected` inheritance. Under `protected` inheritance, public and protected members of `Base` become `protected` in `Derived`. They remain accessible to `Derived` and its further derived classes, but not to outside code.

Let's illustrate `protected` access with an example:

```cpp
#include <iostream>

class Base {
protected:
    int a = 0;
public:
    void f() {
        std::cout << "base" << std::endl;
    }
};

class Derived : public Base {
public:
    int b = 1;
    void g() {
        std::cout << "derived " << a << std::endl;
    }
};

int main() {
    Derived d;
    std::cout << d.a; // Error: 'a' is protected
    return 0;
}
```

This will produce a **CE**:
```ascii
main.cpp:22:20: error: 'a' is a protected member of 'Base'
   22 |     std::cout << d.a;
      |                    ^
```

How does this work? `main` tries to access `d.a`. Because `a` is `protected` in `Base`, outside code cannot access it directly unless `main` is a friend of `Base`. We can fix this by declaring `main` as a friend of `Base`:

```cpp
#include <iostream>

class Base {
    friend int main();
protected:
    int a = 0;
public:
    void f() {
        std::cout << "base" << std::endl;
    }
};

class Derived : public Base {
public:
    int b = 1;
    void g() {
        std::cout << "derived " << a << std::endl;
    }
};

int main() {
    Derived d;
    std::cout << d.a;
    return 0;
}
```

Inheritance can also be `protected`:

```cpp
#include <iostream>

class Base {
protected:
    int a = 0;
public:
    void f() {
        std::cout << "base" << std::endl;
    }
};

class Derived : protected Base {
public:
    int b = 1;
    void g() {
        std::cout << "derived " << a << std::endl;
    }
};

class SubDerived : public Derived {
public:
    void sub() {
        std::cout << a << std::endl; // Accessing Base::a through protected inheritance
    }
};

int main() {
    SubDerived sb;
    sb.sub();
    return 0;
}
```

### Access Level Summary

To understand member accessibility across inheritance levels:
- `public`: Accessible from anywhere.
- `protected`: Accessible inside the class, its derived classes, and friends.
- `private`: Accessible only inside the declaring class and its friends.

When inheriting:
- **Public inheritance**: Access specifiers of base members remain unchanged in the derived class.
- **Protected inheritance**: `public` and `protected` members of the base class become `protected` in the derived class.
- **Private inheritance**: `public` and `protected` members of the base class become `private` in the derived class.

### Accessing Protected Members of a Base Object

A derived class function can only access protected members of a base class instance if that instance is passed via a reference/pointer to the derived class type (or one of its subtypes), not on an independent `Base` instance:

```cpp
#include <iostream>

class Base {
protected:
    int a = 0;
public:
    void f() {
        std::cout << "base" << std::endl;
    }
};

class Derived : public Base {
public:
    int b = 1;
    void g(const Base& x) {
        // std::cout << x.a << std::endl; // Error: cannot access protected member 'a' of an arbitrary Base object
    }
};

int main() {
    Derived d;
    return 0;
}
```

Trying to access `x.a` will result in a **CE**:
`error: 'a' is a protected member of 'Base' - can only access this member on an object of type 'Derived'`.

All violations of inheritance access rules result exclusively in compile-time errors (**CE**). They can never result in runtime errors (**RE**) or undefined behavior (**UB**).
