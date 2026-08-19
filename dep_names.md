## Dependent Names

In C++ templates, a **dependent name** is an identifier whose meaning depends on the type or value of a template parameter.

Because templates are parsed in two phases (during definition and during instantiation), dependent names introduce syntactic ambiguities. The C++ standard defines specific disambiguation rules using the `typename` and `template` keywords, as well as rules for dependent base classes.

---

### The Syntactic Ambiguity Problem

Consider the following template and specialization:

```cpp
template <typename T>
struct S {
    using x = T; // x is a type alias
};

template <>
struct S<int> {
    static const int x = 5; // x is a static data member
};
```

Now, consider a function template attempting to use `S<T>::x`:

```cpp
int a = 10;

template <typename T>
void f() {
    S<T>::x * a; // Ambiguity: pointer declaration or multiplication?
}
```

During Phase 1 (parsing `f`), the compiler does not know what `T` will be:
- If `T` is `double`, `S<double>::x` is a type (`double`), making `S<T>::x * a;` a declaration of a pointer `a` to `double`.
- If `T` is `int`, `S<int>::x` is a value (`5`), making `S<T>::x * a;` an expression multiplying `5` by `a`.

---

### The `typename` Disambiguator

> [!IMPORTANT]
> **Default Rule:** By default, the compiler assumes that any qualified dependent name (e.g., `S<T>::x`) refers to a **value or variable**, not a type.

If you attempt to declare a variable using a dependent type without the `typename` keyword:

```cpp
template <typename T>
void f() {
    S<T>::x a; // Compilation Error!
}
```

Compiler Error:
```ascii
error: expected ';' after expression
    S<T>::x a;
           ^
```

To tell the compiler that `S<T>::x` is a type, prefix it with `typename`:

```cpp
template <typename T>
void f() {
    typename S<T>::x a; // OK: parsed as a type
}
```

---

### The `template` Disambiguator

When a member of a dependent type is itself a template, a similar ambiguity occurs with the `<` character:

```cpp
template <typename T>
struct S {
    template <int N>
    struct A {};
};

int s = 0;

template <typename T>
void f() {
    S<T>::A<10> s; // Ambiguity: template instantiation or (S<T>::A < 10) > s?
}
```

By default, the compiler parses `<` as the "less-than" comparison operator rather than the start of a template argument list.

If we add `typename` without the `template` keyword:

```cpp
typename S<T>::A<10> s; // Compilation Error!
```

Compiler Error:
```ascii
error: use 'template' keyword to treat 'A' as a dependent template name
    typename S<T>::A<10> s;
                   ^
                   template 
```

To resolve this, use `template` after the scope resolution operator (`::`):

```cpp
template <typename T>
void f() {
    typename S<T>::template A<10> s; // OK: parsed as a dependent member template type
}
```

---

### Dependent Base Classes

When a class template derives from a template base class, the base class is a **dependent base class**:

```cpp
template <typename T>
struct Base {
    int x = 0;
};

template <typename T>
struct Derived : Base<T> {
    void f() {
        x = 1; // Compilation Error!
    }
};
```

Compiler Error:
```ascii
error: use of undeclared identifier 'x'
        x = 1;
        ^
```

#### Why Does This Happen?
During Phase 1 name lookup, unqualified names (like `x`) are only looked up in non-dependent scopes. The compiler does not look inside `Base<T>` because a future explicit specialization of `Base<T>` (e.g., `template <> struct Base<int>`) might not even have a member named `x`.

#### Solutions

To access members of a dependent base class, you must explicitly make the lookup dependent on `T`:

```cpp
template <typename T>
struct Derived : Base<T> {
    // Solution 1: Use 'using' declaration
    using Base<T>::x;

    void f() {
        // Solution 2: Explicit 'this->' pointer
        this->x = 1;

        // Solution 3: Fully qualified base class name
        Base<T>::x = 2;

        // Valid due to Solution 1:
        x = 3;
    }
};
```
