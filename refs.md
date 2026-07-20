## REFERENCES

#### Baseline

Let's make a simple swap function - to switch arguments inside of function:

```cpp
#include <iostream>

using namespace std;

void swap(int x, int y){
    int t = x;
    x = y;
    y = t;
}

int main(){
    int x = 1, y = 2;
    swap(x, y);
    cout << x << " " << y << endl;
    return 0;
}
```
compile as:
```sh
g++ -std=c++20 -O2 main.cpp
```
and run `./a.out`. You definitely may expect result as **`1 2`** because nothing changed for **x** / **y**. That happens because we passed into function **swap** _local copies_ of **x/y** variables therefore exchange inside of function did not affect variables defined in the _main_.
This is how C++ defines semantic of passing arguments into function.

Let's fix our program using **pointers**. 
```cpp
void swap(int* x, int* y){
    int* t = x;
    *x = *y;
    y = t;
}
```
Now build and run will show us expected result - `2 1`. However you can keep old _swap_ function semantic using **references**. The idea of _reference_ is looking pretty straightforward - when user defines entity as a reference, he expects to use it not as a copy when passing into function but as the **same** entity just with a **different name**.
so running this function:
```cpp
void swap(int& x, int& y){
    int t = x;
    x = y;
    y = t;
}

```
gives the same correct result `2 1` instead of incorrect `1 1`.

Let's make a different more transparent example of how references work. 
Here is very simple program:
```cpp
#include <vector>

using namespace std;

int main(){
    // create vector instance with name [v]
    vector v = {1,2,3};
    // create different vector with name [vv] and copy into it [v] values
    // [vv] is not a reference, that is another vector instance!
    vector vv = v;
    return 0;
}
```
Here the compiler has to create and destroy two separate objects _v_ and _vv_ when the program is done. However if we change [vv] to be created as a **reference** the compiler will destroy _vv_ at the time _v_ should be destroyed.
```cpp
...
vector& vv = v;
...
```
Let's see one more even more simple scenario of references:
```cpp
#include <iostream>

using namespace std;

int main(){
    int x = 1;
    // y is reference to x => read as the same x but with a different name y
    int& y = x;
    // z is reference to y e.g. another reference to x => read as the same x !! but with a different name z
    int& z = y;

    cout << x << " " << y << " " << z << endl;

    int a = 0;
    // when we do z = a, we actually do x = y = z = a !!
    z = a;

    cout << x << " " << y << " " << z << endl;

    return 0;
}
```
If you build and run it you will see:
```ascii
1 1 1
0 0 0
```
There are important moments this program demonstrates:
- _z_ is not a reference to reference, it is another reference to _x_
- when user does `z = a`, _x_ becomes 0, but NOT _z_ becomes reference to _a_

let's increment _a_ to show _z_ is not reference to _a_:
```cpp
...
a++;

cout << x << " " << y << " " << z << " " << a << endl;
```
result will be as - `0 0 0 1`

Summarizing that, we can say if reference to the entity is defined we have to consider that reference as the referenced object itself just with a different **alias** (name) until the reference is valid. Reference is not distinguished from the referenced object. As soon as user defined _y_ as reference to _x_ he would not be able to get any difference between _x_ and _y_ besides names (besides _decltype_ which will be considered separately).

In most cases, we need _references_ to pass them into functions, however references might be class members or just used as more correct aliases for objects in the functions. 

#### Reference restrictions
You can't create pointer to a reference:
```cpp
int&* x;
```

C++ standard does not allow this operation so compiler will not compile this code.
```ascii
main.cpp:6:9: error: 'x' declared as a pointer to a reference of type 'int &'
    6 |     int&* x;
      |         ^
1 error generated.
```

But you can define a reference to a pointer because a pointer is a C++ type and can be referenced like other types.
```cpp
#include <iostream>

using namespace std;

int main(){
    int x = 0;
    int* p = &x;
    int*& pp = p;
    cout << p << " " << pp << endl; 
    return 0;
}
```
will output two same pointers - `0x16fcbed64 0x16fcbed64`.
As you may see in this example _pp_ is just the alias of the _p_ pointer!

You are not allowed to create reference to reference like:
```cpp
int&& y = x;
```
_&&_ is product of _rvalue_ reference and is not allowed to use as part of _lvalue_ expression (this case will be considered separately later).

Reference must be initialized:
```cpp
int& z;
```
won't compile:
```ascii
main.cpp:10:10: error: declaration of reference variable 'z' requires an initializer
   10 |     int& z;
      |          ^
1 error generated.
```

Reference can't be initialized using _rvalue_. Example:
```cpp
#include <iostream>

using namespace std;

void swap(int& x, int& y){
    int t = x;
    x = y;
    y = t;
}

int main(){
    // won't compile
    /*
     * main.cpp:13:10: error: non-const lvalue reference to type 'int' cannot bind to a temporary of type 'int'
   13 |     int& z = 1;
      |          ^   ~
1 error generated.
    */
    int& z = 1;

    // wont' compile
    /*
     main.cpp:12:5: error: no matching function for call to 'swap'
   12 |     swap(1,2);
      |     ^~~~
    */
    swap(1,2);

    return 0;
}
```
Reference must be initialized with _lvalue_ only (according to the C++ standard every literal besides string literal is _rvalue_).

#### Dangling references
Let's make a simple function _f_ to return reference:
```cpp
#include <iostream>

using namespace std;

int& f(){
    int x = 0;
    return x;
}

int main(){
    int x = f();
    cout << x << endl;
    return 0;
}
```
if we run this example we will get:
```ascii
g++ -std=c++20 -O2 main.cpp
main.cpp:7:12: warning: reference to stack memory associated with local variable 'x' returned [-Wreturn-stack-address]
    7 |     return x;
      |            ^
1 warning generated.

./a.out
1841869832
```
As you may see we try to return a reference to the object already destroyed in the scope of the function _f_ into _main_, which makes reference _x_ in the main alias a non-existent object e.g. a "dead" reference. Execution of this code will raise **ub** e.g. undefined behavior of your C++ program. That may return any value or just cause a segmentation fault, etc.
