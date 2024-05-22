在 C 語言中 [const 關鍵字](https://github.com/JrPhy/C_tutorial/blob/main/CH7-%E7%94%9F%E5%91%BD%E9%80%B1%E6%9C%9F%E8%88%87%E5%8F%AF%E8%A6%96%E7%AF%84%E5%9C%8D.md#5-%E9%97%9C%E9%8D%B5%E5%AD%97-const)表示為只讀(read-only)的意思，也就是該變數不能被程式碼所改變。除了對變數的值作用外，也可以對指標位置設定只讀，而變數的值與位置也可以同時設定只讀。在 C++ 中也可將成員函式用 const 修飾。當然除了最基本的 const，在 C++ 中還多了 constexpr (C++11)、consteval (C++20)、constinit (C++20)，就來看看這些關鍵字的用處是什麼

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

## 2. constexpr 
如果想要拿到一個函數返回值並將其設為 const，在 C 是沒辦法辦到的，因為 square 是要在執行期才知道。
```cpp
int square(int n) {
    return n*n;
}
const int N = 123;
// const int sq_N = square(N); // 報錯
```
在 C++ 就可以用 constexpr 來修飾函數，就可以放入一個 const 變數，而該 const 變數也是無法被改變。
```cpp
constexpr int square(int n) {
    return n*n;
}
const int N = 123;
const int sq_N = square(N);
// sq_N = 55; read-only
```
