## Fold expressions

Fold expressions been introduced in C++17. Let's see an example of fold expression:
```cpp

#include <iostream>

void print(){}

template<typename... Args>
void print(const Args&... args){
    (std::cout << ... << args) << std::endl;
}

int main() {
    print(1, 2, "abc");
    return 0;
}
```
Returns `12abc`.

How does it work and what does it do?

there is syntax of applying fold expression:
_(... operator args);_
that means:
_args1 op args2 op args_n

there are 4 forms of fold expression:
1. (... op args)
2. (args op ...)
3. (x op ... op args)
4. (args op ... op x);



