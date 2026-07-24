## Constants

#### Baseline

Every C++ type can be qualified with the `const` keyword. A type is defined by its state and the set of supported operations. A `const`-qualified type can be thought of as the same underlying type, but with a restricted set of mutating operations allowed on it.

For example, comparing `int` and `const int`: a `const int` has the same representation as an `int`, except operations that modify its value are disallowed at compile time.

#### Const References

It is valid to copy a `const` object into a non-`const` object:
```cpp
#include <string>

using namespace std;

int main() {
    const string s = "test";
    string ss = s;
    return 0;
}
```

However, attempting to bind a non-const reference (`string&`) to a `const string` object results in a **compilation error**:
```cpp
string& ss = s;
```
Error:
```ascii
main.cpp:7:13: error: binding reference of type 'basic_string<...>' to value of type 'const basic_string<...>' drops 'const' qualifier
    7 |     string& ss = s;
      |             ^    ~
1 error generated.
```
This restriction is logical: if `ss` were a non-const reference to `s`, modifying `s` through `ss` would violate `s`'s `const` guarantee.

Conversely, binding a `const` reference to a non-`const` object is fully valid:
```cpp
const string s = "test";
string ss = s;

const string& rs = ss;
```
The compiler allows this because binding a `const` reference (`const string&`) to a mutable object (`ss`) simply restricts modifications through that specific reference `rs`. In C++, adding `const` qualifications (`T&` to `const T&`) is always allowed, but removing `const` is not.

This principle is widely used when passing arguments to functions:
```cpp
void test(string& s1, string& s2) {
}

int main() {
    const string s = "test";
    string ss = s;
    
    const string& rs = ss;

    test(s, s);

    return 0;
}
``` 
Passing `s` to `test(string&, string&)` results in a **compilation error**:
```ascii
main.cpp:14:5: error: no matching function for call to 'test'
   14 |     test(s, s);
      |     ^~~~
main.cpp:5:6: note: candidate function not viable: 1st argument ('const string' (aka 'const basic_string<char>')) would lose const
      qualifier
    5 | void test(string& s1, string& s2){
      |      ^    ~~~~~~~~~~
1 error generated.
```
By adding `const` to the function parameters, `test` can accept both `const` and non-`const` objects:
```cpp
#include <string>

using namespace std;

void test(const string& s1, const string& s2) {
}

int main() {
    const string s = "test";
    string ss = s;
    
    const string& rs = ss;

    test(s, s);

    return 0;
}
```

> **Performance Note**: For small primitive types (such as `int` or `char`), passing by value is typically as fast as or faster than passing by reference (since references are under the hood implemented using pointers). However, for non-trivial types (like `std::string` or `std::vector`), passing by `const` reference is preferred to prevent expensive copy overhead.

Now let's pass a string literal directly into our `test` function:
```cpp
#include <string>

using namespace std;

void test(const string& s1, const string& s2) {
}

int main() {
    const string s = "test";
    string ss = s;
    
    const string& rs = ss;

    test(s, "s");

    return 0;
}
``` 
This code compiles successfully because a **`const` reference can bind to an rvalue** (temporary object).

This behavior involves **reference lifetime extension**:
- When initializing a `const` reference with a temporary rvalue (e.g., `const string& test = "test";`), the temporary object's lifetime is extended to match the lifetime of the reference `test`.
- The temporary object will remain valid until `test` goes out of scope.

However, returning a reference to a local temporary causes **undefined behavior** (UB) due to a dangling reference:
```cpp
#include <iostream>

using namespace std;

const string& f(const string& s) {
    return s;
}

int main() {
    cout << f("test") << endl;
    return 0;
}
```
In this example, the temporary `std::string` created for `"test"` goes out of scope at the end of the full expression in `f("test")`, leaving `f` returning a reference to a destroyed object.

Consider another example:
```cpp
#include <iostream>

using namespace std;

int main() {
    int x = 1;
    const int& y = x;
    int& z = y; 
    return 0;
}
```
This fails to compile with a **compilation error** because converting `const int&` to non-const `int&` drops the `const` qualifier:
```ascii
main.cpp:8:10: error: binding reference of type 'int' to value of type 'const int' drops 'const' qualifier
    8 |     int& z = y;
      |          ^   ~
1 error generated.
```

Finally, note how modifying an underlying variable affects a `const` reference pointing to it:
```cpp
#include <iostream>

using namespace std;

int main() {
    int x = 1;
    const int& y = x;
    x++;
    cout << y << endl;
    return 0;
}
```
Output: `2`

`y` is a `const` reference to `x`, meaning `x` cannot be modified *through `y`*, but `x` itself can still be modified directly, updating the value observed through `y`.

#### Constant Pointers

Let's look at a pointer to a constant value:
```cpp
int a = 1;
const int* p = &a;
++p;
```
This code compiles fine because the `const` qualifier applies to the pointed-to `int` (`*p`), not the pointer itself (`p`). Incrementation (`++p`) alters where `p` points. However, modifying the value through `p` is forbidden:
```cpp
#include <iostream>

using namespace std;

int main() {
    int x = 1;
    const int* p = &x;
    ++p;
    *p = 2;
    return 0;
}
```
Result is a **compilation error**:
```ascii
main.cpp:9:8: error: read-only variable is not assignable
    9 |     *p = 2;
      |     ~~ ^
1 error generated.
```

To make the **pointer itself constant** (preventing `p` from being reassigned) while allowing modifications to the value it points to, place `const` after `*`:
```cpp
#include <iostream>

using namespace std;

int main() {
    int x = 1;
    int* const p = &x;
    ++p;
    *p = 2;
    return 0;
}
```
Result is a **compilation error**:
```ascii
main.cpp:8:5: error: cannot assign to variable 'p' with const-qualified type 'int *const'
    8 |     ++p;
      |     ^ ~
```

If both the pointer and the value it points to must be immutable, declare a constant pointer to a constant value:
```cpp
const int* const p = &x;
```
Here, neither `p` (the pointer address) nor `*p` (the pointed-to value) can be reassigned.
