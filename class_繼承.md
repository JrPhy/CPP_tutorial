C++ 的 class 除了封裝以外，還有繼承的語法。在寫物件導向程式時會考慮到重覆使用性與擴充性，所以在開發時會先想好裡面要有什麼變數跟使用方式，之後在做繼承時就會比較方便。

## 1.父類(base)與子類(derive)
另一種說法就是基類跟派生類。如果要做一個車子展示的頁面，車子會先有共同的部分，例如顏色、排氣量、廠牌、年份跟油電等，那麼就可以有個基類 car，剩下功能例如導航、天窗、敞篷這種並非都有的，就可以在繼承的子類別再去實作。這邊就先用最基本的父類與子類做演。\

#### 1. 繼承方式與成員關係
在[class_封裝](https://github.com/JrPhy/CPP_tutorial/blob/main/class_%E5%B0%81%E8%A3%9D.md)中提到**成員**有公有和私有，但在繼承關係中還有受保護的(protected)跟好友(friend)，對於子類中的成員會有不同的效果。以可見度來說：public > protected > friend > private。\
C++ 中的繼承總共有三種，分別是公有繼承(public base)、私有繼承(private base)與受保護的繼承(protected base)。所以從繼承關係與成員宣告，一共會有 12 種組合
|   | 父類 public 成員 | 父類 private 成員 | 父類 protected 成員 |
| --- | --- | --- | --- |
| public 繼承 | 不變 | 不變 | 不變 |
| private 繼承 | 變為 private 成員 | 不變 | 變為 private 成員 |
| protected 繼承 | 變為 protected 成員 | 不變 | 不變 |

而不管是哪種繼承，父類的 private 成員都不能直接訪問。雖然上面有三種繼承方式，不過在實作時大多是用到**公有繼承**，所以基本上只要了解公有繼承的部分即可。
#### 2. 公有繼承
```cpp
#include<iostream>
class base
{
    int b;
   public:
    int a;
    base(int c, int b){ 
        this->a = c; this->b = b; 
        std::cout << "base construct, in base a = " << a << std::endl;
    }
    base(int c){ a = c; }
    void display() { std::cout << "in base " << std::endl; }
    void display_b() { std::cout << this->b << std::endl; }
    void display(int c) { std::cout << c+10 << std::endl; }
    ~base() { std::cout << "base destruct" << std::endl; }
};

class derived: public base
{
   public:
    derived(int c):base(c) { std::cout << "derived construct, in derived a = "<< a << std::endl; }
    ~derived() { std::cout << "derived destruct" << std::endl; }
};

int main()
{
    base b1(7, 8);
    // 初始化 base 中的 a, b 並印出 base construct, in base a = 7
    derived a1(1000.8989898989);
    // 因為是從 base 繼承而來的，且建構函數也是從基類來
    // 所以會先印 base construct, in base a = 1000
    // 在印出 derived construct, in derived a = 1000
    a1.display();
    // derived 中 display() 印出 in base
    b1.display_b();
    // base 中 display_b() 印出 8
    a1.display_b();
    // derived 沒辦法取得 base 中的 b，故印出 0
    return 0;
}
```
印出的結果為
```
base construct, in base a = 7
base construct, in base a = 1000
derived construct, in derived a = 1000
1000
8
0
derived destruct
base destruct
base destruct
```
因為 derived 是從子類繼承而來，所以先呼叫子類的解構子，再呼叫父類的，而因為這邊也有初始化一次父類，所以父類的解構子會被呼叫兩次。
#### 3. 避免 MEMORY LEAK
如果有使用到 new 的話，就要小心是不是有正確的 delete 物件。在繼承關係中雖然會自動呼叫解構子，但有時候會呼叫失敗，下面就是一個呼叫[失敗的例子](https://stackoverflow.com/questions/461203/when-to-use-virtual-destructors?rq=2)
```cpp
#include<iostream>
class base {
public:
    base() { std::cout << "base constructor\n"; };
    ~base() { std::cout << "base destructor\n"; };
};
class derived: public base {
public:
    derived() { std::cout << "derived constructor\n"; };
    ~derived() { std::cout << "derived destructor\n"; };
};

int main() {
    base *p = new derived;
    delete p;
    return 0;
}
```
在這個例子中雖然在父類與子類都有寫解構子，但是實際跑出來卻沒有呼叫到子類的解構子，因為 *p 原本是 base 類別，在此初始化了一個 derived 的類別，此時兩型別就不同。雖然有呼叫到 delete p，但因為型別不同，在 C++ 中是一個未定義行為，在此可以在父類別的解構子前面加上 ```virtual``` 關鍵字來避免此問題。
```cpp
#include<iostream>
class base {
public:
    base() { std::cout << "base constructor\n"; };
    virtual ~base() { std::cout << "base destructor\n"; };
};
class derived: public base {
public:
    derived() { std::cout << "derived constructor\n"; };
    ~derived() { std::cout << "derived destructor\n"; };
};

int main() {
    base *p = new derived;
    delete p;
    return 0;
}
```
#### 4. virtual 關鍵字
virtual 關鍵字只能用在普通或是解構函數，用來告訴編譯器這個函數**有可能**會在子類被改寫。所以在上述例子中，子類的解構函數被改寫了，如果一個類是會被別的類繼承，那最好在他的解構函數前加上 virtual 關鍵字。
```cpp
#include<iostream>
class base {
public:
    base() { std::cout << "base constructor\n"; };
    virtual ~base() { std::cout << "base destructor\n"; };
    virtual void do_something() { std::cout << "base do_something\n"; };
};
class derived: public base {
public:
    derived() { std::cout << "derived constructor\n"; };
    ~derived() { std::cout << "derived destructor\n"; };
    void do_something() { std::cout << "derived do_something\n"; };
};

int main() {
    base *p = new derived;
    p->do_something();
    delete p;
    return 0;
}
```
這樣一來呼叫到的 do_something() 就是 derived 中的而非 base 中的 do_something()。
```
base constructor
derived constructor
derived do_something
derived destructor
base destructor
```
當然若一函數在基類中被用 virtual 修飾，則被稱為**虛函數**，他的子類也不一定需要有自己的實作，如果子類沒有自己的實作，那就會去呼叫基類中的該函數，以上述例子來說為 base do_something。所以在記憶體洩漏的例子中，即便有把基類的解構子用 virtual 修飾，但若在子類沒另外寫，那麼程式結束時也不會呼叫到子類的解構子。而如果在基類的虛函數僅提供介面不提供時做，則可將其宣告為純虛函數，下方即為一個例子
```cpp
#include<iostream>
class base {
public:
    base() { std::cout << "base constructor\n"; };
    virtual ~base() { std::cout << "base destructor\n"; };
    virtual void do_something() { std::cout << "base do_something\n"; };
    virtual void exec() = 0;
};
class derived: public base {
public:
    derived() { std::cout << "derived constructor\n"; };
    ~derived() { std::cout << "derived destructor\n"; };
    void do_something() { std::cout << "derived do_something\n"; };
    void exec() = {...};
};
```
在 base 中有個 ```virtual void exec() = 0;``` 即告訴開發者，每個繼承此類的函數都需要有一個自己的 exec() 實作，如果沒有的話編譯器就會報錯。
