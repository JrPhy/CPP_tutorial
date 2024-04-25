C/C++ 是一個很接近硬體的語言，所以使用 C/C++ 在開發時很常會使用指標，不過使用指標時會需要自行去銷毀，不然會有記憶體洩漏(Memory Leak)的問題。而指標可以由一個指標 A 指向另一個 B，如果改變了 B 或銷毀了 B，那麼 A 的值也會跟著改變或銷毀。在 C++ 提供了 Smart Pointer 來讓開發者避免這風險。C++11 提供了三種 Smart Pointer，分別是 unique_ptr、shared_ptr、weak_ptr，使用時需要加上 ```<memory>```。C 使用 malloc/free 來宣告與銷毀指標，C++ 使用 new/delete 來宣告與銷毀指標，以下分開來介紹

## 1. Unique_ptr
在 C/C++ 中宣告指標並不會隱式轉型，型別必須一致才可以。
#### 1. 宣告
Unique_ptr 顧名思義就是只有一個擁有者，當宣告了一個 Unique_ptr 無法直接賦值給其他指標。
```cpp
#include <iostream> 
#include <memory> 
  
int main()
{
    std::unique_ptr<int> pInt(new int(5));
    unique_ptr<int> pInt2(pInt);    // 錯誤
    unique_ptr<int> pInt3 = pInt;   // 錯誤
    std::unique_ptr<double> pInt(new int(5));  // 型別不一致
    cout << *pInt;
}
```
以上宣告了一個型別為 int，名稱為 pInt 的 Unique_ptr 指向了一個 int 的指標。如果宣告的型別與指向的型別不同也會報錯，在 C++14 則提供了 make_unique 來避免這個問題
```cpp
#include <iostream> 
#include <memory> 
  
int main()
{
    auto ptr = std::make_unique<int>(5);
    cout << *pInt;
}
```
