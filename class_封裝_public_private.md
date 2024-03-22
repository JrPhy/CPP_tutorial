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
class 的宣告與 C 中的結構宣告非常類似，以 class 為開頭，再來就是型別名稱，並用左右花括號將成員包起來。
#### 1. 公有與私有
雖然 C++ 中類別的成員都是預設私有，但在實作時還是會說明那些為公有哪些為私有
```cpp
class Student {
   public:
    int ID;
    void showName() { std::cout << name << std::endl; };
    void setName(std::string _name) { name = _name; };
   private:
    std::string name;
};
```
如果在 class 中有多個 private 跟 public，那麼在 private ~ public 中的成員為 private，public ~ private 中的成員為 public，private/public ~ } 中的成員為 private/public。
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
若為私有成員，則只有該 class 中的成員可以對他做操作，而公有成員則可以直接作賦值。
#### 1. 公有成員變數初始化
如果在 class 中的成員變數皆為 public，那麼可以像結構一樣給他照順序初始化
```cpp
#include <iostream>
#include <string>
class Student {
   public:
    int ID;
    int gender;
    std::string name;
    double score;
};

int main() {
    Student peter = {10039, 0, "peter", 88.88};
    Student Mary;
    Mary.ID = 10039;
    Mary.gender = 1;
    Mary.name = "Mary";
    Mary.score = 88.88;
}
```
當然也可以在宣告類別時就直接給值，但就無法像結構那樣直接初始化，而且對於之後的使用也需要在去做修改，所以通常會在類別內宣告一個建構子來做初始化。
#### 2. 成員函數實作
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
    void showName() { std::cout << name << std::endl; };
    void setName(std::string _name) { name = _name; };
   private:
    std::string name;
};

class Teacher {
   public:
    std::string subject;
    void showName() { std::cout << name << std::endl; };
    void setName(std::string _name) { name = _name; };
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
若是在成員變數中有需要牽涉 new/malloc 和 delete/free 的操作，則會使用建構子(constructor)與解構子(destructor) 來幫忙做記憶體管理。若是在 class 中沒有實作，則編譯器會幫你實作，但如果是在繼承時可能會造成記憶體洩漏(Memory Leak)。而這兩個必須為公有成員，且名稱皆與類別名稱相同
```cpp
class Student {
   public:
    Student(int ID) { std::cout << "constructor Student" << std::endl; };
    // 建構子
    ~Student() { std::cout << "destructor Student" << std::endl; };
    // 解構子
    int ID;
    void showName() { std::cout << name << std::endl; };
    void setName(std::string _name) { name = _name; };
   private:
    std::string name;
};
```
#### 1. 建構子
類別的物件是在呼叫時才會去要記憶體空間，如同一般變數一樣，若是沒有在宣告時就賦值，那麼裡面的成員變數會預設是 0 跟 '\0'，有了建構子後我們就可以對**公有成員**變數做初始化。而如果一類別有私有成員的話，就無法像結構那樣直接初始化，需要藉由成員函數，但就需要先宣告此類別，然後再呼叫該成員函數，這時我們就可以利用建構子來對私有成員變數初始化。
```cpp
#include <string>
#include <iostream>
class Student {
    int ID;
    int gender;
    std::string name;
    double score;
   public:
    Student(int _id, int _gender, std::string _name, double _score);
    void get() { std::cout << ID << " " << gender << " " << name  << " " << score << std::endl; }
};

Student::Student(int _id, int _gender, std::string _name, double _score)
        :ID(_id), gender(_gender), name(_name), score(_score) {}

int main()
{
    Student Peter(10039, 0, "Peter", 88.88);
    Peter.get();
    return 0;
}
```
因為皆為私有變數，也沒有一個接口可以去修改這些變數，所以被初始化後就無法再修改了。要注意在做初始化時仍是按照類別內的變數順序做初始化，所以在建構時也需要注意順序，下方為一個初始化錯誤的例子
```cpp
#include <string>
#include <iostream>
class Student {
    int ID;
    int gender;
    std::string name;
    double score;
   public:
    Student(int _id, int _gender, std::string _name, double _score);
    void get(){std::cout << ID << " " << gender << " " << name  << " " << score << std::endl;}
};

Student::Student(int _id, int _gender, std::string _name, double _score)
        :gender(_gender), ID(gender),  name(_name), score(_score) {}

int main()
{
    Student Peter(10039, 1, "Peter", 88.88);
    Peter.get();
    return 0;
}
```
上方例子雖然是 gender 寫在前方，但是因為類別內的變數是 ID 寫在前方，此時 gender 尚未被初始化，所以 ID 就會被初始化為 0。

#### 2. this 指標
一般來說函數引數的名稱會盡量明顯讓使用者知道，而私有成員變數名稱也會盡量明顯讓開發者知道，此時如果要用函數來改變私有變數，就有可能會遇到狀名的問題。一個解法就是在其中一個名稱前或後加上底線，另一個則是用 this 指標
```cpp
#include <string>
#include <iostream>
class Student {
    int ID;
    int gender;
    std::string name;
    double score;
   public:
    Student(int _id, int _gender, std::string _name, double _score);
    void get(){std::cout << ID << " " << gender << " " << name  << " " << score << std::endl;}
    void setScore(double score) { this->score = score; }
};

Student::Student(int _id, int _gender, std::string _name, double _score)
        :gender(_gender), ID(gender), name(_name), score(_score) {}

int main()
{
    Student Peter(10039, 1, "Peter", 88.88);
    Peter.get();
    Peter.setScore(95.8);
    Peter.get();
    return 0;
}
```
#### 3. 解構子
在 C/C++ 中如果沒有使用到 new/malloc，那麼在程式結束後記憶體會自動釋放，如果有的話就需要自行去做 delete/free。一個類別中如果有使用到 new/malloc，在解構時就需要有寫 delete/free 來釋放記憶體。解構子不採用任何引數，且沒有傳回類型。無法取得其位址。解構子無法宣告為 const、 volatile、 const volatile 或 static，但可以宣告為 virtual 或 pure virtual。
```cpp
#include <string>
#include <iostream>
class Student {
    int ID;
    int gender;
    int *rank;
    std::string name;
    double score;
   public:
    Student(int _id, int _gender, std::string _name, double _score);
    ~Student() { std::cout << " ~Student() " <<std::endl; delete rank;};
    void get(){ std::cout << ID << " " << gender << " " << name  << " " << score << std::endl; }
    void SetRank(int rank) { *(this->rank) = rank; }
    int GetRank() { return *rank; }
};

Student::Student(int _id, int _gender, std::string _name, double _score)
        :gender(_gender), ID(gender),  name(_name), score(_score) {
    rank = new int;
}

int main()
{
    Student Peter(10039, 1, "Peter", 88.88);
    Peter.get();
    Peter.SetRank(2);
    std::cout << Peter.GetRank();
    return 0;
}
```
一個類別被建構後如果沒有寫解構子，那麼在繼承後到程式結束，就有可能會沒有釋放到記憶體造成記憶體洩漏(Memory Leak)。所以最好是在每個類別中都預設一個建構子與解構子。
