## Classes and Structures

#### Baseline

Classes and structs in C++ are identical except for their default access specifiers and default inheritance access:
- **Class**: members and base classes are `private` by default.
- **Struct**: members and base classes are `public` by default.

You can include as many access specifier sections (`public:`, `private:`, `protected:`) as needed inside a class or struct.

Here is an example of a simple class:
```cpp
#include <iostream>

using namespace std;

class Test {
private:
    int test = 0;

public:
    void getTest() {
        cout << test << endl;
    }
};

int main() {
    Test t;
    t.getTest();
    return 0;
}
```

You cannot instantiate an object using default constructor with parentheses:
```cpp
Test t();
```
This is known as the "most vexing parse" in C++, where the compiler interprets `Test t();` as a function declaration named `t` that takes no arguments and returns a `Test` object.

To instantiate an object using the default constructor, write either:
```cpp
Test t;
```
or explicitly:
```cpp
Test t = Test();
```

We can also define class member functions outside of the class definition using the scope resolution operator (`::`). All private members of the class remain accessible inside member function definitions outside the class body:
```cpp
#include <iostream>

using namespace std;

class Test {
private:
    int test = 0;

public:
    void getTest();
};

void Test::getTest() {
    cout << test << endl;
}

int main() {
    Test t = Test();
    t.getTest();
    return 0;
}
```
