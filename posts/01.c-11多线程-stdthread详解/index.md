# C&#43;&#43;11多线程 Std::thread详解

## 前言

&gt;  我们知道，当程序运行起来，生成一个进程，该进程所属的主线程开始自动运行，C/C&#43;&#43;的主线程就是main函数；当主线程从main()函数返回，则整个进程执行完毕。

所以主线程是从main()开始执行，其中我们自己创建的线程，也需要从一个函数开始运行（初始函数），一旦这个函数运行完毕，线程也结束运行。&lt;font color=&#34;red&#34;&gt;所以整个进程是否执行完毕的标志是：主线程是否执行完，如果主线程执行完毕了，就代表整个进程执行完毕了，此时如果其他子线程还没有执行完，也会被强行终止（符合大部分规律，也有例外）。&lt;/font&gt;

我们需要明白，如果有两个线程在跑，相当于整个程序中有两条线在同时走，即使一条被阻塞，另一条也能运行。

在C&#43;&#43;11以前，使用线程库特别麻烦，C&#43;&#43;11提供了一个新标准线程库，即`thread`，意味着C&#43;&#43;语言本身增加对多线程的支持，意味着可移植性（跨平台），这大大减少开发人员的工作量。

## std::thread

### 构造函数

构造函数如下，其含义分别注释：

```cpp
thread() noexcept; // 创建不代表线程的新线程对象。
thread( thread&amp;&amp; other ) noexcept; // 移动构造函数。构造表示曾为 other 所表示的执行线程的 thread 对象。此调用后 other 不再表示执行线程。一般需要使用move方法
template&lt; class Function, class... Args &gt; 
   explicit thread( Function&amp;&amp; f, Args&amp;&amp;... args ); // 构造新的 std::thread 对象并将它与执行线程关联。新的执行线程开始执行。其中f为可调用函数对象，args为传递的函数参数，一定要相互对应。
thread(const thread&amp;) = delete; // 复制构造函数被删除，线程不可复制。
```

创建线程对象总共有以下几种方法：

```cpp
#include &lt;iostream&gt;
#include &lt;thread&gt;
#include &lt;unistd.h&gt; // 提供sleep函数

void sum(int a, int b) {
    std::cout &lt;&lt; a &#43; b &lt;&lt; std::endl;
}
int main() {
    std::thread t1(); // 不是线程，使用第一种构造方法
    int a = 1, b = 2;
    std::thread t2(sum, a, b); // 是线程，按值传递，使用第三种构造方法
    std::thread t3(sum, std::ref(a), std::ref(b)); // 是线程，按引用传递，使用第三种构造方法
    std::thread t4(std::move(t2)); // t3现在是线程，运行sum，t2不再是线程。这是使用第二种构造方法。
    t2.join();
    t4.join();
    return 0;
}
```

官网上的给的代码如下（这种种类更多，适合用来分析学习，上面列举的内容实际上就够用了）：

```cpp
#include &lt;iostream&gt;
#include &lt;utility&gt;
#include &lt;thread&gt;
#include &lt;chrono&gt;
 
void f1(int n)
{
    for (int i = 0; i &lt; 5; &#43;&#43;i) {
        std::cout &lt;&lt; &#34;Thread 1 executing\n&#34;;
        &#43;&#43;n;
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
}
 
void f2(int&amp; n)
{
    for (int i = 0; i &lt; 5; &#43;&#43;i) {
        std::cout &lt;&lt; &#34;Thread 2 executing\n&#34;;
        &#43;&#43;n;
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
}
 
class foo
{
public:
    void bar()
    {
        for (int i = 0; i &lt; 5; &#43;&#43;i) {
            std::cout &lt;&lt; &#34;Thread 3 executing\n&#34;;
            &#43;&#43;n;
            std::this_thread::sleep_for(std::chrono::milliseconds(10));
        }
    }
    int n = 0;
};
 
class baz
{
public:
    void operator()()
    {
        for (int i = 0; i &lt; 5; &#43;&#43;i) {
            std::cout &lt;&lt; &#34;Thread 4 executing\n&#34;;
            &#43;&#43;n;
            std::this_thread::sleep_for(std::chrono::milliseconds(10));
        }
    }
    int n = 0;
};
 
int main()
{
    int n = 0;
    foo f;
    baz b;
    std::thread t1; // t1 is not a thread
    std::thread t2(f1, n &#43; 1); // pass by value
    std::thread t3(f2, std::ref(n)); // pass by reference
    std::thread t4(std::move(t3)); // t4 is now running f2(). t3 is no longer a thread
    std::thread t5(&amp;foo::bar, &amp;f); // t5 runs foo::bar() on object f
    std::thread t6(b); // t6 runs baz::operator() on a copy of object b
    t2.join();
    t4.join();
    t5.join();
    t6.join();
    std::cout &lt;&lt; &#34;Final value of n is &#34; &lt;&lt; n &lt;&lt; &#39;\n&#39;;
    std::cout &lt;&lt; &#34;Final value of f.n (foo::n) is &#34; &lt;&lt; f.n &lt;&lt; &#39;\n&#39;;
    std::cout &lt;&lt; &#34;Final value of b.n (baz::n) is &#34; &lt;&lt; b.n &lt;&lt; &#39;\n&#39;;
}
```

### 观察器

#### std::thread::joinable

```cpp
bool joinable() const noexcept;
```

这个函数用来检查this是否标识了一个活动的执行线程，即如果this标识了一个活动的执行线程，就返回true，否则返回false。

```cpp
#include &lt;iostream&gt;
#include &lt;thread&gt;
#include &lt;chrono&gt;
 
void foo()
{
    std::this_thread::sleep_for(std::chrono::seconds(1));
}
 
int main()
{
    std::thread t;
    std::cout &lt;&lt; &#34;before starting, joinable: &#34; &lt;&lt; std::boolalpha &lt;&lt; t.joinable()
              &lt;&lt; &#39;\n&#39;;
 
    t = std::thread(foo);
    std::cout &lt;&lt; &#34;after starting, joinable: &#34; &lt;&lt; t.joinable() 
              &lt;&lt; &#39;\n&#39;;
 
    t.join();
    std::cout &lt;&lt; &#34;after joining, joinable: &#34; &lt;&lt; t.joinable() 
              &lt;&lt; &#39;\n&#39;;
}
```

Output:

```
before starting, joinable: false
after starting, joinable: true
after joining, joinable: false
```

#### std::thread::get_id

```cpp
std::thread::id get_id() const noexcept;
```

返回标识与 *this 关联的线程的 `std::thread::id` 。如果没有关联线程，则返回默认构造的`std::thread::id`。

```cpp
#include &lt;iostream&gt;
#include &lt;thread&gt;
#include &lt;chrono&gt;
 
void foo()
{
    std::this_thread::sleep_for(std::chrono::seconds(1));
}
 
int main()
{
    std::thread t1(foo);
    std::thread::id t1_id = t1.get_id();
 
    std::thread t2(foo);
    std::thread::id t2_id = t2.get_id();
 
    std::cout &lt;&lt; &#34;t1&#39;s id: &#34; &lt;&lt; t1_id &lt;&lt; &#39;\n&#39;;
    std::cout &lt;&lt; &#34;t2&#39;s id: &#34; &lt;&lt; t2_id &lt;&lt; &#39;\n&#39;;
 
    t1.join();
    t2.join();
}
```

Possible output:

```
t1&#39;s id: 2
t2&#39;s id: 3
```

### 操作

#### std::thread::join

```cpp
void join();
```

阻塞当前线程直至 `*this`所标识的线程结束其执行。`*this` 所标识的线程的完成同步于对应的从 join() 成功返回。`*this` 自身上不进行同步。同时从多个线程在同一 thread 对象上调用 join() 构成数据竞争，导致未定义行为。

可能引发的异常为：`std::system_error`

```cpp
#include &lt;iostream&gt;
#include &lt;thread&gt;
#include &lt;chrono&gt;
 
void foo()
{
    // simulate expensive operation
    std::this_thread::sleep_for(std::chrono::seconds(1));
}
 
void bar()
{
    // simulate expensive operation
    std::this_thread::sleep_for(std::chrono::seconds(1));
}
 
int main()
{
    std::cout &lt;&lt; &#34;starting first helper...\n&#34;;
    std::thread helper1(foo);
 
    std::cout &lt;&lt; &#34;starting second helper...\n&#34;;
    std::thread helper2(bar);
 
    std::cout &lt;&lt; &#34;waiting for helpers to finish...&#34; &lt;&lt; std::endl;
    helper1.join();
    helper2.join();
 
    std::cout &lt;&lt; &#34;done!\n&#34;;
}
```

Output:

```
starting first helper...
starting second helper...
waiting for helpers to finish...
done!
```

#### std::thread::detach

```cpp
void detach();
```

从 thread 对象分离执行线程，允许执行独立地持续。一旦该线程退出，则释放任何分配的资源。调用 detach 后 *this 不再占有任何线程。

可能引发的异常为：`std::system_error`

```cpp
#include &lt;iostream&gt;
#include &lt;chrono&gt;
#include &lt;thread&gt;
 
void independentThread() 
{
    std::cout &lt;&lt; &#34;Starting concurrent thread.\n&#34;;
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout &lt;&lt; &#34;Exiting concurrent thread.\n&#34;;
}
 
void threadCaller() 
{
    std::cout &lt;&lt; &#34;Starting thread caller.\n&#34;;
    std::thread t(independentThread);
    t.detach();
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout &lt;&lt; &#34;Exiting thread caller.\n&#34;;
}
 
int main() 
{
    threadCaller();
    std::this_thread::sleep_for(std::chrono::seconds(5));
}
```

Possible output:

```
Starting thread caller.
Starting concurrent thread.
Exiting thread caller.
Exiting concurrent thread.
```

这个示例即表明了主线程将先执行完。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://lruihao.cn/posts/01.c-11%E5%A4%9A%E7%BA%BF%E7%A8%8B-stdthread%E8%AF%A6%E8%A7%A3/  

