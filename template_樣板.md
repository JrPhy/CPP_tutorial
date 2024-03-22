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
樣板的使用格式為 ```template<typename T>``` 或是 ```template<class T>```，兩者在 C++ 中完全一樣。樣板可以用在函數跟類別，但是樣板跟函數或類別必須緊臨，中間不可再宣告其他物件。
#### 1. 樣板函數 template function
如果只有宣告一個 template，那麼傳入的型別必須都一樣。例如下方的 add 函數只有用一個 T，則 a, b 跟回傳值的型別必須都為 int 或是 double，如果是分別 int 跟 double，則編譯器會報錯。如果函數宣告跟實作分開，那麼宣告跟實作前面都要加上 ```template<typename T>```
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
也可以寫明型別
```cpp
#include <iostream>
template<typename T>
T add(T a, T b) {return a+b;}
template<typename T> // 不可忽略
void swap(T& a, T& b) {T temp = a; a = b; b = temp;}
int main() {
    std::cout << add<int>(3, 8) << std::endl;
    std::cout << add<float>(4.2, 3.9) << std::endl;
    // std::cout << add(4.2, 9) << std::endl; // 會報錯
    int a = 10, b = 5;
    swap<int>(a, b);
    std::cout << a << "  " << b << std::endl;
    return 0;
}
```
如果要使用不同的型別，可以另外寫明或是在寫一個 template
```cpp
#include <iostream>
template<typename T, typename U>
T add(T a, U b) {return a+b;}
template<typename T>
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
#### 2. 特化樣板
樣板函數可以在特定型別有不同的實作，因為樣板就是讓編譯器自動去推導型別，如果推導到特定的型別，那就會使用特定的函數，當然引數的個數還有傳入的性質都需與同名的函數相同，且特化樣板的尖括號中 < > 不放任何東西。
```cpp
#include <iostream>
template<typename T>
void swap(T& a, T& b) { T temp = a; a = b; b = temp; }
template<>
void swap<int>(int& a, int& b) { }
int main() {
    int a = 10, b = 5;
    swap<int>(a, b);
    std::cout << a << "  " << b << std::endl;
    // 10  5
    float c = 10.5, d = 5.5;
    swap(c, d);
    std::cout << c << "  " << d << std::endl;
    // 5.5  10.5
    return 0;
}
```
上述例子中有個樣板函數 swap，傳了兩個參考(reference)進去且不回傳值，下方有另個特化樣板函數 swap<int>，如果發現是傳入兩個整數型的參考就不做事。
#### 3. 樣板類別 template class
在使用 vector 時會寫成 ```std::vector<int>``` 就是使用樣板類別。在 C 語言中的 linked-list 需要先把結構宣告好，之後加入時就需要把正確的型別放到正確的位置，所以。而在 C++ 中就可以用 template class 來像 vector 一樣有類似的宣告
```cpp
#include <iostream>
template <typename T>
struct Node {
    T data;
    Node *next;
};

template <typename T>
class List{
   private:
    Node<T> *head;
   public:
    List() { head = NULL; }
    void push(T val){
        Node<T> *n = new Node<T>();   
        n->data = val;             
        n->next = head;        
        head = n;              
    }
    
    bool search(T val) {
        Node<T> *temp = head;
        while(temp->next) {
            if(temp->data == val) return true;
            else temp = temp->next;
        }
        return false;
    }

    void print() {
        Node<T> *temp = head;
        while(temp != NULL) {
            std::cout << temp->data << std::endl;
            temp = temp->next;
        }
        std::cout << std::endl;
    }
};

int main() {
    List<int> listi; // 整數型 list
    listi.push(1);
    listi.push(2);
    listi.push(3);
    listi.print();
    List<float> listf; // 浮點數型 list
    listf.push(1.1);
    listf.push(2.1);
    listf.push(3.1);
    listf.print();

    return 0;
}
```
