## Stream Input / Output Operator Overloading

Overloading stream insertion (`<<`) and stream extraction (`>>`) operators is implemented using non-member (free) functions that return a reference to `std::ostream` or `std::istream` to support operator chaining (e.g., `std::cout << a << b;`).

#### Stream Insertion Operator (`<<`)

Here is an example of overloading `operator<<` for custom output formatting:

```cpp
#include <initializer_list>
#include <iostream>
#include <cstring>
#include <ostream>

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
        return str_[idx];
    }

    ~TString() {
        delete[] str_;
    }

    size_t len() const {
        return len_;
    }

private:
    char* str_ = nullptr;
    size_t len_ = 0;
};

// Non-member stream insertion operator
std::ostream& operator<<(std::ostream& out, const TString& s) {
    for (size_t i = 0; i < s.len(); i++) {
        out << s[i];
    }  
    return out; 
}

int main() {
    const TString s({'a', 'b', 'c'});
    std::cout << s << std::endl; // Outputs: abc
    return 0;
}
```

#### Stream Extraction Operator (`>>`) and `friend` Declarations

Overloading the stream extraction operator (`>>`) often requires writing to private data members of the class. To grant a non-member function access to private and protected members, declare the function as a `friend` inside the class definition:

```cpp
#include <initializer_list>
#include <iostream>
#include <cstring>
#include <istream>
#include <ostream>

class TString {
    // Declare friend non-member stream operators
    friend std::ostream& operator<<(std::ostream& out, const TString& s);
    friend std::istream& operator>>(std::istream& in, TString& s);

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
        return str_[idx];
    }

    ~TString() {
        delete[] str_;
    }

    size_t len() const {
        return len_;
    }

private:
    char* str_ = nullptr;
    size_t len_ = 0;
};

std::ostream& operator<<(std::ostream& out, const TString& s) {
    for (size_t i = 0; i < s.len_; i++) { // Direct access to private len_ and str_
        out << s.str_[i];
    }  
    return out; 
}

std::istream& operator>>(std::istream& in, TString& s) {
    char ch;
    if (in >> ch) {
        // Example: append character or modify private members directly
        delete[] s.str_;
        s.len_ = 1;
        s.str_ = new char[1];
        s.str_[0] = ch;
    }
    return in;
}

int main() {
    TString s({'a', 'b', 'c'});
    std::cout << "Original: " << s << std::endl;

    std::cout << "Enter a single character: ";
    std::cin >> s;
    std::cout << "Updated: " << s << std::endl;
    return 0;
}
```

#### Friend Classes and Encapsulation

You can also declare an entire class as a friend, granting all its member functions access to private members:
```cpp
friend class C;
```

> **Design Warning**: The `friend` keyword should be used judiciously because it breaks encapsulation. Note also that friendship is **neither inherited nor transitive** (if class A is a friend of class B, and class B is a friend of class C, class A is not automatically a friend of class C).
