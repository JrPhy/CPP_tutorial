在物件導向中，多型(polymorphism)包含了多載(Overload)與覆寫(Override)兩個功能。不過主要就是說明**同一名稱的函數**，有不同的傳入函數型別、不同傳入的引數，跟(與父類別)不同的函數實作。主要就是為不同資料類型的實體提供統一的介面。

## 1. 多載(Overload)
在同個類別中，有相同的函數名稱，但是有不同的引數，是在**編譯期間**就可以知道要呼叫哪個函數
#### 1. 函數重載
```
class Print {
   public:
    void print(const char * str, int width) {...};  // #1
    void print(double d, int width) {...};          // #2
    void print(long l, int width) {...};            // #3
    void print(int i, int width) {...};             // #4
    void print(const char *str) {...};              // #5
}

int main() {
    Print p;
    p.print("Pancakes", 15);         // use #1
    p.print("Syrup");                // use #5
    p.print(1999.0, 10);             // use #2
    p.print(1999, 12);               // use #4
    p.print(1999L, 15);              // use #3
}
```
以這個例子來說，前面四個除了第一個引數不同外其餘都相同，所以也可以考慮使用樣板(template)來讓函數可以傳入任意型別。
#### 2. 運算子重載
在數學上，相同符號的運算子，會根據不同的元素而有不同的定義，例如數字的乘法可寫成 × 或 *，但是在向量則分別表示外積跟內積。在 C++ 也可以根據不同的物件去重新定義運算子。而在 C++ 中有不可被重載的運算子
|   |  |  |  | |
| --- | --- | --- | --- | --- |
| . | .* | :: | ?: | sizeof |

在此可以寫一個矩陣加法的重載運算子的例子
```cpp
#include <iostream>
#include <vector>

class Matrix
{
   public:
    Matrix() {};
    friend Matrix operator+(Matrix &,Matrix &); // "+" 重載
    void input(std::vector<std::vector<int> > mat);
    void display();
   private:
    int mat[2][3] = {0};
};

Matrix operator+(Matrix &a,Matrix &b) // "+" 重載實作
{
    Matrix c;
    for(int i = 0; i < 2; i++) {
        for(int j = 0; j < 3; j++) {
            c.mat[i][j] = a.mat[i][j] + b.mat[i][j];
        }
    }
    return c;
}
void Matrix::input(std::vector<std::vector<int> > mat)
{
    cout << "input value of matrix:" << endl;
    for (int i = 0; i < mat.size(); i++) { 
        for (int j = 0; j < mat[i].size(); j++) {
            this->mat[i][j] = mat[i][j];
        }
    } 
}
void Matrix::display()
{
    for(int i = 0; i < 2; i++) {
        for(int j = 0; j < 3; j++) {
            cout << mat[i][j] << " ";
        }
        cout<<endl;
    }
}
int main()
{
    Matrix a,b,c;
    std::vector<std::vector<int> > mat_a{ { 1, 2, 3 }, 
                                          { 4, 5, 6 }, 
                                          { 7, 8, 9 } }; 
    std::vector<std::vector<int> > mat_b{ { 1, 2, 3 }, 
                                          { 4, 5, 6 }, 
                                          { 7, 8, 9 } }; 
    a.input(mat_a);
    b.input(mat_b);
    cout << endl << "Matrix a:" << endl;
    a.display();
    cout << endl << "Matrix b:" << endl;
    b.display();
    c = a + b;
    cout << endl << "Matrix c = Matrix a + Matrix b :" << endl;
    c.display();
    return 0;
}
```
## 2. 覆寫(Override)
覆寫就是修改父類相同名稱函數的實作，所以要在需要覆寫的函數前面加上 [virtual](https://github.com/JrPhy/CPP_tutorial/blob/main/class_%E7%B9%BC%E6%89%BF.md#4-virtual-%E9%97%9C%E9%8D%B5%E5%AD%97)，或是將該函數宣告為純虛函數。雖然避免了沒有正確呼叫函數，但也會造成效能下降，C++ 也另外提供了 override 跟 final 兩關鍵字來解決這問題。

#### 1. override
override 在 C++11 為一個關鍵字，用了此關鍵字就會由編譯器提醒開發者是否有正確覆寫父類的函數。因為 virtual 並不強迫一定要覆寫，而 override 則是一定要覆寫，而且若名字或是引數錯誤，也會藉由編譯器提醒。
```cpp
class base {
    virtual void f1(int) const;
    virtual void f2();
    void f3();
};
calss derived:public base{
    void f1(int) const override; //正確
    void f2(int) override;       //錯誤，引數不同
    void F2() override;          //錯誤，名稱不同
    void f3() override;          //錯誤，f3() 非虛函數
    void f4() override;          //錯誤，base 沒有 f4() 這函數
};
```
在子類中的函數加上 override 可以讓編譯器檢查。若子類某個函數同時加上 virtual 跟 override，則是告訴開發者這個函數在他的基類也有並且被覆寫了，且此類的蓋函數有可能需要被別人繼承並覆寫。
```cpp
#include <stdio.h>
class Base
{
public:
    void A_1()  { printf("Base::A_1\n"); };
    virtual void A_2() { printf("Base::A_2\n"); };
    virtual void A_3() { printf("Base::A_3\n"); };
    virtual void A_4() { printf("Base::A_4\n"); };
    void A_5() { printf("Base::A_5\n"); };
};

class Drived : public Base
{
public:
    void A_1() { printf("Drived::A_1\n"); };
    //覆寫了基底類別的函數，用基底類別指標呼叫時呼叫到的時基底類別的A_1，用子類別指標呼叫時呼叫到的時子類別的A_1
    virtual void A_2() { printf("Drived::A_2\n"); };
    //用基底類別或子類別指標都呼叫的是子類別的A_2,之類的virtual說明的是子類別的A_2還可以被virtual
    void A_3() { printf("Drived::A_3\n"); };
    //用基底類別或子類別指標都呼叫的是子類別的A_2
    virtual void A_4() override { printf("Drived::A_4\n"); };
    //子類別加上override，如果基底類別沒有對應virtual函式就會編譯錯誤。避免拼錯和記錯沒有重寫基底類別函數
    //void A_5() override { printf("Drived::A_5\n"); };
    //編譯錯誤'Drived::A_5': method with override specifier 'override' did not override any base class methods
};

class Drived2 : public Drived
{
public:
    void A_2() override { printf("Drived2::A_2\n"); }
    void A_3() override { printf("Drived2::A_3\n"); }
    //用基底類別或子類別指標都呼叫到的是子類別的A_2
};

class VirtualTest
{
public:
    void DoTest()
    {
        Drived *drived = new Drived();
        Base *base = drived;
        base->A_1();
        drived->A_1();
        printf("\n");

        base->A_2();
        drived->A_2();
        printf("\n");

        base->A_3();
        drived->A_3();
        printf("\n");

        base->A_4();
        drived->A_4();
        printf("\n");

        Drived2 *drived2 = new Drived2();
        base = drived2;
        drived = drived2;
        
        base->A_2();
        drived->A_2();
        drived2->A_2();
        printf("\n");

        base->A_3();
        drived->A_3();
        drived2->A_3();
    }
};

int main() {
    VirtualTest t;
    t.DoTest();
}
```
#### 2. final
與 override 不同，final 是告訴編譯器這個函數需要被子類繼承，但是不改變他的實作。
```cpp
class base {
   public:
    virtual void f1(int) const;
    virtual void f2() final { return 1; };
    void f3();
};
calss derived:public base{
    void f1(int) const override; //正確
    void f2() { return 2; };     //錯誤，無法覆寫
};
```
這兩個關鍵字讓開發者避免無心錯誤，算是非常好用。當然 virtual 和 override 與 final 三者是可以同時使用的。即告訴使用者該函數有可能被別的類別繼承，且是從父類別繼承並父寫來的，並無法給他的子類別覆寫。
```cpp
class Drived : public Base
{
public:
    void A_1() { printf("Drived::A_1\n"); };
    //覆寫了基底類別的函數，用基底類別指標呼叫時呼叫到的時基底類別的A_1，用子類別指標呼叫時呼叫到的時子類別的A_1
    virtual void A_2() { printf("Drived::A_2\n"); };
    //用基底類別或子類別指標都呼叫的是子類別的A_2,之類的virtual說明的是子類別的A_2還可以被virtual
    void A_3() { printf("Drived::A_3\n"); };
    //用基底類別或子類別指標都呼叫的是子類別的A_2
    virtual void A_4() override final { printf("Drived::A_4\n"); };
    //子類別加上override，如果基底類別沒有對應virtual函式就會編譯錯誤。避免拼錯和記錯沒有重寫基底類別函數
    //void A_5() override { printf("Drived::A_5\n"); };
    //編譯錯誤'Drived::A_5': method with override specifier 'override' did not override any base class methods
};
```
