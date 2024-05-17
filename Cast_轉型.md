在程式語言中不同的型別在記憶體中有不同的大小與[表示方法](https://github.com/JrPhy/C_tutorial/blob/main/CH1-%E6%95%B8%E5%80%BC%E5%9E%8B%E5%88%A5%E4%BB%8B%E7%B4%B9.md)，各數值間可以互相轉型，但是要小心數值溢位或有誤差，且需要小心隱式轉換。而[指標的轉型](https://github.com/JrPhy/C_tutorial/blob/main/CH5-%E6%8C%87%E6%A8%99%E8%88%87%E5%AD%97%E4%B8%B2.md#3-%E6%B3%9B%E5%9E%8B%E6%8C%87%E6%A8%99-void-)則需要藉由 void* 當作橋梁，C 語言中任何指標都需要經由 void* 來做轉型。在 C++ 因為多了 class，而且也對指標還有 const 類型轉型做了更多規範，並訂了四個轉型方法。

## 1. const_cast
const 在 C/C++ 中表示該物件的值或指標**無法被程式碼改變**，volatile 則是表示告訴編譯器，該變數會被**意想不到**的改變(可能是外部來的)，const_cast 則會把 const 或 volatile 的作用去掉。而比較常用到的場景是將某個 const 變數傳入函數中要將 const 去掉。
```cpp
#include <iostream>

void InputInt(int * num)
{ std::cout << *num << std::endl; }

int main()
{
    const int constant = 21;
    //InputInt(constant);
    //error C2664: invalid conversion from ‘int’ to ‘int*’ [-fpermissive]
    InputInt(const_cast<int*>(&constant));
    return 0;
}
```
再呼叫第三方函式庫時有可能僅提供非 const 的版本，但自己在撰寫時則是用 const 宣告要傳入的變數，此時就可以用到 const_cast 將 const 去掉並傳入函數中

## 2. static_cast
## 3. dynamic_cast
## 4. reinterpret_cast
