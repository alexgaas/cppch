## Nested Classes and Local Classes

In C++, you can define a class inside another class. This is known as a **nested class** (sometimes referred to as an inner class). 

A nested class is a standalone class whose scope is nested within the outer class. Unlike Java, a nested class in C++ does **not** hold an implicit reference or pointer to an instance of the outer class.

---

### Public Nested Classes

If a nested class is declared in the `public` section of an outer class, it can be instantiated outside the outer class using the scope resolution operator (`::`):

```cpp
#include <iostream>

class Outer {
public:
    class Nested {
    public:
        void greet() {
            std::cout << "Hello from Nested class!" << std::endl;
        }
    };
};

int main() {
    // Instantiate the nested class using the Outer:: scope prefix
    Outer::Nested n;
    n.greet();
    return 0;
}
```

---

### Private Nested Classes

If a nested class is declared in the `private` section of an outer class, external code cannot directly reference its type name (e.g., `Outer::PrivateNested`). However, public member functions of the outer class can still return instances or references to it:

```cpp
#include <iostream>

class Outer {
private:
    class PrivateNested {
    public:
        int value = 42;
    };

public:
    PrivateNested getNested() {
        return PrivateNested();
    }
};

int main() {
    Outer o;

    // Outer::PrivateNested cannot be explicitly named here because it is private.
    // However, public members of the returned instance can still be accessed:
    std::cout << o.getNested().value << std::endl;

    // In C++11 and later, 'auto' can be used to store the instance without naming the private type:
    auto nestedObj = o.getNested();
    std::cout << nestedObj.value << std::endl;

    return 0;
}
```

---

### Local Classes

A class defined directly inside a function body is called a **local class**. Its scope is strictly limited to the enclosing function.

```cpp
#include <iostream>

int main() {
    // Local class defined inside main()
    class LocalClass {
    public:
        int x = 10;
    };

    LocalClass lc;
    std::cout << lc.x << std::endl;

    return 0;
}
```

Local classes are useful for creating single-purpose, helper types in a localized scope—such as custom predicates or comparators for standard algorithms—without cluttering the enclosing namespace.

