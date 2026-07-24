## Type Casting

#### C-style Cast

C++ inherits explicit type casting from C (known as a C-style cast):
```cpp
double d = 0.0;
int x = (int)d;
```

A C-style cast attempts a sequence of C++ explicit cast operations in order. From [cppreference](https://en.cppreference.com/w/cpp/language/explicit_cast): 
```text
1) When the C-style cast is encountered, the compiler attempts to interpret it as the following cast expressions, in this order:
a) const_cast<type-id>(unary-expression);
b) static_cast<type-id>(unary-expression), with extensions: pointer or reference to a derived class is additionally allowed to be cast to pointer or reference to unambiguous base class (and vice versa) even if the base class is inaccessible (that is, this cast ignores the private inheritance specifier). Same applies to casting pointer to member to pointer to member of unambiguous non-virtual base;
c) a static_cast (with extensions) followed by const_cast;
d) reinterpret_cast<type-id>(unary-expression);
e) a reinterpret_cast followed by const_cast.
 The first choice that satisfies the requirements of the respective cast operator is selected, even if it is ill-formed. If a static_cast followed by a const_cast is used and the conversion can be interpreted in more than one way as such, the conversion is ill-formed.
 In addition, C-style casts can cast from, to, and between pointers to incomplete class types. If both type-id and the type of unary-expression are pointers to incomplete class types, it is unspecified whether static_cast or reinterpret_cast gets selected.
```

It is highly recommended to use C++ named casts (see below) instead of C-style casts. C-style casts do not strictly enforce safety checks at compile time, instead attempting to force the conversion by cascading through cast types.

#### `static_cast`

Example:
```cpp
#include <iostream>

using namespace std;

void f(int) { cout << "1" << endl; }
void f(double) { cout << "2" << endl; }

int main() {
    int x = 1;
    f(static_cast<double>(x));
    return 0;
}
```
Output: `2`

When we perform a `static_cast`, the compiler creates a converted value or temporary object using implicit and user-defined conversions. If `static_cast` cannot perform the conversion, it results in a **compilation error**.

#### `reinterpret_cast`

When using `reinterpret_cast`, a new object is not constructed; instead, the existing expression's bit pattern is reinterpreted, typically requiring a reference or pointer type:
```cpp
#include <iostream>

using namespace std;

int main() {
    double pi = 3.14;
    cout << reinterpret_cast<int&>(pi) << endl;
    return 0;
}
```

The key thing to understand about `reinterpret_cast` is that reinterpreting unrelated types without strict aliasing compliance leads directly to **undefined behavior** (UB). It is a low-level operation that should be avoided unless strictly necessary.

`reinterpret_cast` is commonly used to convert between different pointer types:
```cpp
int x = 1;
int* p = &x;
double* pd = reinterpret_cast<double*>(p);
```

In summary, `reinterpret_cast` treats the binary data of one type as if it were another. This is dangerous and can cause **undefined behavior** (such as segmentation faults or corrupted data). Use it with extreme caution (typically between pointer types, rather than reference types).

#### `const_cast`

This cast adds or removes `const` (or `volatile`) qualifiers from a pointer or reference.
```cpp
#include <iostream>

using namespace std;

void f(int&) { cout << "1" << endl; }
void f(const int&) { cout << "2" << endl; }

int main() {
    int n = 1;
    f(const_cast<const int&>(n));
    return 0;
}
```
Output: `2`

Please note:
- `const_cast` works only on references and pointers.
- `const_cast` does not construct a new object; it reinterprets the qualifications of the existing reference or pointer.

The example above for `const_cast` is somewhat artificial. If we comment out `void f(int&)` and call `f(n)`, the implicit conversion to `const int&` happens automatically without needing any cast construct:
```cpp
#include <iostream>

using namespace std;

// void f(int&) { cout << "1" << endl; }

void f(const int&) { cout << "2" << endl; }

int main() {
    int n = 1;
    // f(const_cast<const int&>(n));
    f(n);
    return 0;
}
```
Output: `2`

Now let's see how `const_cast` works when casting from a `const` reference to a non-`const` reference:
```cpp
#include <iostream>

using namespace std;

void f(int&) { cout << "1" << endl; }
void f(const int&) { cout << "2" << endl; }

int main() {
    int n = 1;
    const int& cn = n;

    f(const_cast<int&>(cn));

    return 0;
}
```
Output: `1`

While `const_cast` allows removing the `const` qualification from a reference or pointer, attempting to **modify** an object that was originally declared `const` results in **undefined behavior** (UB).

Example:
```cpp
#include <iostream>

using namespace std;

void f(int& n) { 
    ++n; 
    cout << "1" << endl; 
}

void f(const int&) { cout << "2" << endl; }

int main() {
    const int n = 5;
    const int& cn = n;

    f(const_cast<int&>(cn));
    cout << n << endl;

    return 0;
}
```
Output:
```ascii
1
5
```
As shown above, the value of `n` was not modified because `n` was originally declared `const`, enabling compiler optimizations (such as constant folding). Modifying an originally `const` object via `const_cast` is undefined behavior.

#### `dynamic_cast`

Will be discussed in detail alongside inheritance and polymorphism topics.
