## Pointers to Members

In C++, pointers to members allow you to store a pointer referencing a non-static member (data member or member function) of a class or struct, rather than a specific object instance. 

Unlike a regular pointer that stores an absolute memory address, a pointer to a member represents an offset within an instance of the class.

---

### Pointers to Data Members

A pointer to a data member is declared using the syntax `Type ClassName::*ptr_name`. You access the member using the `.*` operator for object instances/references or `->*` for object pointers.

```cpp
#include <iostream>

struct Point {
    int x;
    int y;
};

int main() {
    // Declare a pointer to an int member of Point
    int Point::*memberPtr = &Point::x;

    Point p{10, 20};

    // Access member 'x' using .*
    std::cout << p.*memberPtr << std::endl; // Outputs 10

    // Change memberPtr to point to 'y'
    memberPtr = &Point::y;
    std::cout << p.*memberPtr << std::endl; // Outputs 20

    return 0;
}
```

---

### Pointers to Member Functions

Pointers to member functions store the location of a member function within a class. The syntax to declare a member function pointer is `ReturnType (ClassName::*funcPtr)(ParamTypes...)`.

Because the function call operator `()` has higher precedence than `.*` and `->*`, the dereference expression must be enclosed in parentheses: `(obj.*funcPtr)()` or `(ptr->*funcPtr)()`.

```cpp
#include <iostream>

struct Calculator {
    int add() { return 1; }
    int sub() { return 2; }
};

int main() {
    // Declare a pointer to a member function of Calculator taking no args and returning int
    int (Calculator::*funcPtr)() = &Calculator::add;

    Calculator calc;
    
    // Call add() via member function pointer
    std::cout << (calc.*funcPtr)() << std::endl; // Outputs 1

    return 0;
}
```

---

### Dynamic Dispatch using Pointers to Members

Pointers to member functions allow you to select and call different member functions dynamically at runtime:

```cpp
#include <iostream>

struct Calculator {
    int add() { return 1; }
    int sub() { return 2; }
};

void process(Calculator calc, bool performAdd) {
    int (Calculator::*funcPtr)() = performAdd ? &Calculator::add : &Calculator::sub;
    std::cout << (calc.*funcPtr)();
}

int main() {
    Calculator calc;
    process(calc, true);  // Outputs 1
    process(calc, false); // Outputs 2
    std::cout << std::endl;
    return 0;
}
```

---

### Using the `->*` Operator

When working with a pointer to an object instead of an object instance, use the `->*` operator instead of `.*`:

```cpp
#include <iostream>

struct Calculator {
    int add() { return 1; }
    int sub() { return 2; }
};

void process(Calculator* calcPtr, bool performAdd) {
    int (Calculator::*funcPtr)() = performAdd ? &Calculator::add : &Calculator::sub;
    std::cout << (calcPtr->*funcPtr)() << std::endl;
}

int main() {
    Calculator calc;
    process(&calc, true);  // Outputs 1
    process(&calc, false); // Outputs 2
    return 0;
}
```

