協程這一概念早在 1985 年就已提出，是一個**可以暫停與恢復的函數**。其他語言中已有相關的實作，例如 PYTHON 與 JS 中的 async/await，以及 golang 的 gorouting 等，C++ 則是在 C++20 中才納入標準。在網頁與網頁使用 API 去要資料，或是去跟一些 IOT 設備要資料時，常常是一方去發請求，對方接收到請求後把資料發給你，然後再回一個狀態回去就完成請求。在早期 C/C++ 會使用線程 (Thread) 來實作，但是需要開許多線程，否則若只開一個線程沒收到資料就會一直阻塞到超時。
| server |  | client | 
| :---: | :---: | :---: | 
|  | <--發送請求 |  | 
| 阻塞 | 至 | 連上 | 
|  | 收到請求，發送資料--> |  | 
|  | <--收到資料，回覆成功 |  |

而協程就不需要多線程，當呼叫某函數後可以在某個地方掛起，等待某個事件到來再回到該函數中繼續執行
![img](https://github.com/JrPhy/CPP_tutorial/blob/main/img/coroutine.jpg)\
## 一、線程(Thread)與協程(coroutine)
線程為執行進程(Process)的最小單位，一個進程中可以有多個線程在執行。若是一個進程中有多個線程，就會在該進程的 **stack** 中去分配位置給進程中的線程，若線程數量太多就會需要做 [Context Switch](https://github.com/JrPhy/Multiple_Thread/blob/main/%E4%B8%8A%E4%B8%8B%E6%96%87%E4%BA%A4%E6%8F%9B%E8%88%87%E5%8E%9F%E5%AD%90%E6%93%8D%E4%BD%9C.md#2-%E4%B8%8A%E4%B8%8B%E6%96%87%E4%BA%A4%E6%8F%9B-context-switch)，就會需要額外的資源去記錄目前執行到的位置、優先及等數據，並且把紀錄的數據丟進 CPU 的 Register 中，就是一次的 **system call**，所以會盡量避免這件事。且在多線程為了避免 Race Condition，還會使用鎖或信號量這種由 OS 來管理的函數，所以會占用相當多的資源

協程僅為一個**可以暫停與恢復的函數**，可以在 A 線程中暫停，並在 B 線程中恢復，此種暫停與恢復並不依賴於 OS，所以牽涉到的 sys call 會比線程少非常多，整體來說會比線程更輕量，所以在非阻塞 I/O 或是網路通信會很常看到 coroutine。

## 二、C++ 中的 coroutine
coroutine 在 C++ 20 中才算剛起步，僅有一些關鍵字讓編譯器知道該函數為協程函數，其餘都需要自行實作。在 C++ 20 中的協程是 [stackless](https://en.cppreference.com/w/cpp/language/coroutines)，且當函數中有 ```co_await, co_yield, co_return``` 這三個關鍵字，那麼此函數即為協程函數，編譯器會去認這三個關鍵字。

co_await 協程暫停和恢復的點\
co_yield 是用暫停協程并且放值進 promise\
co_return 放值進 promise 並結束此協程\

#### 1. stackless 無棧的意思
在 C/C++ 中的函數如果有執行到，編譯器都會去分配一個 stack 空間，執行完就釋放。而當把函數放進 Thread 中，那麼在每個 Thread 裡面都會去開一個 stack 放該函數，所以開 n 個 Thread 就會分配 n 個 stack 函數的空間。而如果是協程函數，那在一個 process 中只會有一個 stack 函數的空間，直到要再呼叫時再回到此函數，這也是為什麼協程函數會比線程省資源的原因。

#### 2. promise_type
C++ 20 中僅提供關鍵字讓編譯器去辨認，並沒有一個很好的封裝，所以實作的部分還需要自行實作，在此先來看一個[簡單的例子](https://github.com/chaelim/Coroutine/blob/master/Examples/simple.cpp)
```C++
#include <coroutine>
#include <cstdio>

struct Generator {
    struct promise_type {
        static void* operator new(std::size_t s) {
            printf("new operator size=%zd\n", s);
            return ::operator new(s);
        }

        static void operator delete(void* ptr, std::size_t s) {
            printf("delete operator size=%zd\n", s);
            ::operator delete(ptr);
        }
        int value = 0;
        Generator get_return_object() noexcept { return Generator(std::coroutine_handle<promise_type>::from_promise(*this)); }
        std::suspend_never initial_suspend() noexcept { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        void unhandled_exception() noexcept { }
        void return_value(int v) noexcept { value = v; }
    };
    std::coroutine_handle<promise_type> handle;
    Generator(std::coroutine_handle<promise_type> coro) noexcept : handle(coro) { }
    Generator(Generator&& other) noexcept : handle(other.handle) { other.handle = nullptr; }
    ~Generator() {
        if (handle)
            handle.destroy();
    }
    int value() const noexcept { return handle.promise().value; }
};

Generator generator() noexcept
{ co_return 42; }

int main() {
    Generator t = generator();
    printf("Return value=%d\n", t.value());
}
```
其中的 promise_type 在其他語言中相當於 async 關鍵字，必定會有以下成員，可以當作 promise_type 的最低需求樣板。此例子中的成員分別有以下用途
```c++
simple get_return_object() noexcept { return simple(std::coroutine_handle<promise_type>::from_promise(*this)); }
// 負責從 promise_type 產生 std::coroutine_handle，讓外部可以使用協程的控制權。這樣我們可以在 main() 中使用 simple 物件，存取其返回值。
std::suspend_never initial_suspend() noexcept { return {}; }
// 初始化協程，std::suspend_never 表示協程會直接執行而不會停下來等待。
std::suspend_always final_suspend() noexcept { return {}; }
// 結束協程，std::suspend_always 表示協程執行完畢後 會進入暫停狀態，而不會立即被銷毀，允許外部控制其結束時的行為。
void unhandled_exception() noexcept { }
void return_value(int v) noexcept { value = v; }
// 當 co_return 被使用時，這個函數負責將返回的值存入 value 變數中，讓外部可以使用它。如 Simple() 中的 co_return 42;。若不使用則引數與實作皆為空
```
執行後會得到下列結果，一開始在建構 simple 會先計算整體的大小，然後呼叫協程並印出協程的返回值，最後在銷毀 simple。
```
new operator size=40
Return value=42
delete operator size=40
```
在此用到了 std::suspend_never 與 std::suspend_always，這兩個會根據場用場景的不同而選擇
|   | std::suspend_never | std::suspend_always |
| --- | --- | --- |
| 終止行為 | 直接清理 | 需手動銷毀 |
| 適用場景 | 一次性 | 須延遲或回傳值 |
| 範例 | 計算 | 異步、生成器 |
#### 3. co_return
在上述的例子中若要使用 co_return 並返回值，那就要在 promise_type 中的 return_value 中傳值進去，並將數值給成員變數。當然用到 co_return 時通常意味著函數被呼叫完了，所以 final_suspend 通常搭配 std::suspend_always。若是協程函數不想返回值，則需要在另一個類別中寫 return_void，不能與 return_value 寫在同一個 promise_type。

#### 4. co_yield
協程函數更多時候是在等待從其他地方傳來的值或是賦值，在此將上述例子改成生成器的例子，為了讓其使用起來更方便，在前面加上 template，在此 initial_suspend 使用 suspend_always，因為不希望在函數一建立就呼叫，並加入 ```std::suspend_always yield_value(T v) { value = v; return {}; }``` 來改變值。如此一來就可以寫出類似 python 中的 range。
```c++
#include <coroutine>
#include <cstdio>
#include <iostream>
template<typename T>
struct Generator {
    struct promise_type {
        static void* operator new(std::size_t s) {
            printf("new operator size=%zd\n", s);
            return ::operator new(s);
        }

        static void operator delete(void* ptr, std::size_t s) {
            printf("delete operator size=%zd\n", s);
            ::operator delete(ptr);
        }
        T value;
        Generator get_return_object() noexcept { return Generator(std::coroutine_handle<promise_type>::from_promise(*this)); }
        std::suspend_always initial_suspend() noexcept { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() noexcept { }
        std::suspend_always yield_value(T v) { value = v; return {}; }
    };
    std::coroutine_handle<promise_type> handle;
    Generator(std::coroutine_handle<promise_type> coro) noexcept : handle(coro) { }
    Generator(Generator&& other) noexcept : handle(other.handle) { other.handle = nullptr; }
    ~Generator() {
        if (handle)
            handle.destroy();
    }
    T next() {
        handle.resume(); 
        return handle.promise().value; 
    }
    bool done() const { return handle.done(); }
};

Generator<int> range(int start, int count) {
    for (int i = start; i < start + count; ++i)
        co_yield i;
}

int main() {
    auto gen = range(0, 5);
    while (!gen.done())
        std::cout << gen.next() << " ";
    std::cout << "\n";
    return 0;
}
```
可以看到 C++20 中的協程只是剛起步，甚至可以說僅提供關鍵字讓編譯器辨認，相較之下 JS 中用 async/await 的協程例子就簡潔許多
```JS
function* generator() {
    for (let i = 1; i <= 5; i++) {
        yield i; // 逐步產生數值
        console.log("產生值!");
    }
    return; // 結束產生器
}

async function asyncExample() {
    console.log("開始非同步函數...");
    await new Promise(resolve => setTimeout(resolve, 1000)); // 模擬非同步等待
    console.log("非同步作業完成!");
    return 42; // 返回結果
}

async function main() {
    let gen = generator();
    console.log("開始迭代產生器...");
    for (let value of gen) {
        console.log("收到值:", value);
    }
    console.log("執行非同步函數...");
    let result = await asyncExample();
    console.log("收到非同步結果:", result);
}

main();
```
在 js 中如果函數中有 await 關鍵字，那麼函數就需要用 async 關鍵字修飾，所以 main 函數中有 ```await asyncExample()``` 那就需要用 async 修飾，asyncExample 函數也是如此，generator 中則不需要，而 generator 中有個 yield 關鍵字，會讓函數執行到此段後先暫停，帶回傳該值後再從原本執行到的地方繼續下去。執行結果如下
```
main();
開始迭代產生器...
收到值: 1
產生值!
收到值: 2
產生值!
收到值: 3
產生值!
收到值: 4
產生值!
收到值: 5
產生值!
執行非同步函數...
開始非同步函數...
Promise {<pending>}
非同步作業完成!
收到非同步結果: 42
```
可以看到雖然 generator() 先被執行，但卻不是先將 generator() 執行完再去執行 ```for (let value of gen) { console.log("收到值:", value); }```，而是收到值與產生值交替出現，就是 yield 的特性。交替執行完後就開始順序往下執行，而執行到 ```await new Promise(resolve => setTimeout(resolve, 1000));``` 就會等待 1 秒後在繼續往下執行，最後就會出現 ```收到非同步結果: 42```。若沒有 await，async 函式仍然返回 Promise，但是會在收到非同步結果後才出現，因為 Promise 本身是多線程的。C++ 中雖然有 std::async 關鍵字，但是與 js 中的有非常大的不同，std::async 仍是另外開執行續，並搭配 std::future 中的 future.get() 來阻塞等待直到拿到值，並非是協程，以下即為用 c++ coroutine 改寫
```C++
#include <iostream>
#include <coroutine>
#include <thread>
#include <chrono>

struct Generator {
    struct promise_type {
        int current_value;
        std::suspend_always yield_value(int value) {
            current_value = value;
            return {};
        }
        std::suspend_always initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        Generator get_return_object() { return Generator{this}; }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };

    struct iterator {
        std::coroutine_handle<promise_type> handle;

        iterator(std::coroutine_handle<promise_type> h) : handle(h) {}
        iterator& operator++() {
            handle.resume();
            return *this;
        }
        int operator*() const { return handle.promise().current_value; }
        bool operator!=(const iterator&) const { return !handle.done(); }
    };

    std::coroutine_handle<promise_type> handle;
    Generator(promise_type* p) : handle(std::coroutine_handle<promise_type>::from_promise(*p)) {}
    ~Generator() { if (handle) handle.destroy(); }

    iterator begin() { handle.resume(); return iterator{handle}; }
    iterator end() { return iterator{nullptr}; }
};

Generator generator() {
    for (int i = 1; i <= 5; i++) {
        co_yield i;
        std::cout << "產生值!" << std::endl;
    }
}

struct AsyncExample {
    struct promise_type {
        static void* operator new(std::size_t s) {
            return ::operator new(s);
        }

        static void operator delete(void* ptr, std::size_t s) {
            ::operator delete(ptr);
        }
        int value = 0;
        AsyncExample get_return_object() noexcept { return AsyncExample(std::coroutine_handle<promise_type>::from_promise(*this)); }
        std::suspend_never initial_suspend() noexcept { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        void unhandled_exception() noexcept { }
        void return_value(int v) noexcept { value = v; }
    };
    std::coroutine_handle<promise_type> handle;
    AsyncExample(std::coroutine_handle<promise_type> coro) noexcept : handle(coro) { }
    AsyncExample(AsyncExample&& other) noexcept : handle(other.handle) { other.handle = nullptr; }
    ~AsyncExample() {
        if (handle)
            handle.destroy();
    }
    int value() const noexcept { return handle.promise().value; }
};


AsyncExample asyncExample() {
    std::cout << "開始非同步函數..." << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "非同步作業完成!" << std::endl;
    co_return 42;
}

int main() {
    std::cout << "開始迭代產生器..." << std::endl;
    for (auto value : generator()) {
        std::cout << "收到值: " << value << std::endl;
    }

    std::cout << "執行非同步函數..." << std::endl;
    auto result = asyncExample();
    std::cout << "收到非同步結果: " << result.value() << std::endl;

    return 0;
}
```
#### 5. co_await
在上述例子中仍使用了 thread 來做阻塞，當然用 coroutine 就是不希望開 thread，所以可以改用 co_await．相較於前面兩個關鍵字，在實作 await 時並不需要有 promise_type，而需要以下三個函數
```
await_ready() // 是否阻塞
await_suspend() // 掛起
await_resume() // 恢復
```
若一個類別 A 中含有這三個成員，則 A 為 Awaiter 及 Awaitable，若類別 B 不包含這三個成員，但是包含 A，則 B 為 Awaitable 但不為 Awaiter．
```C++
#include <coroutine>
#include <iostream>

struct Awaiter {
    bool await_ready() {
        std::cout << "await ready or not" << std::endl;
        return true;
    }

    void await_resume()
    { std::cout << "await resumed" << std::endl; }

    void await_suspend(std::coroutine_handle<> h)
    { std::cout << "await suspended" << std::endl; }
};

struct Promise {
  struct promise_type {
    auto get_return_object() noexcept {
      std::cout << "get return object" << std::endl;
      return Promise();
    }

    auto initial_suspend() noexcept {
      std::cout << "initial suspend, return never" << std::endl;
      return std::suspend_never{};
    }

    auto final_suspend() noexcept {
      std::cout << "final suspend, return never" << std::endl;
      return std::suspend_never{};
    }

    void unhandled_exception() {
      std::cout << "unhandle exception" << std::endl;
      std::terminate();
    }

    void return_void() {
      std::cout << "return void" << std::endl;
      return;
    }
  };
};

Promise CoroutineFunc() {
  std::cout << "before co_await" << std::endl;
  co_await Awaiter();
  std::cout << "after co_await" << std::endl;
}

int main() {
  std::cout << "main() start" << std::endl;
  CoroutineFunc();
  std::cout << "main() exit" << std::endl;
}
```
https://www.cnblogs.com/Netsharp/p/17279750.html\
https://juejin.cn/post/7312727134297538594\
