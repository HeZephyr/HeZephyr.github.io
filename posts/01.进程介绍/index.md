# 进程介绍

[github代码](https://github.com/unique-pure/ostep/tree/main/Virtualization/01.Process%20Introduction)
## Program Explanation
`process-run.py`：可以查看进程状态在CPU上运行时如何变化。
进程可以处于以下几种不同的状态：
  * `RUNNING`：进程正在使用CPU
  * `READY`：进程现在可以使用CPU但其他进程正在使用
  * `BLOCKED`：进程正在等待I/O（例如，它向磁盘发送请求）
  * `DONE`：进程已经执行完成

要运行该程序获取其选项，可执行以下操作
`./process-run.py -h`
或者
`python process-run.py -h`
得到：
```
Usage: process-run.py [options]

Options:
  -h, --help            show this help message and exit
  -s SEED, --seed=SEED  the random seed
  -l PROCESS_LIST, --processlist=PROCESS_LIST
                        a comma-separated list of processes to run, in the
                        form X1:Y1,X2:Y2,... where X is the number of
                        instructions that process should run, and Y the
                        chances (from 0 to 100) that an instruction will use
                        the CPU or issue an IO
  -L IO_LENGTH, --iolength=IO_LENGTH
                        how long an IO takes
  -S PROCESS_SWITCH_BEHAVIOR, --switch=PROCESS_SWITCH_BEHAVIOR
                        when to switch between processes: SWITCH_ON_IO,
                        SWITCH_ON_END
  -I IO_DONE_BEHAVIOR, --iodone=IO_DONE_BEHAVIOR
                        type of behavior when IO ends: IO_RUN_LATER,
                        IO_RUN_IMMEDIATE
  -c                    compute answers for me
  -p, --printstats      print statistics at end; only useful with -c flag
                        (otherwise stats are not printed)
```
要理解的最重要的选项是 PROCESS_LIST（由 -l 或 --processlist 参数指定），它准确指定每个正在运行的程序（或“进程”）将执行的操作。一个进程由指令组成，每条指令只能执行以下两件事之一：
&gt; * 使用CPU
&gt; 请求IO（并等待其完成）

当进程使用 CPU（不执行 IO）时，它应该简单地在 CPU 上运行或准备运行之间交替。例如，这是一个简单的运行，仅运行一个程序，并且该程序仅使用 CPU（不执行 IO）。
```
❯ python process-run.py -l 5:100
Produce a trace of what would happen when you run these processes:
Process 0
  cpu
  cpu
  cpu
  cpu
  cpu

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN IO
  After IOs, the process issuing the IO will run LATER (when it is its turn)
```
这里，我们指定的进程是“5:100”，这意味着它应该由5条指令组成，并且每条指令是CPU指令的机会是100%。
我们可以使用-c参数来查看进程发生了什么：
```
❯ python process-run.py -l 5:100 -c
Time        PID: 0           CPU           IOs
  1        RUN:cpu             1          
  2        RUN:cpu             1          
  3        RUN:cpu             1          
  4        RUN:cpu             1          
  5        RUN:cpu             1 
```
这个结果并不是太有趣：该过程在 RUNNING 状态下很简单，然后完成，始终使用 CPU，从而使 CPU 在整个运行过程中保持忙碌，并且不执行任何 I/O。
我们可以运行三个进程看看结果：
```
❯ python process-run.py -l 5:100,5:100,5:100 -c
Time        PID: 0        PID: 1        PID: 2           CPU           IOs
  1        RUN:cpu         READY         READY             1          
  2        RUN:cpu         READY         READY             1          
  3        RUN:cpu         READY         READY             1          
  4        RUN:cpu         READY         READY             1          
  5        RUN:cpu         READY         READY             1          
  6           DONE       RUN:cpu         READY             1          
  7           DONE       RUN:cpu         READY             1          
  8           DONE       RUN:cpu         READY             1          
  9           DONE       RUN:cpu         READY             1          
 10           DONE       RUN:cpu         READY             1          
 11           DONE          DONE       RUN:cpu             1          
 12           DONE          DONE       RUN:cpu             1          
 13           DONE          DONE       RUN:cpu             1          
 14           DONE          DONE       RUN:cpu             1          
 15           DONE          DONE       RUN:cpu             1  
```
首先首先“进程 ID”（或“PID”）为 0 的进程运行，而进程 1 和进程2已准备好运行，但只是等待 0 完成。当0完成时，它进入DONE状态，而1则运行，2继续等待。当 1 完成时，则2才开始运行。

我们继续看一个例子，在此示例中，进程仅发出I/O请求。我们使用`-L`参数指定I/O需要5个时间单位才能完成。
```
❯ python process-run.py -l 3:0 -L 5
Produce a trace of what would happen when you run these processes:
Process 0
  io
  io_done
  io
  io_done
  io
  io_done

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN IO
  After IOs, the process issuing the IO will run LATER (when it is its turn)
```
我们设置跟踪参数看看会是什么样子：
```
❯ python process-run.py -l 3:0 -L 5 -c
Time        PID: 0           CPU           IOs
  1         RUN:io             1          
  2        BLOCKED                           1
  3        BLOCKED                           1
  4        BLOCKED                           1
  5        BLOCKED                           1
  6        BLOCKED                           1
  7*   RUN:io_done             1          
  8         RUN:io             1          
  9        BLOCKED                           1
 10        BLOCKED                           1
 11        BLOCKED                           1
 12        BLOCKED                           1
 13        BLOCKED                           1
 14*   RUN:io_done             1          
 15         RUN:io             1          
 16        BLOCKED                           1
 17        BLOCKED                           1
 18        BLOCKED                           1
 19        BLOCKED                           1
 20        BLOCKED                           1
 21*   RUN:io_done             1          
❯ python process-run.py -l 3:0 -L 5 -c
Time        PID: 0           CPU           IOs
  1         RUN:io             1          
  2        BLOCKED                           1
  3        BLOCKED                           1
  4        BLOCKED                           1
  5        BLOCKED                           1
  6        BLOCKED                           1
  7*   RUN:io_done             1          
  8         RUN:io             1          
  9        BLOCKED                           1
 10        BLOCKED                           1
 11        BLOCKED                           1
 12        BLOCKED                           1
 13        BLOCKED                           1
 14*   RUN:io_done             1          
 15         RUN:io             1          
 16        BLOCKED                           1
 17        BLOCKED                           1
 18        BLOCKED                           1
 19        BLOCKED                           1
 20        BLOCKED                           1
 21*   RUN:io_done             1 
```
如上所示，该程序仅发出三个 I/O。当发出每个 I/O 时，进程将进入 BLOCKED 状态，当设备忙于为 I/O 提供服务时，CPU 处于空闲状态。
为了处理 I/O 的完成，还会发生一项 CPU 操作。请注意，处理 I/O 启动和完成的单个指令并不是特别现实，但这里只是为了简单起见而使用。让我们打印一些统计信息（运行与上面相同的命令，但使用 -p 参数）来查看一些总体行为：
```
Stats: Total Time 21
Stats: CPU Busy 6 (28.57%)
Stats: IO Busy  15 (71.43%)
```
如上所示，跟踪运行需要 21 个时钟周期，但 CPU 忙碌的时间不到 30%。另一方面，I/O 设备却相当忙碌。一般来说，我们希望让所有设备保持忙碌，因为这样可以更好地利用资源。
## QA
1. 运行 `process-run.py`，指定参数：`-l 5:100`。CPU 利用率应该是多少（例如，CPU 使用时间的百分比）？使用 -c 和 -p 参数来查看你是否正确。
   &gt; CPU利用率为100%，因为有5条指令，使用CPU的指令机会为100%。
   ```
   ❯ python process-run.py  -l 5:100 -c -p
    Time        PID: 0           CPU           IOs
    1        RUN:cpu             1          
    2        RUN:cpu             1          
    3        RUN:cpu             1          
    4        RUN:cpu             1          
    5        RUN:cpu             1          

    Stats: Total Time 5
    Stats: CPU Busy 5 (100.00%)
    Stats: IO Busy  0 (0.00%)
   ```
2. 运行：`./process-run.py -l 4:100,1:0`。这些参数指定一个具有 4 条指令的进程（全部使用 CPU），以及一个仅发出 I/O 并等待其完成的进程。完成这两个过程需要多长时间？使用 -c 和 -p 来看看你是否正确。
   &gt; PID为0的进程运行四条指令需要4，PID为1的进程完成IO需要5，但是发出IO指令以及完成IO指令需要CPU，所以为4&#43;5&#43;2=11。
   ```
    ❯ python process-run.py  -l 4:100,1:0 -c -p
    Time        PID: 0        PID: 1           CPU           IOs
      1        RUN:cpu         READY             1          
      2        RUN:cpu         READY             1          
      3        RUN:cpu         READY             1          
      4        RUN:cpu         READY             1          
      5           DONE        RUN:io             1          
      6           DONE       BLOCKED                           1
      7           DONE       BLOCKED                           1
      8           DONE       BLOCKED                           1
      9           DONE       BLOCKED                           1
    10           DONE       BLOCKED                           1
    11*          DONE   RUN:io_done             1          

    Stats: Total Time 11
    Stats: CPU Busy 6 (54.55%)
    Stats: IO Busy  5 (45.45%)
   ```
3. 改变进程的顺序： `./process-run.py -l 1:0,4:100`。现在会发生什么？切换顺序重要吗？为什么？（像往常一样，使用 -c 和 -p 查看你是否正确）
   这样进程0就可以先获得CPU发起IO请求，然后进行I/O。此时CPU空闲，进程1则可以获得CPU。在2中，进程0则需要等到进程1运行完成才可以获得。这节省了时间。
   ```
   ❯ python process-run.py  -l 1:0,4:100 -c -p
    Time        PID: 0        PID: 1           CPU           IOs
      1         RUN:io         READY             1          
      2        BLOCKED       RUN:cpu             1             1
      3        BLOCKED       RUN:cpu             1             1
      4        BLOCKED       RUN:cpu             1             1
      5        BLOCKED       RUN:cpu             1             1
      6        BLOCKED          DONE                           1
      7*   RUN:io_done          DONE             1          

    Stats: Total Time 7
    Stats: CPU Busy 6 (85.71%)
    Stats: IO Busy  5 (71.43%)
   ```
4. 现在我们来看看其他一些参数。其中一个重要的参数是 -S，它决定了当一个进程发出 I/O 时系统的反应。将该标志设置为 SWITCH ON END 时，当一个进程正在进行 I/O 时，系统不会切换到另一个进程，而是等待该进程完全结束。如果运行以下两个进程（`-l 1:0,4:100 -c -S SWITCH_ON_END`），其中一个正在执行 I/O 操作，另一个正在执行 CPU 操作，会发生什么情况？
   &gt; 此时进程0会一直占用CPU，直到I/O操作完成。用时也是11。
   ```
   ❯ python process-run.py  -l 1:0,4:100 -c -S SWITCH_ON_END -p
    Time        PID: 0        PID: 1           CPU           IOs
      1         RUN:io         READY             1          
      2        BLOCKED         READY                           1
      3        BLOCKED         READY                           1
      4        BLOCKED         READY                           1
      5        BLOCKED         READY                           1
      6        BLOCKED         READY                           1
      7*   RUN:io_done         READY             1          
      8           DONE       RUN:cpu             1          
      9           DONE       RUN:cpu             1          
      10           DONE       RUN:cpu             1          
      11           DONE       RUN:cpu             1          

      Stats: Total Time 11
      Stats: CPU Busy 6 (54.55%)
      Stats: IO Busy  5 (45.45%)
   ```
5. 现在，运行相同的进程，但将切换行为设置为每当一个进程等待 I/O 时就切换到另一个进程（`-l 1:0,4:100 -c -S SWITCH_ON_IO`）。现在会发生什么？使用 -c 和 -p 来确认是否正确。
   &gt;此时进程0发起I/O请求后，操作系统就会切换进程，总用时为7
   ```
   ❯ python process-run.py  -l 1:0,4:100 -c -S SWITCH_ON_IO -p
    Time        PID: 0        PID: 1           CPU           IOs
      1         RUN:io         READY             1          
      2        BLOCKED       RUN:cpu             1             1
      3        BLOCKED       RUN:cpu             1             1
      4        BLOCKED       RUN:cpu             1             1
      5        BLOCKED       RUN:cpu             1             1
      6        BLOCKED          DONE                           1
      7*   RUN:io_done          DONE             1          

    Stats: Total Time 7
    Stats: CPU Busy 6 (85.71%)
    Stats: IO Busy  5 (71.43%)
   ```
6. 另一个重要的行为是在 I/O 完成时该做什么。使用 `-I IO RUN LATER` 选项时，当 I/O 完成时，发出它的进程不一定会立即运行；相反，正在运行的内容将继续运行。当您运行这组进程组合时会发生什么？（运行 `python process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p`）系统资源是否得到有效利用？
   &gt; 此时系统资源没有得到有效利用。因为CPU处理I/O完成越早，进程就可以继续处理I/O，它可以在不占用CPU的时候进行。
   ```
   ❯ python process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p
    Time        PID: 0        PID: 1        PID: 2        PID: 3           CPU           IOs
      1         RUN:io         READY         READY         READY             1          
      2        BLOCKED       RUN:cpu         READY         READY             1             1
      3        BLOCKED       RUN:cpu         READY         READY             1             1
      4        BLOCKED       RUN:cpu         READY         READY             1             1
      5        BLOCKED       RUN:cpu         READY         READY             1             1
      6        BLOCKED       RUN:cpu         READY         READY             1             1
      7*         READY          DONE       RUN:cpu         READY             1          
      8          READY          DONE       RUN:cpu         READY             1          
      9          READY          DONE       RUN:cpu         READY             1          
    10          READY          DONE       RUN:cpu         READY             1          
    11          READY          DONE       RUN:cpu         READY             1          
    12          READY          DONE          DONE       RUN:cpu             1          
    13          READY          DONE          DONE       RUN:cpu             1          
    14          READY          DONE          DONE       RUN:cpu             1          
    15          READY          DONE          DONE       RUN:cpu             1          
    16          READY          DONE          DONE       RUN:cpu             1          
    17    RUN:io_done          DONE          DONE          DONE             1          
    18         RUN:io          DONE          DONE          DONE             1          
    19        BLOCKED          DONE          DONE          DONE                           1
    20        BLOCKED          DONE          DONE          DONE                           1
    21        BLOCKED          DONE          DONE          DONE                           1
    22        BLOCKED          DONE          DONE          DONE                           1
    23        BLOCKED          DONE          DONE          DONE                           1
    24*   RUN:io_done          DONE          DONE          DONE             1          
    25         RUN:io          DONE          DONE          DONE             1          
    26        BLOCKED          DONE          DONE          DONE                           1
    27        BLOCKED          DONE          DONE          DONE                           1
    28        BLOCKED          DONE          DONE          DONE                           1
    29        BLOCKED          DONE          DONE          DONE                           1
    30        BLOCKED          DONE          DONE          DONE                           1
    31*   RUN:io_done          DONE          DONE          DONE             1          

    Stats: Total Time 31
    Stats: CPU Busy 21 (67.74%)
    Stats: IO Busy  15 (48.39%)
   ```
7. 现在运行相同的进程，但使用 `-I IO_RUN_IMMEDIATE` 设置，该设置会立即运行发出 I/O 的进程。这种行为有何不同？为什么再次运行刚完成 I/O 的进程可能是个好主意？
   &gt; 这种行为会让进程处理I/O请求的时候还给其他CPU，但完成后立即获得CPU，能让I/O处理和CPU处理一起进行，节省总时间，有效提高CPU利用率。
   ```
   ❯ python process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_IMMEDIATE -c -p
    Time        PID: 0        PID: 1        PID: 2        PID: 3           CPU           IOs
      1         RUN:io         READY         READY         READY             1          
      2        BLOCKED       RUN:cpu         READY         READY             1             1
      3        BLOCKED       RUN:cpu         READY         READY             1             1
      4        BLOCKED       RUN:cpu         READY         READY             1             1
      5        BLOCKED       RUN:cpu         READY         READY             1             1
      6        BLOCKED       RUN:cpu         READY         READY             1             1
      7*   RUN:io_done          DONE         READY         READY             1          
      8         RUN:io          DONE         READY         READY             1          
      9        BLOCKED          DONE       RUN:cpu         READY             1             1
    10        BLOCKED          DONE       RUN:cpu         READY             1             1
    11        BLOCKED          DONE       RUN:cpu         READY             1             1
    12        BLOCKED          DONE       RUN:cpu         READY             1             1
    13        BLOCKED          DONE       RUN:cpu         READY             1             1
    14*   RUN:io_done          DONE          DONE         READY             1          
    15         RUN:io          DONE          DONE         READY             1          
    16        BLOCKED          DONE          DONE       RUN:cpu             1             1
    17        BLOCKED          DONE          DONE       RUN:cpu             1             1
    18        BLOCKED          DONE          DONE       RUN:cpu             1             1
    19        BLOCKED          DONE          DONE       RUN:cpu             1             1
    20        BLOCKED          DONE          DONE       RUN:cpu             1             1
    21*   RUN:io_done          DONE          DONE          DONE             1          

    Stats: Total Time 21
    Stats: CPU Busy 21 (100.00%)
    Stats: IO Busy  15 (71.43%)
   ```
8. 现在运行一些随机生成的进程：`-s 1 -l 3:50,3:50` 或 `-s 2 -l 3:50,3:50` 或 `-s 3 -l 3:50,3:50`。看看你是否可以预测跟踪结果如何。当您使用参数 `-I IO_RUN_IMMEDIATE` 与 `-I IO_RUN_LATER` 时会发生什么？当您使用 `-S SWITCH_ON_IO` 与 `-S SWITCH_ON_END` 时会发生什么？
  &gt; 设置随机种子，可以保证相同的种子生成的随机数都一样，会得到相同的随机数序列。针对问题中的50，则是有50%的机会进行CPU操作或者I/O操作。如果没有使用任何参数，则是正常的切换。
  ```
  ❯ python process-run.py -s 1 -l 3:50,3:50 -c -p
  Time        PID: 0        PID: 1           CPU           IOs
    1        RUN:cpu         READY             1          
    2         RUN:io         READY             1          
    3        BLOCKED       RUN:cpu             1             1
    4        BLOCKED       RUN:cpu             1             1
    5        BLOCKED       RUN:cpu             1             1
    6        BLOCKED          DONE                           1
    7        BLOCKED          DONE                           1
    8*   RUN:io_done          DONE             1          
    9         RUN:io          DONE             1          
  10        BLOCKED          DONE                           1
  11        BLOCKED          DONE                           1
  12        BLOCKED          DONE                           1
  13        BLOCKED          DONE                           1
  14        BLOCKED          DONE                           1
  15*   RUN:io_done          DONE             1          

  Stats: Total Time 15
  Stats: CPU Busy 8 (53.33%)
  Stats: IO Busy  10 (66.67%)
  ```
  

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/01.%E8%BF%9B%E7%A8%8B%E4%BB%8B%E7%BB%8D/  

