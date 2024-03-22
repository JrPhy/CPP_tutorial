在 C/C++ 語言中雖然我們可以利用 *void 指標來模擬泛型，但在撰寫時還是需要開發者自行做轉型，例如 [qsort](https://github.com/JrPhy/C_tutorial/blob/main/CH8-%E6%8C%87%E6%A8%99%E8%88%87%E5%87%BD%E6%95%B8.md#3-%E6%8C%87%E6%A8%99%E5%87%BD%E6%95%B8%E7%95%B6%E4%BD%9C%E5%BC%95%E6%95%B8)。而 #define 雖然也可以達到相同的效果，但是因為編譯器會直接展開，除了會讓 code size 增加外，也有可能會出現一些未預期的問題
```c
#include <iostream>
#define max(a, b) (a > b ? a : b)
#define add(a, b) (a+b)
int main() {
    std::cout << max(10, 3.9) << std::endl;
    std::cout << add(10, 3.9) << std::endl;
    return 0;
}
```
經過編譯器之後該函數會直接被展開，如果大量使用的話會讓整個 code 變得很大。
```cpp
#include <iostream>
#define max(a, b) (a > b ? a : b)
#define add(a, b) (a+b)
int main() {
    std::cout << (10 > 3.9 ? a : b) << std::endl;
    std::cout << (10 + 3.9) << std::endl;
    return 0;
}
```
雖然在 C++ 中可以在類別函數中去[重載函數](https://github.com/JrPhy/CPP_tutorial/blob/main/class_%E5%A4%9A%E5%9E%8B_virtual_override_final.md#1-%E5%87%BD%E6%95%B8%E9%87%8D%E8%BC%89)，但就要寫很多個函數，C++ 另外提供樣板(template)來讓開發者寫泛型函數。
## 1. 樣板 template
樣板的使用格式為 ```template<typename T>``` 或是 ```template<class T>```，兩者在 C++ 中完全一樣。樣板可以用在函數跟類別，但是樣板跟函數必須緊臨，中間不可再宣告其他物件。
#### 1. 樣板函數 template function
如果只有宣告一個 template，那麼傳入的型別必須都一樣。例如下方的 add 函數只有用一個 T，則 a, b 跟回傳值的型別必須都為 int 或是 double，如果是分別 int 跟 double，則編譯器會報錯。
```cpp
#include <iostream>
template<typename T>
T add(T a, T b) {return a+b;}
template<typename T> // 不可忽略
void swap(T& a, T& b) {T temp = a; a = b; b = temp;}
int main() {
    std::cout << add(3, 8) << std::endl;
    std::cout << add(4.2, 3.9) << std::endl;
    // std::cout << add(4.2, 9) << std::endl; // 會報錯
    int a = 10, b = 5;
    swap(a, b);
    std::cout << a << "  " << b << std::endl;
    return 0;
}
```
如果要使用不同的型別，可以另外寫明或是在寫一個 template
```cpp
#include <iostream>
template<typename T, typename U>
T add(T a, U b) {return a+b;}
double dadd(T a, int b) {return a+b;}
int main() {
    std::cout << add(4.2, 9) << std::endl;
    // 第一個為浮點數，故 T 為浮點數，所以會回傳浮點數 13.2
    // 第二個為整數，故 U 為浮點數
    std::cout << add(9, 9.6) << std::endl;
    // 第一個為整數，故 T 為整數，所以會回傳整數 18
    // 第二個為浮點數，故 U 為浮點數
    std::cout << dadd(9, 9.6) << std::endl; // 回傳浮點數 18.0
    return 0;
}
```
#### 2. 樣板類別 template class
在使用 vector 時會寫成 ```std::vector<int>``` 就是使用樣板類別。在 C 語言中的 linked-list 需要先把結構宣告好，之後加入時就需要把正確的型別放到正確的位置。而在 C++ 中就可以用 template class 來像 vector 一樣有類似的宣告
```cpp
template <typename T>
class LinkedList {
    class Node {
    public:
        Node(T value, Node *next) : value(value), next(next) {}
        T value;
        Node *next;
    };

    Node *first = nullptr;

public:
    LinkedList<T>& append(T value);
    T get(int i);
};
```
