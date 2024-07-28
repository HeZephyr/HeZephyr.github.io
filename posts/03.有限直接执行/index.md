# 有限直接执行

[github代码](https://github.com/unique-pure/ostep/blob/main/Virtualization/03.Mechanism%3A%20Limited%20Direct%20Execution/README.md)
在本作业中，您将测量系统调用和上下文切换的成本。测量系统调用的成本相对容易。例如，您可以反复调用一个简单的系统调用（如执行 0 字节读取），并计算所需时间；将时间除以迭代次数，就能估算出系统调用的成本。
您必须考虑的一件事是计时器的精度和准确性。可以使用的典型计时器是 gettimeofday()；阅读手册页了解详细信息。您将看到 gettimeofday() 返回 1970 年以来的时间（以微秒为单位）；然而，这并不意味着计时器精确到微秒。测量对 gettimeofday() 的连续调用，以了解计时器的精确度；这将告诉您必须运行多少次空系统调用测试迭代才能获得良好的测量结果。如果 gettimeofday() 对您来说不够精确，您可以考虑使用 x86 机器上可用的 rdtsc 指令。
测量上下文切换的成本比较麻烦。lmbench 基准测试的方法是在单个 CPU 上运行两个进程，并在它们之间设置两个 UNIX 管道；管道只是 UNIX 系统中进程相互通信的众多方式之一。第一个进程向第一个管道发出写操作，并等待第二个管道的读操作；当看到第一个进程在等待从第二个管道读取数据时，操作系统会将第一个进程置于阻塞状态，并切换到另一个进程，后者从第一个管道读取数据，然后向第二个管道写操作。当第二个进程再次尝试从第一个管道读取数据时，它就会阻塞，这样来回循环的通信就继续进行。通过测量这样反复通信的成本，lmbench 可以很好地估算出上下文切换的成本。您可以尝试使用管道或其他通信机制（如 UNIX 套接字）在这里重新创建类似的功能。
在拥有多个 CPU 的系统中，测量上下文切换成本是个难题；在这样的系统中，你需要做的是确保上下文切换进程位于同一个处理器上。幸运的是，大多数操作系统都有将进程绑定到特定处理器的调用；例如，在 Linux 系统中，调用 sched_setaffinity() 就是你要找的。通过确保两个进程位于同一处理器上，就能确保衡量操作系统在同一 CPU 上停止一个进程并恢复另一个进程的成本。
```c
#include &lt;stdio.h&gt;
#include &lt;stdlib.h&gt;
#include &lt;unistd.h&gt;
#include &lt;sys/time.h&gt;

#define ITERATIONS 1000

// Measure the cost of a system call
void measure_system_call() {
    struct timeval start, end;
    gettimeofday(&amp;start, NULL);
    for (int i = 0; i &lt; ITERATIONS; i&#43;&#43;) {
        read(0, NULL, 0);  // Example of a simple system call
    }
    gettimeofday(&amp;end, NULL);
    double time_per_call = ((double)(end.tv_sec - start.tv_sec) * 1000000 &#43; (end.tv_usec - start.tv_usec)) / ITERATIONS;
    printf(&#34;Estimated cost of a system call: %f microseconds\n&#34;, time_per_call);
}

// Measure the precision of gettimeofday()
void measure_gettimeofday_precision() {
    struct timeval start, end;
    gettimeofday(&amp;start, NULL);
    for (int i = 0; i &lt; ITERATIONS; i&#43;&#43;) {
        gettimeofday(&amp;end, NULL);
    }
    double precision = ((double)(end.tv_sec - start.tv_sec) * 1000000 &#43; (end.tv_usec - start.tv_usec)) / ITERATIONS;
    printf(&#34;Precision of gettimeofday(): %f microseconds\n&#34;, precision);
}

// Measure the cost of a context switch
void measure_context_switch() {
    int pipefd[2];
    if (pipe(pipefd) == -1) {
        perror(&#34;pipe&#34;);
        exit(EXIT_FAILURE);
    }

    pid_t pid = fork();
    if (pid == -1) {
        perror(&#34;fork&#34;);
        exit(EXIT_FAILURE);
    }

    if (pid == 0) {
        // Child process
        close(pipefd[1]);
        struct timeval start, end;
        gettimeofday(&amp;start, NULL);
        read(pipefd[0], NULL, 0);
        gettimeofday(&amp;end, NULL);
        double time_diff = (end.tv_sec - start.tv_sec) * 1000000 &#43; (end.tv_usec - start.tv_usec);
        printf(&#34;Child process context switch time: %f microseconds\n&#34;, time_diff);
        exit(EXIT_SUCCESS);
    } else {
        // Parent process
        close(pipefd[0]);
        struct timeval start, end;
        gettimeofday(&amp;start, NULL);
        write(pipefd[1], &#34;&#34;, 1);
        wait(NULL);
        gettimeofday(&amp;end, NULL);
        double time_diff = (end.tv_sec - start.tv_sec) * 1000000 &#43; (end.tv_usec - start.tv_usec);
        printf(&#34;Parent process context switch time: %f microseconds\n&#34;, time_diff);
    }
}

int main() {
    measure_system_call();
    measure_gettimeofday_precision();
    measure_context_switch();
    return 0;
}

```
&gt; 输出结果如下：
&gt; Estimated cost of a system call: 0.333000 microseconds
Precision of gettimeofday(): 0.014000 microseconds
Child process context switch time: 1.000000 microseconds
Parent process context switch time: 160.000000 microseconds


---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://lruihao.cn/posts/03.%E6%9C%89%E9%99%90%E7%9B%B4%E6%8E%A5%E6%89%A7%E8%A1%8C/  

