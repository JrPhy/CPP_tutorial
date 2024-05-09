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

## 2. 左值引用
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
#### 1. 函數傳入引用
通常來講為左值引用，也就是把該變數的位置與值傳入，並回傳變數的位置與值，寫法上是在型別後加上一個 &，呼叫時直接傳入變數即可。
```cpp
#include <iostream>
void swap(int &i, int &j) {  
    int temp = i;
    i = j;
    j = temp;
}

int main ()
{
    int x = 10, y = 20;
    swap(x, y);
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
## 2. 右值引用
如果右值
