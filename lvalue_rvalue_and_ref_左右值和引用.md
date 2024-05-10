在 C++ 中除了可以傳值以外還可以傳參考(或引用)，**參考(或引用)其實就是變數的別名**，所以傳參考就是把變數的值與位置傳進去。而值又分成左值(lvalue)和右值(rvalue)，在程式語言中是比較底層的概念，比較簡單的分法就是**只能**在等號右邊的為右值，可以在等號左右邊的為左值，即為變數。
```cpp
int a; // a 為左值
a = 3; // 3 為右值
(a+1) = 3; // (a+1) 為左值
3 = a; // 錯誤，右值不能在左邊
int b = a; // b 為左值
```
在上述例子中 a 為左值，3 為右值。
## 1. 左值與右值的定義
最初的定義為**左值是可取其地址，則可修改其值，有持久性。右值一般是不可取其地址，為無名的*臨時*對象。** 在加入了 const 和 volatile 這兩個關鍵字後，我們就必須修改其定義，C 語言中的相關定義在規格書中的 [6.3.2.1](https://www.dii.uchile.cl/~daespino/files/Iso_C_1999_definition.pdf)。有關 C++ 中的相關定義可參考[這篇文章](https://uint128.com/2020/01/17/%E5%AF%B9C-%E5%B7%A6%E5%80%BC%E5%92%8C%E5%8F%B3%E5%80%BC%E7%9A%84%E7%90%86%E8%A7%A3/)。

## 2. 左值引用 & 某個變數引用左值
因為引用就是變數的別名，一個變數包含其名稱，位置與值，所以在定義一個左值引用時，必須要指名是誰的引用。
```cpp
int a = 3; // 變數定義
//int &b = 3; // 錯誤，3 沒有位置
int &b = a; // 正確，引用 a
b = 10; // b 為左值
const int &c = 3; // 正確
// 因為 c 的值已綁定
```
使用方式有點像指標，在上述例子中 b 就為 a，改變其中一個值兩個變數都會一起改動。
#### 1. 函數傳入(左值)引用
C 語言在傳參數時只有 Pass by value，其行為是會先複製一份資料，再把複製的那份傳進去。如果是傳地址，也是會先複製一份**地址的位置**，傳進去後取該地址的值再做其他事。所以當我們在傳遞參數進函數時，其實會先複製，如果是**傳引用(不論是左值或右值引用)**就可以少掉一次複製，在速度上會比較快。通常來講為左值引用，也就是把該變數的位置與值傳入，並回傳變數的位置與值，寫法上是在型別後加上一個 &，呼叫時直接傳入變數即可。
```cpp
#include <iostream>
void swap_ref(int &i, int &j) {  
    int temp = i;
    i = j;
    j = temp;
}

void swap_pointer(int *i, int *j) {  
    int temp = *i;
    *i = *j;
    *j = temp;
}//pointer ver.

int main ()
{
    int x = 10, y = 20;
    swap_ref(x, y);
    printf("%d  %d\n", x, y);
    swap_pointer(&x, &y); //pointer ver.
    printf("%d  %d\n", x, y);
    return 0;
}
```
在 C 語言中的 swap 傳入的部分是要傳指標，並在實作部分取值，呼叫時傳入變數的位置。而若是傳入 C++ 的引用，則只需要直接賦值即可，呼叫時也向傳值一樣呼叫即可，非常直觀。不過如果呼叫時是傳入一個數值而非變數，則需要在前方加上 const
```cpp
#include <iostream>
int add(int &i, int &j) { int temp = i+j; return temp; }
int add_const(const int &i, const int &j) { int temp = i+j; return temp; }

int main ()
{
    int x = 10, y = 20;
    add(x, y);
    add_const(x, y);
    add(10, 20); // 錯誤，引用未綁定值
    add_const(x, y);
}
```
雖然在傳變數的引用可以更有效率，但是在 C++ 中很常遇到需要傳 class 類別進函數中，例如 std::string。傳入時就會遇到一次建構，回傳時可能又會遇到另一次建構，在效能上就會較慢，此為左值引用的短版，在 C++ 中有右值引用來解決這問題。

#### 2. 函數回傳引用
引用回傳時要小心變數的生命週期，在 C 語言中通常是回傳值，所以是回傳右值。但是引用有變數的位置和值，當函數內的變數一離開函數後即被系統釋放，值自然也消失，故回傳引用時要小心變數的生命週期
```cpp
#include <iostream>
int& add(int i, int j) {  
    static int temp = i+j;
    // int temp = i+j;
    //若沒加上 static 則會得到未預期的結果
    return temp;
}

int main ()
{
    int x = 10, y = 20;
    int sum = add(x, y);
    printf("%d\n", sum);
    return 0;
}
```
## 2. 右值引用 && 某個變數引用右值
如果右值本身不是從左值轉來的，那麼當表達式結束後右值就會被系統銷毀。而右值引用的意思是有某個變數引用了右值，該變數本身也為左值，否則離開表達式後也會被銷毀。
```cpp
int a = 3; // 變數定義
int &&b = 3; // 正確，引用 3
// int &&c = a; // 錯誤，a 不為右值
int &&c = b+1; // 正確，b+1 為右值
int &&d = std::move(a);
// 正確，std::move 在此會將 a 轉為右值
```
基本上跟一般宣告變數沒有兩樣。當然在 C++ 中需要區分左右值就必定有其用處，就是在傳 class 型別參數的時候。

#### 1. 函數的行為
而如果是要回傳函數內的區域變數，因為生命週期的關係，函數回傳前會先複製一份另外的臨時對象，再回傳那份臨時對象。也就是會開一個大小與傳入回傳參數相同的臨時變數，再把該變數的值複製到該臨時變數，再回傳該臨時變數。當我們使用右值引用取接回傳值時，就可以直接將原本所創建的臨時對象的值傳出，省去了一次複製，這樣就能夠更快，此即為右值在 C++11 的重要性。當然在 C++ 中還有 class，再傳入值會複製一份再去做建構，所以傳右值的話效能會快非常多。比較常用到的就是在函數內宣告 C++ 字串並回傳
```cpp
std::string to_string(int value) {
    std::string s;
    // do something...
    return s;
}
```
回傳的時候會開一個大小與 s 的臨時變數 a，再把該變數 s 的值複製到該臨時變數 a，再回傳該臨時變數 a 的值，此稱為深拷貝(deep copy)。找了個[例子](https://www.51cto.com/article/714737.html)來說明這件事，下方例子中編譯時加上 ```-fno-elide-constructors``` 選項關閉優化
```cpp
#include <iostream>
#include <vector>

using namespace std;

class StringBuidler {
   public:
    char* str;
    int length;
    StringBuidler() {}
    StringBuidler(int len, char c) {
        this->str = new char[len];
        this->str[0] = c;
        this->length = len;
    }

    StringBuidler(const StringBuidler& s) {
        printf("StringBuidler: deep copy \n");
        this->length = s.length;
        this->str = new char[s.length];
        for (size_t i = 0; i < length; i++)
        { this->str[i] = s.str[i]; }
    }

    StringBuidler operator+(const StringBuidler& p) {

        StringBuidler tmp;

        tmp.length = this->length + p.length;
        tmp.str = new char[tmp.length];
        int index = 0;
        for (size_t i = 0; i < this->length; i++)
        { tmp.str[index++] = this->str[i]; }
        for (size_t i = 0; i < p.length; i++)
        { tmp.str[index++] = p.str[i]; }

        return tmp;
        }
};

int main()
{
    StringBuidler s1(10, 'a');
    StringBuidler s2(5, 'b');
    StringBuidler s3 = s1 + s2;
    printf("s3.length=%d, s1.length=%d, s2.length=%d \n",
             s3.length, s1.length, s2.length);
}
```
在上述例子中，在編譯器未優化的情況下，會呼叫兩次 deep copy，也就是先去建構一份 s1，然後要回傳的時候在複製一份 s1 並且回傳複製的那一份，所以會多一次建構與解構。
```
StringBuidler: deep copy
StringBuidler: deep copy
s3.length=15, s1.length=10, s2.length=5
```
#### 2. 函數傳右值引用 &&
當我們重載 StringBuidler 並傳入右值引用後就少建構與解構一次，也就是直接把 s1 的右值回傳回來。如果說 class 本身很龐大，那就會占掉非常多的記憶體與時間。
```cpp
StringBuidler(StringBuidler&& s) {
    this->str = s.str;
    this->length = s.length;
    s.str = nullptr;
}
```
在 C++11 後得 STL 容器也提供傳入右值引用的版本。
