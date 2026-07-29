## Const Member Functions and `operator[]` Overloading

Overloading `operator[]` is a great example of when to use `const` member functions. Let's start with a basic implementation of `operator[]`:

```cpp
#include <initializer_list>
#include <iostream>
#include <cstring>

class TString {
public:
    TString() = default;

    TString(size_t sz, char ch) : len_(sz), str_(new char[sz]) {
        memset(str_, ch, len_); 
    }

    TString(std::initializer_list<char> list) : len_(list.size()), str_(new char[list.size()]) {
        std::copy(list.begin(), list.end(), str_);
    }

    TString(const TString& t) : len_(t.len_), str_(new char[t.len_]) {
        memcpy(str_, t.str_, len_);
    }

    char& operator[](size_t idx) {
        return str_[idx];
    }

    ~TString() {
        delete[] str_;
    }

private:
    char* str_ = nullptr;
    size_t len_ = 0;
};

int main() {
    TString s({'a', 'b', 'c'});
    std::cout << s[0] << std::endl;
    return 0;
}
```
Output: `a`

However, what happens if we attempt to access elements of a `const` object?
```cpp
const TString s({'a'});
std::cout << s[0];
```
Compiling this results in a **compilation error**:
```ascii
main.cpp:36:19: error: no viable overloaded operator[] for type 'const TString'
   36 |     std::cout << s[0] << std::endl;
      |                  ~^~
```

#### Overloading `operator[]` for Const Objects

To solve this, we need a separate `operator[]` overload marked as a **const member function**:

```cpp
    // Non-const overload: permits reading and writing
    char& operator[](size_t idx) {
        return str_[idx];
    }

    // Const overload: permits reading from const objects
    // Returns char by value since primitive types are cheap to copy (for larger types, return const T&)
    char operator[](size_t idx) const {
        return str_[idx];
    }
```

Placing `const` after a member function signature (`char operator[](size_t idx) const`) guarantees that the function will not modify any non-mutable data members of the class.

Attempting to mutate a data member inside a `const` member function results in a **compilation error**:
```cpp
char operator[](size_t idx) const {
    len_++; // Error!
    return str_[idx];
}
```
Error:
```ascii
main.cpp:25:17: error: cannot assign to non-static data member within const member function 'operator[]'
   25 |             len_++;
      |             ~~~~^
```

#### Best Practices for Const Member Functions

- A `const` member function can be called on both `const` and non-`const` instances of a class.
- If a member function does not modify the logical state of an object, it should always be marked `const`.
- **Note**: Constructors cannot be marked `const` because their purpose is to initialize the object.

#### The `mutable` Keyword

Sometimes a `const` member function needs to modify a specific internal variable (such as a call counter, a cache, or a thread lock). You can mark such data members with the `mutable` keyword, allowing them to be modified even inside `const` member functions:

```cpp
#include <initializer_list>
#include <iostream>
#include <cstring>

class TString {
public:
    TString() = default;

    TString(size_t sz, char ch) : len_(sz), str_(new char[sz]) {
        memset(str_, ch, len_); 
    }

    TString(std::initializer_list<char> list) : len_(list.size()), str_(new char[list.size()]) {
        std::copy(list.begin(), list.end(), str_);
    }

    TString(const TString& t) : len_(t.len_), str_(new char[t.len_]) {
        memcpy(str_, t.str_, len_);
    }

    char& operator[](size_t idx) {
        return str_[idx];
    }

    char operator[](size_t idx) const {
        access_count_++; // Allowed because access_count_ is mutable!
        return str_[idx];
    }

    size_t get_access_count() const {
        return access_count_;
    }

    ~TString() {
        delete[] str_;
    }

private:
    char* str_ = nullptr;
    size_t len_ = 0;
    mutable size_t access_count_ = 0;
};

int main() {
    const TString s({'a', 'b', 'c'});
    std::cout << s[0] << std::endl;
    std::cout << "Access count: " << s.get_access_count() << std::endl;
    return 0;
}
```
Output:
```ascii
a
Access count: 1
```

#### References and Const Member Functions

Consider a scenario where a class contains a reference member:

```cpp
#include <initializer_list>
#include <iostream>
#include <cstring>

int temp = 0;

class TString {
public:
    TString() = default;

    TString(size_t sz, char ch) : len_(sz), str_(new char[sz]) {
        memset(str_, ch, len_); 
    }

    TString(std::initializer_list<char> list) : len_(list.size()), str_(new char[list.size()]) {
        std::copy(list.begin(), list.end(), str_);
    }

    TString(const TString& t) : len_(t.len_), str_(new char[t.len_]) {
        memcpy(str_, t.str_, len_);
    }

    char& operator[](size_t idx) {
        return str_[idx];
    }

    char operator[](size_t idx) const {
        y++; // Modifies 'temp' through reference 'y'
        return str_[idx];
    }

    ~TString() {
        delete[] str_;
    }

private:
    int& y = temp;
    char* str_ = nullptr;
    size_t len_ = 0;
};

int main() {
    const TString s({'a', 'b', 'c'});
    std::cout << s[0] << std::endl;
    std::cout << temp << std::endl;
    return 0;
}
```
Output:
```ascii
a
1
```
`const` member functions enforce immutability on the reference member itself (the reference cannot be rebound), but do not prevent modifying the external object pointed to by a non-const reference member (shallow constness).

Note that combining `const` and `mutable` specifiers on the same data member (e.g., `mutable const int x;`) is invalid and results in a **compilation error**.
