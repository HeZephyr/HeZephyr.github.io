# 并发bug

## 存在哪些类型的并发bug？

第一个也是最明显的问题是：复杂的并发程序中会出现哪些类型的并发bug？这个问题一般来说很难回答，但幸运的是，其他一些人已经为我们完成了这项工作。具体来说，我们依赖于 Lu 等人的一项研究。它详细分析了许多流行的并发应用程序，以了解实践中出现的bug类型。

该研究重点关注四个主要且重要的开源应用程序：MySQL（流行的数据库管理系统）、Apache（著名的 Web 服务器）、Mozilla（著名的 Web 浏览器）和 OpenOffice（MS Office 套件的免费版本，有些人实际使用）。在这项研究中，作者研究了在每个代码库中发现并修复的并发性bug，将开发人员的工作转化为定量bug分析；了解这些结果可以帮助您了解成熟代码库中实际发生的问题类型。

下图显示了 Lu 及其同事研究的bug汇总。从图中可以看出，总共有 105 个bug，其中大部分不是死锁（74 个）；其余 31 个是死锁bug。此外，您还可以看到每个应用程序的bug数量；OpenOffice 的并发bug总数只有 8 个，而 Mozilla 则有近 60 个。

![image-20240411160133229](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/Bugs_In_Modern_Applications.png)

我们现在对这些不同类别的bug（非死锁、死锁）进行更深入的研究。对于第一类非死锁bug，我们将使用研究中的示例进行讨论。对于第二类死锁bug，我们将讨论在预防、避免或处理死锁方面所做的大量工作。

## 非死锁bug

根据 Lu 的研究，非死锁bug占并发bug的大多数。但这些bug属于哪种类型？它们是如何产生的？我们该如何修复它们？我们现在讨论 Lu 等人发现的两大类非死锁bug：原子性违规bug和顺序违规bug。

### 原子性违规bug

遇到的第一类问题被称为原子性违规。下面是一个在 MySQL 中发现的简单示例。

```c
Thread 1::
if (thd-&gt;proc_info) {
    ...
    fputs(thd-&gt;proc_info, ...);
    ...
}

Thread 2::
thd-&gt;proc_info = NULL;
```

在示例中，两个不同的线程访问了结构体 `thd` 中的字段 `proc_info`。第一个线程检查值是否为非空值，然后打印其值；第二个线程将其设置为空值。显然，如果第一个线程执行了检查，但在调用 `fputs` 之前被中断，那么第二个线程可能会在中间运行，从而将指针设置为 NULL；当第一个线程恢复运行时，它将崩溃，因为 NULL 指针将被 `fputs` 解除引用。

根据 Lu 等人的说法，原子性违规的更正式定义是：&lt;font color=&#34;red&#34;&gt;“违反了多个内存访问之间所需的可串行性（即代码区域应该是原子性的，但在执行过程中并未强制执行原子性）。”&lt;/font&gt;在上面的示例中，代码对 `proc_info` 的非 NULL 检查以及 `fputs()` 调用中 `proc_info` 的使用有一个原子性假设（用 Lu 的话说）；当假设不正确时，代码将无法按预期工作。

找到此类问题的解决方案通常（但并非总是）很简单。在此解决方案中，我们只需在共享变量引用周围添加锁，确保当任一线程访问 `proc_info` 字段时，它都持有锁（`proc_info_lock`）。当然，访问该结构的任何其他代码也应该在执行此操作之前获取此锁。

```c
pthread_mutex_t proc_info_lock = PTHREAD_MUTEX_INITIALIZER;

Thread 1::
pthread_mutex_lock(&amp;proc_info_lock);
if (thd-&gt;proc_info) {
    ...
    fputs(thd-&gt;proc_info, ...);
    ...
}
pthread_mutex_unlock(&amp;proc_info_lock);

Thread 2::
pthread_mutex_lock(&amp;proc_info_lock);
thd-&gt;proc_info = NULL;
pthread_mutex_unlock(&amp;proc_info_lock);
```

### 顺序违规bug

Lu 等人发现的另一种常见的非死锁bug被称为 &#34;顺序违规&#34;。下面是另一个简单的例子。

```c
Thread 1::
void init() {
    ...
    mThread = PR_CreateThread(mMain, ...);
    ...
}

Thread 2::
void mMain(...) {
    ...
    mState = mThread-&gt;State;
    ...
}
```

线程 2 中的代码似乎假定变量 `mThread` 已被初始化（并且不是 NULL）；但是，如果线程 2 创建后立即运行，那么在线程 2 的 `mMain()` 中访问 `mThread` 时，它的值将不会被设置，并且很可能会因解引用 NULL 指针而崩溃。请注意，我们假设 `mThread` 的值最初为 NULL；如果不是，那么在线程 2 中通过解引用访问任意内存位置时，可能会发生更奇怪的事情。

违反顺序的更正式定义是这样的：&lt;font color=&#34;red&#34;&gt;&#34;两个（组）内存访问之间的理想顺序被颠倒（即 A 应总是在 B 之前执行，但在执行过程中顺序并没有被强制执行）&#34;。&lt;/font&gt;

解决这类bug的方法一般是强制执行排序。正如我们之前详细讨论过的，使用**条件变量**是将这种同步方式添加到现代代码库中的一种简单而稳健的方法。在上面的例子中，我们可以将代码重写如下：

```c
pthread_mutex_t mtLock = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t 	mtCond = PTHREAD_COND_INITIALIZER;
int mtInit 			   = 0;

Thread 1::
void init() {
    ...
    mThread = PR_CreateThread(mMain, ...);

    // signal that the thread has been created...
    pthread_mutex_lock(&amp;mtLock);
    mtInit = 1;
    pthread_cond_signal(&amp;mtCond);
    pthread_mutex_unlock(&amp;mtLock);
    ...
}

Thread 2::
void mMain(...) {
    ...
    // wait for the thread to be initialized...
    pthread_mutex_lock(&amp;mtLock);
    while (mtInit == 0)
        pthread_cond_wait(&amp;mtCond, &amp;mtLock);
    pthread_mutex_unlock(&amp;mtLock);

    mState = mThread-&gt;State;
    ...
}
```

在这个固定的代码序列中，我们添加了一个锁（`mtLock`）和相应的条件变量（`mtCond`），以及一个状态变量（`mtInit`）。初始化代码运行时，它会将 `mtInit` 的状态设置为 1，并发出信号表示已完成设置。如果线程 2 在这之前运行，它将等待这个信号和相应的状态变化；如果线程 2 在之后运行，它将检查状态，发现初始化已经发生（即 `mtInit` 被设置为 1），从而继续正常运行。需要注意的是，我们可以使用 `mThread` 作为状态变量本身，但为了简单起见，这里不这样做。当线程之间需要排序时，条件变量（或 信号量）就能派上用场。

## 死锁bug

### 基本介绍

除了上面提到的并发bug，在许多具有复杂锁定协议的并发系统中还会出现一个典型的问题，即死锁。例如，当一个线程（例如线程 1）持有一个锁（L1）并等待另一个锁（L2）时，就会出现死锁；不幸的是，持有锁 L2 的线程（线程 2）正在等待 L1 被释放。下面的代码片段演示了这种潜在的死锁：

```c
Thread 1: 					Thread 2:
pthread_mutex_lock(L1); 	pthread_mutex_lock(L2);
pthread_mutex_lock(L2); 	pthread_mutex_lock(L1);
```

注意，如果这段代码运行，并不一定会发生死锁；相反，如果线程 1 获取锁 L1，然后线程 2 发生上下文切换，则可能会发生这种情况。此时，线程 2 获取 L2，并尝试获取 L1。因此，我们遇到了死锁，因为每个线程都在等待另一个线程，而两个线程都无法运行。

如下图所示，图中出现循环就表明出现了死锁。这张图应该能说明问题。程序员应该如何编写代码以便以某种方式处理死锁？我们问题的关键是我们应该如何构建系统来预防、避免或至少检测死锁并从中恢复？这是当今系统中真正的问题吗？

![image-20240411184424257](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/The-Deadlock-Dependency-Graph.png)

### 为什么会出现死锁

你可能会想，像上面这样的简单死锁似乎很容易避免。例如，如果线程 1 和线程 2 都确保以相同的顺序抓取锁，死锁就永远不会出现。那么，为什么会出现死锁呢？

其中一个原因是，在大型代码库中，组件之间会产生复杂的依赖关系。以操作系统为例。虚拟内存系统可能需要访问文件系统，以便从磁盘分页读入一个数据块；文件系统随后可能需要一个内存页来读入该数据块，从而与虚拟内存系统发生关联。因此，在大型系统中设计锁定策略时必须小心谨慎，以避免代码中可能自然出现的循环依赖关系造成死锁。

另一个原因是**封装**的本质。作为软件开发人员，我们被教导要隐藏实现的细节，从而使软件更容易以模块化的方式构建。遗憾的是，这种模块化与锁定并不匹配。正如 Jula 等人所指出的，一些看似无害的接口几乎会让你陷入死锁。例如，以 Java 向量类和 `AddAll()` 方法为例。这个例程的调用过程如下

```c
Vector v1, v2;
v1.AddAll(v2);
```

在内部，由于该方法需要是多线程安全的，因此需要获取添加到 (v1) 的向量和参数 (v2) 的锁。该例程以某种任意顺序获取所述锁（先是 v1，然后是 v2），以便将 v2 的内容添加到 v1。如果其他线程几乎同时调用 `v2.AddAll(v1)`，则可能会出现死锁，而所有这些都对调用应用程序来说是隐藏的。

### 死锁原因

发生死锁需要满足四个条件：

1. **互斥**：线程声明对其所需资源的独占控制（例如，线程获取锁）。
2. **持有并等待**：线程持有分配给它们的资源（例如，它们已经获取的锁），同时等待其他资源（例如，它们希望获取的锁）。
3. **不可抢占**：无法从持有资源的线程中强制删除资源（例如锁）。
4. **循环等待**：存在循环线程链，使得每个线程持有链中下一个线程正在请求的一个或多个资源（例如，锁）。

如果这四个条件中任何一个不满足，就不会发生死锁。因此，我们首先探索防止死锁的技术；这些策略中的每一个都旨在防止出现上述情况之一，因此是处理死锁问题的一种方法。

### 预防死锁

#### 循环等待

最实用的预防方法（当然也是经常使用的方法）可能是编写锁定代码，使其永远不会引起循环等待条件。要做到这一点，最直接的方法就是为锁的获取提供一个**总排序**。例如，如果系统中只有两个锁（`L1` 和`L2`），则可以通过始终在 `L2` 之前获取 `L1` 来防止死锁。这种严格的排序可以确保不会出现循环等待，从而避免死锁。

当然，在更复杂的系统中，会存在两个以上的锁，因此很难实现完全的锁排序（也许根本没有必要）。因此，部分排序可以有效地构建锁获取结构，从而避免死锁。Linux 中的内存映射代码就是一个很好的部分锁排序的真实例子；源代码顶部的注释揭示了十组不同的锁获取顺序，包括简单的如 &#34;`i_mutex` before `i_mmap_mutex` &#34;和更复杂的如 &#34;`i_mmap_mutex` before `private_lock` before `swap_lock` before `mapping-&gt;tree_lock`&#34;。

可以想象，无论是全部排序还是部分排序，都需要精心设计锁定策略，而且必须非常谨慎。此外，排序只是一种惯例，马虎的程序员很容易忽略锁定协议，并可能导致死锁。最后，锁排序要求对代码库以及各种例程的调用方式有深入的了解；只要有一个bug，就可能导致死锁。

&gt; &lt;center&gt;TIP：通过锁地址强制执行锁排序&lt;/center&gt;
&gt;
&gt; 在某些情况下，一个函数必须获取两个（或更多）锁；因此，我们知道我们必须小心，否则可能会出现死锁。想象一个按如下方式调用的函数：`do_something(mutex_t *m1, mutex_t *m2)`。如果代码总是在 `m2` 之前获取 `m1`（或者总是在 `m1` 之前获取 `m2`），则可能会死锁，因为一个线程可以调用 `do_something(L1, L2)`，而另一个线程可以调用 `do_something(L2, L1)`。
&gt;
&gt; 为了避免这个特殊问题，聪明的程序员可以使用每个锁的地址作为获取锁的顺序。通过以从高到低或从低到高的地址顺序获取锁，`do_something()` 可以保证它始终以相同的顺序获取锁，无论它们传入的顺序如何。代码看起来像这样这：
&gt;
&gt; ```c
&gt; if (m1 &gt; m2) { // grab locks in high-to-low address order
&gt;     pthread_mutex_lock(m1);
&gt;     pthread_mutex_lock(m2);
&gt; } else {
&gt;     pthread_mutex_lock(m2);
&gt;     pthread_mutex_lock(m1);
&gt; }
&gt; // Code assumes that m1 != m2 (it is not the same lock)
&gt; ```
&gt;
&gt; 通过使用这种简单的技术，程序员可以确保简单高效地实现无死锁的多锁获取。

#### 持有并等待

通过原子方式一次性获取所有锁，可以避免死锁的保持和等待要求。在实际操作中，可以通过以下方式实现：

```c
pthread_mutex_lock(prevention); // begin lock acquisition
pthread_mutex_lock(L1);
pthread_mutex_lock(L2);
...
pthread_mutex_unlock(prevention); // end
```

通过首先获取锁`prevention`，此代码确保在获取锁时不会发生任何不及时的线程切换，从而再次避免死锁。当然，这要求任何时候任何线程抓取一个锁时，它首先获取全局预防锁。例如，如果另一个线程尝试以不同顺序抓取锁`L1`和`L2`，则是可以的，因为在这样做时它将持有`prevention`锁。

请注意，由于多种原因，该解决方案存在问题。和以前一样，封装对我们不利：当调用例程时，这种方法要求我们准确地知道必须持有哪些锁并提前获取它们。这种技术还可能会降低并发性，因为所有锁都必须尽早（立即）获取，而不是在真正需要时获取。

#### 不可抢占

因为我们通常将锁视为一直保持到调用解锁为止，所以多次获取锁常常会给我们带来麻烦，因为在等待一个锁时，我们正在持有另一个锁。许多线程库提供了一组更灵活的接口来帮助避免这种情况。具体来说，例程 `pthread_mutex_trylock()` 要么获取锁（如果可用）并返回成功，要么返回指示锁已被持有的bug代码；在后一种情况下，如果您想抓住该锁，可以稍后重试。

这样的接口可以按如下方式使用来构建无死锁、有序鲁棒的锁获取协议：

```c
top:
pthread_mutex_lock(L1);
if (pthread_mutex_trylock(L2) != 0) {
    pthread_mutex_unlock(L1);
    goto top;
}
```

需要注意的是，另一个线程可以遵循相同的协议，但以另一种顺序（先 `L2` 后 `L1`）获取锁，这样程序仍然不会出现死锁。然而，一个新的问题出现了：**活锁**。两个线程有可能（虽然可能性不大）都在重复尝试这种顺序，但多次都无法获得两个锁。在这种情况下，两个系统都在重复运行这个代码序列（因此不是死锁），但却没有取得进展，因此被称为活锁。活锁问题也有解决方法：例如，可以在循环之前添加一个随机延迟，然后重新尝试整个过程，从而降低竞争线程之间重复干扰的几率。

关于这个解决方案的一点是：它绕过了使用 `trylock` 方法的难点。第一个可能再次出现的问题是封装：如果这些锁中的一个被埋在某个被调用的例程中，那么跳回起点的实现就会变得更加复杂。&lt;font color=&#34;red&#34;&gt;例如，如果在获取 L1 后，代码分配了一些内存，那么在获取 L2 失败后，就必须释放这些内存，然后再跳回到顶层，重新尝试整个序列。&lt;/font&gt;不过，在有限的情况下（例如前面提到的 Java 向量方法），这种方法可能会很有效。

你可能还会注意到，这种方法并没有真正加入抢占（从拥有锁的线程中强行夺走锁的操作），而是使用 `trylock` 方法允许开发者以一种优雅的方式退出锁的所有权（即抢占自己的所有权）。不过，这是一种实用的方法，因此尽管在这方面并不完美，我们还是将其包含在这里。

#### 互斥

最后一种防范技术是完全避免互斥。一般来说，我们知道这很困难，因为我们希望运行的代码确实存在临界区。那么我们能做些什么呢？

Herlihy 提出了一个想法：我们可以设计各种完全不需要锁的数据结构。这些无锁（以及相关的无等待）方法背后的理念很简单：利用强大的硬件指令，可以以不需要显式锁定的方式构建数据结构。

举个简单的例子，假设我们有一条比较和交换指令，你可能还记得这是一条由硬件提供的原子指令，它的作用如下：

```c
int CompareAndSwap(int *address, int expected, int new) {
    if (*address == expected) {
        *address = new;
        return 1; // success
    }
    return 0; // failure
}
```

想象一下，我们现在想以原子方式将一个值递增一定量。我们可以这样做：

```c
void AtomicIncrement(int *value, int amount) {
    do {
        int old = *value;
    } while (CompareAndSwap(value, old, old &#43; amount) == 0);
}
```

我们没有获取锁，进行更新，然后释放它，而是构建了一种方法，反复尝试将值更新为心智并使用`CompareAndSwap`来执行此操作。通过这种方式，不会获取锁，并且不会出现死锁（尽管活锁仍然是可能的）。

让我们考虑一个稍微复杂的例子：列表插入。以下是在列表头部插入的代码：

```c
void insert(int value) {
    node_t *n = malloc(sizeof(node_t));
    assert(n != NULL);
    n-&gt;value = value;
    n-&gt;next = head;
    head = n;
}
```

这段代码执行简单的插入操作，但如果多个线程 &#34;同时 &#34;调用，就会出现竞争条件。当然，我们可以用获取和释放锁来解决这个问题：

```c
void insert(int value) {
    node_t *n = malloc(sizeof(node_t));
    assert(n != NULL);
    n-&gt;value = value;
    pthread_mutex_lock(listlock); // begin critical section
    n-&gt;next = head;
    head = n;
    pthread_mutex_unlock(listlock); // end critical section
}
```

在这个解决方案中，我们使用了传统的锁。相反，我们可以尝试使用`CompareAndSwap`指令，以无锁定方式执行插入操作。下面是一种可行的方法：

```c
void insert(int value) {
    node_t *n = malloc(sizeof(node_t));
    assert(n != NULL);
    n-&gt;value = value;
    do {
        n-&gt;next = head;
    } while (CompareAndSwap(&amp;head, n-&gt;next, n) == 0);
}
```

这里的代码会更新下一个指针，使其指向当前头部，然后尝试将新创建的节点交换到列表的新头部。但是，如果其他线程在此期间成功地交换了一个新的头，那么这个操作就会失败，从而导致这个线程用新的头再次重试。

当然，建立一个有用的列表需要的不仅仅是列表插入，毫不奇怪，建立一个能以无锁方式插入、删除和执行查找的列表并非易事。

### 通过调度避免死锁

在某些情况下，避免死锁比预防死锁更可取。避免死锁需要一些全局知识，了解各个线程在执行期间可能会获取哪些锁，并随后以保证不会发生死锁的方式调度所述线程。

例如，假设我们有两个处理器和四个线程，必须在它们上进行调度。进一步假设我们知道线程 1 (`T1`) 获取锁 `L1` 和 `L2`（以某种顺序，在执行期间的某个时刻），`T2` 也获取 `L1` 和 `L2`，`T3` 仅获取 `L2`，而 `T4` 根本不获取锁。我们可以以表格形式展示线程的这些锁获取需求：

|      |  T1  |  T2  |  T3  |  T4  |
| :--: | :--: | :--: | :--: | :--: |
|  L1  | yes  | yes  |  no  |  no  |
|  L2  | yes  | yes  | yes  |  no  |

因此，智能调度程序可以计算出，只要 `T1` 和 `T2` 不同时运行，就不会出现死锁。下面就是这样一个调度程序：

![image-20240411195407552](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/CPU_Scheduler_Example.png)

请注意，（`T3` 和 `T1`）或（`T3` 和 `T2`）是可以重叠的。即使 `T3` 抓住了锁 `L2`，它也不会因为与其他线程同时运行而导致死锁，因为它只抓住了一个锁。让我们再看一个例子。在这个例子中，对相同资源（同样是锁 `L1` 和 `L2`）的争用更多，如下面的争用表所示：

|      |  T1  |  T2  |  T3  |  T4  |
| :--: | :--: | :--: | :--: | :--: |
|  L1  | yes  | yes  | Yes  |  no  |
|  L2  | yes  | yes  | yes  |  no  |

其中，线程 `T1`、`T2` 和 `T3` 都需要在执行过程中的某个时刻同时抓住锁 `L1` 和 `L2`。下面是一个可以保证不发生死锁的调度表：

![image-20240411195925218](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/CPU_Scheduler_Example_2.png)

正如你所看到的，静态调度导致了一种保守的方法，即 `T1`、`T2` 和 `T3` 都在同一个处理器上运行，因此完成作业的总时间大大延长。虽然这些任务有可能同时运行，但由于担心死锁，我们无法这样做，而代价就是性能。

Dijkstra 的银行家算法就是这种方法的一个著名例子。遗憾的是，这些方法只在非常有限的环境中有用，例如，在嵌入式系统中，人们完全了解必须运行的全部任务集及其所需的锁。此外，这种方法还会限制并发性，正如我们在上文第二个例子中看到的那样。因此，通过调度避免死锁并不是一种广泛使用的通用解决方案。

### 检测和恢复

最后一个通用策略是允许死锁偶尔发生，然后在检测到死锁后采取一些措施。例如，如果操作系统每年都会出现一次死锁，那么你只需重启操作系统，然后继续愉快地（或暴躁地）工作。如果死锁很少发生，那么这种不解决问题的方法确实非常实用。

许多数据库系统都采用了死锁检测和恢复技术。死锁检测器定期运行，构建资源图并检查其是否存在循环。一旦出现循环（死锁），系统就需要重新启动。如果首先需要对数据结构进行更复杂的修复，则可能需要人工参与，以简化修复过程。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/25.%E5%B9%B6%E5%8F%91bug/  

