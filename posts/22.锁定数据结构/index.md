# 锁定数据结构

在讨论锁之前，我们首先描述如何在一些常见的数据结构中使用锁。向数据结构添加锁以使其可由线程使用，从而使该结构成为&lt;font color=&#34;red&#34;&gt;线程安全&lt;/font&gt;的。如何加锁决定了数据结构的正确性和性能。因此，我们面临着关键挑战：当给定一个特定的数据结构时，我们应该如何向它添加锁，以使其正常工作？此外，我们如何添加锁以使数据结构产生高性能，使许多线程能够同时（即并发）访问该结构？

## 并发计数器

计数器是最简单的数据结构之一。它是一种常用的结构，具有简单的接口。下面代码中中定义了一个简单的非并发计数器。

```c
typedef struct __counter_t {
    int value;
} counter_t;

void init(counter_t *c) {
    c-&gt;value = 0;
}

void increment(counter_t *c) {
    c-&gt;value&#43;&#43;;
}

void decrement(counter_t *c) {
    c-&gt;value--;
}

int get(counter_t *c) {
    return c-&gt;value;
}
```

### 简单但不可扩展

正如您所看到的，非并发计数器是一个简单的数据结构，需要少量的代码来实现。现在我们面临下一个挑战：如何使这段代码线程安全？下面这段代码显示了我们如何做到这一点。 

```c
typedef struct __counter_t {
    int value;
    pthread_mutex_t lock;
} counter_t;

void init(counter_t *c) {
    c-&gt;value = 0;
    pthread_mutex_init(&amp;c-&gt;lock, NULL);
}

void increment(counter_t *c) {
    pthread_mutex_lock(&amp;c-&gt;lock);
    c-&gt;value&#43;&#43;;
    pthread_mutex_unlock(&amp;c-&gt;lock);
}

void decrement(counter_t *c) {
    pthread_mutex_lock(&amp;c-&gt;lock);
    c-&gt;value--;
    pthread_mutex_unlock(&amp;c-&gt;lock);
}

int get(counter_t *c) {
    pthread_mutex_lock(&amp;c-&gt;lock);
    int rc = c-&gt;value;
    pthread_mutex_unlock(&amp;c-&gt;lock);
    return rc;
}
```

这个并发计数器很简单并且工作正常。事实上，它遵循最简单和最基本的并发数据结构常见的设计模式：它只是添加一个锁，该锁在调用操作数据结构的例程时获取，并在从调用返回时释放。通过这种方式，它类似于使用监视器构建的数据结构，当您调用对象方法并从对象方法返回时，会自动获取和释放锁。

至此，你已经有了一个可以运行的并发数据结构。你可能会遇到的问题是性能。如果你的数据结构速度太慢，你需要做的就不仅仅是添加一个锁了；因此，如果需要进行此类优化，这将是本章其余部分的主题。需要注意的是，如果数据结构的运行速度不是太慢，那么你就大功告成了！如果简单的数据结构也能正常工作，那么就没必要做什么花哨的事情了。

为了了解简单方法的性能代价，我们运行了一个基准，其中每个线程更新单个共享计数器的次数是固定的；然后我们改变线程的数量。如下图所示，显示了在一到四个线程活动的情况下所花费的总时间；每个线程更新计数器 100 万次。本实验在配备四颗英特尔 2.7 GHz i5 CPU 的 iMac 上运行；如果激活的 CPU 越多，我们希望单位时间内完成的总工作量就越大。

![image-20240408143721333](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/Performance-Of-Traditional-vs-Approximate-Counters.png)

从图中最上面一行（标注为 &#34;Precise&#34;）可以看出，并发计数器的性能扩展性很差。单个线程可以在极短的时间内（大约 0.03 秒）完成百万次计数器更新，而让两个线程同时更新计数器 100 万次则会导致速度大幅下降（超过 5 秒！）。线程越多，情况越糟糕。

理想情况下，线程在多处理器上完成的速度要和单线程在单处理器上完成的速度一样快。实现这一目标被称为**完美扩展**；即使要完成更多工作，也是并行完成的，因此完成任务所需的时间不会增加。

### 可扩展计数器

令人惊讶的是，研究人员多年来一直在研究如何构建更具可扩展性的计数器。更令人惊叹的是，可扩展计数器的重要性，正如最近在操作系统性能分析方面的工作所显示的那样；如果没有可扩展计数，在 Linux 上运行的一些工作负载在多核机器上就会出现严重的可扩展性问题。

为了解决这个问题，人们开发了许多技术。我们将介绍一种称为**近似计数器**的方法 [C06]。

近似计数器的工作原理是通过众多本地物理计数器（每个 CPU 内核一个）和一个全局计数器来表示一个逻辑计数器。具体来说，在一台有四个 CPU 的机器上，有四个本地计数器和一个全局计数器。除了这些计数器外，还有锁：每个本地计数器和全局计数器各有一个锁。

近似计数器的基本思想如下。当运行在给定内核上的线程希望递增计数器时，它会递增其本地计数器；通过相应的本地锁同步访问该本地计数器。由于每个 CPU 都有自己的本地计数器，因此跨 CPU 的线程可以无竞争地更新本地计数器，因此计数器的更新是可扩展的。

不过，为了保持全局计数器的最新状态（以防线程希望读取其值），本地计数器的值会定期转移到全局计数器上，方法是获取全局锁，并根据本地计数器的值递增；然后将本地计数器重置为零。

这种从本地到全局的转移发生频率由阈值 S 确定。S 越小，计数器的行为就越像上述不可扩展的计数器；S 越大，计数器的可扩展性就越强，但全局值可能会偏离实际计数。我们可以简单地获取所有本地锁和全局锁（按指定顺序，以避免死锁）来获得精确值，但这是不可扩展的。

为了说明这一点，我们来看一个例子，如下图所示。

![image-20240408150033961](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/Tracing-The-Approximate-Counters.png)

在这个例子中，阈值 S 设置为 5，四个 CPU 上都有线程在更新本地计数器 L1 ... L4。全局计数器值 (G) 也显示在跟踪中，随着时间的推移不断向下增加。在每个时间步长，本地计数器都可能递增；如果本地值达到阈值 S，本地值就会转移到全局计数器，然后本地计数器被重置。

在上图中（标注为 &#34;Approximate&#34;）的下线显示了阈值 S 为 1024 的近似计数器的性能。该计数器的性能非常出色；在四个处理器上更新计数器 400 万次所需的时间几乎不超过在一个处理器上更新计数器 100 万次所需的时间。

下图显示了阈值 S 的重要性，四个线程在四个 CPU 上各递增计数器 100 万次。如果 S 值较低，则性能较差（但全局计数总是相当准确）；如果 S 值较高，则性能出色，但全局计数滞后（最多滞后 CPU 数量乘以 S）。近似计数器正是通过这种精度/性能权衡实现的。

![image-20240408150613853](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20240408150613853.png)

近似计数器的粗略版本如下面这段代码所示。

```c
typedef struct __counter_t {
    int global; // global count
    pthread_mutex_t glock; // global lock
    int local[NUMCPUS]; // local count (per cpu)
    pthread_mutex_t llock[NUMCPUS]; // ... and locks
    int threshold; // update frequency
} counter_t;

// init: record threshold, init locks, init values
// of all local counts and global count
void init(counter_t *c, int threshold) {
    c-&gt;threshold = threshold;
    c-&gt;global = 0;
    pthread_mutex_init(&amp;c-&gt;glock, NULL);
    int i;
    for (i = 0; i &lt; NUMCPUS; i&#43;&#43;) {
        c-&gt;local[i] = 0;
        pthread_mutex_init(&amp;c-&gt;llock[i], NULL);
    }
}

// update: usually, just grab local lock and update local amount
// once local count has risen by ’threshold’, grab global
// lock and transfer local values to it
void update(counter_t *c, int threadID, int amt) {
    int cpu = threadID % NUMCPUS;
    pthread_mutex_lock(&amp;c-&gt;llock[cpu]);
    c-&gt;local[cpu] &#43;= amt; // assumes amt &gt; 0
    if (c-&gt;local[cpu] &gt;= c-&gt;threshold) { // transfer to global
        pthread_mutex_lock(&amp;c-&gt;glock);
        c-&gt;global &#43;= c-&gt;local[cpu];
        pthread_mutex_unlock(&amp;c-&gt;glock);
        c-&gt;local[cpu] = 0;
    }
    pthread_mutex_unlock(&amp;c-&gt;llock[cpu]);
}

// get: just return global amount (which may not be perfect)
int get(counter_t *c) {
    pthread_mutex_lock(&amp;c-&gt;glock);
    int val = c-&gt;global;
    pthread_mutex_unlock(&amp;c-&gt;glock);
    return val; // only approximate!
}
```

## 并发链表

### 基本实现

接下来，我们要研究一种更复杂的结构—链表。让我们再次从基本方法开始。为了简单起见，我们将省略这种列表中的一些显而易见的例程，而只关注并发插入；至于查找、删除等，我们将留给读者自己去思考或者可以在仓库中找到更详细的版本。下面显示了这种初级数据结构的代码。

```c
// basic node structure
typedef struct __node_t {
    int key;
    struct __node_t *next;
} node_t;

// basic list structure (one used per list)
typedef struct __list_t {
    node_t *head;
    pthread_mutex_t lock;
} list_t;

void List_Init(list_t *L) {
    L-&gt;head = NULL;
    pthread_mutex_init(&amp;L-&gt;lock, NULL);
}

int List_Insert(list_t *L, int key, int thread_id) {
    pthread_mutex_lock(&amp;L-&gt;lock);
    printf(&#34;Thread %d: Inserting %d\n&#34;, thread_id, key);
    node_t *new = malloc(sizeof(node_t));
    if (new == NULL) {
        perror(&#34;malloc&#34;);
        pthread_mutex_unlock(&amp;L-&gt;lock);
        return -1; // fail
    }
    new-&gt;key = key;
    new-&gt;next = L-&gt;head;
    L-&gt;head = new;
    pthread_mutex_unlock(&amp;L-&gt;lock);
    return 0; // success
}
```

从代码中可以看出，代码只是在进入插入例程时获取一个锁，并在退出时释放它。如果 `malloc()` 恰好失败（这种情况很少见），就会出现一个棘手的小问题；在这种情况下，代码还必须在插入失败前释放锁。

事实证明，这种特殊的控制流非常容易出错；最近对 Linux 内核补丁的一项研究发现，很大一部分错误（近 40%）都是在这种很少使用的代码路径上发现的。因此，我们面临着一个挑战：我们能否重写插入和查找例程，使其在并发插入时保持正确，但避免失败路径也需要我们添加`unlock`调用的情况？

在这种情况下，答案是肯定的。具体来说，我们可以重构一下`List_Insert`代码，使锁定和释放只围绕插入代码中的实际临界区，并在删除代码中使用共同的退出路径。前者之所以有效，是因为部分查找代码实际上无需锁定；假设 `malloc()` 和`printf`本身是线程安全的，那么每个线程都可以调用它，而不必担心出现竞争条件或其他并发错误。只有在更新共享列表时才需要加锁。有关这些修改的详细信息，请见下面这段代码。

```
// basic node structure
typedef struct __node_t {
    int key;
    struct __node_t *next;
} node_t;

// basic list structure (one used per list)
typedef struct __list_t {
    node_t *head;
    pthread_mutex_t lock;
} list_t;

void List_Init(list_t *L) {
    L-&gt;head = NULL;
    pthread_mutex_init(&amp;L-&gt;lock, NULL);
}

int List_Insert(list_t *L, int key, int thread_id) {
    printf(&#34;Thread %d: Inserting %d\n&#34;, thread_id, key);
    node_t *new = malloc(sizeof(node_t));
    if (new == NULL) {
        perror(&#34;malloc&#34;);
        pthread_mutex_unlock(&amp;L-&gt;lock);
        return -1; // fail
    }
    new-&gt;key = key;
    pthread_mutex_lock(&amp;L-&gt;lock);
    new-&gt;next = L-&gt;head;
    L-&gt;head = new;
    pthread_mutex_unlock(&amp;L-&gt;lock);
    return 0; // success
}
```

### 扩展链表

虽然我们又有了一个基本的并发链表，但我们又一次遇到了它不能很好扩展的情况。为了在链表中实现更多并发性，研究人员探索了一种技术，即所谓的 &#34;手拉手锁定&#34;（又称 &#34;锁耦合&#34;）。

这个想法非常简单。你可以为链表的每个节点添加一个锁，而不是为整个链表添加一个锁。当遍历列表时，代码会先抓取下一个节点的锁，然后释放当前节点的锁（这就是 `hand-over-hand` 名称的由来）。

从概念上讲，&#34;交手 &#34;链表是有一定道理的；它可以实现高度并发的操作。然而，在实践中，这种结构很难比简单的单锁方法更快，因为为遍历链表的每个节点获取和释放锁的开销太大。即使是非常大的链表和大量的线程，允许多个正在进行的遍历所带来的并发性也不可能比简单地获取单锁、执行操作和释放锁更快。也许某种混合方式（每隔几个节点就抓取一个新锁）值得研究。

&gt; &lt;center&gt;TIP：并发越多不一定越快&lt;/center&gt;
&gt;
&gt; 如果您设计的方案增加了很多开销（例如，频繁获取和释放锁，而不是一次性），那么它更具并发性可能就不重要了。简单的方案往往效果良好，特别是如果它们很少使用昂贵的例程。增加更多锁和复杂性可能会导致失败。尽管如此，有一种真正需要了解的方法：构建两种替代方案（&lt;font color=&#34;red&#34;&gt;简单但并发性较低、复杂但并发性较高&lt;/font&gt;）并测量它们的表现。最终，您无法在性能上作弊；您的想法要么更快，要么不是。
&gt;
&gt; &lt;center&gt;警惕锁和控制流
&gt; &lt;/center&gt;
&gt;
&gt; 一个通用的设计提示，在并发代码以及其他地方都很有用，就是要警惕导致函数返回、退出或其他类似错误条件从而停止函数执行的控制流变化。因为许多函数会首先获取锁、分配一些内存或进行其他类似的有状态操作，&lt;font color=&#34;red&#34;&gt;当出现错误时，代码必须在返回之前撤销所有状态，这样容易出错。&lt;/font&gt;因此，最好结构化代码以最小化这种模式。

## 并发队列

正如您现在所知，制作并发数据结构总是有一个标准方法：添加一个大锁。对于队列，我们将跳过该方法，假设您可以弄清楚。

相反，我们将看一下由 Michael 和 Scott 设计的并发程度稍高的队列。代码如下所示。

```c
typedef struct __node_t {
    int value;
    struct __node_t *next;
} node_t;

typedef struct __queue_t {
    node_t *head;
    node_t *tail;
    pthread_mutex_t headLock; // Mutex for head pointer
    pthread_mutex_t tailLock; // Mutex for tail pointer
} queue_t;

// Initialize the queue
void Queue_Init(queue_t *q) {
    // Allocate memory for a dummy node to represent the head of the queue
    node_t *tmp = malloc(sizeof(node_t));
    tmp-&gt;next = NULL;
    // Initialize the head and tail pointers to the dummy node
    q-&gt;head = q-&gt;tail = tmp;
    // Initialize mutexes for head and tail pointers
    pthread_mutex_init(&amp;q-&gt;headLock, NULL);
    pthread_mutex_init(&amp;q-&gt;tailLock, NULL);
}

// Enqueue an element into the queue
void Queue_Enqueue(queue_t *q, int value) {
    // Allocate memory for the new node
    node_t *tmp = malloc(sizeof(node_t));
    assert(tmp != NULL);
    tmp-&gt;value = value;
    tmp-&gt;next = NULL;

    // Acquire the tailLock mutex
    pthread_mutex_lock(&amp;q-&gt;tailLock);
    // Insert the new node after the current tail node
    q-&gt;tail-&gt;next = tmp;
    // Update the tail pointer to point to the new node
    q-&gt;tail = tmp;
    // Release the tailLock mutex
    pthread_mutex_unlock(&amp;q-&gt;tailLock);
}

// Dequeue an element from the queue
int Queue_Dequeue(queue_t *q, int *value) {
    // Acquire the headLock mutex
    pthread_mutex_lock(&amp;q-&gt;headLock);
    // Get the current head node
    node_t *tmp = q-&gt;head;
    // Get the next node after the head
    node_t *newHead = tmp-&gt;next;
    // If the queue is empty (newHead is NULL), release the headLock mutex and return -1
    if (newHead == NULL) {
        pthread_mutex_unlock(&amp;q-&gt;headLock);
        return -1; // queue was empty
    }
    // Extract the value from the node to be dequeued
    *value = newHead-&gt;value;
    // Update the head pointer to point to the next node
    q-&gt;head = newHead;
    // Release the headLock mutex
    pthread_mutex_unlock(&amp;q-&gt;headLock);
    // Free the memory of the dequeued node
    free(tmp);
    // Return 0 indicating successful dequeue operation
    return 0;
}
```

如果你仔细研究这段代码，你会发现有两个锁，一个用于队列的头部，一个用于队列的尾部。这两个锁的目标是实现入队和出队操作的并发。在常见情况下，入队例程将仅访问尾部锁，而出队只会访问头部锁。 

Michael 和 Scott 使用的一个技巧是添加一个虚拟节点（在队列初始化代码中分配）；这个虚拟节点可以实现头尾操作的分离。

队列通常用在多线程应用程序中。然而，这里使用的队列类型（仅带有锁）通常不能完全满足此类程序的需求。一个更充分开发的有界队列，使线程能够在队列为空或过满时等待，这是条件变量可以做到的事情。

## 并发哈希表

最后，我们将讨论一种简单而又广泛适用的并发数据结构—哈希表。我们将重点讨论一个不调整大小的简单哈希表，代码如下所示；处理调整大小需要做更多的工作，我们将其作为一个练习留给读者（抱歉！）。

```c
#define BUCKETS (101)

typedef struct __hash_t {
    list_t lists[BUCKETS];
} hash_t;

void Hash_Init(hash_t *H) {
    int i;
    for (i = 0; i &lt; BUCKETS; i&#43;&#43;) {
        List_Init(&amp;H-&gt;lists[i]);
    }
}

int Hash_Insert(hash_t *H, int key, int thread_id) {
    int bucket = key % BUCKETS;
    return List_Insert(&amp;H-&gt;lists[bucket], key, thread_id);
}

int Hash_Remove(hash_t *H, int key, int thread_id) {
    int bucket = key % BUCKETS;
    return List_Remove(&amp;H-&gt;lists[bucket], key, thread_id);
}
```

这个并发哈希表非常简单，使用我们之前开发的并发链表构建，而且运行得非常好。它之所以性能出色，是因为它没有为整个结构设置一个锁，而是为每个哈希桶（每个哈希桶由一个链表表示）设置了一个锁。这样就可以进行许多并发操作。

下图 显示了哈希表在并发更新下的性能（在同一台配备四个 CPU 的 iMac 电脑上，四个线程的并发更新次数从 10,000 次到 50,000 次不等）。为便于比较，图中还显示了链表的性能（使用单锁）。从图中可以看出，这个简单的并发哈希表的扩展能力很强，而链表则不然。

![image-20240408230513598](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/Scaling-Hash-Tables.png)

&gt; &lt;center&gt;避免过早优化（KNUTH定律）&lt;/center&gt;
&gt;
&gt; 在构建并发数据结构时，应从最基本的方法开始，即添加一个大锁以提供同步访问。这样做，你就有可能构建一个正确的锁；如果你发现它存在性能问题，你可以对它进行改进，从而在必要时使它变得更快。正如 Knuth 的名言：&#34;过早优化是万恶之源&#34;。
&gt;
&gt; 在向多处理器过渡之初，许多操作系统都使用单锁，包括 Sun OS 和 Linux。在后者中，这种锁甚至有一个名字，即&lt;font color=&#34;red&#34;&gt;大内核锁（BKL&lt;/font&gt;）。多年来，这种简单的方法一直很好，但当多 CPU 系统成为常态时，内核中每次只允许一个活动线程就成了性能瓶颈。因此，终于到了为这些系统添加改进并发性优化的时候了。在 Linux 系统中，采用了更直接的方法：用多个锁代替一个锁。而在 Sun 内部，则做出了一个更激进的决定：建立一个全新的操作系统，即 Solaris，从一开始就从根本上融入并发性。


---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/22.%E9%94%81%E5%AE%9A%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/  

