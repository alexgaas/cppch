## References

#### Baseline

Let's start with a simple swap function designed to exchange argument values:

```cpp
#include <iostream>

using namespace std;

void swap(int x, int y) {
    int t = x;
    x = y;
    y = t;
}

int main() {
    int x = 1, y = 2;
    swap(x, y);
    cout << x << " " << y << endl;
    return 0;
}
```
Compile and run:
```sh
g++ -std=c++20 -O2 main.cpp
./a.out
```
The output is `1 2` because arguments in C++ are passed by value by default. The parameters `x` and `y` in `swap` are local copies, so modifications inside `swap` do not affect the variables in `main()`.

We can fix this function using **pointers**:
```cpp
void swap(int* x, int* y) {
    int t = *x;
    *x = *y;
    *y = t;
}
```
Building and executing with `swap(&x, &y)` outputs `2 1`.

Alternatively, we can maintain clean call-site syntax using **references**. A reference acts as an **alias** (another name) for an existing object rather than creating a copy:

```cpp
void swap(int& x, int& y) {
    int t = x;
    x = y;
    y = t;
}
```
Calling `swap(x, y)` gives the expected output: `2 1`.

Let's see another example showing how references work:
```cpp
#include <vector>

using namespace std;

int main() {
    // Create a vector instance named 'v'
    vector v = {1, 2, 3};
    // Create a second vector named 'vv' and copy 'v' into it
    // 'vv' is an independent vector object!
    vector vv = v;
    return 0;
}
```
Here, the compiler constructs and destroys two distinct `vector` objects (`v` and `vv`). However, if we declare `vv` as a **reference**, no copy is performed; `vv` becomes an alias for `v`:
```cpp
vector& vv = v;
```

Let's look at another reference example:
```cpp
#include <iostream>

using namespace std;

int main() {
    int x = 1;
    // 'y' is a reference to 'x' (an alias for 'x')
    int& y = x;
    // 'z' is a reference to 'y' (which means 'z' is also an alias for 'x')
    int& z = y;

    cout << x << " " << y << " " << z << endl;

    int a = 0;
    // 'z = a' assigns the value of 'a' to 'x'
    z = a;

    cout << x << " " << y << " " << z << endl;

    return 0;
}
```
Output:
```ascii
1 1 1
0 0 0
```

Key takeaways from this program:
- `z` is not a "reference to a reference"; C++ collapses references so `z` is simply another alias for `x`.
- Executing `z = a` assigns the value of `a` to `x`. References **cannot be rebound** to refer to a different object after initialization.

If we increment `a` afterward:
```cpp
a++;
cout << x << " " << y << " " << z << " " << a << endl;
```
The output is `0 0 0 1`, confirming that `z` did not rebind to `a`.

In summary, a reference is an alias for an existing object. Operations performed on the reference directly mutate or inspect the referenced object. Once initialized, a reference cannot be rebound to point to another object. Aside from `decltype` rules, using a reference is semantically identical to using the object itself.

#### Reference Restrictions

1. **You cannot create a pointer to a reference**:
```cpp
int&* x; // Error
```
The C++ standard forbids pointers to references:
```ascii
main.cpp:6:9: error: 'x' declared as a pointer to a reference of type 'int &'
    6 |     int&* x;
      |         ^
1 error generated.
```

However, you **can** create a reference to a pointer (since a pointer is an object type):
```cpp
#include <iostream>

using namespace std;

int main() {
    int x = 0;
    int* p = &x;
    int*& pp = p;
    cout << p << " " << pp << endl; 
    return 0;
}
```
Output: `0x16fcbed64 0x16fcbed64` (both show the same pointer value, as `pp` is an alias for pointer `p`).

2. **Forming a reference to a reference directly is invalid in variable declarations**:
```cpp
int&& y = x; // Error: '&&' is reserved for rvalue references
```
The `&&` syntax denotes an **rvalue reference** and cannot bind directly to an lvalue like `x` in this context.

3. **A reference must be initialized when declared**:
```cpp
int& z; // Error
```
Compiler error:
```ascii
main.cpp:10:10: error: declaration of reference variable 'z' requires an initializer
   10 |     int& z;
      |          ^
1 error generated.
```

4. **A non-const lvalue reference cannot bind to an rvalue**:
```cpp
#include <iostream>

using namespace std;

void swap(int& x, int& y) {
    int t = x;
    x = y;
    y = t;
}

int main() {
    // Won't compile:
    // int& z = 1;

    // Won't compile:
    // swap(1, 2);

    return 0;
}
```
A non-const lvalue reference must be initialized with an lvalue (in C++, numeric and character literals are rvalues, whereas string literals are lvalues).

#### Dangling References

Consider a function returning a reference to a local variable:
```cpp
#include <iostream>

using namespace std;

int& f() {
    int x = 0;
    return x;
}

int main() {
    int x = f();
    cout << x << endl;
    return 0;
}
```
Compiling this yields a warning:
```ascii
g++ -std=c++20 -O2 main.cpp
main.cpp:7:12: warning: reference to stack memory associated with local variable 'x' returned [-Wreturn-stack-address]
    7 |     return x;
      |            ^
1 warning generated.
```
Returning a reference to a local stack variable (`x`) causes a **dangling reference** because `x` is destroyed when `f()` returns. Accessing this reference in `main()` triggers **undefined behavior** (UB), which may result in garbage values, memory corruption, or a segmentation fault.
