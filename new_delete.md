C/C++ 中除了編譯器會去申請與返還記憶體空間外，開發者也可以自行去申請與返還記憶體空間。在 C 中使用 malloc/free，C++ 中使用 new/delete。在 C/C++ 中使用到上述四個關鍵字來申請記憶體時，我們稱為**動態配置**，以下來說明這兩種的異同。

## 1. 程式的記憶體配置
程式是經由程式碼撰寫後再由編譯器編譯成執行檔執行，編譯器會根據與法規則先去檢查，檢查通過後會編成可執行文件，若沒指定則是 a.out。而程式碼本身也會占用一部分的容量。下圖是 C/C++ 的變數記憶體分配，由左至右分別是從低位置到高位置。
| 程式碼(#define) | 已初始化的靜態變數 | 未初始化的靜態變數 | 動態配置 | 未使用區域 | 區域變數 | 
| --- | --- | --- | --- | :---: | --- |
| TEXT | DATA 包含 const | bss | heap(堆) | -->...<-- | stack(棧) |

上圖中的 heap 與 stack 實際要看編譯器如何實做。在程式開發時的行為會決定變數的生命週期，生命週期越長的就會放在越下面，例如全域變數與 static 變數，較短的就往上放，例如區域變數與函數返回值。而開發者自行申請與釋放的記憶體並不再規範內，所以是放在 heap 中。而 new 申請的空間則稱為自由儲存區，有可能是 heap 或 stack，如果 new 有被重載的話。

## 2. stack/heap 差異
當然記憶體大小也是有限制的，並不會無限成長，而且 stack 空間是 OS 一次就申請好的，用太多會讓程式一跑起來時就占用非常大的記憶體，所以通常不會太大，也有可能出現記憶體不足的錯誤訊息，此時就稱為 stack overflow。且 stack 的記憶體位置是連續的，效能上較好。除了 heap 外，其餘部分的記憶體皆由操作系統來管理，在程式結束時就會自動釋放。\

|  | 管理 | 大小 | 配置方式 | 效能 |
| --- | --- | --- | --- | --- |
| stack | OS | 通常 MB 等級 | 較快 | 連續 |
| heap  | 開發者 | 看主機的記憶體有多大 | 較慢 | 不連續 |

heap 申請的記憶體則需要靠開發者釋放，若在程式結束前沒有釋放，則會造成記憶體洩漏 Memory Leak，在 C++ 中可以用[智慧指標](https://github.com/JrPhy/CPP_tutorial/blob/main/Smart_Pointer_%E6%99%BA%E6%85%A7%E6%8C%87%E6%A8%99.md)來避免。也因為生命週期不固定，所以申請空間時僅會看總大小申請，不會考慮到記憶體位置是否連續，所以效能通常沒有很好。但也可以仿照 stack 的作法，直接申請一個很大的 Memory pool，最後在一起釋放。
下方為一個動態配置記憶體位置不連續的例子。假設目前的可用空間大小剩 32 B，而且被分成兩塊 12 B 和 20 B。如果要申請長度為 6 的 float 陣列，那就會一部分在 A 一部分在 B

| 記憶體 | 未使用 A | 使用中 | 未使用 B | 使用中 |
| --- | --- | --- | --- | --- |
| 大小 | 12 B | XX B | 20 B | XX B |

## 2. malloc/new 與 free/delete 的使用方式
malloc 為函數，會返回一個 void*，開發者申請的物件為指標，需要自行轉型到正確的型別。當然在 C 語言中會自動幫你隱式轉型，不過在 C++ 編譯器中就會報錯。malloc 也需要指定記憶體大小，new 則會自動去判斷該物件的大小。
```cpp
int** ma = (int**)malloc(sizeof(int*));
int** mb = malloc(4);
// C++ Compiler invalid conversion from 'void*' to 'int*'
// but it's OK in C compiler
float* array[10] = (int*)malloc(sizeof(float)*10);
// 申請長度為 10 的整數陣列
int* ne = new int(0);
// do something ...
free(ma);
free(mb);
delete(ne);
```
若只是要返還一維陣列或指標，最後直接 free/delete 就好了。\
而多維陣列實際上是**陣列的陣列**，只是我們習慣以數學上的陣列做使用，所以雙指標實際上也為**指標的指標**。所以在動態配置二維陣列時，需要先去配置一個一維陣列，然後每個一維陣列再去配置一維陣列，其餘使用方式皆與直接分配二維陣列一樣。例如要分配一個 array[m][n] 的陣列。再返還時的順序則是反過來
```cpp
int* ma = (int*)malloc(sizeof(int *)*m);
for(int i = 0; i < m; i++)
{ int* ma[i] = (int*)malloc(sizeof(int *)*n); }
// do something ...
for(int i = 0; i < m; i++)
{ free(ma[i]); }
free(ma);
```
雖然我們習慣照數學的方式去寫陣列，但實際上在記憶體中都是一維的，在分配時也可以直接申請一個 m*n 大小的位置，但這種方式就無法照著數學的方式去取值。而 new 的用法跟 malloc 差不多，delete 的話則與 free 有些許差異
```cpp
for(int i = 0; i < m; i++)
{ delete [] ma[i]; }
delete [] ma;
// 以上三行也可以寫成 delete ma
```
當然會於一般的陣列沒差，但若是需要建構與解構就會有差，如下方說明。

## 3. malloc/free 與 new/delete 混用
C++ 作為 C 的拓展，把 malloc/free 用 new/delete 替代不會有什麼大問題，且在大部分情況 new/delete 其實也是用 malloc/free 實做。不過 new/delete 為運算符，所以有可能被重載，當然 malloc/free 也可以重載，所以就標準中的來看兩者混用會有什麼風險。\
C++ 比 C 多了 class，class 在使用時需要被建構與解構，如果是使用 malloc/free 去做，就有可能會沒有建構與解構到，在 Effective C++ 中就有提到這點
```cpp
string *stringarray1 = static_cast<string*>(malloc(10 * sizeof(string)));
// 實際上沒有建構 string
string *stringarray2 = new string[10];
// 有建構 string 且建構了 10 次
// do something ...
free(stringarray1);
// 沒有解構
delete [] stringarray2;
// [] 是因為建構了 string array，所以需要每個 array 都去解構
```
