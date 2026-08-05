## Virtual Inheritance

To resolve the diamond problem in multiple inheritance, C++ provides **virtual inheritance**. Virtual inheritance guarantees that only one shared instance of a common base class exists within a derived object, regardless of how many inheritance paths lead to it.

To use virtual inheritance, specify the `virtual` keyword when deriving from the common base class:

```cpp
#include <iostream>

struct TZero {
    int z = 0;
};

struct TBase1 : virtual TZero {
    int b1 = 1;
};

struct TBase2 : virtual TZero {
    int b2 = 2;
};

struct TDerived : TBase1, TBase2 {
    int d = 2;
};

int main() {
    TDerived d;
    std::cout << d.z << std::endl; // Unambiguous: single shared TZero instance
    return 0;
}
```

Output:
```ascii
0
```

### Memory Layout with Virtual Inheritance

Without virtual inheritance, memory contains two separate `TZero` subobjects:
`[TBase1::z][TBase1::b1][TBase2::z][TBase2::b2][TDerived::d]`

With virtual inheritance, the shared `TZero` subobject is placed at the end of the memory layout, and the intermediate classes store virtual base pointers (`vbptr`) or offsets to locate the shared `TZero` subobject:
`[TBase1 vbptr][TBase1::b1][TBase2 vbptr][TBase2::b2][TDerived::d][Shared TZero]`

### Trade-offs of Virtual Inheritance

1. **Memory Overhead**: Each class inheriting virtually stores an internal virtual base pointer (`vbptr`) or offset, increasing object size.
2. **Performance Impact**: Accessing members of a virtual base class requires pointer indirection at runtime.
3. **Construction Responsibility**: The most derived class (`TDerived`) is responsible for directly invoking the constructor of the virtual base class (`TZero`). Intermediate constructors do not re-initialize the shared virtual base.
