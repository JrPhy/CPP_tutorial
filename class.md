類別(class)在 C++ 中是用來實現封裝(encapsulation)與抽象(abstraction)，也就是拿來做程式的介面。封裝的目的是為了將程式模組化，封裝出來的東西我們就稱為 Class。類似於 C 語言中的結構(struct)，所以若要用 C 語言來完成物件導向，可以用**結構+指標**來達到 class 的特性。而 C++ 作為 C 語言的拓展，這兩個都有支援，不同的地方在於結構中的成員預設為公有，而類別為私有，所以 class 也是一種使用者自定義的型別。
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
## 1. 宣告與使用
class 的宣告與 C 中的結構宣告非常類似，以 class 為開頭，再來就是型別名稱，並用左右花括號將成員包起來。雖然 C++ 中類別的成員都是預設私有，但在實作時還是會說明那些為公有哪些為私有
```cpp
class Student {
   public:
    int ID;
    void showName() {std::cout << name << std::endl;};
    void setName(std::string _name) {name = _name};
   private:
    std::string name;
};
```
如果在 class 中有多個 private 跟 public，那麼在 private~public 中的成員為 private，public~private 中的成員為 public，private/public~} 中的成員為 private/public。
```cpp
class Student  // class declaration
{
   private:
    私有成員
   public:
    公有成員
   private:
    私有成員
   public:
    公有成員
}; // 結尾一樣需要分號
```
而在 C++ 中的 class 與 struct 成員都可以有函數宣告，函數的實作可以在 class 外也可以在 class 內。
```cpp
void Stock::show() {
    ...
}
```
在 class 外實作時，需要在前面加上 className::。雙冒號 :: 在 C++ 中為告訴編譯器該成員是屬於哪個 class。若為 public 成員，有實作後就可以直接在 main 裡面做使用
```cpp
int main()
{
    Student Mary;
    Mary.show();
}
```
如果在不同的 class 中有相同的成員函數名稱，會根據在哪個 class 中去呼叫，賦值時也會根據是對哪個物件的成員作賦值。
```cpp
#include <iostream>
#include <string>
class Student {
   public:
    int ID;
    void showName() {std::cout << name << std::endl;};
    void setName(std::string _name) {name = _name};
   private:
    std::string name;
};

class Teacher {
   public:
    std::string subject;
    void showName() {std::cout << name << std::endl;};
    void setName(std::string _name) {name = _name};
   private:
    std::string name;
};

int main()
{
    Student Mary, John;
    Mary.ID = 1;
    John.ID = 2;
    Mary.setName("Mary");
    John.setName("John");
    // 把
    Teacher Georgy;
    Georgy.subject = "math\n";
    Georgy.setName("Georgy");
    Mary.showName(); // Mary
    John.showName(); // John
    Georgy.showName(); // Georgy
    std::cout << Mary.ID << "  " << John.ID << "  " << Georgy.subject  << std::endl;
}
```

## 2. 建構子與解構子
