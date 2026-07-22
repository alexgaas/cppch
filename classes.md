## Classes and structures

#### Baseline
Classes and structs in C++ are the same except for default access modifiers (_private_ / _public_). 
- Class fields/methods are _private_ by default
- Struct fields/methods are _public_ by default
- Inheritance by default in classes is _private_
- Inheritance by default in structs is _public_

You can add as many access modifiers as you want in a class/struct.

Let's see an example of how to build a simple class:
```cpp
#include <iostream>

using namespace std;

class Test{
    private:
        int test = 0;

    public:
        void getTest(){
            cout << test << endl;
        }
};

int main(){
    Test t;
    t.getTest();
    return 0;
}
```
You can't initialize a class with the expression:
```cpp
Test t();
```
Since the compiler assumes that _t_ is a function declaration with a return type of _Test_.

You can do:
```cpp
Test t = Test();
```
That will call the default constructor - it is empty for us so we can do it.

We can define class methods outside of the class definition using the class name as a qualified identifier. Class methods and fields will be available in the function defined outside of the class definition:
```cpp
#include <iostream>

using namespace std;

class Test{
    private:
        int test = 0;

    public:
        void getTest();
};

void Test::getTest(){
    cout << test << endl;
}

int main(){
    Test t = Test();
    t.getTest();
    return 0;
}
```
