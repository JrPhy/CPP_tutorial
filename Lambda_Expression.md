C/C++ 是沒有辦法在函數內再寫一個函數的，要在函數裡面呼叫另一個函數，就必須要在外面另外寫一個函數，利用 function pointer 或是直接呼叫的方式使用，例如 [sort](https://github.com/JrPhy/C_tutorial/blob/main/CH8-%E6%8C%87%E6%A8%99%E8%88%87%E5%87%BD%E6%95%B8.md#3-%E6%8C%87%E6%A8%99%E5%87%BD%E6%95%B8%E7%95%B6%E4%BD%9C%E5%BC%95%E6%95%B8)。不過 C++11 之後支援的 Lambda Expression，有點類似在函數內寫函數的做法，所以就可以在 main 裡面直接寫一個比大小的 Lambda Expression 並呼叫
```cpp
#include <iostream> 
#include <algorithm>
using namespace std; 

int main() { 
    int number[] = {3, 5, 1, 6, 9};
    auto cmp = [] (int a, int b) { return a < b; };
    std::sort(begin(number), end(number), cmp);
    for(const auto &i:number) std::cout << i << ", ";
    // 1, 3, 5, 6, 9
    std::cout << std::endl;
    return 0; 
} 
```
## 1. 呼叫
Lambda Expression 在使用上比較偏向函數，所以可以直接在 } 後面緊接著呼叫即可
```cpp
#include <iostream> 
#include <algorithm>
using namespace std; 

int main() { 
    int cmp = [] (double a, double b) {
        return a + b;
    }(6.5, 4.9);
    std::cout << cmp << std::endl;
    // 回傳為 double，但因為 cmp 為整數，故會被轉型
    [] { printf("Hello World\n"); }()
    // 直接呼叫 Lambda Expression
    // Hello World
    return 0; 
} 
```
## 2. 結構
Lambda Expression 以 [] 做開頭，() 裡面放傳入的參數，{} 裡面放實作。所以 ```[](){}```在 C++ 中為合法的寫法，表示有一 Lambda Expression 不抓外部參數也沒有任何實作。接著來對各部分做詳細的介紹。由於 Lambda Expression 是在某個函數內，所以出了該函數範圍即無法作用。

#### 1. lambda-introducer
所有的 Lambda Expression 都是以它來作為開頭，不可以省略。裡面放 = 所有的變數都以傳值的方式抓取，& 則代表都傳參考，不寫就代表不抓外部變數。也可以給特定的值傳值或參考，特定的值傳值可以不用寫 =，傳參考則需要 &。
[]：不抓取外部的變數。
[=]：所有的變數都以傳值的方式抓取。
[&]：所有的變數都以傳參考的方式抓取。
[x, &y]：x 傳值、y 傳參考。
[=, &y]：y 傳參考，其餘的變數皆傳值。
[&, x]：x 傳參值，其餘的變數皆傳參考。
預設抓取的選項要放在第一個，給定抓取的要放在後面。

#### 2. 參數列表與實作
與一般函數幾乎無異，但因為 Lambda Expression 已包含函數實作，所以每個參數都需要有名稱。如果不需要參數的話也可以直接省略小括號。
```cpp
[] (int, int) { return a < b; } // 錯誤，未命名參數
```

#### 3. mutable (非必要)
可以讓 Lambda Expression 直接修改以傳值方式抓取進來的外部變數，也就是若 Lambda Expression 外有個同名的變數，那麼在 Lambda Expression 的該變數可以跟外部的值不同。
```cpp
#include <iostream> 

int main() { 
    int x = 10;
    auto f = [=]() mutable -> void {
        x = 20;
        std::cout << x << std::endl;
    };
    f(); // 20
    std::cout << x << std::endl // 10
    return 0; 
} 
```
#### 4. throw (非必要)
函數會丟出的例外，其使用的方法跟一般函數的例外指定方式相同。

#### 5. -> 回傳值 (非必要)
與一般函數相同
mutable：compound-statement，亦稱為 Lambda 主體（lambda body）。
這個就是匿名函數的內容，就跟一般的函數內容一樣。
