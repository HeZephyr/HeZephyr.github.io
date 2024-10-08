# Linux Execve函数详解


## 基本介绍

```c
#include &lt;unistd.h&gt;
int execve(const char *pathname, char *const argv[], char *const envp[]);
```

* 描述

	&gt; `execve()`执行由`pathname`引用的程序。这会导致当前由调用进程运行的程序被一个新程序替换，该新程序具有新初始化的堆栈、堆和（已初始化和未初始化）数据段。
	&gt;
	&gt; `pathname`必须是二进制可执行文件或以形式为`#!interpreter [optional-arg]`开头的脚本。
	&gt;
	&gt; `argv`是传递给新程序作为其命令行参数的字符串指针数组。按照惯例，这些字符串中第一个（即argv[0]）应包含与正在执行文件相关联的文件名。&lt;font color=&#34;red&#34;&gt;argv数组必须以NULL指针结尾&lt;/font&gt;。（因此，在新程序中，argv[argc]将为NULL）。
	&gt;
	&gt; `envp`是传递给新程序环境变量的字符串指针数组。该数组包含了环境变量。每个环境变量都是一个 `char*` 指针，格式为 &#34;name=value&#34;。envp数组同样必须以NULL指针结尾。
	&gt;
	&gt; ```
	&gt; char *envp[] = {
	&gt;  &#34;PATH=/bin&#34;,
	&gt;  &#34;HOME=/home/user&#34;,
	&gt;  &#34;USER=user&#34;,
	&gt;  NULL // 终止环境变量数组
	&gt; };
	&gt; ```
	&gt;
	&gt; `argv`和`envp`可以从新程序的主函数访问。例如我们编写的C程序，实际上是由操作系统通过`execve()`系统调用执行（这里是操作系统先执行`fork`系统调用，创建一个新的子进程，然后在新的子进程中，操作系统执行`execve()`系统调用），它会传递这些参数给新程序的主函数，即 main 函数。这些参数定义了新程序执行时的环境和命令行参数，在程序启动时由操作系统设置，并在整个程序执行期间保持不变。这使得程序能够根据传递给它的参数和环境变量来执行不同的任务或调整其行为。

* 返回值

	&gt; 成功时，`execve()` 不返回任何值，当 `execve` 成功替换当前进程的映像并开始执行新的程序时，原来的进程（即调用 execve 的进程）已经不再存在，因此无法返回任何值。
	&gt;
	&gt; 在出错时返回 -1，并设置适当的 `errno`。

* 重点

	&gt; 1. `execve`实际上就是将当前运行的状态机重置成另一个程序的初始状态
	&gt; 2. 允许对新状态机设置参数 `argv` (v) 和环境变量 `envp` (e)
	&gt; 3. 在程序启动时，操作系统首先执行 fork 系统调用，创建一个新的子进程。然后，操作系统在子进程中执行 execve 系统调用，以替换子进程的程序映像并开始执行新的程序。原来的父进程继续执行 fork 之后的代码。
	&gt; 4. 在调用 execve 之前，确保释放所有不再需要的资源，如打开的文件描述符、锁等。
	&gt; 5. 在调用 execve 之前，确保子进程已经处理了所有待处理的信号，除非你希望信号处理程序在新程序中执行。
	&gt; 6. 如果 execve 失败，子进程通常应该终止。
	&gt; 7. 在父进程中，通常会在 fork 之后立即调用 wait 或 waitpid 来等待子进程结束，以确保父进程不会过早退出，从而导致子进程的僵尸进程。

## execve实例

### 自定义argv和envp

```c
#include &lt;unistd.h&gt;
#include &lt;stdio.h&gt;

int main() {
  char * const argv[] = {
    &#34;/bin/bash&#34;, &#34;-c&#34;, &#34;env&#34;, NULL,
  };
  char * const envp[] = {
    &#34;HELLO=WORLD&#34;, NULL,
  };
  execve(argv[0], argv, envp);
  printf(&#34;Hello, World!\n&#34;);
  return 0;
}
```

在这段代码中，我们显式的设置了`argv`和`envp`，其中参数 `&#34;/bin/bash&#34;, &#34;-c&#34;, &#34;env&#34;, NULL`，这里的参数实际上是在告诉 `bash` 执行一个命令（由 `-c` 后面的字符串指定），在这个例子中是 `env`，它打印当前的环境变量。

我们运行代码，得到如下输出：

![image-20240313091345219](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20240313091345219.png)

如果我们不传 `-c` 参数和随后的命令，即只传入 `&#34;/bin/bash&#34;, NULL` 作为参数，`bash` 会默认进入交互式模式。在这种模式下，它不会执行任何命令并立即退出，而是会等待用户输入，表现为进入了 shell 环境。

我们发现，打印的当前环境变量除了自定的`envp`，还有一些其他的输出。这是因为除了我们设定的环境变量外，还有一些系统或者 shell 默认的环境变量会被添加到新进程中，例如 `PWD` 表示当前工作目录，`SHLVL` 表示 shell 层级，`_` 是上一个执行的命令。这就是为什么我们会看到额外的环境变量出现在输出中。

### fork后再通过子进程执行execve

```c
#include &lt;stdio.h&gt;
#include &lt;sys/types.h&gt;
#include &lt;sys/wait.h&gt;

int main() {
    pid_t pid = fork();

    if (pid == 0) {
        // 子进程
        char * const argv[] = {&#34;/bin/echo&#34;, &#34;Hello, World!&#34;, NULL};
        char * const envp[] = {NULL};
        execve(&#34;/bin/echo&#34;, argv, envp);
    } else if (pid &gt; 0) {
        // 父进程
        wait(NULL); // 等待子进程结束
        printf(&#34;Child process finished.\n&#34;);
    } else {
        // fork失败
        perror(&#34;fork&#34;);
        return 1;
    }
    return 0;
}
```

这段代码演示了如何使用 `fork()` 系统调用创建一个新的子进程，然后在子进程中执行 `execve()` 系统调用。这是在 Unix-like 系统中常见的操作模式，因为 `execve()` 系统调用有一些关键的限制：

* 一次机会：`execve()` 系统调用只能用于替换当前进程的映像一次。如果一个进程已经调用了 `execve()`，它就不能再调用 `fork()` 或再次执行 `execve()`。
* 无返回值：`execve()` 成功执行时，原来的进程映像被新程序映像替换，原来的进程不再存在，因此无法返回任何值。如果在 `execve()` 执行之前有任何返回值，那么这个返回值是在 `fork()` 调用之后，由父进程获得的。

因此，在实际应用中，我们通常会先使用 `fork()` 创建一个子进程，然后在子进程中调用 `execve()` 执行新的程序。父进程在 `fork()` 之后会继续执行，并通过调用 `wait(NULL)` 来等待子进程结束。这样，父进程可以知道子进程已经成功执行了 `execve()`，并且可以继续执行其他任务或退出。

在多线程程序中，如果一个线程执行了 `fork()` 并尝试在子进程中执行 `execve()`，那么其他线程将继续执行，不受 `fork()` 和 `execve()` 的影响。只有调用 `fork()` 的线程会进入子进程，而其他线程则继续在父进程中运行。

得到的运行结果：

![image-20240313102827423](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/image-20240313102827423.png)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/05.linux-execve%E5%87%BD%E6%95%B0%E8%AF%A6%E8%A7%A3/  

