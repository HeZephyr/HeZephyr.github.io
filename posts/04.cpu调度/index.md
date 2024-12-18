# CPU调度

[github代码](https://github.com/unique-pure/ostep/blob/main/Virtualization/04.CPU%20Scheduling/README.md)
## Program Explanation
程序`scheduler.py`允许您查看不同调度程序在响应时间、周转时间和总等待时间等调度指标下的执行情况。 “实现”了三个调度程序：FIFO、SJF 和 RR。
要运行该程序获取其选项，可执行以下操作：
`./scheduler.py -h`
或者
`python scheduler.py -h`
得到：
```
❯ python scheduler.py -h
Usage: scheduler.py [options]

Options:
  -h, --help            show this help message and exit
  -s SEED, --seed=SEED  the random seed
  -j JOBS, --jobs=JOBS  number of jobs in the system
  -l JLIST, --jlist=JLIST
                        instead of random jobs, provide a comma-separated list
                        of run times
  -m MAXLEN, --maxlen=MAXLEN
                        max length of job
  -p POLICY, --policy=POLICY
                        sched policy to use: SJF, FIFO, RR
  -q QUANTUM, --quantum=QUANTUM
                        length of time slice for RR policy
  -c                    compute answers for me
```
首先，在不带 -c 选项的情况下运行：这会向您显示要解决的问题而不透露答案。例如，如果您想要使用 FIFO 策略计算响应、周转和等待三个作业，请运行以下命令：
`python scheduler.py -p FIFO -j 3 -s 100`
这指定了具有三个作业的 FIFO 策略，而且重要的是，指定了 100 的特定随机种子。如果您想查看该问题的解决方案，则必须再次指定该完全相同的随机种子。让我们运行一下，看看会发生什么。这是您应该看到的：
```
❯ python scheduler.py -p FIFO -j 3 -s 100
ARG policy FIFO
ARG jobs 3
ARG maxlen 10
ARG seed 100

Here is the job list, with the run time of each job: 
  Job 0 ( length = 2 )
  Job 1 ( length = 5 )
  Job 2 ( length = 8 )


Compute the turnaround time, response time, and wait time for each job.
When you are done, run this program again, with the same arguments,
but with -c, which will thus provide you with the answers. You can use
-s &lt;somenumber&gt; or your own job list (-l 10,15,20 for example)
to generate different problems for yourself.
```
计算每个作业的周转时间、响应时间和等待时间。完成后，使用相同的参数再次运行该程序，但使用 -c选项，这将为您提供答案。您可以使用 -s 或您自己的作业列表（例如 -l 10,15,20）为自己生成不同的问题。
从这个例子中可以看出，生成了三个作业：长度为 2 的作业 0、长度为 5 的作业 1 和长度为 8 的作业 2。正如程序所述，您现在可以使用它来计算一些统计数据，看看是否可以掌握基本概念。
完成后，您可以使用相同的程序来“解决”问题并查看您的工作是否正确。为此，请使用“-c”选项。输出：
```
❯ python scheduler.py -p FIFO -j 3 -s 100 -c
ARG policy FIFO
ARG jobs 3
ARG maxlen 10
ARG seed 100

Here is the job list, with the run time of each job: 
  Job 0 ( length = 2 )
  Job 1 ( length = 5 )
  Job 2 ( length = 8 )


** Solutions **

Execution trace:
  [ time   0 ] Run job 0 for 2.00 secs ( DONE at 2.00 )
  [ time   2 ] Run job 1 for 5.00 secs ( DONE at 7.00 )
  [ time   7 ] Run job 2 for 8.00 secs ( DONE at 15.00 )

Final statistics:
  Job   0 -- Response: 0.00  Turnaround 2.00  Wait 0.00
  Job   1 -- Response: 2.00  Turnaround 7.00  Wait 2.00
  Job   2 -- Response: 7.00  Turnaround 15.00  Wait 7.00

  Average -- Response: 3.00  Turnaround 8.00  Wait 3.00
```
从上述输出我们可以看到，作业0首先运行了2秒，作业2接着运行了5秒，最后作业2运行了8秒。
最终的统计数据也很有用：它们计算“响应时间”（作业到达后第一次运行之前等待所花费的时间）、“周转时间”（自第一次到达以来完成作业所花费的时间）以及总时间。 “等待时间”（准备好但未运行的任何时间）。统计数据按作业显示，然后显示所有作业的平均值。当然，您应该在使用“-c”选项运行之前计算出所有这些内容！
如果您想尝试相同类型的问题但使用不同的输入，请尝试更改作业数量或随机种子或同时更改两者。不同的随机种子基本上为您提供了一种为自己生成无数不同问题的方法，并且“-c”选项可以让您检查自己的工作。继续这样做，直到您觉得自己真正理解了这些概念。
另一个有用的选项是“-l”（这是一个小写的 L），它可以让您指定您希望看到的计划的确切作业。例如，如果您想了解 SJF 如何执行长度为 5、10 和 15 的三个作业，您可以运行：
```
❯ python scheduler.py -p SJF -l 5,10,15
ARG policy SJF
ARG jlist 5,10,15

Here is the job list, with the run time of each job: 
  Job 0 ( length = 5.0 )
  Job 1 ( length = 10.0 )
  Job 2 ( length = 15.0 )


Compute the turnaround time, response time, and wait time for each job.
When you are done, run this program again, with the same arguments,
but with -c, which will thus provide you with the answers. You can use
-s &lt;somenumber&gt; or your own job list (-l 10,15,20 for example)
to generate different problems for yourself.
```
计算完后还是通过-c选项获得答案。指定确切的作业时，无须指定随机种子或者作业数量。
我们最后测试一下RR，可以运行：
```
❯ python scheduler.py -p RR -l 5,10,15 -c
ARG policy RR
ARG jlist 5,10,15

Here is the job list, with the run time of each job: 
  Job 0 ( length = 5.0 )
  Job 1 ( length = 10.0 )
  Job 2 ( length = 15.0 )


** Solutions **

Execution trace:
  [ time   0 ] Run job   0 for 1.00 secs
  [ time   1 ] Run job   1 for 1.00 secs
  [ time   2 ] Run job   2 for 1.00 secs
  [ time   3 ] Run job   0 for 1.00 secs
  [ time   4 ] Run job   1 for 1.00 secs
  [ time   5 ] Run job   2 for 1.00 secs
  [ time   6 ] Run job   0 for 1.00 secs
  [ time   7 ] Run job   1 for 1.00 secs
  [ time   8 ] Run job   2 for 1.00 secs
  [ time   9 ] Run job   0 for 1.00 secs
  [ time  10 ] Run job   1 for 1.00 secs
  [ time  11 ] Run job   2 for 1.00 secs
  [ time  12 ] Run job   0 for 1.00 secs ( DONE at 13.00 )
  [ time  13 ] Run job   1 for 1.00 secs
  [ time  14 ] Run job   2 for 1.00 secs
  [ time  15 ] Run job   1 for 1.00 secs
  [ time  16 ] Run job   2 for 1.00 secs
  [ time  17 ] Run job   1 for 1.00 secs
  [ time  18 ] Run job   2 for 1.00 secs
  [ time  19 ] Run job   1 for 1.00 secs
  [ time  20 ] Run job   2 for 1.00 secs
  [ time  21 ] Run job   1 for 1.00 secs
  [ time  22 ] Run job   2 for 1.00 secs
  [ time  23 ] Run job   1 for 1.00 secs ( DONE at 24.00 )
  [ time  24 ] Run job   2 for 1.00 secs
  [ time  25 ] Run job   2 for 1.00 secs
  [ time  26 ] Run job   2 for 1.00 secs
  [ time  27 ] Run job   2 for 1.00 secs
  [ time  28 ] Run job   2 for 1.00 secs
  [ time  29 ] Run job   2 for 1.00 secs ( DONE at 30.00 )

Final statistics:
  Job   0 -- Response: 0.00  Turnaround 13.00  Wait 8.00
  Job   1 -- Response: 1.00  Turnaround 24.00  Wait 14.00
  Job   2 -- Response: 2.00  Turnaround 30.00  Wait 15.00

  Average -- Response: 1.00  Turnaround 22.33  Wait 12.33
```
## QA
&gt;以下问题都可以通过`scheduler.py`进行验证
1. 使用 SJF 和 FIFO 调度器运行长度为 200 的三个作业时，计算响应时间和周转时间
   * SJF
    $turnaround = \frac{200-0&#43;400-0&#43;600-0}{3}=400$
    $response=\frac{0&#43;200&#43;400}{3}=200$
   * FIFO
    跟SJF一样
2. 现在执行相同的操作，但使用不同长度的作业：100、200 和 300。
   * SJF
    $turnaround = \frac{100-0&#43;300-0&#43;600-0}{3}=333$
    $response=\frac{0&#43;100&#43;300}{3}=133$
   * FIFO
    由于到达时间是一样，所以调度具有不确定性，最佳方案就是和SJF一样。
3. 现在执行相同的操作，但也使用 RR 调度程序和时间片 1
   对于RR，$response=\frac{0&#43;1&#43;2}{3}=1$
   $turnaround=\frac{298&#43;499&#43;600}{3}=265.67$
4. 在哪些类型的工作负载中，SJF 的周转时间与 FIFO 相同？
    &gt; SJF 始终提供与 FIFO 相同的周转时间，除非较短的作业在较长的作业之后到达。只要所有作业长度相等或按升序到达，周转时间就会相同。
5. 在哪些类型的工作负载和时间片长度下，SJF 可以提供与 RR 相同的响应时间？
   &gt; 如果所有作业长度(x)相同，且时间片长度(q)=x，在这种情况下，SJF提供与RR相同的响应时间。通过运行以下代码验证：
   `./scheduler.py -p SJF -l 100,100,100 -c`
   `./scheduler.py -p RR -l 100,100,100 -c -q 100`
6. 随着作业长度的增加，SJF 的响应时间会发生什么变化？您能否使用模拟器来演示这一趋势？
   &gt; 如果我们将每个工作的长度加倍，平均响应时间也会加倍。这意味着工作长度和响应时间之间存在线性关系。
7. 随着时间片长度的增加，RR 的响应时间会发生什么变化？你能写出一个方程，在给定 N 个作业的情况下给出最坏情况的响应时间吗？
   &gt; 响应时间随着时间片长度的增加而线性增加。令 N 为作业数量，q 为时间片长度，则最坏情况的响应时间为 $\frac{(N-1)q}{N}$。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/04.cpu%E8%B0%83%E5%BA%A6/  

