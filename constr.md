## Constructors and Destructors

A constructor defines how an object of your class is created and initialized. A constructor might be _public_ or _private_. If you do not define any constructor, the compiler will generate a default constructor for you (again, only if you haven't defined one yourself).

```cpp
#include <iostream>
#include <vector>

using namespace std;

class Test {
private:
    int test = 0;
    vector<int> v;

    Test() {}

public:
    Test(int vn) {
        v.reserve(vn); 
    } 

    void getTest();
};

void Test::getTest() {
    cout << test << endl;
}

int main() {
    Test t(10);
    t.getTest();
    return 0;
}
```

You cannot define the same constructor as both _private_ and _public_—that violates the C++ standard's _One Definition Rule_ (and results in a **compilation error**).

As you can see, once a constructor is defined, you can create an object like this:
```cpp
Test t(10);
```
or even using this notation if a constructor taking an `int` has been defined (`=` is not an assignment operator in this expression; it invokes the constructor):
```cpp
Test t = 10;
```

To access class fields, you can use the `this` pointer. Both dereferencing (`(*this).field`) and using `->` (`this->field`) are valid; however, the idiomatic way is to use `->`:
```cpp
#include <iostream>
#include <cstring>

using namespace std;

class String {
private:
    char* str;
    size_t sz;

public:
    String(size_t l, char c) {
        // (*this).sz = l;
        this->sz = l;
        str = new char[l];
        memset(str, c, l);
    }
};

int main() {
    return 0;
}
```

In standard member functions, explicitly typing `this->` is usually a matter of style. The compiler automatically assumes the field name belongs to the class if there are no local variable conflicts. So we could simply write:
```cpp
sz = l;
```

#### Copy constructors
Let's discuss the **copy constructor** now. A copy constructor creates a new object as a copy of an existing object of the same class. In the code snippet below, the compiler creates a default copy constructor for a `String` object even if we did not define one explicitly:
```cpp
String s(10, 'a');
String ss = s;
```

As we know, there are two types of copy operations:
- **Shallow copy**: copies only the class fields. If a field is a pointer, the copy will point to the same memory arena/location as the original.
- **Deep copy**: makes a full copy, creating a new memory arena for the pointer and copying the underlying data.

The default copy constructor performs a **shallow** copy.

Let's implement our own deep copy constructor:
```cpp
#include <iostream>
#include <cstring>

using namespace std;

class String {
private:
    char* str = nullptr;
    size_t sz = 0;

public:
    String(size_t l, char c) {
        // (*this).sz = l;
        this->sz = l;
        str = new char[l];
        memset(str, c, l);
    }

    String(const String& s) {
        sz = s.sz;
        str = new char[sz];
        memcpy(str, s.str, sz);
    }
};

int main() {
    String s(10, 'a');
    String ss = s;
    return 0;
}
```

Note that we cannot pass `String s` by value to the copy constructor because passing by value requires creating a copy of `s`, which would invoke the copy constructor recursively. A copy constructor must accept its first argument by **reference** (otherwise, you will get a **compilation error**).

Now let's look at **destructors**. A **destructor** is a function called before an object is destroyed. Member fields defined inside the class/struct will also have their destructors called automatically as part of the current destructor call. Example:
```cpp
#include <iostream>

using namespace std;

struct a {
    a() { cout << "a"; }
    ~a() { cout << "~a"; }
};

struct b {
    a a;
    b() { cout << "b"; }
    ~b() { cout << "~b"; }
};

int main() {
    b b;
    return 0;
}
```
will output `ab~b~a`.

- The `a` field inside `b` is constructed first, which calls `a`'s constructor.
- `b`'s constructor is called.
- `b` goes out of scope at the end of `main()`, calling `b`'s destructor.
- `a`'s destructor is called afterward to destroy member `a`.

Destructors are called in **reverse order** of construction.

Let's return to our `String` class. Since we allocated memory on the heap using `new[]`, we need to deallocate it after usage in the destructor:
```cpp
#include <iostream>
#include <cstring>

using namespace std;

class String {
private:
    char* str = nullptr;
    size_t sz = 0;

public:
    String(size_t l, char c) {
        // (*this).sz = l;
        this->sz = l;
        str = new char[l];
        memset(str, c, l);
    }

    ~String() {
        delete[] str; 
    }
};
```

The code above does not include a copy constructor, which is dangerous because the compiler will generate a default copy constructor that performs a shallow copy. This is a recipe for **undefined behavior** (UB) or a segmentation fault if the object is copied and destroyed later:
```cpp
#include <iostream>
#include <cstring>

using namespace std;

class String {
private:
    char* str = nullptr;
    size_t sz = 0;

public:
    String(size_t l, char c) {
        // (*this).sz = l;
        this->sz = l;
        str = new char[l];
        memset(str, c, l);
    }
    /*
    String(const String& s) {
        sz = s.sz;
        str = new char[sz];
        memcpy(str, s.str, sz);
    }
    */

    ~String() {
        delete[] str; 
    }
};

int main() {
    String s(10, 'a');
    // Default copy constructor creates a shallow copy with pointer referencing s's memory
    String ss = s;

    // Destructor called for ss - 'str' memory is freed
    // Destructor called for s  - UB (double free) because 'str' was already freed!
    return 0;
}
```

#### Initializer list
(Note: starting from C++11) `std::vector` can be initialized like this:
```cpp
std::vector v = {1, 2, 3};
```
This initialization feature is called an **initializer list** (`std::initializer_list`). Example:
```cpp
#include <iostream>
#include <cstring>
#include <initializer_list>
#include <algorithm>

using namespace std;

class String {
private:
    char* str = nullptr;
    size_t sz = 0;

public:
    String(const initializer_list<char>& lst) {
        sz = lst.size();
        str = new char[sz];
        std::copy(lst.begin(), lst.end(), str);
    }

    String(size_t l, char c) {
        this->sz = l;
        str = new char[l];
        memset(str, c, l);
    }

    String(const String& s) {
        sz = s.sz;
        str = new char[sz];
        memcpy(str, s.str, sz);
    }

    ~String() {
        delete[] str; 
    }
};

int main() {
    String s = {'a', 'b'};
    return 0;
}
```

#### Aggregate initialization
There is another type of initialization for structures with **all** public fields (if any field is private, you will get a **compilation error**). Let's illustrate it:
```cpp
#include <string>

struct t {
    int i;
    double d;
    std::string s;
};

int main() {
    t t{1, 1.0, "t"};
    return 0;
}
```
#### Default and delete
If you want to generate a default constructor while having other custom constructors defined, you can use `= default`:
```cpp
public:
    String() = default;
```
The compiler will not be able to generate a default constructor if any field in the class cannot be default-initialized:
```cpp
#include <string>

struct t {
    const int x;
    t() = default;
};

int main() {
    t t;
    return 0;
}
```
will result in a **compilation error**:
```ascii
main.cpp:5:5: warning: explicitly defaulted default constructor is implicitly deleted [-Wdefaulted-function-deleted]
    5 |     t() = default;
      |     ^
main.cpp:4:15: note: default constructor of 't' is implicitly deleted because field 'x' of const-qualified type 'const int' would not be
      initialized
    4 |     const int x;
      |               ^
main.cpp:5:11: note: replace 'default' with 'delete'
    5 |     t() = default;
      |           ^~~~~~~
      |           delete
main.cpp:9:7: error: call to implicitly-deleted default constructor of 't'
    9 |     t t;
      |       ^
main.cpp:5:5: note: explicitly defaulted function was implicitly deleted here
    5 |     t() = default;
      |     ^
main.cpp:4:15: note: default constructor of 't' is implicitly deleted because field 'x' of const-qualified type 'const int' would not be
      initialized
    4 |     const int x;
      |               ^
```

Similarly, the compiler can generate a default copy constructor:
```cpp
#include <string>

struct t {
    int x;
    t() = default;
    t(const t&) = default;
};

int main() {
    t x;
    t y = x;
    return 0;
}
```

You can prohibit a specific constructor signature using the `delete` specifier:
```cpp
struct t {
    int x;
    t() = default;
    t(const t&) = delete;
};

int main() {
    t x;
    t y = x;
    return 0;
}
```
will result in a **compilation error**:
```ascii
main.cpp:9:7: error: call to deleted constructor of 't'
    9 |     t y = x;
      |       ^   ~
main.cpp:4:5: note: 't' has been explicitly marked deleted here
    4 |     t(const t&) = delete;
      |     ^
1 error generated.
```

#### Delegating constructors
Delegating constructors allow one constructor to call another constructor in the same class. For example:
```cpp
#include <iostream>

struct t {
    int x = 0;
    t() {
        x++;
    }
    t(int y) : t() {
        x += y;
    }
};

int main() {
    t i;
    std::cout << i.x << std::endl;    

    t i2(10);
    std::cout << i2.x << std::endl;    

    return 0;
}
```
will output:
```ascii
1
11
```
As you can see, we can delegate construction work to another constructor using the member initializer list syntax (`:`).

Please note: you cannot simply call a constructor inside the body of another constructor as a substitute for delegating constructors:
```cpp
#include <iostream>

struct t {
    int x = 0;
    t() {
        x++;
    }
    // WRONG!
    // t() here will create a temporary instance of struct t and immediately destroy it,
    // rather than delegating construction of the current instance!
    t(int y) {
        t();
        x += y;
    }
};

int main() {
    t i;
    std::cout << i.x << std::endl;    

    t i2(10);
    std::cout << i2.x << std::endl;    

    return 0;
}
```
