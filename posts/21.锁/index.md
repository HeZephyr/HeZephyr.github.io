# 锁

## 锁：基本思想

举个例子，假设我们的临界区如下所示，共享变量的规范更新：

```c
balance = balance &#43; 1;
```

当然，其他临界区也是可能的，例如向链表添加元素或对共享结构进行其他更复杂的更新，但我们现在只保留这个简单的示例。要使用锁，我们在临界区周围添加一些代码，如下所示：

```c
lock_t mutex; // some globally-allocated lock ’mutex’
...
lock(&amp;mutex);
balance = balance &#43; 1;
unlock(&amp;mutex);
```

锁只是一个变量，因此要使用它，您必须声明某种类型的**锁变量**（例如上面的`mutex`）。该锁变量（或简称“锁”）保存任意时刻锁的状态。它要么**可用**（或**未锁定**或**空闲**），因此没有线程持有该锁，要么**已获取**（或**锁定**或**持有**），因此恰好有一个线程持有该锁，并且可能位于临界区中。我们还可以在数据类型中存储其他信息，例如哪个线程持有锁，或者用于获取锁的顺序队列，但此类信息对锁的用户是隐藏的。 

`lock()` 和`unlock()` 例程的语义很简单。调用例程 `lock()` 尝试获取锁；如果没有其他线程持有该锁（即它是空闲的），则该线程将获取该锁并进入临界区；该线程有时被称为锁的**所有者**。如果另一个线程随后对同一个锁变量（本例中为`mutex`）调用 `lock()`，则当锁被另一个线程持有时，它不会返回；这样，当第一个持有锁的线程位于临界区时，其他线程就无法进入临界区。

一旦锁的所有者调用`unlock()`，锁就再次可用（空闲）。如果没有其他线程正在等待锁（即没有其他线程调用`lock()` 并被卡在其中），则锁的状态将简单地更改为空闲。如果有等待线程（卡在 lock() 中），其中一个线程将（最终）注意到（或被告知）锁状态的这一变化，获取锁，并进入临界区。

锁为程序员提供了对调度的最小程度的控制。一般来说，我们将线程视为由程序员创建但由操作系统以操作系统选择的任何方式调度的实体。锁将部分控制权交还给程序员；通过在一段代码周围放置一个锁，程序员可以保证该代码中最多只有一个线程处于活动状态。因此，锁有助于将传统操作系统调度的混乱转变为更受控制的活动。

## Pthread 锁

POSIX 库为锁使用的名称是 `mutex`，因为它用于提供线程之间的**互斥**，也就是说，如果一个线程处于临界区，它将禁止其他线程进入，直到它完成该临界区。因此，当你看到下面的 POSIX 线程代码时，你应该明白它在做与上面相同的事情（我们再次使用我们的封装器，在锁定和解锁时检查错误）：

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

Pthread_mutex_lock(&amp;lock); // wrapper; exits on failure
balance = balance &#43; 1;
Pthread_mutex_unlock(&amp;lock);
```

你可能还会注意到，POSIX 版本会通过传递一个变量来加锁和解锁，因为我们可能会使用不同的锁来保护不同的变量。这样做可以提高并发性：我们通常会使用不同的锁来保护不同的数据和数据结构，而不是在访问任何临界区时使用一个大锁（**粗粒度锁定策略**），从而允许更多线程同时进入锁定代码（**更细粒度的方法**）。

## 评估锁—基本标准

在构建任何锁之前，我们首先应该了解我们的目标是什么，因此我们会问如何评估特定锁实现的有效性。要评估一个锁是否有效（而且效果很好），我们首先应该建立一些如下的基本标准。

1. 首先是锁是否完成了它的基本任务，即&lt;font color=&#34;red&#34;&gt;提供互斥&lt;/font&gt;。基本上，锁是否能阻止多个线程进入临界区？

2. 其次是&lt;font color=&#34;red&#34;&gt;公平性&lt;/font&gt;。一旦锁被释放，每个争夺锁的线程是否都能公平地获得锁？另一种方法是考察更极端的情况：是否有任何线程在争夺锁的过程中陷入饥饿，从而永远无法获得锁？

3. 最后一个标准是&lt;font color=&#34;red&#34;&gt;性能&lt;/font&gt;，特别是使用锁所增加的时间开销。这里有几种不同的情况值得考虑。

	* 一种是无竞争的情况；当单线程运行并抓取和释放锁时，这样做的开销是多少？
	* 另一种情况是多个线程在单个 CPU 上争夺锁；在这种情况下，是否存在性能问题？
	* 最后，当涉及多个 CPU 且每个 CPU 上的线程都在争夺锁时，锁的性能如何？

	通过比较这些不同的情况，我们可以更好地了解使用各种锁技术对性能的影响。

## 控制中断

最早用于提供互斥的解决方案之一是禁用临界区的中断；&lt;font color=&#34;red&#34;&gt;这种解决方案是为单处理器系统发明的&lt;/font&gt;。代码如下

```c
void lock() {
    DisableInterrupts();
}
void unlock() {
    EnableInterrupts();
}
```

假设我们正在这样的单处理器系统上运行。通过在进入临界区之前&lt;font color=&#34;red&#34;&gt;关闭中断&lt;/font&gt;（使用某种特殊的硬件指令），我们可以确保临界区内的代码不会被中断，从而像原子一样执行。当我们完成后，我们&lt;font color=&#34;red&#34;&gt;重新启用中断&lt;/font&gt;（再次通过硬件指令），因此程序照常进行。

这种方法的主要优点是它的简单性。当然，您不必绞尽脑汁就能弄清楚为什么会这样。在没有中断的情况下，线程可以确保它执行的代码将会执行，并且没有其他线程会干扰它。

不幸的是，负面因素有很多。首先，这种方法要求我们允许任何调用线程执行特权操作（打开和关闭中断），因此相信该设施不会被滥用。正如您所知，每当我们需要信任任意程序时，我们都可能遇到麻烦。在这里，问题以多种方式表现出来：贪婪的程序可以在执行开始时调用`lock()`，从而独占处理器；更糟糕的是，错误或恶意程序可能调用 `lock()` 并进入无限循环。在后一种情况下，操作系统永远不会重新获得对系统的控制，只有一个办法：重新启动系统。使用中断禁用作为通用同步解决方案需要对应用程序过多的信任。

其次，该方法不适用于多处理器。如果多个线程运行在不同的CPU上，并且每个线程都尝试进入同一个临界区，那么是否禁用中断并不重要；线程将能够在其他处理器上运行，因此可以进入临界区。由于多处理器现在很常见，我们的通用解决方案必须比这做得更好。

第三，长时间关闭中断可能会导致中断丢失，从而导致严重的系统问题。例如，想象一下，如果 CPU 错过了磁盘设备已完成读取请求的事实。操作系统如何知道唤醒等待所述读取的进程？

最后，也许也是最不重要的一点是，这种方法可能效率低下。与正常指令执行相比，现代 CPU 执行屏蔽或取消屏蔽中断的代码往往执行速度较慢。

由于这些原因，关闭中断仅在有限的上下文中用作互斥原语。例如，在某些情况下，操作系统本身将使用中断屏蔽来保证访问其自己的数据结构时的原子性，或者至少防止出现某些混乱的中断处理情况。这种用法是有道理的，因为信任问题在操作系统内部消失了，操作系统始终相信自己能够执行特权操作。

## 失败的尝试：仅使用加载/存储

要想超越基于中断的技术，我们就必须依靠 CPU 硬件及其提供的指令来构建适当的锁。让我们首先尝试使用一个标志变量来构建一个简单的锁。在这次失败的尝试中，我们将看到构建锁所需的一些基本思想，并明白为什么仅仅使用单个变量并通过正常的加载和存储来访问它是不够的。

第一次尝试的代码如下：

```c
typedef struct __lock_t { int flag; } lock_t;

void init(lock_t *mutex) {
    // 0 -&gt; lock is available, 1 -&gt; held
    mutex-&gt;flag = 0;
}

void lock(lock_t *mutex) {
    while (mutex-&gt;flag == 1) // TEST the flag
        ; // spin-wait (do nothing)
    mutex-&gt;flag = 1; // now SET it!
}

void unlock(lock_t *mutex) {
    mutex-&gt;flag = 0;
}
```

在第一次尝试中，我们的想法非常简单：使用一个简单的变量（`flag`）来指示某个线程是否拥有锁。进入临界区的第一个线程将调用 `lock()`，它将测试`flag`是否等于 1（在本例中不等于 1），然后将`flag`设置为 1，表示该线程现在持有锁。完成临界区后，线程会调用 `unlock()` 并清除`flag`，从而表明不再持有锁。

如果另一个线程在第一个线程处于临界区时调用了 `lock()`，那么它只需在 while 循环中自旋等待该线程调用 `unlock()`并清除`flag`。一旦第一个线程这样做了，等待的线程就会退出 `while` 循环，将自己的`flag`设置为 1，然后进入临界区。

不幸的是，该代码有两个问题：一个是正确性，另一个是性能。一旦您习惯于思考并发编程，正确性问题就很容易看出。想象一下下面的的代码交错；假设 `flag=0` 开始。

![image-20240407154637634](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20240407154637634.png)

正如您从这种交错中看到的，通过及时（不及时？）中断，我们可以很容易地产生这样一种情况：两个线程都将`flag`设置为 1，因此两个线程都能进入临界区。我们显然没有满足最基本的要求**：提供互斥**。

我们稍后将详细讨论的性能问题是线程等待获取已持有的锁的方式：它无休止地检查`flag`的值，这是一种称为&lt;font color=&#34;red&#34;&gt;自旋等待&lt;/font&gt;的技术。自旋等待会浪费时间等待另一个线程释放锁。在单处理器上浪费非常高，其中等待者正在等待的线程甚至无法运行（至少在发生上下文切换之前）！

## 用Test-And-Set构建工作自旋锁

由于禁用中断在多处理器上不起作用，而且使用加载和存储（如上所示）的简单方法也不起作用，系统设计人员开始发明硬件支持锁定。最早的多处理器系统，如 20 世纪 60 年代初的 Burroughs B5000，就提供了这种支持；如今，所有系统都提供了这种支持，即使是单 CPU 系统也不例外。

最简单易懂的硬件支持被称为`test-and-set`（或&lt;font color=&#34;red&#34;&gt;原子交换&lt;/font&gt;）指令。我们通过下面的 C 代码片段来定义 `test-and-set` 指令的作用：

```c
int TestAndSet(int *old_ptr, int new) {
    int old = *old_ptr; // fetch old value at old_ptr
    *old_ptr = new; // store ’new’ into old_ptr
    return old; // return the old value
}
```

`test-and-set`指令的作用如下。它返回 `ptr` 指向的`old`，同时将所述值更新为`new`。当然，关键在于这一系列操作都是以原子方式执行的。之所以称为 &#34;test-and-set&#34;，是因为它可以 &#34;test &#34;旧值（即返回值），同时将内存位置 &#34;设置 &#34;为新值；事实证明，这条功能稍强的指令足以构建一个简单的自旋锁，如下面这段代码所示。

```c
typedef struct __lock_t {
    int flag;
} lock_t;

void init(lock_t *lock) {
    // 0: lock is available, 1: lock is held
    lock-&gt;flag = 0;
}

void lock(lock_t *lock) {
    while (TestAndSet(&amp;lock-&gt;flag, 1) == 1)
        ; // spin-wait (do nothing)
}

void unlock(lock_t *lock) {
    lock-&gt;flag = 0;
}
```

让我们先了解一下这种锁的工作原理。首先设想这样一种情况：线程调用 `lock()` 而当前没有其他线程持有锁；因此，`flag` 应为 0。当线程调用 `TestAndSet(flag, 1)` 时，例程将返回 `flag` 的旧值，即 0；因此，正在测试 `flag` 值的调用线程不会陷入 while 循环的自旋，并将获得锁。线程也会原子地将该值设置为 1，从而表明锁已被锁定。当线程完成其临界区后，会调用 unlock() 将`flag`置回 0。

我们可以想象的第二种情况是，一个线程已经持有锁（即`flag`为 1）。在这种情况下，该线程将调用 `lock()` 并调用 `TestAndSet(flag,1)`。这一次，`TestAndSet()` 将返回 flag 的旧值，即 1（因为锁已被持有），同时再次将其设置为 1。只要锁被另一个线程持有，`TestAndSet()` 就会反复返回 1，因此这个线程会自旋，直到锁最终被释放。当`flag`最终被其他线程设置为 0 时，该线程将再次调用 `TestAndSet()`，此时它将返回 0，同时原子地将该值设置为 1，从而获得锁并进入临界区。

通过将`test`（旧锁值）和 `set`（新锁值）作为单个原子操作，我们可以确保只有一个线程获得锁。这就是构建互斥原语的方法！

现在你可能也明白为什么这种锁通常被称为&lt;font color=&#34;red&#34;&gt;自旋锁&lt;/font&gt;了。它是最简单的一种锁，只需使用 CPU 周期旋 转，直到锁可用为止。要在单处理器上正常工作，它需要一个&lt;font color=&#34;red&#34;&gt;抢占式调度程序（即通过定时器中断线程，以便不时运行不同的线程）&lt;/font&gt;。如果没有抢占式调度，自旋锁在单 CPU 上的意义就不大，因为在 CPU 上旋转的线程永远不会放弃自旋锁。

## 评估自旋锁

有了基本的自旋锁，我们现在就可以根据之前描述的标准来评估它的有效性。锁最重要的方面是&lt;font color=&#34;red&#34;&gt;正确性&lt;/font&gt;：它能提供互斥吗？答案是肯定的：自旋锁每次只允许一个线程进入临界区。因此，我们拥有一个正确的锁。

下一个标准是&lt;font color=&#34;red&#34;&gt;公平性&lt;/font&gt;。自旋锁对等待线程的公平性如何？你能保证等待线程永远不会进入临界区吗？不幸的是，自旋锁不提供任何公平性保证。事实上，一个正在自旋的线程可能会在竞争中永远自旋下去。简单的自旋锁（如前面所讨论的）是不公平的，可能会导致饥饿。

最后一个标准是&lt;font color=&#34;red&#34;&gt;性能&lt;/font&gt;。使用自旋锁的代价是什么？为了更仔细地分析这个问题，我们建议考虑几种不同的情况。

* 第一种情况是线程在单个处理器上竞争锁；

	对于自旋锁，在单 CPU 的情况下，性能开销可能会相当大；想象一下持有锁的线程在临界区被抢占的情况。然后，调度程序可能会运行每个其他线程（假设有 N - 1 个其他线程），每个线程都试图获取锁。在这种情况下，每个线程在放弃 CPU 之前都会持续运行一个时间片，浪费了 CPU 周期。

* 第二种情况是线程分布在多个 CPU 上。

	然而，在多个 CPU 上，自旋锁工作得相当好（如果线程数大致等于 CPU 数）。思路如下：想象 CPU 1 上的线程 A 和 CPU 2 上的线程 B，两者都竞争锁。如果线程 A (CPU 1) 获取锁，然后线程 B 尝试获取锁，则 B 将自旋（在 CPU 2 上）。然而，大概临界区很短，因此锁很快就变得可用，并被线程 B 获取。在这种情况下，自旋等待另一个处理器上持有的锁不会浪费很多周期，因此可能是有效的。

## Compare-And-Swap

某些系统提供的另一种硬件原语被称为 **&#34;compare-and-swap &#34;**指令（例如 SPARC 上的名称）或 **&#34;compare-and-exchange &#34;**指令（x86 上的名称）。下面是这条指令的 C 语言伪代码。

```c
int CompareAndSwap(int *ptr, int expected, int new) {
    int actual = *ptr;
    if (actual == expected)
        *ptr = new;
    return actual;
}
```

其基本思想是，`compare-and-swap`指令测试 `ptr` 指定地址上的值是否等于预期值；如果是，则用新值更新 `ptr` 指向的内存位置。如果不相等，则什么也不做。无论哪种情况，都会返回该内存位置的实际值，从而让调用`compare-and-swap`指令的代码知道它是否成功。

有了`compare-and-swap`指令，我们就可以用与`test-and-set`指令类似的方式建立锁。例如，我们可以将上面的 `lock()` 例程替换为下面的例程：

```c
void lock(lock_t *lock) {
    while (CompareAndSwap(&amp;lock-&gt;flag, 0, 1) == 1)
        ; // spin
}
```

代码的其余部分与上面的`test-and-set`示例相同。这段代码的工作原理非常相似；它只需检查`flag`是否为 0，如果为 0，则原子交换 1，从而获取锁。如果线程试图在锁被锁定时获取锁，就会被卡住，直到锁最终被释放。

下面是实际 C 代码（调用汇编）x86版本中显示的`compare-and-swap`的简单示例。

```c
#include &lt;stdio.h&gt;

int global = 0;

// Function to perform a compare and swap operation
// Parameters:
//   ptr: pointer to the memory location where the operation will be performed
//   old: expected value to compare against
//   new: new value to store if the comparison succeeds
// Returns:
//   1 if the comparison succeeded and the value was updated, 0 otherwise
char compare_and_swap(int *ptr, int old, int new) {
    unsigned char ret;
    // Assembly code to perform the compare and swap operation
    // Note: sete sets a byte not the word
    __asm__ __volatile__ (
        &#34; lock\n&#34;
        &#34; cmpxchgl %2,%1\n&#34;
        &#34; sete %0\n&#34;
        : &#34;=q&#34; (ret), &#34;=m&#34; (*ptr)
        : &#34;r&#34; (new), &#34;m&#34; (*ptr), &#34;a&#34; (old)
        : &#34;memory&#34;);
    return ret;
}

int main(int argc, char *argv[]) {
    // Before successful compare and swap
    printf(&#34;before successful cas: %d\n&#34;, global);
    // Perform successful compare and swap operation
    int success = compare_and_swap(&amp;global, 0, 100);
    // Print result after successful compare and swap
    printf(&#34;after successful cas: %d (success: %d)\n&#34;, global, success);
    
    // Before failing compare and swap
    printf(&#34;before failing cas: %d\n&#34;, global);
    // Perform failing compare and swap operation
    success = compare_and_swap(&amp;global, 0, 200);
    // Print result after failing compare and swap
    printf(&#34;after failing cas: %d (old: %d)\n&#34;, global, success);

    return 0;
}
```

 最后，正如你可能已经感觉到的，`compare-and-swap`是一条比`test-and-set`功能更强大的指令。今后，当我们深入探讨&lt;font color=&#34;red&#34;&gt;无锁同步&lt;/font&gt;等主题时，我们将利用这一功能。不过，如果我们只是用它构建一个简单的自旋锁，其行为与我们上面分析的自旋锁完全相同。

## Load-Linked and Store-Conditional

一些平台提供了一对协同工作的指令来帮助构建临界区。例如，在 MIPS 架构上，**加载链接指令**和**存储条件指令**可以串联使用来构建锁和其他并发结构。这些指令的 C 伪代码如下所示。 Alpha、PowerPC 和 ARM 提供类似的指令。

```c
int LoadLinked(int *ptr) {
    return *ptr;
}

int StoreConditional(int *ptr, int value) {
    if (no update to *ptr since LoadLinked to this address) {
        *ptr = value;
        return 1; // success!
    } else {
        return 0; // failed to update
    }
}
```

加载链接的操作与典型的加载指令非常相似，只是从内存中获取一个值并将其放入寄存器中。关键的区别在于条件存储，只有在没有对地址进行干预存储的情况下，条件存储才会成功（并更新存储在刚刚加载链接的地址处的值）。如果成功，`store-conditional` 返回 1 并将 ptr 处的值更新为 value；如果失败，则 ptr 处的值不会更新并返回 0。

下面这段代码展示了如何使用加载链接和存储条件来构建自旋锁。

```c
void lock(lock_t *lock) {
    while (1) {
        while (LoadLinked(&amp;lock-&gt;flag) == 1)
            ; // spin until it’s zero
        if (StoreConditional(&amp;lock-&gt;flag, 1) == 1)
            return; // if set-it-to-1 was a success: all done
        // otherwise: try it all over again
    }
}

void unlock(lock_t *lock) {
    lock-&gt;flag = 0;
}
```

`lock()` 代码是唯一有趣的部分。首先，线程自旋，等待`flag`设置为 0（从而指示未持有锁）。一旦这样，线程尝试通过`store-conditional`获取锁；如果成功，线程会自动将`flag`的值更改为 1，从而可以进入临界区。

注意存储条件失败可能出现的情况。一个线程调用`lock()`并执行`load-linked`，返回0表示锁未被持有。在它尝试`store-conditional`之前，它被中断，另一个线程进入锁代码，同样执行`load-linked`指令，并得到0继续执行。此时，两个线程各自执行了`load-linked`，并且即将尝试`store-conditional`。这些指令的关键特点是这两个线程中只有一个会成功地将`flag`更新为1从而获取锁；第二个尝试`store-conditional`的线程会失败（因为另一个线程在其`load-linked`和`store-conditional`之间更新了flag值），因此必须再次尝试获取锁。

一种更简洁的`lock()`写法如下：

```c
void lock(lock_t *lock) {
    while (1) {
        while (LoadLinked(&amp;lock-&gt;flag) == 1 || !StoreConditional(&amp;lock-&gt;flag, 1))
            ; // spin until the condition does not hold
    }
}
```

## Fetch-And-Add

最后一个硬件原语是`fetch-and-add`指令，它以原子方式递增一个值，同时返回特定地址上的旧值。`fetch-and-add`指令的 C 语言伪代码如下：

```c
int FetchAndAdd(int *ptr) {
    int old = *ptr;
    *ptr = old &#43; 1;
    return old;
}
```

`lock`和`unlock`代码如下所示。在这个例子中，我们将使用fetch-and-add来构建一个更有趣的票据锁，由Mellor-Crummey和Scott引入。  

```c
typedef struct __lock_t {
    int ticket;
    int turn;
} lock_t;

void lock_init(lock_t *lock) {
    lock-&gt;ticket = 0;
    lock-&gt;turn = 0;
}

void lock(lock_t *lock) {
    int myturn = FetchAndAdd(&amp;lock-&gt;ticket);
    while (lock-&gt;turn != myturn)
        ; // spin
}

void unlock(lock_t *lock) {
    lock-&gt;turn = lock-&gt;turn &#43; 1;
}
```

这个解决方案不是使用单一值，而是结合使用`ticket`和`turn`变量来构建一个自旋锁。 基本操作非常简单：当线程希望获取锁时，首先对`ticket`进行原子`fetch-and-add`操作；该值现在被视为此线程的“轮次”（`myturn`）。 然后全局共享的`lock-&gt;turn`用于确定哪个线程的轮次；当给定线程的(`myturn == turn`)时，就是该线程进入临界区的机会。 `unlock`只需通过增加`turn`即可完成，以便下一个等待的线程（如果有）现在可以进入临界区。 

请注意这种解决方案与我们之前尝试过的方法之间存在一个重要差异：它确保所有线程都能取得进展。 一旦分配了某个线程自己的`ticket`，它将在未来某个时间点被调度执行（一旦排队等候其前面已经通过临界区并释放了锁）。 在我们之前尝试过的情况下，并不存在这样保证；例如，在`test-and-set`上自旋着等待，即使其他线程获取并释放了锁也可能永远自旋下去。

## 太多自旋：现在怎么办？

我们基于硬件的简单锁非常简单（只有几行代码），而且能正常工作，这是任何系统或代码的两个优秀特性。然而，在某些情况下，这些解决方案的效率可能很低。想象一下，你在单个处理器上运行两个线程。现在假设一个线程（线程 0）正处于临界区，因此被锁定，并不幸被中断。第二个线程（线程 1）现在试图获取锁，但发现锁已被锁定。于是，它开始不停地自旋。然后又自旋了几圈。最后，定时器中断，线程 0 再次运行，释放了锁，最后（比如说下次运行时），线程 1 就不用自旋那么多圈了，就能获得锁。因此，在这种情况下，线程一旦陷入自旋，就会浪费一整个时间片，而只是检查一个不会改变的值外，什么也不做！如果有 N 个线程在争夺一个锁，问题就更严重了；N - 1 个时间片可能会以类似的方式被浪费掉，仅仅是在自旋并等待一个线程释放锁。这就是我们的下一个问题：如何避免自旋？我们怎样才能开发出一种不会无谓地浪费时间在 CPU 上自旋的锁？仅靠硬件支持无法解决问题。我们还需要操作系统的支持！现在就让我们来看看如何实现这一点。

## 一个简单的方法：只需yield

硬件支持让我们取得了相当大的进展：工作锁，甚至（如票据锁）锁获取的公平性。然而，我们仍然面临一个问题：当上下文切换发生在临界区时，线程开始无休止地自旋，等待被中断的（持有锁的）线程再次运行，这时该怎么办？

我们首先尝试的是一种简单而友好的方法：当你要自旋时，把 CPU 让给另一个线程。或者，就像Al Davis说的那样，&#34;让一让，宝贝！&#34;。下面这段代码展示了这种方法。

```c
void init() {
    flag = 0;
}

void lock() {
    while (TestAndSet(&amp;flag, 1) == 1)
        yield(); // 放弃 CPU 控制权
}

void unlock() {
    flag = 0;
}
```

在这种方法中，我们假定一个线程在想要放弃 CPU 并让另一个线程运行时，可以调用操作系统原语 `yield()`。一个线程可以处于三种状态（运行、就绪或阻塞）之一；`yield` 只是一个系统调用，它将调用者从**运行状态**移到**就绪状态**，从而将另一个线程提升到**运行状态**。因此，`yield`过程本质上是自我取消调度的。

回想一下一个 CPU 上有两个线程的例子，在这种情况下，我们基于 `yield` 的方法就能很好地发挥作用。如果一个线程碰巧调用了 `lock()` 并发现锁被锁定，它就会简单地让出 CPU，这样另一个线程就会运行并完成其临界区。在这种简单的情况下，`yield`方法运行良好。

现在让我们考虑一下有多个线程（比如 100 个）重复争夺一个锁的情况。在这种情况下，如果一个线程获得了锁，并在释放锁之前被抢占，那么其他 99 个线程将分别调用 `lock()`，发现锁被持有，并让出 CPU。假设有某种循环调度程序，那么 99 个线程中的每一个都会在持有锁的线程再次运行之前执行这种`run-and-yield`模式。虽然这种方法比我们的 &#34;自旋&#34;方法要好（&#34;自旋 &#34;会浪费 99 个时间片），但上下文切换的成本可能很高，因此会造成大量浪费。

更糟糕的是，我们根本没有解决饥饿问题。当其他线程反复进入和退出临界区时，一个线程可能会陷入无休止的`yield`循环。显然，我们需要一种能直接解决这一问题的方法。

## 使用队列：睡眠而不是自旋

我们之前的方法的真正问题在于，它们留下了太多的偶然性。调度器决定下一个运行的线程；如果调度器做出了错误的选择，运行的线程要么必须自旋等待锁（我们的第一种方法），要么立即让出 CPU（我们的第二种方法）。无论哪种方法，都有可能造成浪费，而且无法避免饥饿。

因此，我们必须明确控制在当前持有者释放锁后，下一个获得锁的线程。为此，我们需要更多的操作系统支持，以及一个队列来跟踪哪些线程正在等待获取锁。

为简单起见，我们将使用 Solaris 提供的支持，即两个调用：`park()` 用于使调用线程休眠，`unpark(threadID)` 用于唤醒`threadID` 指定的特定线程。这两个例程可以配合使用，构建一个锁，当调用者试图获取一个被锁定的锁时，该锁会使调用者休眠，而当锁被释放时，调用者会被唤醒。

下面这段代码实现了简单的锁。

```c
typedef struct __lock_t {
    int flag;
    int guard;
    queue_t *q;
} lock_t;

void lock_init(lock_t *m) {
    m-&gt;flag = 0;
    m-&gt;guard = 0;
    queue_init(m-&gt;q);
}

void lock(lock_t *m) {
    while (TestAndSet(&amp;m-&gt;guard, 1) == 1)
        ; // 通过自旋获取 guard 锁
    if (m-&gt;flag == 0) {
        m-&gt;flag = 1; // 获取锁
        m-&gt;guard = 0;
    } else {
        queue_add(m-&gt;q, gettid());
        m-&gt;guard = 0;
        park();
    }
}

void unlock(lock_t *m) {
    while (TestAndSet(&amp;m-&gt;guard, 1) == 1)
        ; // 通过自旋获取 guard 锁
    if (queue_empty(m-&gt;q))
        m-&gt;flag = 0; // 释放锁；没有线程要获取它
    else
        unpark(queue_remove(m-&gt;q)); // 保持锁（给下一个线程！）
    m-&gt;guard = 0;
}
```

在这个例子中，我们做了几件有趣的事情。首先，我们将老式的`test-and-set`思想与显式的锁等待者队列相结合，以实现更高效的锁。其次，我们使用队列来帮助控制下一个获得锁的人，从而避免饥饿。

你可能会注意到 `guard` 是如何使用的，它基本上是围绕锁使用的标志和队列操作的自旋锁。因此，这种方法并不能完全避免自旋等待；线程在获取或释放锁时可能会被中断，从而导致其他线程自旋等待该线程再次运行。不过，自旋所花费的时间非常有限（只是`lock()`和`unlock()`代码中的几条指令，而不是用户定义的临界区），因此这种方法可能是合理的。

其次，你可能会注意到，在 `lock()` 中，当一个线程无法获取锁（它已被持有）时，我们会小心地将该线程加入一个队列（通过调用 `gettid()` 函数获取当前线程的`threadID`），将 `guard` 设为 0，并让出 CPU。但如果在 `park()` 之后而不是之前释放`guard`锁，会发生什么情况？这会导致其他线程暂时无法再获得 `guard` 锁，因为该锁仍然由先前的线程持有。如果其他线程试图获取锁，则会一直自旋等待 `guard` 锁被释放。而由于无法获取`guard`锁，其他线程无法执行 `unlock()` 函数，所以即使某个线程被唤醒，也无法继续执行`unlock`操作。这就导致了死锁状态，所有线程都被阻塞，无法正常运行。

你可能还会注意到一个有趣的事实，那就是当另一个线程被唤醒时，`flag`不会被设置回 0。这是为什么呢？这并不是一个错误，而是一种必然！当一个线程被唤醒时，它就像从 `park()` 返回一样；然而，它在代码中的这一点上并不持有`guard`，因此甚至无法尝试将`flag`设置为 1。因此，我们只是将锁从释放锁的线程直接传递给下一个获取锁的线程；在这中间，`flag`不会被设置为 0。

最后，你可能会注意到在解决方案中，就在调用 `park()` 之前出现了竞争条件。例如线程B在调用`park()`之前线程A释放了锁，那么线程B可能会永远休眠，因为在调用 `park()` 之前它已经错过了唤醒的时机。这个问题有时被称为&lt;font colo=&#34;red&#34;&gt;唤醒/等待竞争&lt;/font&gt;。

Solaris 通过添加第三个系统调用 `setpark()` 来解决这个问题。通过调用该例程，线程可以表明它即将`park`。如果线程在调用 `park()` 之前被中断（例如收到了一个信号），而另一个线程在这个时候调用了 `unpark()` 来唤醒当前线程，则当前线程下一次调用 `park()` 不会真正进入休眠状态，而是立即返回，继续执行后续的代码。`lock()`内部的代码修改很小：

```c
queue_add(m-&gt;q, gettid());
setpark(); // new code , (before open guard)
m-&gt;guard = 0;
```

另一种不同的解决方案是将`guard`传递给内核。在这种情况下，内核可以采取预防措施，以原子方式释放锁，并将运行中的线程出列。

## 不同的操作系统，不同的支持

到目前为止，我们已经看到了操作系统为在线程库中建立更高效的锁而提供的一种支持。其他操作系统也提供类似的支持，但细节各有不同。

例如，Linux 提供的 **futex** 与 Solaris 接口类似，但提供了更多内核功能。具体来说，每个 futex 都关联了一个特定的物理内存位置，以及每个 futex 的内核队列。调用者可以根据需要使用 futex 调用来休眠和唤醒。具体来说，

有两种调用方式可用。调用 `futex_wait(address,expected)`后，调用线程将进入休眠状态，前提是地址中的值等于预期值。如果不相等，调用将立即返回。调用例程 `futex_wake(address)` 会唤醒一个正在队列中等待的线程。这些调用在 Linux `mutex`中的用法如下所示。

```c
void mutex_lock(int *mutex) {
    int v;
    /* Bit 31 was clear, we got the mutex (the fastpath) */
    if (atomic_bit_test_set(mutex, 31) == 0)
        return;

    // Atomic operations increase the number of waiters on the lock
    atomic_increment(mutex);

    while (1) {
        // Spin waits for other threads to release the lock
        if (atomic_bit_test_set(mutex, 31) == 0) {
            // The current thread holds the lock and the number of waiters is reduced by 1
            atomic_decrement(mutex);
            return;
        }

        /* We have to wait. First make sure the futex value
           we are monitoring is truly negative (locked). */
        v = *mutex;
        if (v &gt;= 0)
            continue;
        futex_wait(mutex, v);
    }
}

void mutex_unlock(int *mutex) {
    /* Adding 0x80000000 to counter results in 0 if and
       only if there are not other interested threads */
    if (atomic_add_zero(mutex, 0x80000000))
        return;

    /* There are other threads waiting for this mutex,
       wake one of them up. */
    futex_wake(mutex);
}
```

nptl 库（gnu libc 库的一部分） 中 lowlevellock.h的这段 代码片段之所以有趣，有几个原因。首先，它使用一个整数来跟踪锁是否被持有（整数的高位）以及锁上等待者的数量（所有其他位）。因此，如果锁是负数，它就是被持有的（因为高位被设置，而该位决定了整数的符号）。

其次，代码片段展示了如何针对常见情况进行优化，特别是在没有锁竞争的情况下；只有一个线程获取和释放锁，只需完成很少的工作（原子位`test-and-set`锁定以及原子位添加释放锁）。

## 两阶段锁

最后一点：Linux 方法有一种旧方法的味道，这种方法已经断断续续地使用了很多年，至少可以追溯到 1960 年代初的 Dahm Locks ，现在被称为两阶段锁。两阶段锁意识到自旋可能很有用，特别是在锁即将被释放的情况下。所以在第一阶段，锁会自旋一段时间，希望能够获取到锁。

但是，如果在第一个自旋阶段没有获取锁，则会进入第二个阶段，调用者将进入睡眠状态，只有在锁稍后释放时才会被唤醒。上面的 Linux 锁就是这种锁的一种形式，但它只自旋一次；对此的概括可以是在使用 futex 支持进入睡眠之前循环自旋固定的时间。两阶段锁是混合方法的另一个实例，其中结合两个好的想法确实可能会产生更好的想法。当然，它是否确实在很大程度上取决于很多因素，包括硬件环境、线程数量和其他工作负载细节。与往常一样，制作一个适合所有可能用例的通用锁是一个相当大的挑战。

## Dekker算法和Peterson算法

20 世纪 60 年代，Dijkstra 向他的朋友们提出了并发问题，其中一位名叫 Theodorus Jozef Dekker 的数学家提出了解决方案。与我们在此讨论的使用特殊硬件指令甚至操作系统支持的解决方案不同，Dekker 的算法仅使用加载和存储（假定它们之间是原子关系，这在早期的硬件上是正确的）。下面是Dekker算法的代码：

```c
int flag[2];
int turn;

void init() {
    flag[0] = flag[1] = 0;
    turn = 0;
}

void lock() {
    flag[self] = 1;
    while (flag[1 - self] == 1) {
        if (turn != self) {
            flag[self] = 0;
            while (turn != self)
                ;
            flag[self] = 1;
        }
    }
}

void unlock() {
    flag[self] = 0;
    turn = 1 - self;
}
```

Dekker的方法后来被Peterson改进。同样，该算法只使用加载和存储，其目的是确保两个线程不会同时进入临界区。下面是 Peterson 的算法（两个线程）：

```c
int flag[2];
int turn;

void init() {
    // indicate you intend to hold the lock w/ ’flag’
    flag[0] = flag[1] = 0;
    // whose turn is it? (thread 0 or 1)
    turn = 0;
}

void lock() {
    // ’self’ is the thread ID of caller
    flag[self] = 1;
    // make it other thread’s turn
    turn = 1 - self;
    while ((flag[1 - self] == 1) &amp;&amp; (turn == 1 - self))
        ; // spin-wait while it’s not your turn
}

void unlock() {
    // simply undo your intent
    flag[self] = 0;
}

```

出于某种原因，开发无需特殊硬件支持就能工作的锁曾风靡一时，给理论家们带来了许多难题。当然，当人们意识到假定有一点硬件支持会容易得多时，这一行就变得毫无用处了（事实上，这种支持在多进程的早期就已经存在了）。此外，类似上述的算法在现代硬件上也行不通（因为内存一致性模型被放宽了），因此它们的用处比以前更小了。然而，更多的研究已被历史尘封......

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/21.%E9%94%81/  

