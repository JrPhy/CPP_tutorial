C/C++ 是一個很接近硬體的語言，所以使用 C/C++ 在開發時很常會使用指標，不過使用指標時會需要自行去銷毀，不然會有記憶體洩漏(Memory Leak)的問題。而指標可以由一個指標 A 指向另一個 B，如果改變了 B 或銷毀了 B，那麼 A 的值也會跟著改變或銷毀。在 C++ 提供了 Smart Pointer 來讓開發者避免這風險。C++11 提供了三種 Smart Pointer，分別是 unique_ptr、shared_ptr、weak_ptr，使用時需要加上 ```<memory>```。C 使用 malloc/free 來宣告與銷毀指標，C++ 使用 new/delete 來宣告與銷毀指標，以下分開來介紹

## 1. unique_ptr
在 C/C++ 中宣告指標並不會隱式轉型，型別必須一致才可以。
#### 1. 宣告
unique_ptr 顧名思義就是只有一個擁有者，當宣告了一個 Unique_ptr 無法直接賦值給其他指標，如果需要這麼做，則需要用到 ```std::move```，經過 move 後原指標已不存在，其餘使用上的操作則與一般 pointer 沒兩樣。
```cpp
#include <iostream> 
#include <memory> 
  
int main()
{
    std::unique_ptr<int> pInt(new int(5));
    std::unique_ptr<int> pNull{};
    //std::unique_ptr<int> pInt2(pInt);    // 錯誤
    //std::unique_ptr<int> pInt3 = pInt;   // 錯誤
    std::unique_ptr<int> b = std::move(pInt);
    //std::unique_ptr<double> pInt1(new int(5));  // 型別不一致
    std::cout << *b; // 取值
}
```
以上宣告了一個型別為 int，名稱為 pInt 的 Unique_ptr 指向了一個 int 的指標。如果宣告的型別與指向的型別不同也會報錯，在 ```C++14``` 則提供了 make_unique 來避免這個問題。除此之外因為不直接使用 new，所以當程式拋出異常後也會把前面建構的 class 解構。
```cpp
#include <iostream> 
#include <memory>

class C {
    // ...
}

void pass_unique_ptr(std::unique_ptr<C> a, std::unique_ptr<C> b){
    // ...
}
  
int main()
{
    std::unique_ptr<int> pInt(new MyClass());
    std::unique_ptr<MyClass> pInt1 = std::make_unique<MyClass>(5);
    // 如果第一個 new 失敗那就不會 delte，第二個就不會執行，
    pass_unique_ptr(std::unique_ptr<C>(new C()), std::unique_ptr<C>(new C()));
    // make_unique 保證所有參數都會被建構
    pass_unique_ptr(std::make_unique<C>(), std::make_unique<C>());
    std::cout << *pInt;
}
```
當然也可以自行實作，利用回傳一個 unique_ptr 來實現 make_unique。
```cpp
template<typename T, typename... Args>
std::unique_ptr<T> make_unique(Args&&... args)
{
    return std::unique_ptr<T>( new T(std::forward<Args>(args)...) );
}
```
#### 2. 傳遞與回傳 unique_ptr
因為 unique_ptr 只能有**一個實體**，所以在傳遞時都是傳 ref 或是用 get() 而非使用 std::move，兩種寫法對應不同的引數寫法。如果使用 std::move，那就是把 unique_ptr 的所有權轉移到 local variable 上，函數結束就會自動被銷毀而造成錯誤。而函數回傳則與一般函數寫法沒兩樣，如果有賦值給其他指標則會預設使用 std::move，否則直接釋放。
```cpp
#include <iostream> 
#include <memory>
void func1(unique_ptr& a) {
    // ...
}

void func2(int *b) {
    // ...
}

unique_ptr<int> ret_unique_ptr()
{
    return std::make_unique<int>(5)
}

int main() {
    std::unique_ptr<int> pInt(new int(5));
    func1(pInt);
    //func2(pInt); // 錯誤
    func2(pInt.get());
    auto ptr{ ret_unique_ptr() };
    // 把 ret_unique_ptr 建立的指標所有權轉給 ptr
    return 0;
}
```
#### 3. 不要混用一般指標與 unique_ptr
unique_ptr 提供了 get() 函數來取得原始指標，因為 unique_ptr 只有一個實體，所以當函數內有個普通指標 A 指向 unique_ptr，那函數結束前銷毀普通指標 A，那麼 unique_ptr 也會一併被銷毀，這時就會造成程式 crash。
```cpp
#include <iostream> 
#include <memory>
void func2(int *b) {
    // b 指向了 unique_ptr
    std::cout << *b;
    *b = 7;
    // 刪除 b 也就刪除了 unique_ptr
    delete b;
}

int main() {
    std::unique_ptr<int> pInt(new int(5));
    func2(pInt.get());
    std::cout << *pInt;  // pInt 已被 delete
    return 0;
}
```
## 2. shared_ptr
顧名思義是可以讓多個 std::shared_ptr 共享一份記憶體，在執行時可有多個實例共享該指標，並且在最後一個 std::shared_ptr 生命週期結束時時自動釋放記憶體。所以在內部有一個 count 來知道有幾分共用，並提供了 use_count() 函數來取得目前共用的分數，當 count == 0 時才會銷毀。
#### 1. 宣告
宣告方式與 unique_ptr 類似，不過 shared_ptr 可以直接賦值，C++14 也提供了 make_shared 來讓開發者實例化一個 shared_ptr。
```cpp
#include <iostream> 
#include <memory> 
  
int main()
{
    std::shared_ptr<int> pInt(new int(5));
    std::shared_ptr<int> pInt2(pInt);
    std::shared_ptr<int> pInt3 = pInt;
    //std::unique_ptr<double> pInt1(new int(5));  // 型別不一致
    std::cout << *pInt; // 取值
    std::cout << pInt.use_count(); // 3
}
```
#### 2. [引用計數](https://www.cnblogs.com/tianshihao/p/14378918.html)
當一個 shared_ptr 被建構或是被拷貝時，引用計數就會 +1，解構時 -1，實際上是有一個私有變數在 shared_ptr 類別中，而非使用 static 變數來實作。

#### 3. 避免循環
因為 shared_ptr 可以被共用，所以有可能出現 A 指向 B，然後 B 也指向 A，這樣就形成了循環，最後就無法自動釋放 shared_ptr。
```cpp
#include<iostream>
#include<memory>
class B;
class A{
   public:
    A()  { std::cout << "A" << std::endl; }
    ~A() { std::cout << "delete A" << std::endl; }
    std::shared_ptr<B> a_;
};
class B{
   public:
    B()  { std::cout << "B" << std::endl; }
    ~B() { std::cout << "delete B" << std::endl; }
    std::shared_ptr<A> b_;
};
int main()
{
    std::shared_ptr<A> a = std::make_shared<A>();
    std::shared_ptr<B> b = std::make_shared<B>();
    {
        a->a_ = b;
        b->b_ = a;
        std::cout << a.use_count() << endl;
        std::cout << b.use_count() << endl;
    }
    std::cout << a.use_count() << endl;
    std::cout << b.use_count() << endl;
    return 0;
}
```
上述在 class A 中有 shared_ptr 指向 B，class B 中有 shared_ptr 指向 A，導致最後出了作用域後無法去解構，使得 count 的值無法 -1。這可以將 class 中的 shared_ptr 改為 weak_ptr 來解決。

## 3. weak_ptr 
weak_ptr 是弱共享的智慧指標，它一般指向 shared_ptr 指向的物件，但不會增加 use_count 的計數。此指標大部分與 shared_ptr 同時使用。

## 4. [使用情境](https://www.zhihu.com/question/648170767/answer/3428590625)
基本上 unique_ptr 會使用的比較多，因為效能幾乎跟一般指標一樣，除非要用再多執行緒的程式。而 shared_ptr 只有計數的部分是原子操作，在多執行緒使用 shared_ptr 改值時還是需要上鎖才能取到正確的值。
