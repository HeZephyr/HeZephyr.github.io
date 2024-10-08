# 线程API

## 线程创建

编写多线程程序的第一件事就是创建新线程，因此必须有某种线程创建接口。在 POSIX 中，这很容易：

```c
#include &lt;pthread.h&gt;
int pthread_create(     pthread_t*              thread, 
                  const pthread_attr_t*         attr, 
                        void*                   (*start_routine)(void*),
                        void*                   arg);
```

这个函数声明有四个参数：`thread`、`attr`、`start_routine` 和 `arg`。第一个参数 `thread` 是指向 `pthread_t` 类型结构的指针；我们将使用该结构与线程交互，因此需要将其传递给 `pthread_create()` 以对其进行初始化。

第二个参数 `attr` 用于指定该线程可能具有的任何属性。例如包括设置栈大小或可能有关线程的调度优先级的信息。通过单独调用 `pthread_attr_init()` 来初始化属性；有关详细信息，请参阅手册页：`man pthread_create`。然而，在大多数情况下，默认值就可以了，在这种情况下，我们将简单地传递 NULL 值。

第三个参数是最复杂的，但实际上只是询问：这个线程应该开始在哪个函数中运行？在 C 中，我们将其称为函数指针，这告诉我们预期的内容：函数名称（`start_routine`），它传递一个类型为 `void *` 的单个参数（如`start_routine`后面的括号中所示），以及它返回一个 `void *` 类型的值（即，一个 void 指针）。如果此例程需要整数参数而不是 void 指针，则声明将如下所示：

```c
int pthread_create(..., // first two args are the same
                    void *  (*start_routine)(int),
                    int     arg);
```

如果例程的参数是一个 void 指针，但返回值是一个整数，那么就会是这样：

```c
int pthread_create(..., // first two args are the same
                    int     (*start_routine)(void *),
                    void *  arg);
```

最后，第四个参数 `arg` 正是要传递给线程开始执行的函数的参数。你可能会问：为什么我们需要这些 void 指针？答案其实很简单：&lt;font color=&#34;red&#34;&gt;将 void 指针作为函数`start_routine`的参数，可以让我们传递任何类型的参数&lt;/font&gt;；将它作为返回值，可以让线程返回任何类型的结果。

还有函数的返回值，如果运行正常，则返回 0（否则为错误代码：EAGAIN、EINVAL、EPERM）。

让我们看看下面这段代码。

```c
#include &lt;assert.h&gt;
#include &lt;stdio.h&gt;

#include &#34;common.h&#34;
#include &#34;common_threads.h&#34;

typedef struct {
    int a;
    int b;
} myarg_t;

void *mythread(void *arg) {
    myarg_t *args = (myarg_t *) arg;
    printf(&#34;%d %d\n&#34;, args-&gt;a, args-&gt;b);
    return NULL;
}

int main(int argc, char *argv[]) {
    pthread_t p;
    myarg_t args = { 10, 20 };

    int rc = pthread_create(&amp;p, NULL, mythread, &amp;args);
    assert(rc == 0);
    (void) pthread_join(p, NULL);
    printf(&#34;done\n&#34;);
    return 0;
}
```

在这里，我们只是创建了一个线程，它传递了两个参数，并打包成我们自己定义的单一类型（`myarg t`）。线程创建后，可以简单地将其参数转换为它所期望的类型，从而根据需要解包参数。就是这样！一旦创建了线程，你就真正拥有了另一个活生生的执行实体，它拥有自己的调用栈，与程序中当前存在的所有线程运行在同一地址空间。程序的运行结果如下：

```shell
❯ make thread_create
gcc -o thread_create thread_create.c -Wall -Werror -I../include -pthread
❯ ./thread_create
10 20
done
```

## 等待线程完成

上面的例子展示了如何创建一个线程。但是，如果您想等待线程完成，会发生什么情况？你需要做一些特别的事情才能等待完成；特别是，您必须调用例程 `pthread_join()`。

```c
int pthread_join(pthread_t thread, void **value_ptr);
```

此例程需要两个参数。第一个参数的类型是 `pthread_t`，用于指定等待哪个线程。该变量由线程创建例程初始化（将指针作为参数传递给 `pthread create()`）；如果保留该变量，就可以用它来等待该线程终止。

第二个参数是指向你期望返回值的指针。由于该例程可以返回任何值，因此它被定义为返回 void 的指针；由于 `pthread_join() `例程会改变传入参数的值，因此你需要传入指向该值的指针，而不仅仅是该值本身。

让我们看下面这段代码，在代码中，再次创建了一个单线程，并通过 `myarg_t` 结构传递了几个参数。返回值使用 `myret_t` 类型。一旦线程运行完毕，一直在 `pthread_join() `例程 中等待的主线程就会返回，我们就可以访问从线程返回的值，即 `myret_t` 中的值。

```c
#include &lt;stdio.h&gt;
#include &lt;stdlib.h&gt;

#include &#34;common.h&#34;
#include &#34;common_threads.h&#34;

typedef struct {
    int a;
    int b;
} myarg_t;

typedef struct {
    int x;
    int y;
} myret_t;

void *mythread(void *arg) {
    myarg_t *args = (myarg_t *) arg;
    printf(&#34;args %d %d\n&#34;, args-&gt;a, args-&gt;b);
    myret_t *rvals = malloc(sizeof(myret_t));
    assert(rvals != NULL);
    rvals-&gt;x = 1;
    rvals-&gt;y = 2;
    return (void *) rvals;
}

int main(int argc, char *argv[]) {
    pthread_t p;
    myret_t *rvals;
    myarg_t args = { 10, 20 };
    Pthread_create(&amp;p, NULL, mythread, &amp;args);
    Pthread_join(p, (void **) &amp;rvals);
    printf(&#34;returned %d %d\n&#34;, rvals-&gt;x, rvals-&gt;y);
    free(rvals);
    return 0;
}
```

关于这个例子，有几点需要注意。首先，很多时候我们不必对参数进行这些痛苦的打包和拆包。例如，如果我们只是创建一个不带参数的线程，我们可以在创建线程时将 NULL 作为参数传递进去。同样，如果我们不关心返回值，也可以将 NULL 传递给 `pthread_join()`。

其次，如果我们只传递一个值（如 int），就不必将其打包为参数。

如下面这段代码所示，在这种情况下，我们不必将参数和返回值打包到结构内部。

```c
#include &lt;stdio.h&gt;

#include &#34;common.h&#34;
#include &#34;common_threads.h&#34;

void *mythread(void *arg) {
    long long int value = (long long int) arg;
    printf(&#34;%lld\n&#34;, value);
    return (void *) (value &#43; 1);
}

int main(int argc, char *argv[]) {
    pthread_t p;
    long long int rvalue;
    Pthread_create(&amp;p, NULL, mythread, (void *) 100);
    Pthread_join(p, (void **) &amp;rvalue);
    printf(&#34;returned %lld\n&#34;, rvalue);
    return 0;
}
```

第三，我们应该注意，从线程返回值的方式必须非常谨慎。尤其是，千万不要返回指向线程调用栈中分配的指针。如果这样做，你觉得会发生什么？(想想吧！）下面是一段危险代码的示例，它是根据上面的示例修改的。

```c
void *mythread(void *arg) {
    myarg_t *m = (myarg_t *)arg;
    printf(&#34;%d %d\n&#34;, m-&gt;a, m-&gt;b);
    myret_t r; // ALLOCATED ON STACK: BAD!
    r.x = 1;
    r.y = 2;
    return (void *)&amp;r;
}
```

在这种情况下，变量 r 被分配到 `mythread` 的栈中。然而，当它返回时，该值会被自动解除分配（毕竟这就是栈如此易于使用的原因！），因此，将指向已解除分配的变量的指针传回会导致各种糟糕的结果。

最后，你可能会注意到，使用 `pthread_create()` 创建线程，然后立即调用 `pthread_join()` 是一种非常奇怪的创建线程的方法。事实上，有一种更简单的方法可以完成这一任务，那就是&lt;font color=&#34;red&#34;&gt;过程调用&lt;/font&gt;。显然，我们通常要创建不止一个线程并等待它完成，否则使用线程就没有什么意义了。

我们应该注意，并非所有多线程代码都使用`join`例程。例如，多线程网络服务器可能会创建许多工作线程，然后使用主线程接受请求并将请求无限期地传递给工作线程。因此，这种长寿命程序可能不需要`join`。然而，创建线程执行特定任务（并行）的并行程序可能会使用 `join` 来确保在退出或进入下一阶段计算之前，所有这些工作都已完成。

## 锁

除了线程创建和等待线程完成之外，POSIX 线程库提供的下一组最有用的函数可能就是那些通过**锁**为临界区提供互斥的函数了。为此目的使用的最基本的一对例程如下：

```c
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

例程应该易于理解和使用。当您的代码区域是临界区，因此需要受到保护以确保正确操作时，锁非常有用。你大概可以想象代码的样子：

```c
pthread_mutex_t lock;
pthread_mutex_lock(&amp;lock);
x = x &#43; 1; // 或者不管你的临界区是什么
pthread_mutex_unlock(&amp;lock);
```

代码的意图如下：如果调用 `pthread_mutex_lock()` 时没有其他线程持有锁，则该线程将获取锁并进入临界区。如果另一个线程确实持有锁，则尝试获取锁的线程将不会从调用中返回，直到它获得锁（这意味着持有锁的线程已通过unlock调用释放了锁）。当然，在给定时间，许多线程可能会卡在锁获取函数内等待；然而，只有获得锁的线程才应该调用`unlock`。

不幸的是，这段代码在两个重要方面被破坏了。第一个问题是&lt;font color=&#34;red&#34;&gt;缺乏正确的初始化&lt;/font&gt;。所有锁都必须正确初始化，以保证它们具有正确的值，从而在调用lock和unlock时按需要工作。

对于 POSIX 线程，有两种初始化锁的方法。一种方法是使用 `PTHREAD_MUTEX_INITIALIZER`，如下所示：

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
```

这样做会将锁设置为默认值，从而使锁可用。动态方法（即在运行时）是调用 `pthread_mutex_init()`，如下所示：

```c
int rc = pthread_mutex_init(&amp;lock, NULL);
assert(rc == 0 &amp;&amp; “Error in mutex init”);
```

该例程的第一个参数是锁本身的地址，第二个参数是一组可选属性。传递 NULL 即只需使用默认值即可。两种方法都可以，但我们通常使用动态（后一种）方法。需要注意的是，在使用完锁后，还需要调用 `pthread_mutex_destroy()`。

上述代码的第二个问题是，它在调用lock和unlock时没有检查错误代码。就像你在 UNIX 系统中调用的几乎所有库例程一样，这些例程也可能失败！如果你的代码没有正确检查错误代码，失败就会无声无息地发生，在这种情况下，可能会允许多个线程进入临界区段。在最低限度上，应使用包装器来断言例程成功（如下面这段代码所示）；更复杂的（非玩具）程序在出错时不能简单地退出，而应检查失败，并在lock或unlock不成功时采取适当的措施。

```c
// Use this to keep your code clean but check for failures
// Only use if exiting program is OK upon failure
void Pthread_mutex_lock(pthread_mutex_t *mutex) {
    int rc = pthread_mutex_lock(mutex);
    assert(rc == 0 &amp;&amp; “Error in acquire”);
}
```

lock和unlock例程并不是 `pthreads` 库中与锁交互的唯一例程。这里还有两个例程可能值得关注：

```c
int pthread_mutex_trylock(pthread_mutex_t *mutex);
int pthread_mutex_timedlock(pthread_mutex_t *mutex,
                            struct timespec *abs_timeout);
```

这两个调用用于获取锁。如果锁已被持有，`trylock` 版本会返回失败；获取锁的 `timedlock` 版本会在超时或获取锁后返回，以先发生者为准。因此，超时后的 `timedlock` 会退化为 `trylock`。一般来说，这两种情况都应该避免；不过，在某些情况下，避免卡在（也许是无限期地）锁获取例程中是有用的。

## 条件变量

任何线程库的另一个主要组成部分，当然也包括 POSIX 线程，就是**条件变量**的存在。&lt;font color=&#34;red&#34;&gt;当线程之间必须进行某种信号传递时，如果一个线程正在等待另一个线程做某事，然后才能继续，那么条件变量就非常有用&lt;/font&gt;。希望以这种方式进行交互的程序主要使用两个例程：

```c
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
int pthread_cond_signal(pthread_cond_t *cond);
```

要使用条件变量，还必须拥有与该条件关联的锁。当调用上述任一例程时，应保持此锁。

第一个例程 `pthread_cond_wait()` 使调用线程进入睡眠状态，从而等待其他线程向其发出信号，通常是在程序中的某些内容发生更改而现在正在睡眠的线程可能关心的情况下。典型的用法如下：

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

Pthread_mutex_lock(&amp;lock);
while (ready == 0)
    Pthread_cond_wait(&amp;cond, &amp;lock);
Pthread_mutex_unlock(&amp;lock);
```

在此代码中，在初始化相关锁和条件之后，线程检查变量`ready`是否已设置为非零的值。如果没有，该线程只需调用等待例程即可休眠，直到其他线程将其唤醒。唤醒一个线程的代码如下所示，该代码将在其他线程中运行：

```c
Pthread_mutex_lock(&amp;lock);
ready = 1;
Pthread_cond_signal(&amp;cond);
Pthread_mutex_unlock(&amp;lock);
```

关于这个代码序列，有几点需要注意。首先，在发送信号时（以及修改全局变量 `ready` 时），我们始终要确保`lock`。这样可以确保我们的代码不会意外引入竞争条件。

其次，你可能会注意到，`wait` 调用的第二个参数是锁，而 `signal` 调用只需要一个条件。造成这种差异的原因是，wait 调用除了让调用线程休眠外，还会在让调用者休眠时释放锁。试想一下，如果不这样做，其他线程怎么可能获得锁并发出信号唤醒它呢？不过，在被唤醒后返回之前，`pthread_cond_wait()` 会重新获取锁，从而确保在等待序列开始时获取锁和结束时释放锁之间的任何时间，等待线程都持有锁。

最后一个奇怪的现象：等待线程在 while 循环中重新检查条件，而不是简单的 if 语句。因为使用 while 循环是简单安全的做法。虽然它会重新检查条件（可能会增加一点开销），但有些 pthread 实现可能会错误地唤醒等待线程；在这种情况下，如果不重新检查，等待线程就会继续认为条件改变，即使它并没有改变。例如，如果有多个线程在等待，而只有一个线程应该抓取数据（生产者-消费者）。因此，更安全的做法是将唤醒视为可能已发生变化的提示，而不是绝对的事实。

需要注意的是，有时在两个线程之间使用一个简单的标志来发出信号，而不是使用条件变量和相关的锁，这很有诱惑力。例如，我们可以重写上面的等待代码，在等待代码中看起来更像这样：

```c
while (ready == 0)
    ; // spin
```

相关的信号代码如下：

```c
ready = 1;
```

永远不要这样做，原因如下。首先，它在很多情况下表现不佳（长时间自旋，即持续检查某个条件是否满足，这只会浪费 CPU 周期）。其次，容易出错。使用标志（如上所述）在线程之间进行同步时非常容易出错。

## 线程API指南

当你使用POSIX线程库（或者实际上，任何线程库）构建一个多线程程序时，有一些小但重要的事情需要记住。它们包括：

- **保持简单。**最重要的是，任何涉及线程之间的锁定或信号的代码都应尽可能简单。复杂的线程交互会导致错误。
- **最小化线程交互。**尽量减少线程之间交互的方式。每个交互都应该经过深思熟虑，并用经过验证的方法构建。
- **初始化锁和条件变量。**未初始化将导致代码有时能够正常工作，有时会以非常奇怪的方式失败。
- **检查返回码。**当然，在你所做的任何C和UNIX编程中，你都应该检查每一个返回码，这在这里也是正确的。不这样做将导致奇怪且难以理解的行为。
- **在传递参数给线程和从线程返回值时要小心。**特别是，任何时候你传递指向栈上分配的变量的引用时，你可能在做一些错误的事情。
- **每个线程都有自己的栈。**与上面的观点相关，请记住每个线程都有自己的栈。因此，如果你在某个线程执行的函数中有一个在本地分配的变量，它基本上是私有的，其他线程无法（轻易）访问它。要在线程之间共享数据，这些值必须在堆上或者以其他全局可访问的位置。
- **总是使用条件变量来在线程之间进行信号传递。**虽然使用简单的标志往往很诱人，但不要这样做。
- **使用手册页面。**特别是在Linux上，pthread手册页面非常有信息量。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/20.%E7%BA%BF%E7%A8%8Bapi/  

