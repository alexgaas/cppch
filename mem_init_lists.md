## Member Initializer Lists in Constructors

You may need to include reference (`&`) or `const` members in your class, or include a member variable of a class type that lacks a default constructor (or has a deleted default constructor).

In such cases, default-constructing the outer class will fail to compile because reference members, `const` members, and objects without default constructors must be explicitly initialized upon creation:

```cpp
#include <iostream>

class C {
public:
    C() = delete;
    C(int val) {}
};

class TType {
public:
    TType() { // Compilation Error!
    }

private:
    int& x;
    const int y;
    C c;
};

int main() {
    return 0;
}
```

#### Using Member Initializer Lists

To solve this, use a **member initializer list** in the constructor header (introduced by a colon `:` after the parameter list):

```cpp
#include <iostream>

class C {
public:
    C() = delete;
    C(int val) : data(val) {}

private:
    int data;
};

class TType {
public:
    TType(int& ref, int constVal, int cVal) 
        : x(ref), y(constVal), c(cVal) {
    }

    void print() const {
        std::cout << "x: " << x << ", y: " << y << std::endl;
    }

private:
    int& x;
    const int y;
    C c;
};

int main() {
    int val = 10;
    TType t(val, 20, 30);
    t.print();
    return 0;
}
```

#### Benefits of Member Initializer Lists

Member initializer lists directly construct class fields rather than first default-constructing them and then assigning values inside the constructor body. This avoids unnecessary copy/assignment overhead and is the standard, idiomatic way to write C++ constructors:

```cpp
#include <iostream>

class TType {
public:
    TType(int i, int j) : x(i), y(j) {
    }

    void print() const {
        std::cout << "x: " << x << ", y: " << y << std::endl;
    } 

private:
    int x;
    int y;
};

int main() {
    TType t(1, 2);
    t.print();
    return 0;
}
```

Member initializer lists take precedence over in-class default member initializers (e.g., `int x = 0;`).

#### Initialization Order Warning

> **CRITICAL NOTE**: Member variables are **ALWAYS** initialized in the order they are declared in the class definition, **NOT** in the order they appear in the member initializer list!

Example of undefined behavior due to declaration order:
```cpp
#include <iostream>

class TType {
public:
    // WRONG: y is listed first in the initializer list, but x is declared first in the class!
    // Therefore, x(new int[y]) is evaluated BEFORE y(i), reading uninitialized garbage for 'y'!
    TType(int i) : y(i), x(new int[y]) {
    }

    ~TType() {
        delete[] x;
    }

private:
    int* x; // Declared first -> Initialized FIRST!
    int y;  // Declared second -> Initialized SECOND!
};

int main() {
    TType t(5);
    return 0;
}
```
Compiler warning:
```ascii
main.cpp:6:39: warning: field 'y' is uninitialized when used here [-Wuninitialized]
    6 |         TType(int i): y(i), x(new int[y]) {
      |                                       ^
1 warning generated.
```

To fix this issue, reorder the field declarations inside the class so that `y` is declared before `x`.

#### Restriction with Delegating Constructors

You cannot combine a delegating constructor call with other member initializers in the member initializer list:

```cpp
class TType {
public:
    TType() : y(0) {}

    // ERROR: A delegating constructor cannot be combined with member initializers!
    TType(int i) : TType(), y(i) {
    }

private:
    int y;
};

int main() {
    TType t(1);
    return 0;
}
```
Compiler error:
```ascii
main.cpp:6:23: error: an initializer for a delegating constructor must appear alone
    6 |         TType(int i): TType(),  y(i) {
      |                       ^~~~~~~   ~~~~
1 error generated.
```
