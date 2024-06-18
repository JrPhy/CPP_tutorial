C++ 原本為 C 語言的拓展，雖然從 1989 之後兩者就分道揚鑣，但是 C 語言大部分仍然可看成 C++ 的子集合，當然這也表示有些 C 語言的寫法用 C++ 編譯器是會報錯的，例如
```C
int n = 10;
int A[n]; // C++ 編譯器會報錯，C 不會
```
上述程式碼在 C++ 會報錯，原因是 C++ 不支援 VLA，在 C++ 中通常會使用 vector 來取代 array。\
除了兼容多數 C 的語法外，C++ 則為物件導向的語言 (本身的語法就支持類別)，而 C 只是過程導向的語言 (語法不支援類，但也可以達到物件導向的目的)。當然這方面在網路上都已經有人[整理](https://www.1ju.org/cplusplus/c-vs-cpp)了，在此就不贅述。

而 C++ 標準從早期的 C++89、C++03，然後從 C++11 開始幾乎是三年就一個新標準，每個標準加入的東西也非常多，C++14 支援了 lambda 運算式，詳細可以去看各個標準的規格書。當然因為 C++ 包含大部分 C 語言的特性，所以在撰寫時會以 C 語言沒有的特性去撰寫，其餘請參考 C 語言的文章。

C++ 可以當作是有 class 的 C，而 class 為物件導向的語法，用來做封裝與繼承，且在 C++ 中支援樣板 (template) 來達成[泛型函數](https://github.com/JrPhy/C_tutorial/blob/main/CH8-%E6%8C%87%E6%A8%99%E8%88%87%E5%87%BD%E6%95%B8.md#3-%E6%8C%87%E6%A8%99%E5%87%BD%E6%95%B8%E7%95%B6%E4%BD%9C%E5%BC%95%E6%95%B8)，不過樣板會有比較高的編譯時間，但對於大型程式開發可以換來更好的可讀性與維護性。

https://meetingcpp.com/mcpp/books/book.php?hash=d6176cbb9cae6aa6304cfb89c1cbe61cd7c9455d

從 C++11 起被稱為 Modern C++，裡面除了新增許多特性，如 Smart pointer 可以不用讓開發者不用去想指標是否被釋放，auto (與 C 的不同) 讓編譯器去自動推導型別，Lambda function 讓函數裡面可以寫函數，右值引用讓傳入 class 可以少呼叫一次複製的建構子，還有將 thread 野放進 std 中，比 pthread 更好用。
