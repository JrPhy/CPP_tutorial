協程這一概念早在 1985 年就已提出，是一個**可以暫停與恢復的函數**。其他語言中已有相關的實作，例如 PYTHON 與 JS 中的 async/await，以及 golang 的 gorouting 等，C++ 則是在 C++20 中才納入標準。在網頁與網頁使用 API 去要資料，或是去跟一些 IOT 設備要資料時，常常是一方去發請求，對方接收到請求後把資料發給你，然後再回一個狀態回去就完成請求。在早期 C/C++ 會使用線程 (Thread) 來實作，但是需要開許多線程，否則若一個線程沒收到資料就會一直阻塞到超時。
| server |  | client | 
| :---: | :---: | :---: | 
|  | <--發送請求 |  | 
| 阻塞 | 至 | 連上 | 
|  | 收到請求，發送資料--> |  | 
|  | <--收到資料，回覆成功 |  |

## 一、線程(Thread)與協程(coroutine)
線程為執行進程(Process)的最小單位，一個進程中可以有多個線程在執行。若是一個進程中有多個線程，就會在該進程的 **stack** 中去分配位置給進程中的線程，若線程數量太多就會需要做 [Context Switch](https://github.com/JrPhy/Multiple_Thread/blob/main/%E4%B8%8A%E4%B8%8B%E6%96%87%E4%BA%A4%E6%8F%9B%E8%88%87%E5%8E%9F%E5%AD%90%E6%93%8D%E4%BD%9C.md#2-%E4%B8%8A%E4%B8%8B%E6%96%87%E4%BA%A4%E6%8F%9B-context-switch)，就會需要額外的資源去記錄目前執行到的位置、優先及等數據，並且把紀錄的數據丟進 CPU 的 Register 中，就是一次的 **system call**，所以會盡量避免這件事。且在多線程為了避免 Race Condition，還會使用鎖或信號量這種由 OS 來管理的函數，所以會占用相當多的資源

協程僅為一個**可以暫停與恢復的函數**，可以在 A 線程中暫停，並在 B 線程中恢復，此種暫停與恢復並不依賴於 OS，所以牽涉到的 sys call 會比線程少非常多，整體來說會比線程更輕量，所以在非阻塞 I/O 或是網路通信會很常看到 coroutine。

## 二、C++ 中的 coroutine
coroutine 在 C++ 20 中才算剛起步，僅有一些關鍵字讓編譯器知道該函數為協程函數，其餘都需要自行實作。在 C++ 20 中的協程是 [stackless](https://en.cppreference.com/w/cpp/language/coroutines)，且當函數中有 ```co_await, co_yield, co_return``` 這三個關鍵字，那麼此函數即為協程函數，編譯器會去認這三個關鍵字。

co_await：等待一個可等待的物件完成。當執行到 co_await 時，協程的執行會暫停，讓出控制權給調度器，直到可等待物件完成後再繼續執行。\
co_yield：這個關鍵字用於暫停當前的協程並返回一個值，而協程的狀態會被保留下來，當再次調用時，它會從上次暫停的地方繼續執行。\
co_return：這個關鍵字用來終止協程並返回一個結果值。它類似於一般函數中的 return，但專門用於協程，確保適當處理返回值的類型。

#### 1. stackless 無棧的意思
在 C/C++ 中的函數如果有執行到，編譯器都會去分配一個 stack 空間，執行完就釋放。而當把函數放進 Thread 中，那麼在每個 Thread 裡面都會去開一個 stack 放該函數，所以開 n 個 Thread 就會分配 n 個 stack 函數的空間。而如果是協程函數，那在一個 process 中只會有一個 stack 函數的空間，直到要再呼叫時再回到此函數，這也是為什麼協程函數會比線程省資源的原因。

#### 2. 協程函數範例
C++ 20 中僅提供關鍵字讓編譯器去辨認，並沒有一個很好的封裝，所以實作的部分還需要自行實作，在此先來看 JS 中用 async/await 的協程例子
```JS
function* generator() {
    for (let i = 1; i <= 5; i++) {
        yield i; // 逐步產生數值
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
```C++
#include <iostream>
#include <coroutine>
#include <thread>
#include <chrono>

struct Generator {
    struct promise_type {
        int current_value;
        Generator get_return_object() { 
            return Generator{std::coroutine_handle<promise_type>::from_promise(*this)}; 
        }
        std::suspend_always initial_suspend() { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        std::suspend_always yield_value(int value) {
            current_value = value;
            return {};
        }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };

    std::coroutine_handle<promise_type> handle;

    Generator(std::coroutine_handle<promise_type> h) : handle(h) {}
    ~Generator() { if (handle) handle.destroy(); }

    int next() {
        if (!handle || handle.done()) return -1;
        handle.resume();
        return handle.promise().current_value;
    }
};

Generator counter() {
    for (int i = 1; i <= 5; ++i) {
        co_yield i; // 使用 co_yield 逐步返回數值
    }
    co_return; // 使用 co_return 結束協程
}

struct Awaitable {
    bool await_ready() { return false; }
    void await_suspend(std::coroutine_handle<>) {
        std::this_thread::sleep_for(std::chrono::seconds(1)); // 模擬非同步等待
    }
    void await_resume() {}
};

Generator async_example() {
    std::cout << "開始非同步協程...\n";
    co_await Awaitable{}; // 使用 co_await 來模擬等待非同步作業
    std::cout << "非同步作業完成!\n";
    co_yield 42; // 使用 co_yield 產生數值
    co_return;   // 使用 co_return 結束協程
}

int main() {
    Generator gen = async_example();

    while (int value = gen.next() && value != -1) {
        std::cout << "收到值: " << value << '\n';
    }

    std::cout << "協程執行完畢！\n";
    return 0;
}

```

