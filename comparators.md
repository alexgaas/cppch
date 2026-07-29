## Function Call Overloading and Comparators

In C++, you can overload the function call operator (`operator()`) for your classes, allowing class objects to be used like functions. Let's illustrate this with a simple example:

```cpp
#include <iostream>

class GreaterThanZero {
public:
    bool operator()(int x) const {
        return x > 0;
    }
};

int main() {
    GreaterThanZero g;
    std::cout << g(10) << std::endl;
    return 0;
}
```

As you can see, an instance of our class can now be called like a function. The `operator()` can have any return type and accept any number of arguments. You can also overload `operator()` with different parameter types.

This allows function objects to be passed directly to standard algorithms:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

class GreaterThanZero {
public:
    bool operator()(int x) const {
        return x > 0;
    }
};

int main() {
    std::vector<int> v = {1, 2, -1};
    std::cout << *std::find_if(v.begin(), v.end(), GreaterThanZero()) << std::endl;
    return 0;
}
```

This will output `1`.

Classes that overload `operator()` are known as function objects, and an instance of such a class is called a **functor**.

Now let's look at comparators. A comparator is a function object whose `operator()` compares two elements (typically implementing a less-than comparison for strict weak ordering).

```cpp
#include <iostream>
#include <map>
#include <cmath>

struct cmp {
    bool operator()(int x, int y) const {
        return std::abs(x) < std::abs(y);
    }
};

int main() {
    std::map<int, int, cmp> m;
    m[-20] = 2;
    m[-10] = 1;
    std::cout << m.begin()->second << std::endl;
    return 0;
}
```

This outputs `1`, demonstrating that the comparator worked as expected by ordering keys by their absolute values.
