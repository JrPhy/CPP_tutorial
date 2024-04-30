C/C++ 是沒有辦法在函數內再寫一個函數的，要在函數裡面呼叫另一個函數，就必須要在外面另外寫一個函數，利用 function pointer 或是直接呼叫的方式使用，例如 [sort](https://github.com/JrPhy/C_tutorial/blob/main/CH8-%E6%8C%87%E6%A8%99%E8%88%87%E5%87%BD%E6%95%B8.md#3-%E6%8C%87%E6%A8%99%E5%87%BD%E6%95%B8%E7%95%B6%E4%BD%9C%E5%BC%95%E6%95%B8)。不過 C++11 之後支援的 Lambda Expression，有點類似在函數內寫函數的做法，所以就可以在 main 裡面直接寫一個比大小的 Lambda Expression 並呼叫
```cpp
#include <iostream> 
#include <algorithm>
using namespace std; 

int main() { 
    int number[] = {3, 5, 1, 6, 9};
    std::sort(begin(number), end(number),
            [] (int a, int b) { return a < b; });
    for(const auto &i:number) std::cout << i << ", ";
    // 1, 3, 5, 6, 9
    std::cout << std::endl;
    return 0; 
} 
```
## 1. 呼叫
Lambda Expression 在使用上比較偏向函數，如果沒有回傳值則必須緊接在 } 後面呼叫，否則編譯器會報錯。
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
所有的 Lambda Expression 都是以它來作為開頭，不可以省略。裡面放 = 所有的變數都以傳值的方式抓取，& 則代表都傳參考，不寫就代表不抓外部變數。也可以給特定的值傳值或參考，特定的值傳值可以不用寫 =，傳參考則需要 &。整理如下
1. []：不抓取外部的變數。
2. [=]：所有的變數都以傳值的方式抓取。
3. [&]：所有的變數都以傳參考的方式抓取。
4. [x, &y]：x 傳值、y 傳參考。
5. [=, &y]：y 傳參考，其餘的變數皆傳值。
6. [&, x]：x 傳參值，其餘的變數皆傳參考。\
預設抓取的選項要放在第一個，給定抓取的要放在後面。如果不抓取外部變數的話，則可以像 function pointer 一樣使用

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
與一般函數相同，但也可以不寫。若有回傳值則可以接在等號後面，沒有的話可以直呼叫。
```cpp
int cmp = [] (double a, double b) { return a + b; }(6.5, 4.9);
// 回傳為 double，但因為 cmp 為整數，故會被轉型
[] { printf("Hello World\n"); }()
// 直接呼叫 Lambda Expression
// Hello World
```

## 3. Lambda Expression 的好處
1. 在寫 Lambda Expression 時因為可以緊鄰著使用處，不像呼叫某個函數時會離得很遠，所以要修改也很方便
2. 在寫一些簡短的表達式或判斷是就不用再去另外寫個 inline 函數，可以減少編譯的時間
3. 對於相同表達式但不同值的函數可以直接做修改，例如在 sort 的最後一個引數式決定要升序還降序，此時就需要寫兩個函數，但是使用 Lambda Expression，我們就可以直接寫在最後面，不用另外在寫一個函數。
```cpp
std::vector<int> numbers { 1, 2, 3, 4, 5, 10, 15, 20, 25, 35, 45, 50 };
std::sort(numbers.begin(), numbers.end(),
        [] (int a, int b) { return a < b; });
// 升序，由小到大
std::sort(numbers.begin(), numbers.end(),
        [] (int a, int b) { return a > b; });
// 降序，由小到大
```
std::count_if 是個更好的例子，數出一個陣列內有多少 > n 的元素
```cpp
#include <iostream>
#include <algorithm>
#include <vector>
int main() {
    std::vector<int> numbers { 1, 2, 3, 4, 5, 10, 15, 20, 25, 35, 45, 50 };
    // 使用 lambda expression 替代原有的 condition 函數
    int count_10 = std::count_if(numbers.begin(), numbers.end(),
                                [](int x) { return (x > 10); });
    std::cout << "Count > 10: " << count_10 << std::endl;
    int count_15 = std::count_if(numbers.begin(), numbers.end(),
                                [](int x) { return (x > 15); });
    std::cout << "Count > 15: " << count_15 << std::endl;
}
```
## 4. 泛型 Lambda Expression
在 C++11 中僅支援特定型別的 Lambda Expression，所以在寫 sort 時仍然需要寫許多個 cmp。
```cpp
#include <iostream> 
#include <algorithm>
using namespace std; 

int main() { 
    int number[] = {3, 5, 1, 6, 9};
    double numberd[] = {4.3, 2.5, 5.1, 6.6, 4.9};
    auto icmp = [] (int a, int b) { return a < b; };
    auto dcmp = [] (double a, double b) { return a < b; };
    // 回傳雖然為 int，但如果前面用 int 宣告
    // 則表示 icmp 與 dcmp 為一個變數，就不能傳入 sort 中。
    // 但可以另外宣告一個函數指標去接再傳入
    // bool(*icmp)(int, int) = [] (int a, int b) { return a < b; };
    std::sort(begin(number), end(number), icmp);
    std::sort(begin(numberd), end(numberd), dcmp);
    for(const auto &i:number) std::cout << i << ", ";
    // 1, 3, 5, 6, 9
    std::cout << std::endl;
    for(const auto &i:numberd) std::cout << i << ", ";
    // 1, 3, 5, 6, 9
    std::cout << std::endl;
    return 0; 
} 
```
在 C++14 之後的版本就支援傳入 auto 的參數讓編譯器自動推導，這樣就不用再寫個別型別的 cmp 了，可以讓 Lambda Expression 更泛用。
```cpp
#include <iostream> 
#include <algorithm>
using namespace std; 

int main() { 
    int number[] = {3, 5, 1, 6, 9};
    double numberd[] = {4.3, 2.5, 5.1, 6.6, 4.9};
    auto cmp = [] (auto a, auto b) { return a < b; };
    // 回傳雖然為 int，但如果前面用 int 宣告
    // 則表示 icmp 與 dcmp 為一個變數，就不能傳入 sort 中。
    std::sort(begin(number), end(number), cmp);
    std::sort(begin(numberd), end(numberd), cmp);
    for(const auto &i:number) std::cout << i << ", ";
    // 1, 3, 5, 6, 9
    std::cout << std::endl;
    for(const auto &i:numberd) std::cout << i << ", ";
    // 1, 3, 5, 6, 9
    std::cout << std::endl;
    return 0; 
} 
```
