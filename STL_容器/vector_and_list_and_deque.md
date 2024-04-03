在 C++ 中提供了三種序列式容器，分別是向量（vector）(ARRAY 封裝)、雙端列隊（deque）([deque](https://github.com/JrPhy/DS-AL/tree/master/Stack_and_Queue) 封裝)、列表（list）([雙向 LIST 封裝](https://github.com/JrPhy/DS-AL/blob/master/List_and_Tree/LinkedList-%E9%9B%99%E5%90%91%E9%80%A3%E7%B5%90.md))
## 1. 向量（vector）
此說的向量並非數學上的向量，可以當作是 C++ 中更好用的 ARRAY，除了有 ARRAY 的特性外，還可以隨時改變他的大小。
```cpp
#include <vector> // 使用時須引入
#include <iostream>
using namespace std; // 若不使用則須在 vector 前面加上 std::
int main(int argc, char* argv[])
{
    vector<int> vecInt; // 若無第九行，則為 std::vector<int> vecInt;
    // 宣告一個型別為 int 的 vector，不需要給定大小
    vector<double> vecDou = {10.0, 20.0, 30.0}; // 如同陣列初始化
    vector<int> vec(2, 9); // 也可以這樣初始化，前面放大小，後面放值
    for (int i = 0; i < 4; i++) {
        vec.push_back(i); // 每 push 一次大小就會增加 1
        cout << vec.size() << " "; // 輸出：3 4 5 6
    }
    for (int i = 0; i<vec.size(); i++) cout << vec[i] <<" ";
    // 輸出：9 9 0 1 2 3
    // 若無第九行，則為 std::cout ...
    return 0;
}
```
![img](https://github.com/JrPhy/CPP_tutorial/blob/main/img/vector_size.jpg)
## 2. 向量的資料結構
雖然看起來好像是用動態增加長度，但實際上是開一個較大的 array 然後去裝物件，並呼叫 begin() 回傳起始 start, end() 回傳使用大小 finish，和 capacity() 回傳陣列大小 end_of_storage。如果已經滿了但還要 push_back 的話，則會**再開一個更大的向量**，然後把先前的資料放進去新的向量，並不會每放個東西進去就去申請一次空間。capacity 為陣列的實際大小，而 size 僅為目前使用了多少個 index 而已，所以 capacity >= size。

## 3. 向量操作的複雜度
在向量中提供了一些函數，如 push_back, insert 可放入元素, pop_back, erase, clear 可刪除元素。
#### 1. push_back(value)
由於向量底層是用陣列實作，再加上一些整數來記錄位置，所以可以直接在尾端插入，如果陣列大小足夠的話複雜度為 O(1) (大部分情況)，不夠的話則需要再開新陣列並把原本的值複製到新的向量，這時為 O(n) (少數情況)。
#### 2. insert(position, value)
除了在尾端插入元素，也可以用 insert 在任意位置插入元素，但是因為插入後要把該位置後面的元素在往後移，所以複雜度為 O(n)，取決於後面還有多少元素，如果陣列大小不足，則還需要新開陣列並複製原始元素到新的陣列中。一般來說要在向量中增加元素，盡量使用 push_back 會有較好的效能。如果是要頻繁的插入則可考慮 [list](https://github.com/JrPhy/DS-AL/blob/master/List_and_Tree/LinkedList-%E9%9B%99%E5%90%91%E9%80%A3%E7%B5%90.md)。所以在新增時的步驟為 \
1. 檢查長度是否足夠
2.1. 足夠
2.2. 找到位置插入
2.3. 把後面的值往後移
3.1. 不足夠
3.2. 配置新空間
3.3. 找到位置插
3.4. 把後面的值往後移
#### 3. pop_back()
把最尾端的元素拿掉，並且 size 會少 1，capacity 並不會改變。
#### 4. erase(iterator position), erase(iterator first, iterator last)
erase 傳入和返回**迭帶器**(可以當作一個指標)，所以並不能像前面的函數一樣直接呼叫，必須先用 iterator 才行，可以使用 begin() 這個成員函數先指向起始位置，再去指定位置，或是刪除特定元素，且呼叫 erase 後 size 也會變小。可以刪除指定位置的元素，也可刪除某個範圍內的元素。
```cpp
#include <vector>
#include <iostream>
using namespace std;
int main(int argc, char* argv[])
{
    vector<int> vec(2, 9);
    for (int i = 0; i < 4; i++) vec.push_back(i);
    cout << vec.size() << " "; // 此時為 6
    vec.erase(vec.begin()+1); // 刪除 vec[1] (9)
    vec.erase(vec.begin()+3, vec.begin()+5);
    // 刪除 vec[3] (2), vec[4] (3), vec[5] (已超出 size，不會有動作)
    for (int i = 0; i < vec.size(); i++) cout << vec[i] << " ";
    // 9 0 1
    return 0;
}
```
因為 erase 是用迭帶器操作，所以如果有重複的資料要刪除，要去移動迭帶器指向下一個位置，不然只會刪除第一個找到的
```cpp
#include <vector>
#include <iostream>
using namespace std;
int main(int argc, char* argv[])
{
    vector<int> vec(2, 9);
    for (int i = 0; i < 4; i++) vec.push_back(i);
    for (vector<int>::iterator it = vec.begin(); it != vec.end();) {
        if (*it == 9) vec.erase(it);
        else ++it; //沒有這段就只會刪除第一個 9
    }
}
```
#### 5. clear
會把向量中的元素清除並把 size 設為 0，但不改變 capacity 的大小

## 3. 多維向量
在宣告陣列時我們會宣告多維陣列，如 ```int A[10][10]; ``` 表示 10*10 的陣列，但實際上在記憶體中都是一維的，而要宣告多維向量則是
```cpp
#include <iostream> 
#include <vector> 
int main() 
{
    // 如同 2 維陣列初始化
    std::vector<std::vector<int> > vec{ { 1, 2, 3, 4}, 
                                        { 5, 6, 7, 8 }, 
                                        { 9, 10, 11, 12 } };
    // 空格為必要
    vec[2].pop_back(); // 移除 3rd row 的尾端元素
    vec[1].pop_back(); // 移除 2nd row 的尾端元素
  
    // 如同 2 維陣列的大小
    for (int i = 0; i < vec.size(); i++) { 
        for (int j = 0; j < vec[i].size(); j++) {
            std::cout << vec[i][j] << " "; 
        }
        std::cout << std::endl; 
    } 
    return 0; 
} 
```
## 2. 列表 LIST
C++ STL 中的 list 為[雙向 LIST 封裝](https://github.com/JrPhy/DS-AL/blob/master/List_and_Tree/LinkedList-%E9%9B%99%E5%90%91%E9%80%A3%E7%B5%90.md)，所以除了資料本身外，還需要多**兩個指標**來指向前後的節點。與向量不同，列表中每個節點都是新開的指標，所以為**不連續**的記憶體位置，而且一定要從開頭去走訪。而 STL 對於 list 的封裝讓 list 可以像 array 一樣的方式初始化
```cpp
#include <iostream> 
#include <list>

int main() {
    std::list<int> ListInt = {1, 2, 3};
    int array[] = {3, 2, 1}; // C++98 會報錯，C++11 可以
    std::list<int> ListInt1(array, array+3);
    std::list<double> ListDou(5, 10.0); // 長度為 5 且值皆為 10
    for (const auto &n : ListInt) {
        std::cout << n << ", "; // 1, 2, 3,
    }
    std::cout << "\n";
    return 0;
}
```
## 4. 列表的資料結構
除了放資料本身外，還需要有兩個指標指向前後節點，所以雖然 LIST 可以有多少資料就放多少節點，但每多開一個節點就需要多開兩個指標的空間。

## 5. 列表操作的複雜度
除了向量中提供的函數外，列表還支援 push_front, pop_front, [merge, sort](https://github.com/JrPhy/DS-AL/blob/master/Sort_and_Search/Sorting_for_list.md), [reverse](https://github.com/JrPhy/DS-AL/blob/master/List_and_Tree/LinkedList-%E9%9B%99%E5%90%91%E9%80%A3%E7%B5%90.md#6-%E9%9B%99%E5%90%91-list-%E5%8F%8D%E8%BD%89) 等函數。
#### 1. push_back(value), push_front(value) 與 pop_back(value), pop_front(value)
因為本身為雙端列表，所以可以從頭尾開始去存取，所以這四種操作複雜度皆為 O(1)，但就對尾端元素的操作，vector 效能還是比 list 高的。
#### 2. insert(position, value)
因為是用指標做連接，所以在任意位置插入，只須找到該位置，然後讓指標重新串接即可，不需把後面的資料在往後移，故複雜度取決於該位置離起始位置有多遠，複雜度為 O(n)。

## 6. 雙端[隊列 DEQUE](https://github.com/JrPhy/DS-AL/blob/master/Stack_and_Queue/Queue-%E4%BD%87%E5%88%97.md)
為 double-ended queue，一般的隊列底層是陣列，然後是 FIFO，所以會去記錄第一個 start 跟最後一個 finish。而 STL 的雙端隊列**融合了 vector 和 list** 的優缺點。
#### 1. 隊列的資料結構
底層是用很多段陣列，最後將其串起來。所以底層其實是有很多的指標，這些指標會分別指向一段連續的記憶體，不同段的記憶體就不一定是連續的，在 std::deque 中稱其為 map，與 std::map 不同。
![img](https://github.com/JrPhy/CPP_tutorial/blob/main/img/deque.jpg)
deque 因為底層是用陣列實作，所以在取值也可直些取 index 的值，初始化方式也能跟 vector 一樣。也可以在前後端 push 與 pop 和 insert。
```cpp
#include <iostream> 
#include <deque>

int main() {
    std::deque<int> DequeInt = {1, 2, 3};
    std::deque<double> DequeDou(5, 10.0); // 長度為 5 且值皆為 10
    for (int i = 0; i < DequeInt.size(); i++) {
        std::cout << DequeInt[i] << ", "; // 1, 2, 3,
    }
    std::cout << "\n";
    DequeInt.push_back(4);
    DequeInt.push_front(5);
    DequeInt.push_back(6);
    for (int i = 0; i < DequeInt.size(); i++) {
        std::cout << DequeInt[i] << ", "; // 1, 2, 3,
    }
    std::cout << "\n";
    return 0;
}
```
#### 2. 隊列走訪的效能
前面提到對列底層並非完全連續的記憶體位置，雖然也可以像陣列一樣用 index 取值，但在底層其實是重載了 [] 算子，必須要先找到在 map 的位置，然後再找到 index 的位置，所以在走訪的時候效能會比 vector 用 index 走訪[慢約 4 倍](https://blog.csdn.net/u011428210/article/details/111869627)，但是用 iterator 則會差不多。
## 7. 隊列操作的複雜度
列表中有的對列也都有
#### 1. push_back(value), push_front(value) 與 pop_back(value), pop_front(value)
因為本身為雙端隊列，所以可以從頭尾開始去存取，所以這四種操作複雜度皆為 O(1)，但就對尾端元素的操作，vector 效能還是比 list 高的。
#### 2. insert(position, value)
因為是用指標做連接，所以在任意位置插入，只須找到該位置，然後讓指標重新串接即可，不需把後面的資料在往後移，故複雜度取決於該位置離起始位置有多遠，複雜度為 O(n)。

## 8. 比較
|  | vector | list | deque |
| --- | --- | --- | --- |
| 記憶體 | 連續 | 不連續 | 部分連續 |
| 首端操作 | 不支援 | 中等 | 最快 |
| 任意位置存取 | 最快 | 最慢 | 中等 |
| 中間插入 | 最慢 | 中等 | 最快 |
| 尾端操作 | 最快 | 最慢 | 中等 |
| 靜態無連結 | 靜態 | 區塊 | 無 |

1. 需要隨機存取、已知儲存元素數量，選擇 vector
2. 若需要隨機插入/刪除，選擇 list
3. 若需要隨機插入/刪除和存取，選擇 才选择deque
