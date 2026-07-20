## Constants

#### Baseline

Every C++ type can be extended with the **const** keyword. A type is defined by its data and the number of operations on it. A simple definition of a **const** type would be that it is the same type with a reduced number of operations.
So if we consider two types: `int` and `const int`, we can say `const int` is the same `int` except that some `int` operations are not allowed for the **const int** type or are defined in a different way than on **int**.

#### Const references

It is allowed to copy a **const** type into a non-const type. Example:
```cpp
#include <string>

using namespace std;

int main(){
    const string s = "test";
    string ss = s;
    return 0;
}
```
However if we try to create a reference of string type to a _const string_ type, we will get **ce** (compile time error/exception):
```cpp
...
string& ss = s;
...
```
result:
```ascii
main.cpp:7:13: error: binding reference of type 'basic_string<...>' to value of type 'const basic_string<...>' drops 'const' qualifier
    7 |     string& ss = s;
      |             ^    ~
1 error generated.
```
That makes sense because when we initialize a reference to the object we define, _ss_ and _s_ must be **indistinguishable** from each other; however, _s_, according to the _const string_ type definition, must have a restricted number of operations compared to _ss_.
However, you are allowed to do:
```cpp
const string s = "test";
string ss = s;

const string& rs = ss;
```
Compiler allows that because you can apply operation restrictions to a **const string** reference. You actually can propagate this logic between `type` -> `const type` all around - a type can be derived to a _const_ type with a restricted number of operations, but not in reverse.

That is actually widely used to pass arguments into functions. Let's take this example to illustrate it:
```cpp
...
void test(string& s1, string& s2){
}

int main(){
    const string s = "test";
    string ss = s;
    
    const string& rs = ss;

    test(s, s);

    return 0;
}
``` 
If you run this example you will get **ce**:
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
So if we just update the definition of arguments by adding the _const_ modifier, we will be able to use the function for both _const_ and normal types.
```cpp
#include <string>

using namespace std;

void test(const string& s1, const string& s2){
}

int main(){
    const string s = "test";
    string ss = s;
    
    const string& rs = ss;

    test(s, s);

    return 0;
}
```

Note: since a reference is implemented using a pointer at the low level, if your function gets `int` types it would be cheaper to copy them than to create a _const_ reference (or just a reference). However, for any types bigger than 4 bytes, a constant reference must be chosen by default.

Now let's try to pass a string literal explicitly into our _test_ function:
```cpp
#include <string>

using namespace std;

void test(const string& s1, const string& s2){
}

int main(){
    const string s = "test";
    string ss = s;
    
    const string& rs = ss;

    test(s, "s");

    return 0;
}
``` 
Program compiles fine; however, you can see the passed _rvalue_ value "s". It means it is allowed to **initialize a const reference with an _rvalue_**.
This case is called **lifetime expansion**:
- you can create const reference like `const string& test = "test";` initialized with _rvalue_ literal
- this object won't be destroyed until variable _test_ leaves its scope (function by example)

This example can illustrate **UB** (undefined behavior) with a dangling reference.
```cpp
#include <iostream>

using namespace std;

const string& f(const string& s){
    return s;
}

int main(){
    cout << f("test") << endl;
    return 0;
}
```
That may output "test"; however, if the stack has some different values, you also can get a segmentation fault.

Let's consider a different example here:
```cpp
#include <iostream>

using namespace std;

int main(){
    int x = 1;
    const int& y = x;
    int& z = y; 
    return 0;
}
```
This program compiles with **ce** - it does not matter if const _y_ reference to _x_ is cast correctly - we can't cast back from _const int&_ to _int&_ since C++ is a statically typed language:
```cpp
main.cpp:8:10: error: binding reference of type 'int' to value of type 'const int' drops 'const' qualifier
    8 |     int& z = y;
      |          ^   ~
1 error generated.
```

Finally, here is an example of how we can implicitly "change" a constant reference:
```cpp
#include <iostream>

using namespace std;

int main(){
    int x = 1;
    const int& y = x;
    x++;
    cout << y << endl;
    return 0;
}
```
The output will be - `2`.


#### Constant pointers

Let's make a simple program with const pointer:
```cpp
int a = 1;
const int* p = &a;
++p;
```

This program will compile fine because by applying _const_, we apply it to the value, not the pointer. The pointer is just pointing to the constant value. However, changing the value under the pointer (the result of dereferencing the pointer) is not allowed:
```cpp
#include <iostream>

using namespace std;

int main(){
    int x = 1;
    const int* p = &x;
    ++p;
    *p = 2;
    return 0;
}
```
result is **ce**:
```ascii
main.cpp:9:8: error: read-only variable is not assignable
    9 |     *p = 2;
      |     ~~ ^
1 error generated.
```

How do we make a constant pointer but not a constant value under the pointer? We just need the pointer definition in front of the _const_ keyword:
```cpp
#include <iostream>

using namespace std;

int main(){
    int x = 1;
    int* const p = &x;
    ++p;
    *p = 2;
    return 0;
}
```
will result in **ce**:
```ascii
main.cpp:8:5: error: cannot assign to variable 'p' with const-qualified type 'int *const'
    8 |     ++p;
      |     ^ ~
```

Accordingly, if we need a const pointer to a constant, we can write it as:
```cpp
const int* const p = &x;
```
Then both pointer and value under pointer should be constants.
