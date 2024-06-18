在 C 語言中有時會不小心寫出一些錯誤，或是使用者未按照規則使用造成一些錯誤，例如除以 0，此時就可以用 if-else 來檢查並回報，當然還有其他種錯誤，之後會在分別來說。而在 C++ 中引入了 try-catch-throw 來做異常處理。

## 1. if-else 的錯誤管理
if-else 只做條件比對，所以必須自己去定義一些錯誤，例如除以 0、超過陣列索引、溢位等許多例外情況，所以就會寫成
```C
int a = 0, b = 0, c = 0;
...
if (0 == b) {
    printf("divide by 0\n");
    return;
}
```
而這類的錯誤若是在客戶端則很難發現問題，因為不知道客戶會如何使用，當然也可以一直寫 if-else 去做管理，但是程式碼會變得非常複雜。

## 2. try-catch-throw 的錯誤管理
try-catch 則是將這類的錯誤已經整理好，開發者只需使用其寫法就可以作錯誤管理。上述程式碼就可改成
```C++
int a = 0, b = 0, c = 0;
try {
   if( b == 0 )
   {
      throw "divide by 0\n";
   }
   std::cout << "before catch\n";
} catch (const char* msg) {
    std::cerr << msg << std::endl;
}
std::cout << "after catch\n";
```
當程式碼遇到 throw 的時候就會跳到 catch 部分，後面就不執行，執行完 catch 部分後程式碼就會繼續往下走，所以會印出
```
Division by zero condition!
after catch
```

## 3. what()
在 std 中定義了許多例外如下圖\
![image](https://d8it4huxumps7.cloudfront.net/uploads/images/64e7046ee621f_c_exception_handling_4.jpg)\
所以在寫 throw 跟 catch 時就可以搭配 std::exception 去寫，需要 ```#include <stdexcept>```，然後用 what() 來印出例外訊息。
```c++
#include <iostream>
#include <stdexcept>

int divide(int a, int b) {
    if (0 == b) {
        throw std::runtime_error("Division by zero error");
    }
    return a / b;
}

int main() {
    try {
        int a = 0, b = 0, c = 0;
        int result = divide(a, b);
        std::cout << "Result: " << result << std::endl;
    } catch (const std::exception& e) {
        std::cout << "Exception caught: " << e.what() << std::endl;
    }
    return 0;
}
```
上述例子中因為進去了```(0 == b)```，所以會拋出異常，catch 捕獲到異常後就印出異常，其中 what() 就是 error msg。

## 4. 自訂例外
C++ 中也可以去自訂例外情況，只要公開繼承 exception 就可以去複寫了。
```C++
#include <iostream>
#include <stdexcept>
class MyException : public std::exception {
   public:
    const char* what() const noexcept override {
    return "Custom exception message";}
};

int divide(int a, int b) {
    if (b == 0) {
        throw MyException();
    }
    return a / b;
}

int main() {
    try {
        int result = divide(10, 0);
        std::cout << "Result: " << result << std::endl;
    } catch (const MyException& ex) {
        std::cout << "Exception caught: " << ex.what() << std::endl;
    }
    return 0;
}
}

int main() {
try {
int result = divide(10, 0);
std::cout << "Result: " << result << std::endl;
} catch (const MyException& ex) {
std::cout << "Exception caught: " << ex.what() << std::endl;
}

return 0;
}
```
