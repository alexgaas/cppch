## Static fields and methods
Classes may have static fields and methods. There is special qualifier _static_ to mark field / method for that. For compiler _static_ means object is created on the program initialization and lives to the end of program, all static objects preserved in the special memory arena called "static memory".
For us as users that mean correspodning static object belong not the instance of class but class itself.
```cpp
struct TType {
    static int x;
};

int main(){
    TType::x = 1;
    return 0;
}
```
Static fields can be very convinient to save any unified values for all class instances or class counters, etc.

Also we can make static methods same way. Best example of usage static method is an _Singleton_ pattern.
```cpp
class TType {
    private:
        static TType* obj = nullptr;

        TType(){}
    public:
        static TType& Get(){
            if (obj) return *obj;
            obj = new TType();
            return *obj;
        }
};

int main(){
    return 0;
}
```
this program will return **ce**:
```ascii
main.cpp:3:23: error: non-const static data member must be initialized out of line
    3 |         static TType* obj = nullptr;
      |                       ^     ~~~~~~~
1 error generated.
```

