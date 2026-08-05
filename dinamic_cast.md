## RTTI and _dynamic_cast_
Let's look on the simple example:
```cpp

#include <iostream>

struct TBase{
    virtual void f() { std::cout << "base\n"; }
};

struct TDerived: TBase {
    void f() override { std::cout << "derived\n"; }
};

int main(){

    return 0;
}
```
the question - how compiler really knows what function to call in this example? Compiler is not able to get this information from comile-time analysis, however it can generate some code in runtime to help itself make choice.

