在 C 語言中 [const 關鍵字](https://github.com/JrPhy/C_tutorial/blob/main/CH7-%E7%94%9F%E5%91%BD%E9%80%B1%E6%9C%9F%E8%88%87%E5%8F%AF%E8%A6%96%E7%AF%84%E5%9C%8D.md#5-%E9%97%9C%E9%8D%B5%E5%AD%97-const)表示為只讀(read-only)的意思，也就是該變數不能被程式碼所改變。除了對變數的值作用外，也可以對指標位置設定只讀，而變數的值與位置也可以同時設定只讀。在 C++ 中也可將成員函式用 const 修飾。當然除了最基本的 const，在 C++ 中還多了 constexpr (C++11)、consteval (C++20)、constinit (C++20)，就來看看這些關鍵字的用處是什麼

## 1. const
C 語言中有的用法在 C++ 中也相同，但在 C++ 中 const 可置於函數前後
#### 1. const 置於函數前
