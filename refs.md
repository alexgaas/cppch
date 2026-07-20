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
and run `./a.out`. You defentiely may expect result as **`1 2`** because nothing changed for **x** / **y**. That happens because we passed into function **swap** _local copies_ of **x/y** varibles therefore exchange inside of function did not affect variable defined in the _main_.
This is how C++ defines semantic of passing arguments into function.

Let's fix our program using **pointers**. 
```cpp
void swap(int* x, int* y){
    int* t = x;
    *x = *y;
    y = t;
}
```
Now build and run will show us expected result - `2 1`. However you can keep old _swap_ function semantic using **references**. The idea of _reference_ is looking pretty strightforward - when user defines entity as a reference, he expects to use not as a copy when passing into function but as a **same** enityy just with a **different name**.
so running this function:
```cpp
void swap(int& x, int& y){
    int t = x;
    x = y;
    y = t;
}

```
gives a same correct result `2 1` instead of incorrect `1 1`.

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
Here is compiler have to create and destroy to separate objects _v_ and _vv_ when program is done. However if we change [vv] to be created as a **reference** compiler will destroy _vv_ at the time _v_ should be destroyed.
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
    // y is reference of x => read as same x but within a different name y
    int& y = x;
    // z is reference on y eg another reference to x  => read as same x !! but within a different name z
    int& z = y;

    cout << x << " " << y << " " << z << endl;

    int a = 0;
    // when we do z = a, we actually do x = y = z = a !!
    z = a;

    cout << x << " " << y << " " << z << endl;

    return 0;
}
```
If ypu build and run it you will see:
```ascii
1 1 1
0 0 0
```
There is important moments this program demonstrates:
- _z_ is not reference to reference, it is another reference to _x_
- when user does `z = a`, _x_ becames 0, but NOT _z_ becomes reference to _a_

let's incremenet _a_ to show _z_ is not reference to _a_:
```cpp
...
a++;

cout << x << " " << y << " " << z << " " << a << endl;
```
result will be as - `0 0 0 1`

Summarizing that we can say if reference to the entity is defined we have to consider that reference as just referenced object just with different **alias** (name) until reference is valid. Reference is not distingushed from referenced object. As soon as user defined _y_ as reference on _x_ he would not be able to get difference between _x_ and _y_ besides names (beside of _decltype_ what will be considered separately).

In most of cases, we need _references_ to pass them into functions, however references might be class members or just used as more correct aliases for objects in the functions. 

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

But you can define reference for a pointer b/c pointer is C++ type and can be referenced as other types.
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
As you may see in this example _pp_ is just alias of the _p_ pointer!

You not allowed to create reference on reference like:
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
Reference must initialized with _lvalue_ only (in according to C++ standard every literal besides string literal is _rvalue_).

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
As you may see we try to return reference on the object already destroyed in the scope of the function _f_ into _main_ what makes reference _x_ in the main aliasing non-existed object e.g "dead" reference. Execution of this code will rise **ub** e.g. undefined behaivor of your C++ program. That may return any value or just make segmentation fault, etc.
