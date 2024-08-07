在程式語言中不同的型別在記憶體中有不同的大小與[表示方法](https://github.com/JrPhy/C_tutorial/blob/main/CH1-%E6%95%B8%E5%80%BC%E5%9E%8B%E5%88%A5%E4%BB%8B%E7%B4%B9.md)，各數值間可以互相轉型，但是要小心數值溢位或有誤差，且需要小心隱式轉換。而[指標的轉型](https://github.com/JrPhy/C_tutorial/blob/main/CH5-%E6%8C%87%E6%A8%99%E8%88%87%E5%AD%97%E4%B8%B2.md#3-%E6%B3%9B%E5%9E%8B%E6%8C%87%E6%A8%99-void-)則需要藉由 void* 當作橋梁，C 語言中任何指標都需要經由 void* 來做轉型。在 C++ 因為多了 class，而且也對指標還有 const 類型轉型做了更多規範，並訂了四個轉型方法。
```cpp
#include <iostream>

int main()
{
    int a = 21, b = 13, c = b/a;
    // b/a 為小數，但因為 c 為整數，所以會被隱式轉換成整數
    float d = (float)b/(float)a;
    // 在變數前面寫明型別，稱為顯示轉換
    return 0;
}
```
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
    //error C2664: invalid conversion from 'int' to 'int*' [-fpermissive]
    InputInt(const_cast<int*>(&constant));
    return 0;
}
```
再呼叫第三方函式庫時有可能僅提供非 const 的版本，但自己在撰寫時則是用 const 宣告要傳入的變數，此時就可以用到 const_cast 將 const 去掉並傳入函數中

## 2. static_cast 主要用於數值
static_cast 為顯式轉換的操作，即各變數之間的轉換都可使用 static_cast，可看成 C++ 版本的顯示轉型，對於各種數值類型的變數和指標的顯式轉換。
```cpp
double a = 1.999;
int b = static_cast<int>(a);
// 相當於 int b = (int)a ;
char* cptr = nullptr;
void* vptr = static_cast<void*>(cptr); // OK
// 相當於 void* vptr = (void*)cptr ;
// int* iptr = static_cast<int*>(cptr);
// error: invalid 'static_cast' from type 'char*' to type 'int*'
// 相當於 int* vptr = (int*)cptr ;
```
而在類別中的轉換，從子類轉為父類(向上轉)是安全的可以使用 static_cast，因為子類中的成員在父類都會有，反過來就不是，所以從父類轉子類(向下轉)就會有風險，因為沒有動態型別的檢查，通常這種情況需要使用 dynamic_cast。

## 3. dynamic_cast 主要用於父轉子
在繼承時如果基類中有個虛函數，表示繼承的子類有可能會複寫該函數，在執行時會生成一個虛函數表，動態時才會決定使用哪個函數。使用 dynamic_cast 時會先檢查是否為父子類的關係，如果不是則會拋出 bad_cast，並使用 try...catch 去捕捉錯誤
```cpp
#include <iostream>
#include <typeinfo>
using namespace std;

class MyBaseClass {
public:
    virtual void print() {
        std::cout << "Printing from MyBaseClass" << std::endl;
    }
};

class MyClass1 : public MyBaseClass {
public:
    void print() override {
        std::cout << "Printing from MyClass1" << std::endl;
    }
};

class MyClass2 : public MyBaseClass {
public:
    void print() override {
        std::cout << "Printing from MyClass2" << std::endl;
    }
};

class MyClass3 : public MyBaseClass {
public:
    void print() override {
        std::cout << "Printing from MyClass3" << std::endl;
    }
};

int main() {
    MyBaseClass *base1 = new MyBaseClass();
    base1->print();
    MyClass2 *pd2 = dynamic_cast<MyClass2 *>(base1);
    base1->print();
    MyClass2 *pd3 = new MyClass2();
    base1 = pd3;
    base1->print();
    
    MyClass1 derive1;
    try {
        MyClass2 c = dynamic_cast<MyClass2&>(derive1);
        c.print();
    }
    catch(bad_cast) {
        cout << "bad_cast 轉型失敗" << endl;
    }
}
```

## 4. reinterpret_cast 主要用於指標
在 C 語言中指標轉型必須先轉到 void* 再轉到其他指標，reinterpret_cast 可以不用經過 void* 而直接轉型。
```cpp
#include <iostream>

int main()
{
    int num = 55;
    int* p_int = &num;
    char* p_char = reinterpret_cast<char *>(p_int);
	
    std::cout << *p_int << std::endl;
    std::cout << std::hex  << *p_int << std::endl;
    // 0x37
    std::cout << *p_char << std::endl;
    // 7 (ascii code 55)
}
```

## 5. 轉型選擇
先看是不是要將 const 或 volatile 拿掉，如果是則用 const_cast。再來看是不是有結構或類別之間的轉換，是的話就用 dynamic_cast。接著再看是不是有指標類型的轉換，如果是的話就用 reinterpret_cast。最後在選 static_cast。
const_cast --> dynamic_cast --> reinterpret_cast --> static_cast
