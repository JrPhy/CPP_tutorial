在 C 語言中 [const 關鍵字](https://github.com/JrPhy/C_tutorial/blob/main/CH7-%E7%94%9F%E5%91%BD%E9%80%B1%E6%9C%9F%E8%88%87%E5%8F%AF%E8%A6%96%E7%AF%84%E5%9C%8D.md#5-%E9%97%9C%E9%8D%B5%E5%AD%97-const)表示為只讀(read-only)的意思，也就是該變數不能被程式碼所改變。除了對變數的值作用外，也可以對指標位置設定只讀，而變數的值與位置也可以同時設定只讀。在 C++ 中也可將成員函式用 const 修飾。而 [static 關鍵字](https://github.com/JrPhy/C_tutorial/blob/main/CH7-%E7%94%9F%E5%91%BD%E9%80%B1%E6%9C%9F%E8%88%87%E5%8F%AF%E8%A6%96%E7%AF%84%E5%9C%8D.md#1-static)則是延長區域變數的生命週期與函數，和改變全域變數的可視範圍。而在 C++ 中 const/static 也是對類別成員多了其他效果。當然除了最基本的 const，在 C++ 中還多了 constexpr (C++11)、consteval (C++20)、constinit (C++20)，就來看看這些關鍵字的用處是什麼

## 1. const
C 語言中有的用法在 C++ 中也相同，但在 C++ 中 const 可置於**成員**函數前後。在前面表示此函數的回傳值不可被改變，而在後面表示此函數不會修改到成員變數，常用在 get 相關的函數。
```cpp
#include <string>
#include <iostream>
class Student {
    int ID;
    int gender;
    std::string name;
    std::string nickName = "TEST";
    double score;
   public:
    Student(int _id, int _gender, std::string _name, double _score);
    void get(){std::cout << ID << " " << gender << " " << name  << " " << score << std::endl;}
    void setScore(double score) { this->score = score; }
    int getID() const { /*ID = 10; 報錯*/ return ID; }
    const std::string& getName() { return name; }
    void setName(std::string name) { this->name = name; }
};
int main() {
    Student s;
    const std::string& Name = s.getName();
    std::cout << Name << "\n";
    // Name = "Peter"; // 報錯
    s.setName("Tom");
    std::cout << Name << "\n";
    return 0;
}
```

## 2. constexpr C++11
如果想要拿到一個函數返回值並將其設為 const，在 C 是沒辦法辦到的，因為 square 是要在執行期才知道。
```cpp
int square(int n)
{ return n*n; }
const int N = 123;
// const int sq_N = square(N); // 報錯
```
在 C++ 就可以用 constexpr 來修飾函數，就可以放入一個 const 變數，而該 const 變數也是無法被改變。
```cpp
constexpr int square(int n)
{ return n*n; }

int main() {
    const int N = 123;
    const int sq_N_c = square(44);
    const int sq_N_r = square(N);
    // sq_N = 55; read-only
    return 0;
}
```
從上方例子可以知道，不論在編譯或執行期都可成功編譯與執行，因為編譯器都會直接做掉。在 C++11 只能寫一些簡單的函數，若內部有其他流程或變數宣告則會錯誤，但在 C++14 之後就可以支援較複雜的函數。
```cpp
constexpr int factorial(int n) {
    int r = 1; // C++11 會報錯
    do {r *= n;} while (--n);
    return r;
}

int main() {
    const int N = 123;
    const int sq_N_c = square(44);
    const int sq_N_r = square(N);
    // sq_N = 55; read-only
    return 0;
}
```

## 2. consteval/constinit C++20
constexpr 是在編譯與執行期都可以成功執行。而在 C++20 中則是將 consteval/constinit 獨立出來，必須要在**編譯期**就能知道結果。這可以讓編譯性能提高。

## 3. static
在類別成員用 static 修飾，表示對於相同類別都**共用**一份靜態成員變數，且因為是共用，必須在外面賦值，不能在類別內賦值。也因為是共用的，所以靜態變數沒有 this 指標，靜態函數也無法取得 this 指標。
```cpp
#include <iostream>

class Object {
public:
    Object() {
        ++counter;
        std::cout << "counter = " << counter << std::endl;
    }
private:
    static int counter;
};

int Object::counter = 0; // initializing the static int

int main() {
    Object obj1; // 1
    Object obj2; // 2 
    Object obj3; // 3
    return 0;
}
```
也因為只有一份，所以在設計模式中有一種是**單例模式 Singleton**，一般來說是當一個 class 的 member function 做了很多事花很多資源，那就會使用到，通常會設計成 private static。
