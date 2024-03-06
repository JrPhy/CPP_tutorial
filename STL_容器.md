在 C 語言中只提供最基本的數值型別跟組合型別，其餘的如 [list](https://github.com/JrPhy/DS-AL/tree/master/List_and_Tree)、map、set、[queue](https://github.com/JrPhy/DS-AL/tree/master/Stack_and_Queue) 都需要自行實作。在 C++ 中就已經提供高效且方便的函數庫。C++ 中提供了以下七種容器，分別是
序列式容器：向量（vector）、雙端列隊（deque）、列表（list）
關聯式容器：集合（set）、多重集合（multiset）、映射（map）和多重映射（multimap）

## 1. 向量（vector）
此說的向量並非數學上的向量，可以當作是 C++ 中更好用的 ARRAY，除了有 ARRAY 的特性外，還可以隨時改變他的大小。
```cpp
#include <vector> // 使用時須引入
using namespace std; // 若不使用則須在 vector 前面加上 std::
int main(int argc, char* argv[])
{
	vector<int> vecInt; // 若無第九行，則為 std::vector<int> vecInt;
    // 宣告一個型別為 int 的 vector，不需要給定大小
    vector<double> vecDou = {10.0, 20.0, 30.0}; // 如同陣列初始化
	for (int i = 0; i<6; i++)
		vecInt.push_back(i); // 每 push 一次大小就會增加 1
    cout << vecInt.size() << " "; // 输出：0 1 2 3 4 5
	for (int i = 0; i<vecInt.size(); i++)
		cout << vecInt[i] <<" "; // 输出：0 1 2 3 4 5
        // 若無第九行，則為 std::cout ...
	return 0;
```
