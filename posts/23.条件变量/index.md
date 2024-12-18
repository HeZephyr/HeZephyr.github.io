# 条件变量

到目前为止，我们已经开发了锁的概念，并了解了如何通过正确的硬件和操作系统支持组合来正确构建锁。不幸的是，锁并不是构建并发程序所需的唯一原语。

特别是，在很多情况下，线程希望在继续执行之前检查**条件**是否为真。例如，父线程可能希望在继续之前检查子线程是否已完成（这通常称为 join()）；这样的等待应该如何实现呢？我们来看下面这段代码。

```c
#include &lt;stdio.h&gt;
#include &#34;common_threads.h&#34;

void *child(void *arg) {
    printf(&#34;child\n&#34;);
    // XXX how to indicate we are done?
    return NULL;
}

int main(int argc, char *argv[]) {
    printf(&#34;parent: begin\n&#34;);
    pthread_t c;
    Pthread_create(&amp;c, NULL, child, NULL); // create child
    // XXX how to wait for child?
    printf(&#34;parent: end\n&#34;);
    return 0;
}
```

我们希望在这里看到以下输出：

```c
parent: begin
child
parent: end
```

我们可以尝试使用共享变量，如下面这段代码所示。此解决方案通常可以工作，但效率非常低，因为父进程会自旋并浪费 CPU 时间。我们在这里想要的是某种方法让父进程进入睡眠状态，直到我们等待的条件（例如，子进程完成执行）实现为止。 

```c
#include &lt;stdio.h&gt;
#include &#34;common_threads.h&#34;

volatile int done = 0;
void *child(void *arg) {
    printf(&#34;child\n&#34;);
    done = 1;
    return NULL;
}

int main(int argc, char *argv[]) {
    printf(&#34;parent: begin\n&#34;);
    pthread_t c;
    Pthread_create(&amp;c, NULL, child, NULL); // create child
    while (done == 0) // spin
        ;
    printf(&#34;parent: end\n&#34;);
    return 0;
}
```

&gt; &lt;center&gt;关键：如何等待条件？&lt;/center&gt;
&gt;
&gt;  在多线程程序中，线程在继续操作之前等待某些条件变为真通常很有用。这种简单的方法，即只是自旋直到条件成立，效率非常低并且浪费 CPU 周期，并且在某些情况下可能是不正确的。那么，线程应该如何等待条件呢？

## 定义和例程

为了等待条件成真，线程可以使用所谓的**条件变量**。条件变量是一个显式队列，当某些执行状态（即某些条件）不符合预期时（通过**等待条件**），线程可以将自己置于该队列中；当其他线程改变上述状态时，可以唤醒一个（或多个）等待的线程，从而允许它们继续执行（通过**向条件发出信号**）。这个想法可以追溯到 Dijkstra 使用的 &#34;私有信号&#34;；后来，Hoare 在他关于监控器的工作中将类似的想法命名为 &#34;条件变量&#34;。

要声明这样一个条件变量，只需这样写：`pthread_cond_t c`;，将 `c` 声明为条件变量（注意：还需要适当的初始化）。条件变量有两个相关操作：`wait()` 和 `signal()`。`wait()`调用在线程希望进入休眠状态时执行；`signal()`调用在线程改变了程序中的某些内容，从而希望唤醒在此条件下等待的休眠线程时执行。具体来说，POSIX 调用是这样的：

```c
pthread_cond_wait(pthread_cond_t *c, pthread_mutex_t *m);
pthread_cond_signal(pthread_cond_t *c);
```

为简单起见，我们通常将其称为 `wait(`) 和 `signal()`。关于 `wait()`调用，有一点你可能会注意到，它也将一个`mutex`作为参数；它假定在调用 `wait()` 时这个`mutex`已被锁定。`wait()`的职责是释放锁并让调用线程休眠（**原子式**）；当线程醒来时（在其他线程发出信号后），它必须在返回调用者之前重新获取锁。之所以如此复杂，是因为我们希望在线程试图让自己进入休眠状态时，防止出现某些竞争条件。

让我们来看看`join`问题的解决方案，以便更好地理解这一点，代码如下所示。

```c
#include &lt;stdio.h&gt;
#include &lt;pthread.h&gt;
#include &#34;common_threads.h&#34;

int done = 0;
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t c = PTHREAD_COND_INITIALIZER;

void thr_exit() {
    Pthread_mutex_lock(&amp;m);
    done = 1;
    Pthread_cond_signal(&amp;c);    // wake up waiting thread
    Pthread_mutex_unlock(&amp;m);
}

void *child(void *arg) {
    printf(&#34;child\n&#34;);
    thr_exit();
    return NULL;
}

void thr_join() {
    Pthread_mutex_lock(&amp;m);
    while (done == 0)
        Pthread_cond_wait(&amp;c, &amp;m);  // unlock m and wait for signal
    Pthread_mutex_unlock(&amp;m);
}

int main(int argc, char *argv[]) {
    printf(&#34;parent: begin\n&#34;);
    pthread_t p;
    Pthread_create(&amp;p, NULL, child, NULL);
    thr_join();
    printf(&#34;parent: end\n&#34;);
    return 0;
}
```

有两种情况需要考虑。第一种情况是父线程创建了子线程，但自己继续运行（假设我们只有一个处理器），因此立即调用 `thr_join()` 等待子线程完成。在这种情况下，父线程会获取锁，检查子线程是否完成（未完成），然后调用 `wait()` 使自己进入休眠状态（从而释放锁）。子线程最终将运行，打印信息 &#34;child&#34;，并调用 `thr_exit()` 来唤醒父线程；该代码只是获取锁、设置状态变量 `done`，并向父线程发出信号，从而唤醒父线程。最后，父线程将运行（从 `wait()`返回时锁已被锁定）、解锁并打印最终信息 &#34;parent:end：结束&#34;。 

在第二种情况下，子进程在创建后立即运行，将 `done` 设为 1，调用信号唤醒睡眠线程（但没有，所以直接返回），然后完成。然后父线程运行，调用 `thr_join()`，发现 done 为 1，于是不再等待，直接返回。

最后一点：你可能会发现父进程在决定是否等待条件时使用了 `while` 循环而不是 `if` 语句。虽然从程序逻辑上看，这并非绝对必要，但这始终是个好主意，我们将在下文中看到。

为了确保你理解 `thr_exit()` 和 `thr_join()` 代码中每一段代码的重要性，让我们尝试几种不同的实现方法。首先，你可能想知道我们是否需要完成状态变量。如果代码看起来像下面的示例呢？这样行得通吗？

```c
void thr_exit() {
    Pthread_mutex_lock(&amp;m);
    Pthread_cond_signal(&amp;c);
    Pthread_mutex_unlock(&amp;m);
}

void thr_join() {
    Pthread_mutex_lock(&amp;m);
    Pthread_cond_wait(&amp;c, &amp;m);
    Pthread_mutex_unlock(&amp;m);
}
```

不幸的是，这种方法是有问题的。想象一下，如果子进程立即运行并立即调用`thr_exit()`；在这种情况下，子进程会发出信号，但条件上没有任何线程处于休眠状态。当父进程运行时，它将简单地调用`wait`并被卡住；没有任何线程会唤醒它。从这个例子中，你应该意识到状态变量`done`的重要性；它记录了线程感兴趣的值。睡眠、唤醒和锁定都围绕着它构建。

以下是另一个糟糕的实现方式。在这个例子中，我们假设不需要持有锁来发出信号和等待。可能会出现什么问题？思考一下！

```c
void thr_exit() {
    done = 1;
    Pthread_cond_signal(&amp;c);
}

void thr_join() {
    if (done == 0) {
        Pthread_cond_wait(&amp;c);
    }
}
```

这里的问题是一个微妙的竞争条件。具体来说，如果父线程调用`thr_join()`，然后检查done的值，它会发现它是0，从而尝试进入睡眠状态。但就在它调用 `wait` 进入睡眠状态之前，父线程被中断，子线程开始运行。子线程将状态变量 `done` 更改为 1 并发出信号，但没有线程在等待，因此没有线程被唤醒。当父线程再次运行时，它就永远沉睡了，这是可悲的。

&gt; &lt;center&gt;TIP：在发出信号时始终保持锁定
&gt;     
&gt; &lt;/center&gt;
&gt;
&gt;  虽然并非在所有情况下都严格要求保持锁定，但在使用条件变量时，在发出信号时保持锁定可能是最简单且最好的方法。上面的示例显示了必须持有锁才能正确的情况；然而，在其他一些情况下，不这样做也可以，但可能是您应该避免的事情。因此，为了简单起见，&lt;font color=&#34;red&#34;&gt;在调用信号时保持锁定&lt;/font&gt;。
&gt;
&gt; 本技巧的反面，即在调用 `wait` 时保持锁定，不仅仅是一个技巧，而是 `wait` 语义所强制的，因为 `wait` 总是 
&gt;
&gt; * 假设在调用它时锁定已被持有，
&gt; * 释放当让调用者进入睡眠状态时所说的锁
&gt; * 在返回之前重新获取锁。
&gt;
&gt; 因此，这个技巧的概括是正确的：&lt;font color=&#34;red&#34;&gt;在调用 `signal` 或 `wait` 时保持锁定，你将永远处于良好状态。&lt;/font&gt; 

## 生产者—消费者（有界缓冲区）问题

### 基本概念

在本章中，我们将面对的下一个同步问题被称为**生产者/消费者问题**，有时也被称为**有界缓冲区问题**，它是由 Dijkstra 首次提出的。事实上，正是这个生产者/消费者问题促使 Dijkstra 和他的同事们发明了广义的 `semaphore`（可用作锁或条件变量）。

设想一个或多个生产者线程和一个或多个消费者线程。生产者生成数据项并将其放入缓冲区；消费者从缓冲区中抓取上述数据项，并以某种方式消费它们。

这种安排在许多实际系统中都会出现。例如，在多线程网络服务器中，生产者将 HTTP 请求放入工作队列（即有界缓冲区）；消费者线程从队列中取出请求并进行处理。

有界缓冲区也用于将一个程序的输出导入另一个程序，例如，`grep foo file.txt | wc -l`。此示例同时运行两个进程：`grep` 将 `file.txt` 中含有 `foo` 字符串的行写入它认为的标准输出；`UNIX shell` 将输出重定向到所谓的 `UNIX` 管道（通过`pipe`系统调用创建）。管道的另一端连接到 `wc` 进程的标准输入，该进程只需计算输入流的行数并打印出结果。因此，`grep` 进程是生产者，`wc` 进程是消费者，它们之间是一个内核有界缓冲区。

由于有界缓冲区是共享资源，我们当然必须要求同步访问它，以免出现竞争条件。为了更好地理解这个问题，让我们来看看一些实际的代码。我们首先需要一个共享缓冲区，生产者将数据放入缓冲区，消费者从缓冲区中取出数据。为了简单起见，我们只使用一个整数（当然，你也可以想象把一个数据结构的指针放到这个槽中），以及两个内部例程，分别用于向共享缓冲区中放入一个值，以及从缓冲区中取出一个值。代码如下所示：

```c
int buffer;
int count = 0; // initially, empty

void put(int value) {
    assert(count == 0);
    count = 1;
    buffer = value;
}

int get() {
    assert(count == 1);
    count = 0;
    return buffer;
}
```

`put()` 例程假定缓冲区为空（并通过`assert`进行检查），然后简单地将一个值放入共享缓冲区，并通过将`counter`设为 1 来标记缓冲区已满。不用担心这个共享缓冲区只有一个入口；稍后，我们将把它推广到可以容纳多个入口的队列，这将比听起来更有趣。

现在我们需要编写一些例程来知道何时可以访问缓冲区以将数据放入其中或从其中取出数据。其条件应该是显而易见的：仅当 `count` 为零时（即缓冲区为空时）才将数据放入缓冲区，并且仅当 `count` 为 1 时（即缓冲区已满时）从缓冲区中获取数据。如果我们编写同步代码，使得生产者将数据放入已满的缓冲区中，或者消费者从空缓冲区中获取数据，那么我们就做错了（在这段代码中，**将触发`assert`**）。

这项工作将由两种类型的线程完成，其中一组我们称为**生产者线程**，另一组我们称为**消费者线程**。如下所示， 显示了生产者将整数放入共享缓冲区循环次数的代码，以及消费者从共享缓冲区中获取数据（永远）的代码，每次打印从共享缓冲区中拉出的数据项。

```c
void *producer(void *arg) {
    int i;
    int loops = (int)arg;
    for (i = 0; i &lt; loops; i&#43;&#43;) {
        put(i);
    }
}

void *consumer(void *arg) {
    int i;
    while (1) {
        int tmp = get();
        printf(&#34;%d\n&#34;, tmp);
    }
}
```

### 一个残缺的解决方案

现在想象一下，我们只有一个生产者和一个消费者。显然，`put()` 和 `get() `例程都有临界区，因为 `put()` 会更新缓冲区，而 `get()` 会从缓冲区读取数据。然而，在代码周围加锁是行不通的；我们需要更多的东西。毫不奇怪，我们需要的是一些条件变量。如下代码所示：在这个（残缺的）首次尝试中，我们只有一个条件变量 `cond` 和相关的锁`mutex`。

```c
int loops; // must initialize somewhere...
cond_t cond;
mutex_t mutex;

void *producer(void *arg) {
    int i;
    for (i = 0; i &lt; loops; i&#43;&#43;) {
        Pthread_mutex_lock(&amp;mutex); // p1
        if (count == 1) // p2
            Pthread_cond_wait(&amp;cond, &amp;mutex); // p3
        put(i); // p4
        Pthread_cond_signal(&amp;cond); // p5
        Pthread_mutex_unlock(&amp;mutex); // p6
    }
}

void *consumer(void *arg) {
    int i;
    for (i = 0; i &lt; loops; i&#43;&#43;) {
        Pthread_mutex_lock(&amp;mutex); // c1
        if (count == 0) // c2
            Pthread_cond_wait(&amp;cond, &amp;mutex); // c3
        int tmp = get(); // c4
        Pthread_cond_signal(&amp;cond); // c5
        Pthread_mutex_unlock(&amp;mutex); // c6
        printf(&#34;%d\n&#34;, tmp);
    }
}
```

让我们来看看生产者和消费者之间的信号逻辑。当生产者想要填满缓冲区时，它会等待缓冲区为空（p1-p3）。消费者的逻辑完全相同，但等待的条件不同：缓冲区满（c1-c3）。如果只有一个生产者和一个消费者，上面代码就能正常工作。但是，如果我们有多个线程（例如两个消费者），解决方案就会出现两个关键问题。它们是什么？

让我们来了解第一个问题，它与等待之前的 `if` 语句有关。假设有两个消费者（$T_{c_1}$ 和 $T_{c_2}$）和一个生产者（$T_p$）。首先，运行消费者 ($T_{c_1}$)；它获取锁 (c1)，检查是否有缓冲区可供使用 (c2)，如果没有，则等待 (c3)（释放锁）。

然后运行生产者 ($T_p$)。它获取锁 (p1)，检查所有缓冲区是否已满 (p2)，如果没有，则继续填充缓冲区 (p4)。然后，生产者发出缓冲区已填满的信号（p5）。重要的是，这将第一个消费者（$T_{c_1}$ ）从条件变量的休眠状态移到就绪队列；$T_{c_1}$  现在可以运行（但尚未运行）。然后，生产者继续运行，直到发现缓冲区已满，这时它才进入休眠状态（p6, p1-p3）。

问题就出现在这里：另一个消费者（$T_{c_2}$ ）悄悄进入并消耗了缓冲区中的一个现有值（c1、c2、c4、c5、c6，由于缓冲区已满，跳过了 c3 处的`wait`）。现在假设 $T_{c_1}$运行，在从等待返回之前，它会重新获取锁，然后返回。然后它调用 `get()` (c4)，但没有缓冲区要使用！断言触发了，代码没有按预期运行。显然，我们应该以某种方式阻止  $T_{c_1}$ 尝试消耗，因为$T_{c_2}$ 偷偷地进入并消耗了缓冲区中产生的一个值。下图显示了每个线程执行的操作及其随时间变化的调度器状态（就绪、运行或休眠）。

![image-20240410111405759](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/Thread_Trace_Broken_Solution_v1.png)

出现问题的原因很简单：在生产者唤醒 $T_{c_1}$ 之后，但在 $T_{c_1}$ 运行之前，有界缓冲区的状态发生了变化（这要归功于 $T_{c_2}$）。向线程发出信号只能唤醒它们，因此它只是提示世界的状态已经发生了变化（在本例中，缓冲区中已经放入了一个值），但并不能保证当被唤醒的线程运行时，状态仍然如愿以偿。对信号含义的这种解释通常被称为 **Mesa 语义**，这是以首次以这种方式构建条件变量的研究命名的；与之相对的是 **Hoare** 语义，它更难构建，但能更有力地保证被唤醒的线程在被唤醒后立即运行。几乎所有已构建的系统都采用了 Mesa 语义。

### 更好，但仍然残缺：while，而不是if

幸运的是，解决方法很简单：将 `if` 改为 `while`。代码如下所示：

```c
int loops;
cond_t cond;
mutex_t mutex;

void *producer(void *arg) {
    int i;
    for (i = 0; i &lt; loops; i&#43;&#43;) {
        Pthread_mutex_lock(&amp;mutex); // p1
        while (count == 1) // p2
            Pthread_cond_wait(&amp;cond, &amp;mutex); // p3
        put(i); // p4
        Pthread_cond_signal(&amp;cond); // p5
        Pthread_mutex_unlock(&amp;mutex); // p6
    }
}

void *consumer(void *arg) {
    int i;
    for (i = 0; i &lt; loops; i&#43;&#43;) {
        Pthread_mutex_lock(&amp;mutex); // c1
        while (count == 0) // c2
            Pthread_cond_wait(&amp;cond, &amp;mutex); // c3
        int tmp = get(); // c4
        Pthread_cond_signal(&amp;cond); // c5
        Pthread_mutex_unlock(&amp;mutex); // c6
        printf(&#34;%d\n&#34;, tmp);
    }
}
```

想一想为什么会这样：现在消费者 $T_{c_1}$ 会醒来，并（在锁定的情况下）立即重新检查共享变量 (c2) 的状态。如果此时缓冲区是空的，消费者就会继续休眠 (c3)。在生产者中，`if` 的推论也被改为 `while` (p2)。

&lt;font color=&#34;red&#34;&gt;得益于 Mesa 语义，使用条件变量时要记住一条简单的规则，那就是始终使用 while 循环。&lt;/font&gt;有时不必重新检查条件，但这样做总是安全的。

然而，这段代码仍然有一个错误，也就是上面提到的两个问题中的第二个。你能发现吗？它与只有一个条件变量有关。

当两个消费者首先运行（$T_{c_1}$  和 $T_{c_2}$ ）并都进入睡眠状态（c3）时，问题就出现了。然后，生产者运行，将一个值放入缓冲区，并唤醒其中一个消费者（例如 $T_{c_1}$ ）。然后，生产者返回循环（沿途释放并重新获取锁），并尝试将更多数据放入缓冲区；由于缓冲区已满，生产者转而等待条件（因此进入睡眠）。现在，一个消费者已准备好运行（$T_{c_1}$ ），两个线程正在等待一个条件（$T_{c_2}$  和 $T_{p}$ ）。

关键问题来了。然后，消费者 $T_{c_1}$  从 `wait()` 返回 (c3) 唤醒，重新检查条件 (c2)，发现缓冲区已满，于是消耗值 (c4)。重要的是，这个消费者会根据条件（c5）发出信号，只唤醒一个处于睡眠状态的线程。然而，它应该唤醒哪个线程呢？

因为消费者清空了缓冲区，显然应该唤醒生产者。但是，如果唤醒消费者 $T_{c_2}$（这是绝对可能的，取决于等待队列的管理方式），我们就会遇到问题。具体来说，消费者 $T_{c_2}$ 会在醒来时发现缓冲区是空的（c2），然后继续休眠（c3）。生产者$T_{p}$ 有一个值要放入缓冲区，但却处于休眠状态。另一个消费者线程 $T_{c_1}$ 也继续休眠。所有三个线程都处于休眠状态，这是一个明显的`bug`；如下图所示，显示了有关这一可怕灾难的残酷步骤。

![image-20240410113350433](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/Thread_Trace_Broken_Solution_v2.png)

信号显然是需要的，但必须更有针对性。消费者不应唤醒其他消费者，只能唤醒生产者，反之亦然。

### 单一缓冲区生产者/消费者解决方案

这里的解决方案又是一个小解决方案：使用两个条件变量而不是一个，以便在系统状态发生变化时正确地发出应该唤醒哪种类型的线程的信号。改进的代码如下所示。

```c
cond_t empty, fill;
mutex_t mutex;

void *producer(void *arg) {
    int i;
    for (i = 0; i &lt; loops; i&#43;&#43;) {
        Pthread_mutex_lock(&amp;mutex);
        while (count == 1)
            Pthread_cond_wait(&amp;empty, &amp;mutex);
        put(i);
        Pthread_cond_signal(&amp;fill);
        Pthread_mutex_unlock(&amp;mutex);
    }
}

void *consumer(void *arg) {
    int i;
    for (i = 0; i &lt; loops; i&#43;&#43;) {
        Pthread_mutex_lock(&amp;mutex);
        while (count == 0)
            Pthread_cond_wait(&amp;fill, &amp;mutex);
        int tmp = get();
        Pthread_cond_signal(&amp;empty);
        Pthread_mutex_unlock(&amp;mutex);
        printf(&#34;%d\n&#34;, tmp);
    }
}
```

在上面的代码中，生产者线程等待条件为`empty`，并发出`fill`信号。相反，消费者线程等待`fill`并发出`empty`信号。通过这样做，上面的第二个问题在设计上就得到了避免：消费者永远不会意外唤醒消费者，生产者也永远不会意外唤醒生产者。

### 最终的生产者/消费者解决方案

我们现在有了一个有效的生产者/消费者解决方案，尽管不是一个完全通用的解决方案。我们所做的最后一个改变是实现更高的并发性和效率；具体来说，我们添加更多的缓冲区槽，以便在睡眠前可以生成多个值，同样可以在睡眠前消耗多个值。由于只有一个生产者和消费者，这种方法更加高效，因为它减少了上下文切换；对于多个生产者或消费者（或两者），它甚至允许并发生产或消费，从而增加并发性。幸运的是，这与我们当前的解决方案相比只是一个小小的改变。

要实现这一正确的解决方案，首先要改变的是缓冲区结构本身以及相应的 `put()` 和 `get()`，代码如下所示。

```c
#define MAX 10
int buffer[MAX];
int fill_ptr = 0;
int use_ptr = 0;
int count = 0;
mutex_t mutex;
cond_t empty, fill;

void put(int value) {
    Pthread_mutex_lock(&amp;mutex);
    while (count == MAX)
        Pthread_cond_wait(&amp;empty, &amp;mutex);
    buffer[fill_ptr] = value;
    fill_ptr = (fill_ptr &#43; 1) % MAX;
    count&#43;&#43;;
    Pthread_cond_signal(&amp;fill);
    Pthread_mutex_unlock(&amp;mutex);
}

int get() {
    int tmp;
    Pthread_mutex_lock(&amp;mutex);
    while (count == 0)
        Pthread_cond_wait(&amp;fill, &amp;mutex);
    tmp = buffer[use_ptr];
    use_ptr = (use_ptr &#43; 1) % MAX;
    count--;
    Pthread_cond_signal(&amp;empty);
    Pthread_mutex_unlock(&amp;mutex);
    return tmp;
}
```

我们还稍微修改了生产者和消费者为确定是否休眠而检查的条件。下面代码显示了正确的等待和信号逻辑。

```c
cond_t empty, fill;
mutex_t mutex;

void *producer(void *arg) {
    int i;
    for (i = 0; i &lt; loops; i&#43;&#43;) {
        Pthread_mutex_lock(&amp;mutex); // p1
        while (count == MAX) // p2
            Pthread_cond_wait(&amp;empty, &amp;mutex); // p3
        put(i); // p4
        Pthread_cond_signal(&amp;fill); // p5
        Pthread_mutex_unlock(&amp;mutex); // p6
    }
}

void *consumer(void *arg) {
    int i;
    for (i = 0; i &lt; loops; i&#43;&#43;) {
        Pthread_mutex_lock(&amp;mutex); // c1
        while (count == 0) // c2
            Pthread_cond_wait(&amp;fill, &amp;mutex); // c3
        int tmp = get(); // c4
        Pthread_cond_signal(&amp;empty); // c5
        Pthread_mutex_unlock(&amp;mutex); // c6
        printf(&#34;%d\n&#34;, tmp);
    }
}
```

生产者只有在所有缓冲区都被填满的情况下才会休眠（p2）；同样，消费者只有在所有缓冲区都被清空的情况下才会休眠（c2）。这样，我们就解决了生产者/消费者的问题。

&gt; &lt;center&gt;TIP：对条件使用 WHILE（而非 IF）&lt;/center&gt; 
&gt;
&gt; 在多线程程序中检查条件时，使用 `while` 循环始终是正确的；而仅使用 `if` 语句可能是错误的，这取决于信号的语义。因此，始终使用 `while` 语句，你的代码就会按照预期运行。
&gt;
&gt; 在条件检查周围使用 `while` 循环还能处理发生**虚假唤醒**的情况。在某些线程包中，由于实现的细节问题，可能会出现两个线程被唤醒的情况，尽管只发生了一个信号。虚假唤醒是重新检查线程正在等待的条件的进一步理由。

## 覆盖条件

现在我们再来看一个如何使用条件变量的例子。本代码研究摘自 Lampson 和 Redell 关于 Pilot 的论文 ，正是他们首次实现了上文所述的 Mesa 语义（他们使用的语言是 Mesa，因此得名）。

他们遇到的问题最好通过简单的示例来说明，这里的示例是一个简单的多线程内存分配库，代码如下所示。

```c
// how many bytes of the heap are free?
int bytesLeft = MAX_HEAP_SIZE;

// need lock and condition too
cond_t c;
mutex_t m;

void * allocate(int size) {
    Pthread_mutex_lock(&amp;m);
    while (bytesLeft &lt; size)
        Pthread_cond_wait(&amp;c, &amp;m);
    void *ptr = ...; // get mem from heap
    bytesLeft -= size;
    Pthread_mutex_unlock(&amp;m);
    return ptr;
}

void free(void *ptr, int size) {
    Pthread_mutex_lock(&amp;m);
    bytesLeft &#43;= size;
    Pthread_cond_signal(&amp;c); // Signal waiting threads
    Pthread_mutex_unlock(&amp;m);
}
```

正如你在代码中看到的，当线程调用内存分配代码时，可能需要等待更多内存被释放。反之，当线程释放内存时，就会发出更多内存可用的信号。然而，我们上面的代码有一个问题：哪个等待的线程（可能不止一个）应该被唤醒？

请考虑以下情况。假设空闲字节数为零；线程 $T_a$ 调用 `allocate(100)`，紧随其后的线程 $T_b$ 调用 `allocate(10)`，要求获得更少的内存。因此， $T_a$和 $T_b$ 都等待条件并进入休眠；没有足够的空闲字节来满足这两个请求。

这时，假设第三个线程 $T_c$调用 `free(50)`。不幸的是，当它调用 `signal` 来唤醒一个等待线程时，可能没有唤醒正确的等待线程 $T_b$，因为 $T_b$ 只等待释放 10 个字节；$T_a$ 应该继续等待，因为还没有足够的空闲内存。因此，上面的代码不起作用，因为唤醒其他线程的线程不知道该唤醒哪个（或哪些）线程。

Lampson 和 Redell 提出的解决方案非常简单：用调用 `pthread_cond_broadcast()` 代替上面代码中的 `pthread_cond_signal()`，唤醒所有等待的线程。这样，我们就能保证所有应该被唤醒的线程都被唤醒了。当然，这样做的缺点是可能会对性能产生负面影响，因为我们可能会不必要地唤醒许多其他不应该（尚未）被唤醒的等待线程。这些线程会简单地唤醒，重新检查条件，然后立即回到睡眠状态。

Lampson 和 Redell 将这种条件称为**覆盖条件**，因为它涵盖了（保守地）需要唤醒线程的所有情况；正如我们已经讨论过的，代价是可能会唤醒过多的线程。精明的读者可能也注意到了，我们本可以在更早的时候使用这种方法（参见只有一个条件变量的生产者/消费者问题）。不过，在这种情况下，我们有一个更好的解决方案，因此我们使用了它。一般来说，如果你发现只有当你将信号改为广播时，你的程序才能运行（但你认为它不需要这样），那么你可能遇到了一个错误；请修复它！但在类似上述内存分配器的情况下，广播可能是最直接的解决方案。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/23.%E6%9D%A1%E4%BB%B6%E5%8F%98%E9%87%8F/  

