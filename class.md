類別(class)在 C++ 中是用來實現封裝(encapsulation)與抽象(abstraction)，也就是拿來做程式的介面。封裝的目的是為了將程式模組化，封裝出來的東西我們就稱為 Class。類似於 C 語言中的結構(struct)，所以若要用 C 語言來完成物件導向，可以用結構+指標來達到 class 的特性。而 C++ 作為 C 語言的拓展，這兩個都有支援，不同的地方在於結構中的成員預設為公有，而類別為私有。
```cpp
#include <iostream>

class cT {
    int val;
};

struct sT {
    int val;
};

int main()
{
    cT t1;
    sT t2;
    t1.val = 10; // error: ‘int cT::val’ is private within this context
    t2.val = 10;
}
```

## 1. 封裝 Encapsulation
