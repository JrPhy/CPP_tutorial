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
#### 2. 運算子重載
在數學上，相同符號的運算子，會根據不同的元素而有不同的定義，例如數字的乘法可寫成 × 或 *，但是在向量則分別表示外積跟內積。在 C++ 也可以根據不同的物件去重新定義運算子。而在 C++ 中有不可被重載的運算子
|   |  |  |  | |
| --- | --- | --- | --- | --- |
| . | .* | :: | ?: | sizeof |
2. ## 覆寫(Override)
