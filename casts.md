## Type casting

#### C-style cast

C++ has a simple way to cast inherited from C (it's called C-style cast):
```cpp
double d = 0.0;
int x = (int) d;
```

C-style cast uses a combination of the casts below to use all possible ways to cast. From [cppreference](https://en.cppreference.com/cpp/language/explicit_cast): 
```html
1) When the C-style cast is encountered, the compiler attempts to interpret it as the following cast expressions, in this order:
a) const_cast<type-id ﻿>(unary-expression ﻿);
b) static_cast<type-id ﻿>(unary-expression ﻿), with extensions: pointer or reference to a derived class is additionally allowed to be cast to pointer or reference to unambiguous base class (and vice versa) even if the base class is inaccessible (that is, this cast ignores the private inheritance specifier). Same applies to casting pointer to member to pointer to member of unambiguous non-virtual base;
c) a static_cast (with extensions) followed by const_cast;
d) reinterpret_cast<type-id ﻿>(unary-expression ﻿);
e) a reinterpret_cast followed by const_cast.
 The first choice that satisfies the requirements of the respective cast operator is selected, even if it is ill-formed (see example). If a static_cast followed by a const_cast is used and the conversion can be interpreted in more than one way as such, the conversion is ill-formed.
 In addition, C-style casts can cast from, to, and between pointers to incomplete class type. If both type-id and the type of unary-expression are pointers to incomplete class types, it is unspecified whether static_cast or reinterpret_cast gets selected.
```

It is highly recommended to use specialized types of casts (see below) instead of C-style casts. C-style cast does not provide any **ce** in case of error, instead trying to combine casts "to the end".

#### _static_cast_
Example:
```cpp
#include <iostream>

using namespace std;

void f(int) { cout << "1" << endl; }
void f(double) { cout << "2" << endl; }

int main(){
    int x = 1;
    f(static_cast<double>(x));
    return 0;
}
```
Output: `2`

When we make a _static_cast_, we create a new object from the old one as a result of the cast (e.g., a copy of the object). If _static_cast_ can't cast, it will produce a **ce**.

#### _reinterpret_cast_
When you use _reinterpret_cast_, you do not create a new object; you cast into the current object, so you need to do a cast on the reference of the object.
```cpp
#include <iostream>

using namespace std;

int main(){
    double pi = 3.14;
    cout << reinterpret_cast<int&>(pi) << endl;
    return 0;
}
```
The key thing you need to know about _reinterpret_cast_: if the casted type is not derived from the base type, it is clearly **ub**. Mostly, it is a very dangerous way to cast anything and this cast should not be used without good reasoning.
This kind of cast is usually used to cast _pointers_ between each other:
```cpp
int x = 1;
int* p = &x;
double* pd = reinterpret_cast<double*>(p);
...

```
As a result - _reinterpret_cast_ casts bytes of one type as bytes of another. This is a dangerous way that may lead to **ub** (in fact a segmentation fault) and this cast must be used very carefully (likely between pointers only, not references!).

#### _const_cast_
This cast allows casting a non-constant type to const or vice versa.
```cpp
#include <iostream>

using namespace std;

void f(int&) { cout << "1" << endl; }

void f(const int&) { cout << "2" << endl; }


int main(){
    int n = 1;
    f(const_cast<const int&>(n));
    return 0;
}
```
Will output `2`. 

Please note: 
- _const_cast_ works only with references and pointers
- _const_cast_ does not make a new copy of the object, it reinterprets the same object according to cast rules, the same as _reinterpret_cast_

As you may see, the example above for _const_cast_ is a bit weird - if we comment `void f(int&) { cout << "1" << endl; }` and run the code again without _const_cast_, it will cast implicitly without any cast constructs:
```cpp
#include <iostream>

using namespace std;

//void f(int&) { cout << "1" << endl; }

void f(const int&) { cout << "2" << endl; }


int main(){
    int n = 1;
    //f(const_cast<const int&>(n));
    f(n);
    return 0;
}
```
The result is also `2`!

Now let's see how _const_cast_ works to cast from a const type to a non-const type:
```cpp
#include <iostream>

using namespace std;

void f(int&) { cout << "1" << endl; }

void f(const int&) { cout << "2" << endl; }


int main(){
    int n = 1;
    const int& cn = n;

    f(const_cast<int&>(cn));

    return 0;
}
```
Will result in `1` as expected.

As you may see, _const_cast_ allows removing the _const_ restriction from an object. However, this type of cast is in fact **ub**. Example:
```cpp

#include <iostream>

using namespace std;

void f(int& n) { 
    // since reference must incr n in [main]
    ++n; 
    cout << "1" << endl; 
}

void f(const int&) { cout << "2" << endl; }


int main(){
    const int n = 5;
    const int& cn = n;

    f(const_cast<int&>(cn));
    cout << n << endl;

    return 0;
}
```
Will output
```ascii
1
5
```
As you may see, the value casted from _const_ to _int_ was not incremented correctly because the compiler does not save const ints on the stack as normal ints; instead, it just puts them into the code at compile time. That's why casting from a _const_ to a normal type is mostly a bad idea.

#### _dynamic_cast_
Will be considered later as part of inheritance/polymorphism topics.

