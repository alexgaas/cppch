## Arithmetic Operator Overloading

As an example of arithmetic operator overloading, let's overload the `+`, `+=`, and `++` operators for a complex number class `TType`.

#### Overloading `+` and `+=`

Here is an initial implementation of `+=` and binary `+`:

```cpp
#include <iostream>

class TType {
public:
    TType(double re, double im) : re(re), im(im) {}

    TType(const TType& t) : re(t.re), im(t.im) {
        std::cout << "copy" << std::endl;
    }

    // Overload compound assignment +=
    TType& operator+=(const TType& t) {
        re += t.re;
        im += t.im;
        return *this;
    }

    // Overload binary + (implemented in terms of +=)
    TType operator+(const TType& t) const {
        TType tt = *this;
        tt += t;
        return tt;
    }

private:
    double re;
    double im;
};

int main() {
    TType t(1, 2);
    TType tt = t + t;
    return 0;
}
```

Running this code prints `"copy"` only once. Naively, we might expect up to three copy operations:
1. Creating a local copy `TType tt = *this;` inside `operator+`.
2. Returning a copy of `tt` by value from `operator+`.
3. Initializing `tt` in `main()` via `TType tt = t + t;`.

However, C++ compilers perform **Return Value Optimization (RVO)** and **Named Return Value Optimization (NRVO)**. Instead of constructing `tt` locally on the function's stack frame and subsequently copying it into the caller's frame, the compiler constructs the return object directly inside the caller's target memory.

The compiler cannot always apply NRVO. For example, if we rewrite `operator+` as:
```cpp
// NRVO disabled because (tt += t) returns a reference
TType operator+(const TType& t) {
    TType tt = *this;
    return tt += t;
}
``` 
Here, `tt += t` returns a reference (`TType&`). The compiler cannot apply NRVO to construct the local variable directly in the return slot, forcing an additional copy from the reference into the returned value.

In addition, C++ compilers perform **copy elision** during initialization (`TType tt = t + t;`), ensuring no redundant temporary object is created when initializing `tt` from the function's return value.

#### Implementing `+` vs `+=` Order

What if we implement the operators in reverse—implementing `+=` in terms of binary `+`?
```cpp
TType operator+(const TType& t) const {
    TType tt = *this;
    tt.re += t.re;
    tt.im += t.im;
    return tt;
}

TType& operator+=(const TType& t) {
    // Requires creating a temporary copy!
    *this = *this + t;
    return *this;
}
```
Implementing `+=` in terms of `+` forces an unnecessary temporary creation and assignment. Implementing binary `+` in terms of `+=` is much more efficient and idiomatic.

#### Non-Member (Free) Operator Overloading

Now let's examine non-member operator functions. Suppose `TType` has a single-argument constructor:
```cpp
#include <iostream>

class TType {
public:
    TType(double re, double im) : re(re), im(im) {}
    TType(double im) : re(0.0), im(im) {}

    TType(const TType& t) : re(t.re), im(t.im) {}

    TType& operator+=(const TType& t) {
        re += t.re;
        im += t.im;
        return *this;
    }

    TType operator+(const TType& t) const {
        TType tt = *this;
        tt += t;
        return tt;
    }

private:
    double re;
    double im;
};

int main() {
    TType t(1, 2);
    t + 5.0; // Works via member operator+ (implicit conversion of 5.0 to TType)
    return 0;
}
```

What if we try the reverse expression: `5.0 + t`?
Attempting `5.0 + t` causes a **compilation error**: `Invalid operands to binary expression ('double' and 'TType')`.

This fails because member operators require the left operand to be an instance of the class:
```cpp
TType t(1, 2);
t.operator+(5.0); // Valid: equivalent to t + 5.0

// Invalid: 5.0 is a primitive double, not a class with member functions!
// 5.0.operator+(t);
```

To support symmetric operations where implicit conversions can apply to the left operand (e.g., `5.0 + t`), define `operator+` as a non-member (free) function:

```cpp
#include <iostream>

class TType {
public:
    TType(double re, double im) : re(re), im(im) {}
    TType(double im) : re(0.0), im(im) {}

    TType(const TType& t) : re(t.re), im(t.im) {}

    TType& operator+=(const TType& t) {
        re += t.re;
        im += t.im;
        return *this;
    }

private:
    double re;
    double im;
};

// Non-member (free) binary operator+
TType operator+(const TType& l, const TType& r) {
    TType sum = l;
    sum += r;
    return sum;
}

int main() {
    TType t(1, 2);
    TType res1 = t + 5.0; // Implicit conversion of 5.0 to TType(5.0) for right operand
    TType res2 = 5.0 + t; // Implicit conversion of 5.0 to TType(5.0) for left operand
    return 0;
}
```

#### Overloading the Increment Operator (`++`)

The increment operator has two forms: **prefix** (`++t`) and **postfix** (`t++`).

Prefix increment (`++t`) modifies the object and returns a reference to `*this`:

```cpp
#include <iostream>

class TType {
public:
    TType(double re, double im) : re(re), im(im) {}
    TType(double im) : re(0.0), im(im) {}
    TType(const TType& t) : re(t.re), im(t.im) {}

    TType& operator+=(const TType& t) {
        re += t.re;
        im += t.im;
        return *this;
    }

    // Prefix increment: ++t
    TType& operator++() {
        *this += 1.0;
        return *this;
    }

private:
    double re;
    double im;
};

int main() {
    TType t(1.0);
    ++t;
    return 0;
}
```

Postfix increment (`t++`) returns the object's previous value by value (as a copy). In C++, postfix `operator++` is distinguished from prefix `operator++` by taking a dummy `int` parameter:

```cpp
#include <iostream>

class TType {
public:
    TType(double re, double im) : re(re), im(im) {}
    TType(double im) : re(0.0), im(im) {}
    TType(const TType& t) : re(t.re), im(t.im) {}

    TType& operator+=(const TType& t) {
        re += t.re;
        im += t.im;
        return *this;
    }

    // Prefix increment: ++t (returns reference)
    TType& operator++() {
        *this += 1.0;
        return *this;
    }

    // Postfix increment: t++ (returns copy of old state)
    // The dummy 'int' parameter distinguishes postfix from prefix
    TType operator++(int) {
        TType copy = *this;
        ++(*this); // Delegate to prefix increment
        return copy;
    }

private:
    double re;
    double im;
};

int main() {
    TType t(1.0);
    ++t; // Calls prefix operator++()
    t++; // Calls postfix operator++(int)
    return 0;
}
```

> **Performance Tip**: Prefer prefix increment (`++t`) over postfix increment (`t++`) for custom types, as postfix increment must construct and return a temporary copy of the previous state.
