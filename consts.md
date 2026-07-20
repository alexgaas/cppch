## Constants

#### Baseline

Every C++ type can be extended with **const** keyword. Type is defined by it's data and number of operations on it.  Simple definition of **const** type would be is that same type with reduced  number of operations.
So if we consider two types: `int` and `const int` we can say `const int` is a same `int` besides the some `int` operations are not allowed for **const int** type or defined different way then on **int**.

#### Const references

There is allowed to copy **const** type into type. Example:
```cpp
#include <string>

using namespace std;

int main(){
    const string s = "test";
    string ss = s;
    return 0;
}
```
However if we try to create a reference of string type into _const string_ type, we will ge **ce** (compile time exception):
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
That make sense b/c when we initialzie reference on the object we define _ss_ and _s_ must **indistinguishable** against each other however _s_ in accoridng to _const string_ type definition must have restricted number of operations according to _ss_.
However you allowed to do:
```cpp
const string s = "test";
string ss = s;

const string& rs = ss;
```
Compiler allows that b/c you can apply operation restrictions to **const string** reference. You actually can propagate this logic between `type` -> `const type` all around - type can be derived to _const_ type with restricted number of operation but not in reverse.

That actually widely used to pass arguments into functions. Let's get this example to illustrate it:
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
If you will run this example you will get **ce**:
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
So if we just update definition of arguments adding _const_ modifier we will be able to use function for a both _const_ and normal types.
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

Note: since reference implement over pointer on the low level if your function gets `int` types it would be cheaper to copy them than create a _const_ reference (or just reference). However for any types bigger than 4 bytes constant reference must choice by default.

Now let's try to pass into our _find_ function string literal explicitly:
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
Program compiles fine however you can see passed _rvalue_ value "s". It means, it is allowed to **initialize const reference with _rvalue_**.
This case is called **lifetime expansion**:
- you can create const reference like `const string& test = "test";` initialized with _rvalue_ literal
- this object won't be destroyed until variable _test_ leaves its scope (function by example)

This example can illustrate **UB** (undefined behaivor) with dangling reference.
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
That may output "test" however if stack have some different values you also can get segmentation fault.

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
This program compiles with **ce** - does not matter if const _y_ reference on the _x_ casted correctly - we can't cast back from _const int&_ to _int&_ since C++ is statically typed language:
```cpp
main.cpp:8:10: error: binding reference of type 'int' to value of type 'const int' drops 'const' qualifier
    8 |     int& z = y;
      |          ^   ~
1 error generated.
```

Finally there is example how we can implicitly "change" constant reference:
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
output will be - `2`.


#### Constant pointers

Let's make a simple program with const pointer:
```cpp
int a = 1;
const int* p = &a;
++p;
```

This program will compile fine because applying _const_ we apply into value, not a pointer. Pointer just pointing on the constant value. However to change value under pointer (result of dereferencing of pointing) is not allowed:
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

how to make constant pointer but not constant value under pointer? we just need pointer definition in front of _const_ keyword:
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
will result **ce**:
```ascii
main.cpp:8:5: error: cannot assign to variable 'p' with const-qualified type 'int *const'
    8 |     ++p;
      |     ^ ~
```

Accordingly if we need const pointer on the constant we can write it up as:
```cpp
const int* const p = &x;
```
Then both pointer and value under pointer should be constants.
