# IO设备

## 系统架构

为了开始我们的讨论，让我们看一下典型系统的“经典”图。

![image-20240412145750053](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/Prototypical_System_Architecture_Example.png)

该图显示了通过&lt;font color=&#34;red&#34;&gt;某种内存总线或互连&lt;/font&gt;连接到系统主内存的单个 CPU。一些设备通过&lt;font color=&#34;red&#34;&gt;通用 I/O 总线&lt;/font&gt;连接到系统，在许多现代系统中，该总线是 **PCI**（或其众多衍生产品之一）；显卡和其他一些更高性能的 I/O 设备可能会在这里找到。最后，更底层的是我们所说的一种或多种&lt;font color=&#34;red&#34;&gt;外围总线&lt;/font&gt;，例如 **SCSI**、**SATA** 或 USB。它们将慢速设备连接到系统，包括磁盘、鼠标和键盘。

您可能会问的一个问题是：为什么我们需要这样的层次结构？简而言之：物理和成本。公共汽车速度越快，其长度就必须越短；因此，高性能内存总线没有太多空间来插入设备等。此外，设计高性能总线的成本相当高。因此，系统设计人员采用了这种分层方法，其中需要高性能的组件（例如显卡）靠近 CPU。性能较低的组件距离较远。将磁盘和其他慢速设备放置在外围总线上的好处是多方面的。特别是，您可以在其上放置大量设备。

当然，现代系统越来越多地使用专用芯片组和更快的点对点互连来提高性能。下图为Intel Z270 芯片组的示意图。

![image-20240412150741468](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/Modern_System_Architecture.png)

在顶部，CPU 与内存系统的连接最紧密，但也与显卡（以及显示器）有高性能连接，以支持游戏和其他图形密集型应用程序。

CPU 通过&lt;font color=&#34;red&#34;&gt;英特尔专有的 DMI（Direct Media Interface, 直接媒体接口）&lt;/font&gt;连接到 I/O 芯片，其余设备通过许多不同的互连连接到该芯片。右侧，一个或多个硬盘通过&lt;font color=&#34;red&#34;&gt;eSATA&lt;/font&gt;接口连接到系统； ATA（AT 附件，指提供与 IBM PC AT 的连接）、SATA（串行 ATA）和现在的 eSATA（外部 SATA）代表了过去几十年来存储接口的演变，每向前一步都在增加性能与现代存储设备保持同步。

 I/O 芯片下方是许多 &lt;font color=&#34;red&#34;&gt;USB（Universal Serial Bus, 通用串行总线）&lt;/font&gt;连接，在此描述中，这些连接使键盘和鼠标能够连接到计算机。在许多现代系统中，USB 用于此类低性能设备。

最后，在左侧，其他更高性能的设备可以通过 &lt;font color=&#34;red&#34;&gt;PCIe（Peripheral Component Interconnect Express，外围组件互连扩展）&lt;/font&gt;连接到系统。在此图中，网络接口连接到系统；更高性能的存储设备（例如NVMe持久存储设备）通常连接在这里。

下图是一个真实的配备 Xeon E5-2600 v4 的双插槽服务器，展示了两个CPU（中央处理器）及其相关组件的连接方式。每个CPU都具有多达22个核心，这两个CPU通过QPI链接连接，然后通过内存控制器与DRAM连接（DDR4（Double Data Rate 4）是一种内存标准，它提供了更高的数据传输速率和更低的功耗，是当前常见的内存类型）。此外，它们还与PCIe总线相连，并分别有40条PCIe线路到CPU1和CPU2。

CPU与众多外设的连接通常是通过主板上的I/O芯片组实现的，这个芯片组包括北桥（PCH，Platform Controller Hub）和南桥（ICH，I/O Controller Hub）。在较新的系统中，这些功能可能被集成到一个单一的PCH芯片中。DMI（Direct Media Interface）是CPU与PCH之间的一种高速连接接口，它允许CPU通过PCH与各种外设进行通信。

![img](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/Real_CPU_System_Architecture_Example.png)

下图显示了一个计算机系统的PCIe连接布局。从CPU Package开始，它通过Host Bridge与Root Complex(RC)相连。在RC中，有多个Core与Memory Controller相连。PCIe Switch位于RC和外设之间，管理着数据传输。此外，还有多个PCIe Bus、PCIe Bridge和PCIe Endpoint。每个部分都有其对应的bus、dev、fun、pri、sec和sub的标识。

![img](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/Pcle_Structure_Example.png)

## 规范设备

现在让我们看一个规范设备（不是真实的设备），并使用该设备来加深我们对提高设备交互效率所需的一些机制的理解。如下图所示，我们可以看到设备有两个重要的组件。第一个是它向系统其余部分提供的硬件接口。就像软件一样，硬件也必须提供某种接口，允许系统软件控制其操作。因此，所有设备都有一些特定的接口和协议用于典型的交互。

![image-20240412155318734](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/Canonical_Device_Example.png)

任何设备的第二部分是其内部结构。设备的这一部分是特定于实现的，负责实现设备向系统呈现的抽象。非常简单的设备将有一个或几个硬件芯片来实现其功能；更复杂的设备将包括一个简单的 CPU、一些通用内存和其他特定于设备的芯片来完成其工作。例如，现代 RAID 控制器可能包含数十万行固件（即硬件设备内的软件）来实现其功能。

## 规范协议

在上图中，（简化的）设备接口由三个寄存器组成：

* &lt;font color=&#34;red&#34;&gt;状态寄存器&lt;/font&gt;，可通过读取状态寄存器来查看设备的当前状态；
* &lt;font color=&#34;red&#34;&gt;命令寄存器&lt;/font&gt;，用于通知设备执行某项任务；
* &lt;font color=&#34;red&#34;&gt;数据寄存器&lt;/font&gt;，用于向设备传递数据或从设备获取数据。

通过读写这些寄存器，操作系统可以控制设备的行为。现在，让我们来描述一下操作系统与设备之间可能进行的典型交互，以便让设备代表自己做一些事情。协议如下：

```c
While (STATUS == BUSY)
    ; // wait until device is not busy

Write data to DATA register
Write command to COMMAND register
(Doing so starts the device and executes the command)

While (STATUS == BUSY)
    ; // wait until device is done with your request
```

该协议有四个步骤。

1. 首先，操作系统通过重复读取状态寄存器来等待设备准备好接收命令；我们称之为**轮询**设备（基本上，只是询问发生了什么）。
2. 其次，操作系统将一些数据发送到数据寄存器；例如，可以想象，如果这是一个磁盘，则需要进行多次写入才能将磁盘块（例如 4KB）传输到设备。当主 CPU 参与数据移动时（如本示例协议所示），我们将其称为&lt;font color=&#34;red&#34;&gt;编程 I/O (PIO)&lt;/font&gt;。
3. 然后，操作系统向命令寄存器写入命令；这样做会隐式地让设备知道数据存在并且它应该开始处理命令。
4. 最后，操作系统通过再次循环轮询设备来等待设备完成，等待查看它是否完成（然后可能会收到一个错误代码来指示成功或失败）。

这个基本协议的优点是简单且有效。然而，这也存在一些低效率和不便之处。您可能在协议中注意到的第一个问题是轮询似乎效率低下；具体来说，它浪费了大量的 CPU 时间来等待（可能很慢的）设备完成其活动，而不是切换到另一个就绪进程，从而更好地利用 CPU。

所以问题关键是操作系统如何在不频繁轮询的情况下检查设备状态，从而降低管理设备所需的 CPU 开销？

## 通过中断降低 CPU 开销

多年前，许多工程师为了改善这种交互方式，发明了我们已经见过的：中断。操作系统可以发出一个请求，让调用进程进入休眠状态，然后切换到另一个任务，而不是反复轮询设备。当设备最终完成操作时，它会引发硬件中断，导致 CPU 在预定的中断服务例程（ISR）或更简单的中断处理程序处跳转到操作系统。处理程序只是一段操作系统代码，它将完成请求（例如，从设备读取数据和错误代码），并唤醒等待 I/O 的进程，然后该进程可按需要继续运行。

因此，中断允许计算和 I/O 重叠，这是提高利用率的关键。这条时间线显示了问题所在：

![image-20240412161030799](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/CPU_Utilization_By_Polling_Example.png)

在图中，进程 1 在 CPU 上运行一段时间（由 CPU 线上重复的 1 表示），然后向磁盘发出 I/O 请求以读取一些数据。在没有中断的情况下，系统只是简单地自旋，重复轮询设备的状态，直到 I/O 完成（由 p 表示）。磁盘服务该请求，最后进程 1 可以再次运行。

相反，如果我们利用中断并允许重叠，操作系统可以在等待磁盘时执行其他操作：

![image-20240412161507503](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/CPU_Utilization_By_Interrupt.png)

在此示例中，操作系统在 CPU 上运行进程 2，同时磁盘服务进程 1 的请求。当磁盘请求完成时，会发生中断，操作系统唤醒进程1并再次运行它。这样，CPU和磁盘在中间的一段时间内都得到了适当的利用。

请注意，使用中断并不总是最好的解决方案。例如，假设一个设备执行任务的速度非常快：第一次轮询通常会发现该设备已完成任务。在这种情况下使用中断实际上会减慢系统速度：切换到另一个进程、处理中断以及切换回发出进程的成本很高。&lt;font color=&#34;red&#34;&gt;因此，如果设备速度很快，最好进行轮询；如果速度很慢，那么允许重叠的中断是最好的。如果设备的速度未知，或者有时快有时慢，最好使用混合轮询一段时间，然后如果设备尚未完成，则使用中断。&lt;/font&gt;这种分两阶段的方法可能会达到两全其美的效果。

不使用中断的另一个原因出现在网络中。当大量传入数据包均产生中断时，操作系统可能会发生活锁，即发现自己只处理中断，而不允许用户级进程运行并实际服务请求。例如，假设一个 Web 服务器由于成为黑客新闻上排名最高的条目而经历了负载爆发。在这种情况下，最好偶尔使用轮询来更好地控制系统中发生的情况，并允许 Web 服务器在返回设备检查更多数据包到达之前为某些请求提供服务。

另一种基于中断的优化是&lt;font color=&#34;red&#34;&gt;合并&lt;/font&gt;。在这样的设置中，需要引发中断的设备首先等待一段时间，然后再将中断传递给 CPU。在等待期间，其他请求可能很快完成，因此可以将多个中断合并为单个中断传递，从而降低中断处理的开销。当然，等待太久会增加请求的延迟，这是系统中常见的权衡。

## 通过 DMA 实现更高效的数据移动

不幸的是，我们的规范协议还有另一个方面需要我们注意。特别是，当使用编程 I/O (PIO) 将大量数据传输到设备时，CPU 再次因一项相当琐碎的任务而负担过重，从而浪费了大量的时间和精力，而这些时间和精力本来可以更好地花在运行上其他流程。这个时间线说明了这个问题：

![image-20240413084020326](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/CPU_Utilization_Example_3.png)

在时间线中，进程 1 正在运行，然后希望将一些数据写入磁盘。然后它启动 I/O，该 I/O 必须将数据从内存显式复制到设备，一次一个字（图中标记为 c）。复制完成后，I/O 开始在磁盘上进行，CPU 最终可以用于其他用途。

&gt; &lt;center&gt;关键：如何降低 PIO 开销&lt;/center&gt;
&gt;
&gt; 使用 PIO，CPU 会花费太多时间手动将数据移入和移出设备。我们如何才能卸载这项工作，从而更有效地利用 CPU？

解决这一问题的方法就是我们所说的直接内存访问（DMA）。DMA 引擎本质上是系统中一个非常特殊的设备，它可以在设备和主内存之间协调传输，而无需 CPU 的过多干预。DMA 的工作原理如下。以向设备传输数据为例，操作系统将对 DMA 引擎进行编程，告诉它数据在内存中的位置、需要复制多少数据以及发送到哪个设备。

```c
void setup_dma_transfer(void *source_address, void *destination_address, size_t data_size);
```

此时，操作系统就完成了传输，可以继续其他工作。当 DMA 完成时，DMA 控制器会发出中断，操作系统因此知道传输已经完成。修改后的时间线如下：

![image-20240413084446102](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/CPU_Utilization_By_DMA_Example.png)

从时间轴上可以看到，复制数据的工作现在由 DMA 控制器负责。因为在这段时间内 CPU 是空闲的，操作系统可以做其他事情，这里选择运行进程 2。这样，在进程 1 再次运行之前，进程 2 可以使用更多的 CPU。

## 设备交互

现在我们对执行 I/O 所涉及的效率问题有了一定的了解，为了将设备合并到现代系统中，我们还需要处理一些其他问题。到目前为止，您可能已经注意到一个问题：我们还没有真正谈论操作系统如何与设备实际通信！因此，关键问题是：

&gt; 硬件应该如何与设备通信？是否应该有明确的指示？或者还有其他方法可以做到吗？

随着时间的推移，已经开发出两种主要的设备通信方法。第一种也是最古老的方法（IBM 大型机使用了很多年）是使用&lt;font color=&#34;red&#34;&gt;显式 I/O 指令&lt;/font&gt;。这些指令指定操作系统将数据发送到特定设备寄存器的方式，从而允许构建上述协议。

例如，在 x86 系统中，`in` 和 `out` 指令可用于与设备通信。例如，要向设备发送数据，调用者需要指定&lt;font color=&#34;red&#34;&gt;一个包含数据的寄存器和一个命名设备的特定端口&lt;/font&gt;。执行该指令后，就会产生所需的行为。

此类指令通常具有特权。操作系统控制着设备，因此操作系统是唯一允许与设备直接通信的实体。试想一下，如果任何程序都能读写磁盘，那么整个系统都会陷入混乱（一如既往），因为任何用户程序都可以利用这个漏洞完全控制机器。

与设备交互的第二种方法称为&lt;font color=&#34;red&#34;&gt;内存映射 I/O&lt;/font&gt;。采用这种方法时，硬件会将设备寄存器当作内存位置来使用。要访问特定寄存器，操作系统会发出加载（读取）或存储（写入）地址；然后硬件会将加载/存储路由到设备，而不是主内存。

这两种方法并没有很大的优势。内存映射方法的优点是不需要新指令来支持，但这两种方法目前仍在使用。

## 适配操作系统：设备驱动程序

我们要讨论的最后一个问题是：我们将讨论的最后一个问题是：如何将具有非常特定接口的设备整合到操作系统中，而且我们希望保持尽可能通用。例如，考虑一个文件系统。我们希望构建一个可以在`SCSI`磁盘、`IDE`磁盘、`USB`闪存驱动器等设备上运行的文件系统，我们希望文件系统能相对忽略如何向这些不同类型的驱动器发出读取或写入请求的所有细节。

因此，我们面临的关键问题是：如何让操作系统的大部分功能保持设备中立，从而将设备交互的细节从主要的操作系统子系统中隐藏起来？

这个问题可以通过古老的**抽象**技术来解决。在最底层，操作系统中的一个软件必须详细了解设备是如何工作的。我们称这一软件为&lt;font color=&#34;red&#34;&gt;设备驱动程序&lt;/font&gt;，设备交互的任何细节都封装在其中。

让我们通过研究 Linux 文件系统软件栈，看看这种抽象如何帮助操作系统的设计和实现。下图粗略描绘了 Linux 软件的组织结构。

![image-20240413091848773](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/The_File_System_Stack_Example.png)

从图中可以看出，文件系统（当然也包括上面的应用程序）完全不关心它所使用的磁盘类的具体情况；它只需向通用块层发出块读写请求，通用块层会将这些请求路由到相应的设备驱动程序，设备驱动程序会处理发出具体请求的细节。虽然经过简化，但该图显示了这些细节是如何从操作系统的大部分功能中隐藏起来的。

该图还显示了设备的&lt;font color=&#34;red&#34;&gt;原始接口&lt;/font&gt;，它使特殊应用程序（例如&lt;font color=&#34;red&#34;&gt;文件系统检查器&lt;/font&gt;或&lt;font color=&#34;red&#34;&gt;磁盘碎片整理工具&lt;/font&gt;）能够直接读取和写入块，而不使用文件抽象。大多数系统提供这种类型的接口来支持这些低级存储管理应用程序。

请注意，上面看到的封装也有其缺点。例如，如果有一个设备具有许多特殊功能，但必须向内核的其余部分提供通用接口，那么这些特殊功能将不会被使用。例如，在具有 SCSI 设备的 Linux 中就会出现这种情况，这些设备具有非常丰富的错误报告；因为其他块设备（例如 ATA/IDE）的错误处理要简单得多，所以更高级别的软件收到的只是通用 `EIO`（通用 IO 错误）错误代码；因此，`SCSI` 可能提供的任何额外细节都会在文件系统中丢失。

有趣的是，由于您可能插入系统的任何设备都需要设备驱动程序，因此随着时间的推移，它们已经占据了内核代码的很大一部分。对 Linux 内核的研究表明，超过 70% 的操作系统代码都存在于设备驱动程序中 ；对于基于 Windows 的系统，该值可能也相当高。因此，&lt;font color=&#34;red&#34;&gt;当人们告诉您操作系统有数百万行代码时，他们真正说的是操作系统有数百万行设备驱动程序代码。&lt;/font&gt;当然，对于任何给定的安装，大部分代码可能并不活跃（即，一次只有少数设备连接到系统）。也许更令人沮丧的是，由于驱动程序通常是由“业余爱好者”（而不是全职内核开发人员）编写的，因此它们往往会存在更多错误，因此是导致内核崩溃的主要因素。

## 案例研究：简单的 IDE 磁盘驱动程序（xv6 使用 QEMU IDE）

为了更深入地研究，让我们快速看一下实际的设备：IDE 磁盘驱动器。我们将查看 xv6 源代码，以获取工作 IDE 驱动程序的简单示例。

IDE 磁盘为系统提供了一个简单的接口，由四种类型的寄存器组成：

* **控制寄存器**

	Address 0x3F6 = 0x80 (0000 1RE0): R=reset, E=0 means &#34;enable interrupt”

* **命令块寄存器**

	Address 0x1F0 = Data Port Address 

	0x1F1 = Error Address

	 0x1F2 = Sector Count Address 

	0x1F3 = LBA low byte (Logical Block Address) Address 

	0x1F4 = LBA mid byte Address 

	0x1F5 = LBA hi byte Address 

	0x1F6 = 1B1D TOP4LBA: B=LBA, D=drive Address 

	0x1F7 = Command/status

* **状态寄存器**

	Address 0x1F7

	| 7    | 6     | 5     | 4    | 3    | 2    | 1      | 0     |
	| ---- | ----- | ----- | ---- | ---- | ---- | ------ | ----- |
	| BUSY | READY | FAULT | SEEK | DRQ  | CORR | IDX/EX | ERROR |

* **错误寄存器**

	Address 0x1F1

	| 7    | 6    | 5    | 4    | 3    | 2    | 1    | 0    |
	| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
	| BBK  | UNC  | MC   | IDNF | MCR  | ABRT | T0NF | AMNF |

	- BBK: Bad Block
	- UNC: Uncorrectable data error
	- MC: Media Changed
	- IDNF: ID mark Not Found
	- MCR: Media Change Requested
	- ABRT: Command aborted
	- T0NF: Track 0 Not Found
	- AMNF: Address Mark Not Found

&lt;font color=&#34;red&#34;&gt;控制、命令块、状态和错误&lt;/font&gt;。通过使用（在 x86 上）`in`和`out` I/O 指令读取或写入特定的“I/O 地址”（例如下面的 `0x3F6`），可以使用这些寄存器。

与设备交互的基本协议如下，假设设备已经初始化。

* **等待驱动器准备就绪。**读取状态寄存器 (`0x1F7`)，直到驱动器就绪且不繁忙。
* **将参数写入命令寄存器。**将扇区计数、要访问的扇区的逻辑块地址 (LBA) 和驱动器编号（主驱动器 = 0x00 或从驱动器 = 0x10，因为 IDE 只允许两个驱动器）写入命令寄存器 (`0x1F2-0x1F6`)。
* **启动I/O。**通过向命令寄存器发出读/写命令。将 `READ—WRITE` 命令写入命令寄存器 (0x1F7)。
* **数据传输（用于写入）**：等待驱动器状态为READY 和DRQ（驱动器数据请求）；将数据写入数据端口。
* **处理中断。**在最简单的情况下，为每个传输的扇区处理一个中断；更复杂的方法允许批处理，从而在整个传输完成时进行最后一次中断。
* **错误处理。**每次操作后，读取状态寄存器。如果 ERROR 位打开，读取错误寄存器以了解详细信息。

该协议的大部分内容可以在 xv6 IDE 驱动程序中找到，如下面这段代码所示：

```c
struct buf { //chunk of 512B to read/write
    int flags;
    uint dev;
    uint sector;
    struct buf *prev; // LRU cache list
    struct buf *next;
    struct buf *qnext; // disk queue
    uchar data[512];
};

#define B_BUSY  0x1  // buffer is locked by some process
#define B_VALID 0x2  // buffer has been read from disk
#define B_DIRTY 0x4  // buffer needs to be written to disk

static int ide_wait_ready() {
    while (((int r = inb(0x1f7)) &amp; IDE_BSY) || !(r &amp; IDE_DRDY))
    ; // loop until drive isn’t busy
}

static void ide_start_request(struct buf *b) {
    ide_wait_ready();
    outb(0x3f6, 0); // generate interrupt
    outb(0x1f2, 1); // how many sectors? one
    // write LBA to command register
    outb(0x1f3, b-&gt;sector &amp; 0xff); // LBA goes here ... LBA 0-7
    outb(0x1f4, (b-&gt;sector &gt;&gt; 8) &amp; 0xff); // ... and here LBA 8-15
    outb(0x1f5, (b-&gt;sector &gt;&gt; 16) &amp; 0xff); // ... and here! LBA 16-23
    // 0x30 set LBA mode, ((b-&gt;dev&amp;1)&lt;&lt;4) 根据设备号（b-&gt;dev）来设置主/从设备位。设备号为 0 或 1，左移 4 位后会设置到控制寄存器中相应的位置。((b-&gt;sector&gt;&gt;24)&amp;0x0f) 用于提取 LBA 的第24到第27位，这些位表示扇区地址的高四位，然后通过与 0x0f 位掩码进行按位与运算来获取这些位。
    outb(0x1f6, 0xe0 | ((b-&gt;dev&amp;1)&lt;&lt;4) | ((b-&gt;sector&gt;&gt;24)&amp;0x0f));
    // B_DIRTY：缓冲区数据被修改过；B_VALID：缓冲区数据有效
    if(b-&gt;flags &amp; B_DIRTY){
        outb(0x1f7, IDE_CMD_WRITE); // this is a WRITE，告诉磁盘控制器执行写入操作
        outsl(0x1f0, b-&gt;data, 512/4); // transfer data too!写入数据寄存器，outsl函数以32位（4字节）为单位传输数据。
    } else {
        outb(0x1f7, IDE_CMD_READ); // this is a READ (no data) 告诉磁盘控制器执行读取操作。
    }
}

void ide_rw(struct buf *b) {
    acquire(&amp;ide_lock);	// 获取锁，确保对队列的访问是安全的
    for (struct buf **pp = &amp;ide_queue; *pp; pp=&amp;(*pp)-&gt;qnext)
    ; // 遍历磁盘I/O 请求队列，移动到队尾
    *pp = b; // add request to end
    if (ide_queue == b) // if q is empty，则立即发送请求
        ide_start_request(b); // send req to disk
    while ((b-&gt;flags &amp; (B_VALID|B_DIRTY)) != B_VALID) // 直到请求的数据状态变为有效为止
        sleep(b, &amp;ide_lock); // wait for completion
    release(&amp;ide_lock);
}

void ide_intr() {
    struct buf *b;
    acquire(&amp;ide_lock);
    // 检查当前请求是否是读取操作并且磁盘处于就绪状态
    if (!(b-&gt;flags &amp; B_DIRTY) &amp;&amp; ide_wait_ready() &gt;= 0)
        insl(0x1f0, b-&gt;data, 512/4); // if READ: get data
    b-&gt;flags |= B_VALID;	// set flag to VALID，表示数据已经被读取或写入
    b-&gt;flags &amp;= ~B_DIRTY;	// 清除DIRTY flag
    wakeup(b); // wake waiting process
    if ((ide_queue = b-&gt;qnext) != 0) // start next request
        ide_start_request(ide_queue); // (if one exists)
    release(&amp;ide_lock);
}
```

该驱动程序（初始化后）通过四个主要函数工作。

第一个是 `ide_rw()`，它将请求排队（如果还有其他待处理的请求），或者直接将其发送到磁盘（通过 `ide_start_request()`）；无论哪种情况，例程都会等待请求完成并将调用进程置于睡眠状态。

第二个是 `ide_start_request()`，用于向磁盘发送请求（在写入的情况下可能还有数据）； `in` 和 `out` x86 指令分别被调用来读取和写入设备寄存器。

启动请求例程使用第三个函数 `ide_wait_read()`，以确保驱动器在向其发出请求之前已准备就绪。

最后，当发生中断时，会调用 `ide_intr()`；它从设备读取数据（如果请求是读，而不是写），唤醒等待 I/O 完成的进程，并且（如果 I/O 队列中有更多请求），通过 `ide_start_request()`启动下一个 I/O。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/27.io%E8%AE%BE%E5%A4%87/  

