在 C 語言中只提供最基本的數值型別跟組合型別，其餘的如 [list](https://github.com/JrPhy/DS-AL/tree/master/List_and_Tree)、map、set、[queue](https://github.com/JrPhy/DS-AL/tree/master/Stack_and_Queue) 都需要自行實作。在 C++ 中就已經提供高效且方便的函數庫。C++ 中提供了以下七種容器，分別是
序列式容器：向量（vector）、雙端列隊（deque）、列表（list）
關聯式容器：集合（set）、多重集合（multiset）、映射（map）和多重映射（multimap）

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
除了在尾端插入元素，也可以用 insert 在任意位置插入元素，但是因為插入後要把該位置後面的元素在往後移，所以複雜度為 O(n)，取決於後面還有多少元素，如果陣列大小不足，則還需要新開陣列並複製原始元素到新的陣列中。一般來說要在向量中增加元素，盡量使用 push_back 會有較好的效能。如果是要頻繁的插入則可考慮 [list](https://github.com/JrPhy/DS-AL/blob/master/List_and_Tree/LinkedList-%E9%9B%99%E5%90%91%E9%80%A3%E7%B5%90.md)。
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
