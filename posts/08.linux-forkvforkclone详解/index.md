# Linux 系统调用函数fork、vfork、clone详解

## fork

### 基本介绍

```c
#include &lt;sys/types.h&gt;
#include &lt;unistd.h&gt;

pid_t fork(void)
```

* 描述

	&gt; fork用于创建一个子进程，它与父进程的唯一区别在于其PID和PPID，以及资源利用设置为0。&lt;font color=&#34;red&#34;&gt;文件锁和挂起信号&lt;/font&gt;（指已经被内核发送给一个进程，但尚未被该进程处理的信号）不会被继承，其他和父进程几乎完全相同：&lt;font color=&#34;red&#34;&gt;会获得父进程的内存空间、栈、数据段、堆、打开的文件描述符、信号处理函数、进程优先级、环境变量等资源的副本。&lt;/font&gt;

* 返回值

	&gt; 成功时，在父进程中返回子进程的 PID，在子进程中返回 $0$。失败时，父进程返回 $-1$，不创建子进程，并适当设置 errno。
	&gt;
	&gt; 其中errno是一个全局变量，它用于表示最近一次系统调用或库函数调用产生的错误代码。当系统调用或库函数失败时，它们通常会设置 errno 以指示错误的原因。
	&gt;
	&gt; 以下是一些常见的 errno 错误代码及其含义：
	&gt;
	&gt; * EAGAIN：资源暂时不可用，通常是因为达到了系统限制，如文件描述符或内存限制。
	&gt; * ENOMEM：内存不足，无法分配请求的资源。
	&gt; * EACCES：权限不足，无法访问某个资源。
	&gt; * EINTR：系统调用被信号中断。
	&gt; * EINVAL：无效的参数。

* 重点

	&gt; fork() 函数创建的子进程会从父进程复制执行顺序。具体来说，子进程会从父进程复制当前的执行上下文，包括指令指针（instruction pointer）和寄存器的状态。这意味着子进程将从 fork() 系统调用之后的指令开始执行，与父进程在 fork() 之后应该执行的指令完全相同。&lt;font color=&#34;red&#34;&gt;因此，fork() 之后通常会有一个基于返回值的分支结构，以区分父进程和子进程的执行路径。&lt;/font&gt;

### fork实例

####.1多个fork返回值

```c
#include &lt;stdio.h&gt;
#include &lt;unistd.h&gt;

int main() {
    pid_t pid1 = fork();
    pid_t pid2 = fork();
    pid_t pid3 = fork();
    pid_t pid  = getpid();
    printf(&#34;The PID of the current process is %d\n Hello World from (%d, %d, %d)\n&#34;, pid, pid1, pid2, pid3);
    return 0;
}
```

这段程序包含了三个 fork() 调用，每个 fork() 都会创建一个新的子进程。由于每次 fork() 调用都会导致进程数翻倍，所以总共会有$2^3=8$个进程 （包括最初的父进程）。每个进程都会打印出它的进程 ID (pid) 以及三个 fork() 调用的返回值 (pid1, pid2, pid3)。

得到的输出结果如下：

![image-20240312193151556](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20240312193151556.png)

我们画个状态机来理解它们的输出，假设最初的父进程PID为291871：

![fork_information](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/fork_information.png)

#### C语言 fork与输出

```c
#include &lt;stdio.h&gt;
#include &lt;unistd.h&gt;
#include &lt;sys/wait.h&gt;

int main(int argc, char *argv[]) {
  int n = 2;
  for (int i = 0; i &lt; n; i&#43;&#43;) {
    fork();
    printf(&#34;Hello\n&#34;);
  }
  for (int i = 0; i &lt; n; i&#43;&#43;) {
    wait(NULL);
  }
}
```

这段代码中，按我们的理解，第一次fork后有2个进程，然后一起执行printf输出，得到两个`Hello`，然后第二次fork后有4个进程，然后执行printf，得到四个`Hello`，则会有6个``Hello`，如下：

![image-20240312200038027](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20240312200038027.png)

但是当我们将输出通过管道传给`cat`等命令时，会看到8个`Hello`：

![image-20240312200714610](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20240312200714610.png)

这是因为标准输出一般是行缓冲的，碰到`\n`，缓冲区中的内容会被刷新，即输出到终端或文件中。这种缓冲方式的目的是为了提高效率，因为这样可以减少对磁盘 I/O 的调用次数。

如果标准输出被重定向到管道，它可能不再是行缓冲的，而是变为全缓冲的。这意味着缓冲区可能会在填满时刷新，而不是在每次遇到换行符时刷新。如果缓冲区足够大，以至于可以容纳所有的 `Hello` 输出，那么fork的时候子进程也会复制缓冲区，导致最后每个进程中的缓冲区都有2个`Hello`，最后输出为8个。

如果为了确保缓冲区在需要的时候被刷新，可以在 printf 调用之后显式地调用 `fflush(stdout)` 来刷新标准输出缓冲区。这样可以确保所有的输出都被立即写入，而不会受到缓冲行为的影响。

```c
#include &lt;stdio.h&gt;
#include &lt;unistd.h&gt;
#include &lt;sys/wait.h&gt;

int main(int argc, char *argv[]) {
  int n = 2;
  for (int i = 0; i &lt; n; i&#43;&#43;) {
    fork();
    printf(&#34;Hello\n&#34;);
    fflush(stdout);
  }
  for (int i = 0; i &lt; n; i&#43;&#43;) {
    wait(NULL);
  }
  return 0;
}
```

![image-20240312201140424](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20240312201140424.png)

#### fork 💣

```c
#include &lt;stdio.h&gt;
#include &lt;unistd.h&gt;
#include &lt;sys/wait.h&gt;

int main(int argc, char *argv[]) {
  while(1) {
      fork();
  }
  return 0;
}
```

这段代码会无限循环地调用 fork() 函数，每次循环都会创建一个新进程。由于每次 fork() 调用都会成功创建一个新进程，而且这个新进程又会立即进入下一次循环并再次调用 fork()，因此进程的数量会以指数速度增长，很快就会耗尽系统的可用资源。

&lt;font color=&#34;red&#34;&gt;绝对不要在任何生产环境或您没有权限的任何系统上运行fork炸弹。&lt;/font&gt;

## vfork

### 基本介绍

* 描述

	```c
	#include &lt;sys/types.h&gt;
	#include &lt;unistd.h&gt;
	
	pid_t vfork(void);
	```

	`vfork()` 系统调用用于创建一个子进程，与 `fork()` 类似，但它使用父进程的地址空间，而不是复制父进程的地址空间。&lt;font color=&#34;red&#34;&gt;`vfork()` 调用后，父进程会阻塞，直到子进程调用 `exec` 函数或执行了 exit 函数。&lt;/font&gt;这是因为子进程需要独占父进程的地址空间，以确保数据一致性。一。&lt;font color=&#34;red&#34;&gt;在子进程调用 `exec` 函数或执行了 `exit` 函数之后，子进程将获得自己的内存空间。&lt;/font&gt;

* 返回值

	和`fork`一致

* 重点

	&gt; 1. `vfork()` 创建的子进程会继承父进程的环境，但不会继承父进程的堆栈。
	&gt; 2. 在子进程执行这些`exec`或`exit`操作之前，父进程和子进程可能会访问相同的内存地址，这可能导致数据竞争和不一致。
	&gt; 3. 在 `vfork()` 调用成功后，子进程应该立即调用 `exec` 函数或执行 `exit` 函数。如果在子进程中修改除了用于存储从 `vfork()` 返回值的 `pid_t` 类型变量之外的任何数据，或者从调用 `vfork()` 的函数返回，或在成功调用 `_exit()` 或 `exec()` 函数族中的一个函数之前调用其他任何函数，则行为是未定义的。这可能会导致程序崩溃或表现出不可预测的行为。
	&gt; 	因此，使用 `vfork()` 时，必须确保子进程在调用 `exec` 函数或执行 `exit` 函数之前不执行任何可能影响共享内存的操作。
	&gt; 4. `vfork()` 系统调用会阻塞父进程，直到子进程完成 `exec` 调用或 `exit` 调用。父进程不需要显式调用 `wait()` 或 `waitpid()` 来等待子进程结束。

### 验证vfork共享内存

```c
#include &lt;stdio.h&gt;
#include &lt;stdlib.h&gt;
#include &lt;unistd.h&gt;
#include &lt;sys/wait.h&gt;
#include &lt;string.h&gt;
#include &lt;assert.h&gt;

int main() {
    // 在父进程中分配内存并初始化
    char *buffer = malloc(1024);
    assert(buffer != NULL);
    memset(buffer, &#39;A&#39;, 1024);

    // 使用vfork创建子进程
    pid_t pid = vfork();
    if (pid &lt; 0) {
        perror(&#34;vfork error&#34;);
        exit(EXIT_FAILURE);
    } else if (pid == 0) {
        // 子进程
        printf(&#34;Child process: PID = %d\n&#34;, getpid());

        // 修改内容
        memset(buffer, &#39;B&#39;, 1024);
        // 子进程
        char * const argv[] = {&#34;/bin/echo&#34;, &#34;Hello, Linux!&#34;, NULL};
        char * const envp[] = {NULL};
        // 执行exec函数
        execve(argv[0], argv, envp);
    } else {
        // 父进程
        printf(&#34;Parent process: PID = %d, child&#39;s PID = %d\n&#34;, getpid(), pid);
        // 验证内存内容是否被子进程修改
        for (int i = 0; i &lt; 1024; i&#43;&#43;) {
            if (buffer[i] != &#39;B&#39;) {
                printf(&#34;Memory corruption detected at index %d\n&#34;, i);
                exit(EXIT_FAILURE);
            }
        }
        printf(&#34;Memory is consistent\n&#34;);
    }
    return 0;
}
```

这个程序的目的是验证在 `vfork()` 之后，子进程和父进程是否共享内存。首先在父进程中分配一块内存 ，并将其初始化为字符` ‘A’`。然后，父进程调用 `vfork()` 创建一个子进程。在子进程中，程序试图将内容修改为字符 `‘B’`，并执行 `execve()`。在父进程中，程序检查缓冲区的内容是否被修改为字符 `‘B’`，以验证内存是否被正确共享。

程序运行结果如下：

![image-20240313132239776](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20240313132239776.png)

## clone

### 基本介绍

* 描述

	```c
	#define _GNU_SOURCE
	#include &lt;sched.h&gt;
	int clone(int (*fn)(void *), void *child_stack, int flags, void *arg, ...
	                 /* pid_t *parent_tid, void *tls, pid_t *child_tid */ );
	```

	`clone`与`fork`类似，是用于创建新进程的系统调用，但`clone`提供了更精确的控制，可以确定在调用进程（父进程）和子进程之间共享哪些执行上下文的部分。例如，调用者可以控制两个进程是否共享虚拟地址空间、文件描述符表和信号处理程序表。这些系统调用还允许将新的子进程放置在单独的命名空间中。

* 参数

	&gt; * `fn`是指向新进程要执行的函数的指针，这个函数接受一个 void* 参数，并返回一个 int 类型的值，这个返回值将被 clone 系统调用捕获，并作为子进程的退出状态；
	&gt;
	&gt; * `child_stack`是新进程的堆栈地址，由于子进程和调用进程可能共享内存，因此子进程不可能与调用进程在同一堆栈中执行。调用进程必须为子堆栈设置内存空间，并将指向该空间的指针传递给`clone()`。
	&gt;
	&gt; * `flags`可以设置新进程的属性（通过二进制位设置），包括是否与原进程共享地址空间（CLONE_VM）、是否共享文件描述符表（CLONE_FILES）、是否共享信号处理器（CLONE_SIGHAND）等等；
	&gt;
	&gt; 	`int flags = CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGHAND | CLONE_THREAD | CLONE_SYSVSEM | CLONE_SETTLS;`
	&gt;
	&gt; 	|     标志      |                             含义                             |
	&gt; 	| :-----------: | :----------------------------------------------------------: |
	&gt; 	| CLONE_PARENT  | 创建的子进程的父进程是调用者的父进程，新进程与创建它的进程成了“兄弟”而不是“父子” |
	&gt; 	|   CLONE_FS    | 子进程与父进程共享相同的文件系统，包括root、当前目录、umask  |
	&gt; 	|  CLONE_FILES  |   子进程与父进程共享相同的文件描述符（file descriptor）表    |
	&gt; 	|  CLONE_NEWNS  | 在新的namespace启动子进程，namespace描述了进程的文件hierarchy |
	&gt; 	| CLONE_SIGHAND |     子进程与父进程共享相同的信号处理（signal handler）表     |
	&gt; 	| CLONE_PTRACE  |               若父进程被trace，子进程也被trace               |
	&gt; 	|  CLONE_VFORK  |           父进程被挂起，直至子进程释放虚拟内存资源           |
	&gt; 	|   CLONE_VM    |              子进程与父进程运行于相同的内存空间              |
	&gt; 	|   CLONE_PID   |                子进程在创建时PID与父进程一致                 |
	&gt; 	| CLONE_THREAD  | Linux 2.4中增加以支持POSIX线程标准，子进程与父进程共享相同的线程群 |
	&gt;
	&gt; * `arg`是传递给新进程的参数；
	&gt;
	&gt; * 可选参数，包括 `pid_t *parent_tid`等。

* 返回值

	&gt; 成功时，在父进程中返回子进程的 PID。失败时，父进程返回 $-1$，不创建子进程，并适当设置 `errno`。

* 重点

	&gt; 1. `clone` 可以创建新的进程或线程，Linux创建线程使用的系统调用就是`clone`。而 `fork` 和`vfork`只能创建进程。这意味着 `clone` 可以在单个进程中创建多个线程，而 `fork` 则总是创建一个新的进程。
	&gt; 2. `clone` 提供比 `fork` 和 `vfork` 更多的选项，可以指定子进程或线程的堆栈、信号处理、权限等。
	&gt; 3. `clone` 的使用比 `fork` 和 `vfork` 更复杂，需要正确设置 flags、child_stack、parent_pidptr、ptr、stack_size 和 tls 等参数。

### clone使用

```c
#define _GNU_SOURCE
#include &lt;sched.h&gt;
#include &lt;stdio.h&gt;
#include &lt;stdlib.h&gt;
#include &lt;string.h&gt;
#include &lt;sys/wait.h&gt;
#include &lt;unistd.h&gt;

#define STACK_SIZE (1024 * 1024) /* Stack size for cloned child */
// 宏，简化错误处理
#define ERREXIT(msg) { perror(msg); exit(EXIT_FAILURE); }
// 安全分配内存函数，分配失败报告错误
#define CHECKALLOC(ptr, msg)  ({ void *p = ptr; if (p == NULL) {ERREXIT(msg);} p;})

/*
 * 子进程函数
 * params: 接受一个void *类型参数，但是没有被使用过，后面的声明是用于告诉编译器这个参数是未被使用的
 */
static int childFunc(void *arg __attribute__((unused))) {
    puts(&#34;child: start&#34;);
    sleep(2);
    puts(&#34;child: terminate&#34;);
    return 0; /* Child terminates now */

}

int main(int argc, char *argv[]) {
    /* Start of stack buffer */
    char **stacks;
    /* Child process&#39;s pids */
    pid_t *pids;
    size_t nproc, i;

    // 接受两个参数
    if (argc != 2) {
        puts(&#34;Wrong way to execute the program:\n&#34;
                &#34;\t\t./waitpid nProcesses\n&#34;
                &#34;example:\t./waitpid 2&#34;);

        return EXIT_FAILURE;

    }
    // 初始化nproc，表示要创建的子进程数
    nproc = atol(argv[1]);  /* Process count */

    // 分配内存空间
    stacks = CHECKALLOC(malloc(nproc * sizeof(void *)), &#34;malloc&#34;);
    pids = CHECKALLOC(malloc(nproc * sizeof(pid_t)), &#34;malloc&#34;);

    for (i = 0; i &lt; nproc; i&#43;&#43;) {
        char *stackTop; /* End of stack buffer */
        stacks[i] = CHECKALLOC(malloc(STACK_SIZE), &#34;stack malloc&#34;);
        // 得到栈顶位置
        stackTop = stacks[i] &#43; STACK_SIZE;

        /*
         * 创建子进程
         * 第一个标志表示在子进程清除线程组ID（TID），目的是为了避免子进程与父进程或其他子进程的线程组ID冲突
         * 第二个表示告诉在子进程中设置线程ID，目的是为了允许父进程在子进程中追踪线程
         * 告诉 clone 系统调用在子进程中重新安装信号处理程序，目的是为了允许子进程捕获和处理信号，而不是传递给父进程。
         */
        pids[i] = clone(childFunc, stackTop, CLONE_CHILD_CLEARTID | CLONE_CHILD_SETTID | SIGCHLD, NULL);
        if (pids[i] == -1)
            ERREXIT(&#34;clone&#34;);
        printf(&#34;clone() returned %ld\n&#34;, (long)pids[i]);

    }

    sleep(1);

    // 等待所有子进程
    for (i = 0; i &lt; nproc; i&#43;&#43;) {
        // 第一个参数为子进程id，第二个参数表示不关心子进程的退出状态，第三个参数表示等待任何子进程
        if (waitpid(pids[i], NULL, 0) == -1)
            ERREXIT(&#34;waitpid&#34;);
        printf(&#34;child %ld has terminated\n&#34;, (long)pids[i]);

    }

    // 回收内存空间
    for (i = 0; i &lt; nproc; i&#43;&#43;)
        free(stacks[i]);
    free(stacks);
    free(pids);
    return EXIT_SUCCESS;

}
```

运行：`gcc clone-example.c &amp;&amp; ./a.out 5`，其中5为`nproc`，表示要创建的进程数。

运行结果如下：

![image-20240313212733984](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20240313212733984.png)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/08.linux-forkvforkclone%E8%AF%A6%E8%A7%A3/  

