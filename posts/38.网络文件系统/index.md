# 网络文件系统

## 基本的分布式文件系统

分布式客户端/服务器计算最早应用于分布式文件系统领域。在这种环境中，**有多台客户机和一台（或几台）服务器**；服务器将数据存储在磁盘上，客户机通过格式良好的协议信息请求数据。下图描述了基本设置。

&lt;img src=&#34;https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/A_Generic_Client_Server_System.png&#34; alt=&#34;image-20240423230842288&#34; style=&#34;zoom:50%;&#34; /&gt;

从图中可以看出，服务器拥有磁盘，客户端通过网络发送消息来访问这些磁盘上的目录和文件。我们为什么要费心这样的安排呢？ （即，为什么我们不让客户端使用他们的本地磁盘？）嗯，&lt;font color=&#34;red&#34;&gt;主要是这种设置允许在客户端之间轻松共享数据。&lt;/font&gt;因此，如果您访问一台计算机（客户端 0）上的文件，然后使用另一台计算机（客户端 2），您将拥有相同的文件系统视图。您的数据自然会在这些不同的机器之间共享。第二个好处是**集中管理**；例如，备份文件可以从少数服务器计算机而不是多个客户端完成。另一个优势可能是安全性。将所有服务器放在上锁的机房中可以防止出现某些类型的问题。

因此关键问题是：

&gt; 如何构建分布式文件系统？需要考虑哪些关键方面？什么容易出错？我们可以从现有系统中学到什么？

我们现在将研究简化的分布式文件系统的架构。一个简单的客户端/服务器分布式文件系统比我们迄今为止研究的文件系统具有更多的组件。在客户端，有一些客户端应用程序通过**客户端文件系统**访问文件和目录。客户端应用程序向客户端文件系统发出系统调用（例如 `open()`、`read()`、`write()`、`close()`、`mkdir()` 等），以便访问存储在服务器上的文件。因此，对于客户端应用程序来说，除了性能之外，文件系统似乎与本地（基于磁盘的）文件系统没有任何不同。这样，分布式文件系统提供了对文件的**透明**访问，这是一个显而易见的目标；毕竟，谁会愿意使用一个需要不同的应用程序接口或者使用起来很麻烦的文件系统呢？

客户端文件系统的作用是执行服务这些系统调用所需的操作。例如，如果客户端发出 `read()` 请求，客户端文件系统可能会向**服务器端文件系统**（或者通常称为**文件服务器**）发送消息以读取特定块；然后，文件服务器将从磁盘（或其自己的内存缓存）读取该块，并将包含请求数据的消息发送回客户端。然后，客户端文件系统会将数据复制到提供给 `read()` 系统调用的用户缓冲区中，从而完成请求。请注意，客户端上同一块的后续 `read()` 可能会缓存在客户端内存中，甚至缓存在客户端磁盘上；在最好的情况下，不需要产生网络流量。

![image-20240423231648503](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/Distributed_File_System_Architecture.png)

从这个简单的概述中，您应该了解到客户端/服务器分布式文件系统中有两个重要的软件部分：客户端文件系统和文件服务器。它们的行为共同决定了分布式文件系统的行为。现在是时候研究一个特定的系统了：Sun 的**网络文件系统 (NFS)**。

## 网络文件系统（NFS）

Sun Microsystems 公司开发的分布式系统是最早也是相当成功的分布式系统之一，被称为 Sun 网络文件系统（或 NFS）。在定义 NFS 时，Sun 采用了一种不同寻常的方法：Sun 没有构建一个专有的封闭系统，而是开发了一个开放协议，简单地规定了客户机和服务器用于通信的确切信息格式。不同的团体可以开发自己的 NFS 服务器，从而在 NFS 市场上竞争，同时保持互操作性。这种做法取得了成功：如今有许多公司都在销售 NFS 服务器（包括 Oracle/Sun、NetApp、EMC、IBM 和其他公司），NFS 的广泛成功很可能归功于这种 &#34;开放市场 &#34;方法。

### 重点：简单快速的服务器崩溃恢复

在本章中，我们将讨论经典的 NFS 协议（版本 2，又称 NFSv2），该协议是多年来的标准；在向 NFSv3 迁移时进行了一些小改动，在向 NFSv4 迁移时进行了更大规模的协议改动。然而，NFSv2 既精彩又令人沮丧，因此成为我们关注的焦点。在 NFSv2 中，协议设计的主要目标是简单快速地恢复服务器崩溃。在多客户端、单服务器的环境中，这一目标是非常合理的；服务器宕机（或不可用）的任何一分钟都会让所有客户端机器（及其用户）不高兴，并影响工作效率。因此，服务器垮了，整个系统也就垮了。

#### 快速崩溃恢复的关键：无状态

NFSv2 通过设计无状态协议实现了这一简单目标。根据设计，服务器不会跟踪每个客户端正在发生的任何事情。例如，服务器不知道哪些客户端正在缓存哪些数据块，也不知道每个客户端当前打开了哪些文件，更不知道文件的当前文件指针位置等。简而言之，服务器不会跟踪客户端正在做的任何事情；相反，协议的设计目的是在每个协议请求中提供完成请求所需的所有信息。

有状态（非无状态）协议的一个例子是 `open()` 系统调用。给定一个路径名，`open()` 返回一个文件描述符（整数）。该描述符将用于后续的 `read()` 或 `write()` 请求，以访问各种文件块，如下面这段代码所示：

```c
char buffer[MAX];
int fd = open(&#34;foo&#34;, O_RDONLY); // get descriptor &#34;fd&#34;
read(fd, buffer, MAX); // read MAX bytes from foo (via fd)
read(fd, buffer, MAX); // read MAX bytes from foo
...
read(fd, buffer, MAX); // read MAX bytes from foo
close(fd); // close file
```

现在想象一下，客户端文件系统通过向服务器发送一条协议消息“打开文件‘foo’并给我返回一个描述符”来打开文件。然后，文件服务器在其本地打开该文件并将描述符发送回客户端。在后续读取中，客户端应用程序使用该描述符来调用 `read()` 系统调用；然后，客户端文件系统将消息中的描述符传递给文件服务器，表示“从我传递给您的描述符所引用的文件中读取一些字节”。

在这个例子中，文件描述符是客户端和服务器之间的一段**共享状态**（Ousterhout 称之为**分布式状态**）。正如我们上面所暗示的，共享状态使崩溃恢复变得复杂。想象一下，服务器在第一次读取完成后、客户端发出第二次读取之前崩溃了。服务器重新启动并运行后，客户端会发出第二次读取。不幸的是，服务器不知道 `fd` 引用的是哪个文件；该信息是短暂的（即在内存中），因此当服务器崩溃时就会丢失。为了处理这种情况，客户端和服务器必须参与某种**恢复协议**，客户端将确保在其内存中保留足够的信息，以便能够告诉服务器它需要知道什么（在这种情况下 ，该文件描述符 `fd` 引用文件 `foo`)

当您考虑有状态服务器必须处理客户端崩溃这一事实时，情况会变得更糟。例如，想象一下，一个客户端打开一个文件然后崩溃了。 `open()` 使用了服务器上的文件描述符；服务器如何知道可以关闭给定文件？在正常操作中，客户端最终会调用 `close()` ，从而通知服务器应该关闭文件。然而，当客户端崩溃时，服务器永远不会收到 `close()`，因此必须注意到客户端已崩溃才能关闭文件。

出于这些原因，NFS 的设计者决定采用无状态方法：每个客户端操作都包含完成请求所需的所有信息。不需要花哨的崩溃恢复；服务器刚刚重新开始运行，而客户端在最坏的情况下可能必须重试请求。

#### NFSv2 协议

由此，我们得出了 NFSv2 协议的定义。我们的问题陈述很简单：

&gt; &lt;center&gt;如何定义无状态文件协议
&gt;     
&gt; &lt;/center&gt;
&gt;
&gt; 如何定义网络协议以实现无状态操作？显然，像 `open()` 这样的有状态调用不能作为讨论的一部分（因为这需要服务器跟踪打开的文件）；但是，客户端应用程序会希望调用 `open()`、`read()`、`write()`、`close()` 和其他标准 API 调用来访问文件和目录。因此，作为一个细化的问题，我们该如何定义协议才能既无状态又支持 POSIX 文件系统 API 呢？

理解 NFS 协议设计的关键之一是理解**文件句柄**。文件句柄用于唯一描述特定操作要操作的文件或目录；因此，许多协议请求都包含一个文件句柄。

你可以认为文件句柄有三个重要组成部分：**卷标识符**、**inode号**和生成号；这三者共同构成了客户端希望访问的文件或目录的唯一标识符。

* 卷标识符告知服务器该请求指向哪个文件系统（一个 NFS 服务器可以导出多个文件系统）；
* Inode号告诉服务器该请求访问的是该分区中的哪个文件。
* 最后，在重复使用inode号时需要使用生成号；每当重复使用一个inode号时，服务器就会递增生成号，以确保使用旧文件句柄的客户端不会意外访问新分配的文件。

以下是协议中一些重要部分的摘要；完整的协议可从其他地方获得。

```yml
NFSPROC_GETATTR:
  # 期望：文件句柄
  # 返回：属性
  expects: file handle
  returns: attributes

NFSPROC_SETATTR:
  # 期望：文件句柄，属性
  # 返回：无
  expects: file handle, attributes
  returns: nothing

NFSPROC_LOOKUP:
  # 期望：目录文件句柄，要查找的文件/目录的名称
  # 返回：文件句柄
  expects: directory file handle, name of file/directory to look up
  returns: file handle

NFSPROC_READ:
  # 期望：文件句柄，偏移量，计数
  # 返回：数据，属性
  expects: file handle, offset, count
  returns: data, attributes

NFSPROC_WRITE:
  # 期望：文件句柄，偏移量，计数，数据
  # 返回：属性
  expects: file handle, offset, count, data
  returns: attributes

NFSPROC_CREATE:
  # 期望：目录文件句柄，文件名，属性
  # 返回：无
  expects: directory file handle, name of file, attributes
  returns: nothing

NFSPROC_REMOVE:
  # 期望：目录文件句柄，要移除的文件名
  # 返回：无
  expects: directory file handle, name of file to be removed
  returns: nothing

NFSPROC_MKDIR:
  # 期望：目录文件句柄，目录名，属性
  # 返回：文件句柄
  expects: directory file handle, name of directory, attributes
  returns: file handle

NFSPROC_RMDIR:
  # 期望：目录文件句柄，要移除的目录名
  # 返回：无
  expects: directory file handle, name of directory to be removed
  returns: nothing

NFSPROC_READDIR:
  # 期望：目录句柄，要读取的字节数，标识符
  # 返回：目录条目，标识符（以获取更多条目）
  expects: directory handle, count of bytes to read, cookie
  returns: directory entries, cookie (to get more entries)
```

我们简要介绍一下协议的重要组成部分。首先，LOOKUP 协议报文用于获取文件句柄，然后使用该句柄访问文件数据。客户端传递**一个目录文件句柄和要查找的文件名**，服务器会将该文件（或目录）的句柄及其属性传回客户端。

例如，假设客户端已经拥有文件系统根目录 (`/`) 的目录文件句柄（实际上，这可以通过 NFS **挂载协议**获得，这是客户端和服务器首次连接的方式；为简洁起见，我们在此不讨论挂载协议）。如果在客户端运行的应用程序打开文件 `/foo.txt`，客户端文件系统就会向服务器发送一个查找请求，将根文件句柄和文件名 `foo.txt` 传递给服务器；如果请求成功，就会返回 `foo.txt` 的文件句柄（和属性）。

如果你想知道，属性只是文件系统跟踪每个文件的元数据，包括文件创建时间、最后修改时间、大小、所有权和权限信息等字段，也就是在文件上调用 `stat()` 时会返回的信息。

一旦有了文件句柄，客户端就可以对文件发出 READ 和 WRITE 协议消息，分别读取或写入文件。读取协议消息要求传递文件句柄、文件偏移量和要读取的字节数。然后，服务器就能发出读取命令（毕竟，句柄会告诉服务器要从哪个卷和哪个 inode 读取，偏移量和字节数会告诉服务器要读取文件的哪个字节），并将数据返回给客户端（如果读取失败，则返回错误信息）。写入（WRITE）的处理方式与此类似，只是数据从客户端传递到服务器，并只返回一个成功代码。

最后一个有趣的协议信息是 GETATTR 请求；给定一个文件句柄后，它只需获取该文件的属性，包括文件的最后修改时间。在下文讨论缓存时，我们将看到该协议请求在 NFSv2 中的重要性。

### 从协议到分布式文件系统

希望您现在已经了解如何将该协议转变为跨客户端文件系统和文件服务器的文件系统。&lt;font color=&#34;red&#34;&gt;客户端文件系统跟踪打开的文件，并且通常将应用程序请求转换为相关的协议消息集。服务器只是响应协议消息，每个消息都包含完成请求所需的所有信息。&lt;/font&gt;

例如，让我们考虑一个读取文件的简单应用程序。在下图中，我们显示了应用程序进行了哪些系统调用，以及客户端文件系统和文件服务器响应此类调用时执行的操作。

![image-20240424131608950](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/NFS_Reading_A_File.png)

首先，注意客户端如何跟踪文件访问的所有相关状态，包括**整数文件描述符**到 **NFS 文件句柄**的映射以及当前文件指针。这使得客户端能够将每个读取请求（您可能已经注意到，没有明确指定要读取的偏移量）转换为格式正确的读取协议消息，该消息准确地告诉服务器要读取文件中的哪些字节。成功读取后，客户端更新当前文件位置；后续读取将使用相同的文件句柄但偏移量不同。

其次，您可能会注意到服务器交互发生的位置。当文件第一次打开时，客户端文件系统会发送一个LOOKUP请求消息。事实上，如果必须遍历长路径名（例如，`/home/zfhe/foo.txt`），客户端将发送三个 LOOKUP：一个在目录 `/` 中查找 `home`，一个在 `home` 中查找 `zfhe`，最后一个在 `zfhe` 中查找 `foo.txt`。

第三，您可能会注意每个服务器请求都包含完成请求所需的全部信息。这个设计要点对于从服务器故障中从容恢复至关重要，我们现在将详细讨论；它确保服务器不需要状态就能响应请求。

### 使用幂等操作处理服务器故障

客户端向服务器发送信息时，有时会收不到回复。造成这种无法回复的原因有很多。在某些情况下，信息可能被网络丢弃；网络确实会丢失信息，因此请求或回复都可能丢失，这样客户端就永远不会收到回复。

也有可能是服务器崩溃了，因此目前没有响应信息。一段时间后，服务器将重新启动并重新开始运行，但在此期间，所有请求都已丢失。在所有这些情况下，客户端都会遇到一个问题：当服务器未能及时回复时该怎么办？

在 NFSv2 中，客户端以一种单一、统一和优雅的方式处理所有这些故障：&lt;font color=&#34;red&#34;&gt;只需重试请求即可。&lt;/font&gt;具体来说，在发送请求后，客户端会设置一个计时器，在指定时间段后关闭。如果在定时器关闭前收到了回复，定时器就会被取消，一切正常。但是，如果在收到任何回复之前计时器就关闭了，客户端就会认为请求没有被处理，并重新发送请求。如果服务器回复了，则一切正常，客户端也顺利地解决了问题。

客户端之所以能简单地重试请求（不管失败的原因是什么），是因为大多数 NFS 请求都有一个重要特性：它们都是**幂等**的。&lt;font color=&#34;red&#34;&gt;当多次执行该操作的效果等同于单次执行该操作的效果时，我们就称该操作为幂等操作。&lt;/font&gt;例如，如果将一个值存储到内存位置三次，与存储一次的效果相同；因此，&#34;将值存储到内存 &#34;就是一个幂等操作。但是，如果将一个计数器递增三次，其结果与只递增一次的结果不同，因此 &#34;递增计数器 &#34;不是幂等操作。&lt;font color=&#34;red&#34;&gt;更一般地说，任何只读取数据的操作显然都是幂等的，而更新数据的操作则必须更仔细地考虑，以确定它是否具有这种特性。&lt;/font&gt;

NFS 崩溃恢复设计的核心是大多数常见操作的幂等性。LOOKUP 和 READ 请求具有微不足道的幂等性，因为它们只从文件服务器读取信息，而不更新信息。更有趣的是，WRITE 请求也是幂等的。例如，如果 WRITE 失败，客户端只需重试即可。&lt;font color=&#34;red&#34;&gt;WRITE 消息包含数据、计数和（重要的）要写入数据的准确偏移量。&lt;/font&gt;因此，只要知道多次写入的结果与单次写入的结果相同，就可以重复写入。

![image-20240424140254771](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/The_Three_Types_Of_Loss.png)

这样，客户端就能以统一的方式处理所有超时。

* 如果只是丢失了 WRITE 请求（上述情况 1），客户端将重试，服务器将执行写入操作，一切正常。
* 同样的情况也会发生，如果在发送请求时服务器碰巧宕机，但在发送第二个请求时又恢复运行，那么一切都会如愿以偿（情况 2）。
* 最后，服务器可能会接收到写入请求，向磁盘发出写入操作，并发送回复。这个回复可能会丢失（情况 3），再次导致客户机重新发送请求。当服务器再次收到请求时，它只会做完全相同的事情：将数据写入磁盘并回复说已经写入。如果这次客户端收到了回复，那么一切又都正常了，这样客户端就以统一的方式处理了信息丢失和服务器故障。

不过有些操作很难做到幂等。例如，当你试图创建一个已经存在的目录时，你会被告知 mkdir 请求失败。因此，在 NFS 中，如果文件服务器接收到 MKDIR 协议信息并成功执行，但却丢失了回复，那么客户端可能会重复该操作并遭遇失败，而实际上该操作一开始是成功的，只是在重试时失败了。因此，生活并不完美。

&gt; &lt;center&gt;TIP：幂等性非常强大 
&gt;     
&gt; &lt;/center&gt;
&gt;
&gt; 在构建可靠系统时，幂等性是一个有用的属性。当一个操作可以多次发出时，处理操作失败就容易得多；你可以重试。如果操作不是幂等的，生活就会变得更加困难。

## 提高性能：客户端缓存

### 基本介绍

分布式文件系统的优点有很多，但通过网络发送所有读写请求可能会导致很大的性能问题：网络通常速度不快，尤其是与本地内存或磁盘相比。那么，又一个问题：如何提高分布式文件系统的性能？

答案是**客户端缓存**。 NFS 客户端文件系统将从服务器读取的文件数据（和元数据）缓存在客户端内存中。因此，虽然第一次访问的成本很高（即，它需要网络通信），但后续访问很快就会从客户端内存中得到服务。

高速缓存还充当写入的临时缓冲区。当客户端应用程序首次写入文件时，客户端会先将数据缓冲在客户端内存中（与从文件服务器读取的数据位于同一缓存中），然后再将数据写出到服务器。这种写入缓冲非常有用，因为它将应用程序 `write()` 延迟与实际写入性能解耦，即应用程序对 `write()` 的调用立即成功（并且只是将数据放入客户端文件系统的缓存中）；只有稍后数据才会被写出到文件服务器。

因此，NFS 客户端缓存数据并且性能通常很好，我们就完成了，对吗？不幸的是，不完全是。将缓存添加到具有多个客户端缓存的任何类型的系统中都会带来一个巨大且有趣的挑战，我们将其称为**缓存一致性问题**。

### 缓存一致性问题

缓存一致性问题最好用两个客户端和一个服务器来说明。假设客户端 C1 读取文件 F，并在其本地缓存中保留该文件的副本。现在想象一个不同的客户端 C2 覆盖文件 F，从而更改其内容；我们将文件的新版本称为 F（版本 2）或 F[v2]，将旧版本称为 F[v1]，这样我们就可以保持两者不同（当然，文件具有相同的名称，只是内容不同）。最后，还有第三个客户端 C3，它尚未访问文件 F。

您可能会看到即将出现的问题，如下图所示。

![image-20240424142910926](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/NFS_The_Cache_Consistency_Problem.png)

事实上，有两个子问题。

* 第一个子问题是客户端 C2 在将其写入传播到服务器之前可能会在其缓存中缓冲一段时间；在这种情况下，虽然 F[v2] 位于 C2 的内存中，但来自另一个客户端（例如 C3）对 F 的任何访问都将获取文件的旧版本 (F[v1])。因此，通过在客户端缓冲写入，其他客户端可能会获得该文件的过时版本，这可能是不合需要的；事实上，想象一下这样的情况：您登录到机器 C2，更新 F，然后登录到 C3 并尝试读取文件，结果却得到旧副本！当然，这可能会令人沮丧。因此，我们将这方面的缓存一致性问题称为&lt;font color=&#34;red&#34;&gt;更新可见性&lt;/font&gt;；一个客户端的更新何时对其他客户端可见？
* 缓存一致性的第二个子问题是&lt;font color=&#34;red&#34;&gt;陈旧的缓存&lt;/font&gt;；在这种情况下，C2 最终将其写入刷新到文件服务器，因此服务器具有最新版本（F[v2]）。然而，C1 的缓存中仍然有 F[v1]；如果在 C1 上运行的程序读取文件 F，它将获得陈旧版本 (F[v1])，而不是最新副本 (F[v2])，这（通常）是不可取的。

NFSv2 实现通过两种方式解决这些缓存一致性问题。

* 首先，为了解决更新可见性，客户端实现有时称为“关闭时刷新”（也称为“关闭到打开”）一致性语义；具体来说，&lt;font color=&#34;red&#34;&gt;当客户端应用程序写入文件并随后关闭文件时，客户端会将所有更新（即缓存中的脏页）刷新到服务器&lt;/font&gt;。通过关闭时刷新一致性，NFS 可确保后续从另一个节点打开时将看到最新的文件版本。
* 其次，为了解决陈旧缓存问题，NFSv2 客户端在使用其缓存内容之前首先检查文件是否已更改。具体来说，&lt;font color=&#34;red&#34;&gt;在使用缓存块之前，客户端文件系统将向服务器发出 GETATTR 请求以获取文件的属性。&lt;/font&gt;重要的是，属性包括有关文件最后一次在服务器上修改的时间的信息；如果修改时间比文件被提取到客户端缓存的时间更新，则客户端会使该文件无效，从而将其从客户端缓存中删除，并确保后续读取将转到服务器并检索最新的文件的版本。另一方面，如果客户端发现它具有该文件的最新版本，它将继续使用缓存的内容，从而提高性能。

当 Sun 的原始团队针对陈旧缓存问题实施此解决方案时，他们意识到了一个新问题：突然，NFS 服务器被 GETATTR 请求淹没。遵循的一个良好的工程原则是针对常见情况进行设计，并使其运行良好；在这里，尽管常见情况是仅从单个客户端访问文件（可能重复），**但客户端始终必须向服务器发送 GETATTR 请求以确保没有其他人更改该文件。**因此，客户端轰炸服务器，不断询问“有人更改了这个文件吗？”，而大多数时候没有人更改过。

为了（在某种程度上）解决这种情况，每个客户端都添加了属性缓存。客户端在访问文件之前仍然会验证文件，但大多数情况下只会查看属性缓存以获取属性。特定文件的属性在第一次访问该文件时被放置在缓存中，然后在一定时间（例如 3 秒）后超时。因此，在这三秒钟内，所有文件访问都将确定可以使用缓存的文件，从而在与服务器没有网络通信的情况下执行此操作。

### 评估 NFS 缓存一致性

关于 NFS 缓存一致性的最后几句话。添加关闭时刷新行为是为了“有意义”，但引入了一定的性能问题。具体来说，如果在客户端上创建了临时或短暂的文件，然后很快将其删除，它仍然会被强制发送到服务器。更理想的实现可能会将这些短暂的文件保留在内存中，直到它们被删除，从而完全消除服务器交互，也许会提高性能。

更重要的是，在 NFS 中添加属性缓存使得人们很难理解或推断到底获得的文件版本是什么。有时你会得到最新版本；有时，您会得到旧版本，只是因为您的属性缓存尚未超时，因此客户端很乐意为您提供客户端内存中的内容。尽管这在大多数情况下都很好，但它偶尔会（而且仍然如此！）导致奇怪的行为。这样我们就描述了 NFS 客户端缓存的奇怪之处。它是一个有趣的例子，其中实现的细节用于定义用户可观察的语义，而不是相反。

## 对服务器端写缓冲的影响

到目前为止，我们的重点一直放在客户端缓存上，这也是最有趣的问题所在。不过，NFS 服务器往往也是拥有大量内存的装备精良的机器，因此它们也有缓存问题。当从磁盘读取数据（和元数据）时，NFS 服务器会将其保存在内存中，随后对数据（和元数据）的读取将不会转到磁盘，这有可能（小幅）提高性能。

更有趣的是写缓冲。NFS 服务器在强制写入稳定存储（如磁盘或其他持久性设备）之前，绝对不会返回成功的 WRITE 协议请求。虽然它们可以在服务器内存中放置数据副本，但向客户端返回 WRITE 协议请求成功可能会导致不正确的行为。

答案就在于我们对客户端如何处理服务器故障的假设。想象一下客户端发出的以下写入序列：

```c
write(fd, a_buffer, size); // fill first block with a’s
write(fd, b_buffer, size); // fill second block with b’s
write(fd, c_buffer, size); // fill third block with c’s
```

这些写入会先用 a 块、b 块和 c 块覆盖文件的三个块。因此，如果文件最初看起来像这样：

```
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz
```

我们可能期望这些写入后的最终结果是这样的，其中 x、y 和 z 将分别被 a、b 和 c 覆盖。

```
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc
```

现在，为了举例，我们假设这三个客户端写入作为三个不同的 WRITE 协议消息发送到服务器。假设服务器收到第一个 WRITE 消息并将其发送到磁盘，并且客户端通知其成功。现在假设第二次写入只是缓冲在内存中，并且服务器在将其强制写入磁盘之前也会向客户端报告其成功；不幸的是，服务器在将其写入磁盘之前崩溃了。服务器很快重启，收到第三个写请求，也成功了。

因此，对于客户端来说，所有请求都成功了，但令我们惊讶的是文件内容如下所示：

```
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy &lt;--- oops
cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc
```

因为服务器在将第二次写入提交到磁盘之前告诉客户端第二次写入成功，所以文件中会留下旧的块，这可能是灾难性的，具体取决于应用程序。

为了避免这个问题，NFS 服务器必须将每次写入提交到稳定（持久）存储，然后再通知客户端成功；这样做使客户端能够在写入期间检测到服务器故障，从而重试直到最终成功。这样做可以确保我们永远不会像上面的示例一样最终出现文件内容混合的情况。

这一要求在 NFS 服务器实现中引起的问题是，如果不小心的话，写入性能可能会成为主要的性能瓶颈。事实上，一些公司（例如 Network Appliance）的成立只是为了构建一个可以快速执行写入的 NFS 服务器；他们使用的一个技巧是首先将**写入操作放入电池支持的内存中**，从而能够快速回复写入请求，而不必担心丢失数据，并且无需立即写入磁盘；第二个技巧是使用专门设计的文件系统设计，以便在最终需要时快速写入磁盘。

## 总结

* 在 NFS 中实现快速、简单的崩溃恢复这一主要目标的关键在于**无状态协议的设计**。崩溃后，服务器可以快速重新启动并再次开始服务请求；客户端只需**重试**请求，直到成功为止。
* 使请求具有**幂等性**是NFS 协议的一个核心方面。当多次执行某个操作的效果与执行一次相同时，该操作就是幂等的。在NFS中，幂等性使客户端能够无忧重试，并统一客户端丢失消息重传以及客户端处理服务器崩溃的方式。
* 性能问题决定了对**客户端缓存**和**写缓冲**的需求，但会带来**缓存一致性问题**。
* NFS 实现提供了一种通过多种方式实现缓存一致性的工程解决方案：**关闭时刷新（关闭到打开）**方法可确保当文件关闭时，其内容被强制传输到服务器，从而使其他客户端能够观察到文件的更新。属性缓存减少了向服务器检查文件是否已更改的频率（通过 GETATTR 请求）。
* &lt;font color=&#34;red&#34;&gt;NFS 服务器必须在返回成功之前向持久介质提交写入；否则，可能会导致数据丢失。&lt;/font&gt;
* 为了支持NFS 集成到操作系统中，Sun 引入了**VFS/Vnode** 接口，使多个文件系统实现能够在同一操作系统中共存。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/38.%E7%BD%91%E7%BB%9C%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F/  

