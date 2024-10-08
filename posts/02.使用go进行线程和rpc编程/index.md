# 【MIT 6.5840(6.824)学习笔记】使用Go进行线程和RPC编程

## 为什么选择Go

在实现分布式系统时，选择合适的编程语言非常重要。Go有以下特点：

* 优秀的线程支持；
* 便捷的RPC机制、类型；
* 内存安全以及垃圾回收机制。

这使Go成为了一个理想的选择。Go不仅相对简单，而且其垃圾回收机制使线程管理更加容易，避免了使用后释放问题。由于这些优势，Go在分布式系统中被广泛应用。

[Go Tutorial](https://golang.org/doc/effective_go.html)

## 线程与Go中的Goroutine

线程是一种有用的结构工具，允许一个程序同时执行多项任务，每个线程串行执行，就像非线程程序一样。Go中称线程为Goroutine，每个Goroutine在执行时包含自己的程序计数器、寄存器和栈，但共享内存。使用线程可以提高I/O并发性和多核性能，同时也方便后台任务的处理。

### 为什么使用线程？

1. **I/O并发性**：客户端可以并行向多个服务器发送请求并等待回复，服务器可以同时处理多个客户端请求。
2. **多核性能**：在多核处理器上并行执行代码，提高计算效率。
3. **便捷性**：后台线程可以定期检查各个`worker`线程是否仍然活跃。

### 线程的替代方案

事件驱动编程（Event-driven programming）是一种替代传统多线程编程的方式，通过在单线程中显式交错处理活动来实现I/O并发性。这种编程模型常用于处理大量I/O操作的场景，例如网络服务器和图形用户界面（GUI）应用。

在事件驱动编程中，系统维护一个事件循环（event loop），不断检查并处理事件队列中的事件。每个事件通常对应某种外部输入或状态变化，如网络请求到达、用户点击按钮或定时器到期。事件处理程序（event handler）被注册到特定事件上，当相应事件发生时，处理程序被调用来执行预定义的操作。

### 线程编程挑战

1. **安全共享数据**：多个线程同时访问共享数据时可能导致竞争条件，常用的解决方案是使用锁（如Go的`sync.Mutex`）或者避免共享可变数据。
2. **线程间协调**：一个线程生产数据，另一个线程消费数据，需要使用Go的通道（`channel`）或条件变量（`sync.Cond`）或等待组（`sync.WaitGroup`）进行协调。
3. **死锁**：线程之间通过锁或`channel`或RPC相互等待资源时可能导致死锁，需要小心避免。

## 以网络爬虫为例的线程应用

网络爬虫的目标是抓取所有网页内容，常见的实现方式有串行和并发两种。并发爬虫利用线程提高抓取效率，但也需要解决避免重复抓取和循环依赖的问题。

在本例中，我们使用一个填充的`Fetcher`来模拟抓取。`fetcher`结构如下：

```go
type fakeResult struct {
	body string
	urls []string
}

func (f fakeFetcher) Fetch(url string) ([]string, error) {
	if res, ok := f[url]; ok {
		fmt.Printf(&#34;found:   %s\n&#34;, url)
		return res.urls, nil
	}
	fmt.Printf(&#34;missing: %s\n&#34;, url)
	return nil, fmt.Errorf(&#34;not found: %s&#34;, url)
}

// fetcher is a populated fakeFetcher.
var fetcher = fakeFetcher{
	&#34;http://golang.org/&#34;: &amp;fakeResult{
		&#34;The Go Programming Language&#34;,
		[]string{
			&#34;http://golang.org/pkg/&#34;,
			&#34;http://golang.org/cmd/&#34;,
		},
	},
	&#34;http://golang.org/pkg/&#34;: &amp;fakeResult{
		&#34;Packages&#34;,
		[]string{
			&#34;http://golang.org/&#34;,
			&#34;http://golang.org/cmd/&#34;,
			&#34;http://golang.org/pkg/fmt/&#34;,
			&#34;http://golang.org/pkg/os/&#34;,
		},
	},
	&#34;http://golang.org/pkg/fmt/&#34;: &amp;fakeResult{
		&#34;Package fmt&#34;,
		[]string{
			&#34;http://golang.org/&#34;,
			&#34;http://golang.org/pkg/&#34;,
		},
	},
	&#34;http://golang.org/pkg/os/&#34;: &amp;fakeResult{
		&#34;Package os&#34;,
		[]string{
			&#34;http://golang.org/&#34;,
			&#34;http://golang.org/pkg/&#34;,
		},
	},
}
```

### 串行爬虫

串行爬虫通过递归调用实现深度优先搜索，使用一个共享的map记录已抓取的URL，防止重复抓取。然而，这种方式只能一次抓取一个页面，速度较慢。代码如下：

```go
func Serial(url string, fetcher Fetcher, fetched map[string]bool) {
	// Use a map to keep track of fetched URLs
	if fetched[url] {
		return
	}
	fetched[url] = true
	urls, err := fetcher.Fetch(url)
	if err != nil {
		return
	}
	// Recursively fetch URLs
	for _, u := range urls {
		Serial(u, fetcher, fetched)
	}
	return
}
```

&gt; 我们是否可以在`Serial()`调用前面放一个`go`呢

在 `Serial()` 调用前添加 `go` 关键字会导致并发执行多个爬虫任务，从而可能导致重复抓取相同的页面。这是因为每个爬虫任务都会尝试从未抓取过的页面开始递归抓取，而并发执行可能导致多个爬虫同时选择相同的页面作为起始点，进而重复抓取。

### 并发爬虫

### 使用锁

每个页面的抓取在独立的线程中进行，为了确保抓取过程的正确性，我们使用了互斥锁来保护共享的 `fetchState` 结构体，避免了重复抓取和并发冲突的问题。在抓取过程中，我们使用了递归调用 `ConcurrentMutex` 函数来处理当前页面的所有子链接。每当发现一个新的子链接时，我们启动一个新的 Goroutine 来并发地抓取该链接，从而实现了多个页面的并行抓取。使用 `sync.WaitGroup` 来等待所有的子链接抓取任务完成，确保主线程在所有任务完成后才返回，以避免提前结束抓取过程。代码如下：

```go
type fetchState struct {
	mu      sync.Mutex      // protect concurrent crawls
	fetched map[string]bool // Used to store crawled URLs. The key is URL and the value is whether it has been crawled.
}

func (fs *fetchState) testAndSet(url string) bool {
	fs.mu.Lock()
	defer fs.mu.Unlock()
	r := fs.fetched[url]
	fs.fetched[url] = true
	return r // Return to previous crawling status
}

func ConcurrentMutex(url string, fetcher Fetcher, fs *fetchState) {
	if fs.testAndSet(url) {
		return
	}
	urls, err := fetcher.Fetch(url)
	if err != nil {
		return
	}
	var done sync.WaitGroup // Create a wait group that waits for all subtasks to complete
	for _, u := range urls {
		done.Add(1)         // Increase the counter of the waiting group
		go func(u string) { // Start a Go coroutine to concurrently crawl sub-links
			defer done.Done()
			ConcurrentMutex(u, fetcher, fs) // Recursive call ConcurrentMutex, fetching sub-links.
		}(u)
	}
	done.Wait() // Wait for all subtasks to complete
	return
}
```

### 使用通道

每个`worker`线程将抓取到的URL发送到一个通道，`coordinator`从通道中读取URL并启动新的`worker`线程。这种方式避免了锁的使用，但需要小心避免通道阻塞导致的死锁。代码如下：

```go
func worker(url string, ch chan []string, fetcher Fetcher) {
	urls, err := fetcher.Fetch(url)
	if err != nil {
		ch &lt;- []string{}
	} else {
		ch &lt;- urls
	}
}

func coordinator(ch chan []string, fetcher Fetcher) {
	n := 1                           // 记录正在处理的任务数量，初始值为 1，因为最开始只有一个初始 URL
	fetched := make(map[string]bool) // 记录已经抓取的 URL
	for urls := range ch {           // 不断从通道中接收抓取到的链接列表，直到通道被关闭
		for _, u := range urls { // 遍历接收到的链接列表
			if fetched[u] == false {
				fetched[u] = true
				n &#43;= 1
				go worker(u, ch, fetcher) // 启动一个新的 worker 协程抓取该链接的子链接
			}
		}
		n -= 1
		if n == 0 { // 如果当前没有正在处理的任务，则退出循环，结束并发抓取过程
			break
		}
	}
}

// ConcurrentChannel 函数是并发抓取的入口函数，利用通道协调并发抓取的过程。
func ConcurrentChannel(url string, fetcher Fetcher) {
	ch := make(chan []string) // 创建一个字符串切片类型的通道
	go func() {               // 启动一个匿名函数的 Go 协程，用于向通道发送 URL
		ch &lt;- []string{url} // 向通道发送包含初始 URL 的字符串切片
	}()
	coordinator(ch, fetcher) // 调用 coordinator 函数，开始并发抓取的协调过程
}
```

在 `worker` 函数中，每个`worker`线程会尝试抓取指定的URL，并将抓取到的子链接发送到通道中。如果抓取失败，则发送一个空的字符串切片到通道，以便通知`coordinator`任务失败。

在 `coordinator` 函数中，`coordinator`不断从通道中读取抓取到的链接列表，然后遍历这些链接，如果发现之前未抓取过的新链接，则将其标记为已抓取并启动一个新的`worker`线程进行抓取。同时，`coordinator`会维护一个计数器 `n` 来记录当前正在处理的任务数量，当所有任务都处理完成后，`coordinator`结束并发抓取的过程。

在 `ConcurrentChannel` 函数中，我们首先创建了一个字符串切片类型的通道，并启动了一个匿名的 Goroutine 来向通道发送初始的 URL。然后，调用 `coordinator` 函数开始并发抓取的协调过程。

&gt; 1. `coordinator`如何知道它已经完成？
&gt;
&gt; 	`coordinator`知道它已完成的条件是 `n` 计数器的值归零。`coordinator`通过维护 `n` 计数器来跟踪当前正在处理的任务数量，每个`worker`线程处理完成后会将 `n` 减一。当 `n` 计数器的值为零时，表示所有的任务都已经完成，`coordinator`就知道自己的工作已经完成。
&gt;
&gt; 2. 通道在这里有两个作用：
&gt;
&gt; 	- 通信值：`worker`线程将抓取到的链接列表发送到通道中，以便`coordinator`可以读取并处理。
&gt; 	- 事件通知：通道的关闭可作为事件通知，当通道关闭时，`coordinator`会知道所有的`worker`线程都已完成，并且没有新的任务需要处理。

## 远程过程调用（RPC）

### 介绍

### 远程过程调用（RPC）

远程过程调用（RPC）是分布式系统中的关键技术之一，它使得客户端和服务器之间的通信变得简单而直观。在分布式系统中，不同的节点可能分布在不同的物理机器上，RPC允许这些节点之间进行远程通信，就像调用本地函数一样，无需了解底层的网络协议细节。

RPC的目标是实现易于编程的客户端/服务器通信，它隐藏了底层网络通信的复杂性，为开发人员提供了简单的接口。通过RPC，开发人员可以专注于业务逻辑的实现，而无需担心网络通信的细节。

在RPC中，数据在客户端和服务器之间通过网络传输，因此需要将数据转换为“有线格式”（wire format）。RPC库负责处理数据的序列化和反序列化，以确保数据可以在网络上传输并在另一端正确解析。

RPC消息的基本结构是请求-响应模式。&lt;font color=&#34;red&#34;&gt;客户端发送请求给服务器，服务器处理请求并发送响应给客户端。&lt;/font&gt;这种简单的请求-响应模式使得RPC成为了一种非常有效的通信方式。

在RPC的软件结构中，通常会有以下几个组件：

- 客户端应用程序：负责发起RPC请求的应用程序。
- 存根函数（Stub functions）：客户端应用程序调用的接口函数，实际上是一个本地代理，负责将RPC调用转发给远程服务器。
- 服务器处理函数（Handler functions）：服务器端实际执行业务逻辑的函数。
- 调度器（Dispatcher）：负责将RPC请求分发给正确的处理函数。
- RPC库：提供了RPC通信所需的基本功能，例如序列化、网络通信等。

通过RPC，不同语言编写的客户端和服务器可以进行通信，实现了跨语言的可移植性和互操作性。

### Go的RPC实现

在Go中，实现RPC需要定义请求和回复的结构体，并使用Go的RPC库来处理通信。下面是一个示例，展示了如何在Go中实现一个简单的键值存储服务器（key/value storage server），并使用RPC进行通信。

* 请求回复结构体

	在键值存储服务器的示例中，我们定义了用于Put和Get操作的请求和回复结构体：

	```go
	type PutArgs struct {
		Key   string
		Value string
	}
	
	type PutReply struct {
	}
	
	type GetArgs struct {
		Key string
	}
	
	type GetReply struct {
		Value string
	}
	```

* 服务器端（Server）

	在服务器端，首先需要定义一个对象，并在该对象上注册处理函数作为RPC处理程序。这些处理函数将处理客户端发送的RPC请求。服务器接受TCP连接并将其传递给RPC库。RPC库负责读取每个请求，并为每个请求创建一个新的Goroutine进行处理。处理函数会读取请求参数，并根据请求调用相应的方法。处理完请求后，服务器将回复信息进行序列化，并通过TCP连接发送回客户端。&lt;font color=&#34;red&#34;&gt;服务器端的处理函数必须使用锁进行同步，因为RPC库为每个请求创建了一个新的Goroutine。&lt;/font&gt;处理函数需要读取请求参数并修改回复信息，因此需要确保并发访问的安全性。

	```go
	type KV struct {
		mu   sync.Mutex // 互斥锁，保护数据并发访问
		data map[string]string
	}
	
	func server() {
		kv := &amp;KV{data: map[string]string{}} // 创建键值存储服务器实例
		rpcs := rpc.NewServer()              // 创建一个 RPC 服务器
		rpcs.Register(kv)                    // 注册 kv 为 RPC 服务器的服务对象
		l, e := net.Listen(&#34;tcp&#34;, &#34;:1234&#34;)   // 监听 TCP 端口 1234
		if e != nil {
			log.Fatal(&#34;listen error:&#34;, e)
		}
		go func() { // 启动一个协程来处理客户端连接
			for {
				conn, err := l.Accept() // 接受客户端连接
				if err == nil {
					go rpcs.ServeConn(conn) // 启动一个协程来为客户端提供服务
				} else {
					break
				}
			}
			l.Close() // 关闭监听器
		}()
	}
	
	// Get 方法用于处理客户端发送的 Get 请求，获取指定键的值。
	func (kv *KV) Get(args *GetArgs, reply *GetReply) error {
		kv.mu.Lock()
		defer kv.mu.Unlock()
	
		reply.Value = kv.data[args.Key]
	
		return nil
	}
	
	// Put 方法用于处理客户端发送的 Put 请求，存储键值对。
	func (kv *KV) Put(args *PutArgs, reply *PutReply) error {
		kv.mu.Lock()
		defer kv.mu.Unlock()
	
		kv.data[args.Key] = args.Value
	
		return nil
	}
	```

* 客户端（Client）

	在客户端，首先需要使用Dial函数建立与服务器的TCP连接。然后，客户端需要定义RPC请求的参数结构体和回复结构体，并实现对应的处理函数。客户端通过调用Call函数发起RPC调用，指定连接、函数名称、参数以及存放回复的位置。RPC库负责对参数进行序列化，并将请求发送给服务器。然后，客户端等待并接收服务器的回复，并将回复反序列化为指定的回复结构体。Call函数的返回值指示是否成功接收到了回复，通常还包括服务级别的错误信息。

	```go
	// connect 函数用于与键值存储服务器建立连接，并返回一个 RPC 客户端对象。
	func connect() *rpc.Client {
		client, err := rpc.Dial(&#34;tcp&#34;, &#34;:1234&#34;) // 使用 TCP 协议连接服务器
		if err != nil {
			log.Fatal(&#34;dialing:&#34;, err) // 如果连接失败，则记录错误并终止程序
		}
		return client
	}
	
	func get(key string) string {
		client := connect()                         // 建立连接
		args := GetArgs{key}                        // 构造 Get 请求的参数
		reply := GetReply{}                         // 准备接收服务器的响应
		err := client.Call(&#34;KV.Get&#34;, &amp;args, &amp;reply) // 调用远程方法 Get，并传递参数 args，将响应写入 reply
		if err != nil {
			log.Fatal(&#34;error:&#34;, err)
		}
		client.Close()     // 关闭连接
		return reply.Value // 返回服务器返回的值
	}
	
	// put 函数用于向键值存储服务器发送 Put 请求，存储键值对。
	func put(key string, val string) {
		client := connect()
		args := PutArgs{key, val}
		reply := PutReply{}
		err := client.Call(&#34;KV.Put&#34;, &amp;args, &amp;reply)
		if err != nil {
			log.Fatal(&#34;error:&#34;, err)
		}
		client.Close()
	}
	```

其他细节：

- 绑定（Binding）：客户端如何知道要与哪台服务器通信？

	在Go的RPC中，服务器的名称和端口是Dial函数的参数。在大型系统中，通常会有一种名称或配置服务器来管理这些信息。

- 序列化（Marshalling）：数据格式化为数据包。Go的RPC库可以传递字符串、数组、对象、映射等类型的数据。Go通过复制指向的数据来传递指针，但不能传递通道或函数。RPC库只序列化导出字段（即大写字母开头的字段）。

### 处理RPC中的失败

在分布式系统中，网络故障和服务器故障是不可避免的。简单的解决方案是“尽力而为”的RPC，即在超时后重试请求，但这可能导致重复操作。

这种方法的缺点是，重试请求可能导致操作被重复执行。例如：

```go
client.Put(&#34;k&#34;, 10)
client.Put(&#34;k&#34;, 20)
```

如果第二个`Put`请求因网络故障重试多次，可能会导致不一致的结果：

更好的解决方案是“至多一次”的RPC，“至多一次”的RPC通过以下机制实现更可靠的行为：

- 客户端在未收到响应时重新发送请求。
- 服务器检测重复请求，并返回之前的响应，而不是重新执行处理函数。

为了检测重复请求，客户端在每个请求中包含一个唯一的ID（XID）。每个请求使用相同的 XID 重新发送服务器：

```
if seen[xid] {
  reply = old[xid]
} else {
  reply = handler()
  old[xid] = reply
  seen[xid] = true
}
```

**一些“至多一次”复杂性问题：**

- 如果两个客户端使用相同的XID？

	- 解决方案：使用大随机数生成唯一ID。

- 如何避免seen[xid]表过大？

	- 每个客户端有一个唯一ID，使用序列号。
	- 客户端在每次RPC中包含“已见到的最大回复”信息，类似于TCP序列号和确认号。

- **服务器崩溃和重启：**

	- 如果“至多一次”信息保存在内存中，服务器重启后将忘记这些信息，可能会接受重复请求。

		解决方案：将重复检测信息写入磁盘，或使用复制服务器同步这些信息。

Go的RPC库是“至多一次”策略的简单实现：

- 打开TCP连接。
- 将请求写入TCP连接。
- Go RPC从不重发请求，因此服务器不会看到重复请求。
- 如果未收到回复，Go RPC代码返回错误（可能是由于TCP超时）。

&gt; 关于“恰好一次”的RPC
&gt;
&gt; “恰好一次”的RPC包括无限重试、重复检测和容错服务，这种方法更复杂，在实际系统中需要实现容错机制。
&gt;
&gt; 例如，lab 4中将探讨“恰好一次”RPC的实现。

## FAQ

&gt; 1. Go通道是如何工作的？Go如何确保它们在多个goroutines之间同步？
&gt;
&gt; 	可以在[这里](https://go.dev/src/runtime/chan.go)查看源码，尽管不易理解。高层次上，&lt;font color=&#34;red&#34;&gt;通道是一个包含缓冲区和锁的结构体&lt;/font&gt;。发送到通道涉及获取锁，等待（可能释放CPU）直到某个线程接收，并交付消息。接收涉及获取锁并等待发送者。可以使用Go的sync.Mutex和sync.Cond自己实现通道。
&gt;
&gt; 2. 我使用通道唤醒另一个goroutine，通过在通道上发送一个虚拟的bool值。但如果另一个goroutine已经在运行（因此没有在通道上接收），发送goroutine会阻塞。我应该怎么做？
&gt;
&gt; 	尝试使用条件变量（Go的sync.Cond）而不是通道。条件变量非常适合通知可能（或可能不）等待某事的goroutines。由于通道是同步的，如果不确定通道另一端是否有goroutine在等待，使用通道会显得很尴尬。
&gt;
&gt; 3.  如何让一个goroutine等待来自多个不同通道的输入？如果没有任何内容可读取，则尝试在任何一个通道上接收都会阻塞，从而阻止 goroutine 检查其他通道。
&gt;
&gt; 	尝试为每个通道创建一个单独的goroutine，每个goroutine阻塞在其通道上。这不是总能实现，但在可行时通常是最简单的方法。否则，尝试使用Go的select。
&gt;
&gt; 4. 什么时候应该使用sync.WaitGroup而不是通道？反之亦然？
&gt;
&gt; 	WaitGroup用途较为特殊；它仅在等待一堆活动完成时有用。通道用途更广泛；例如，可以通过通道传递值。尽管比WaitGroup需要多写几行代码，但也可以使用通道等待多个goroutines。
&gt;
&gt; 5. 如何创建一个通过互联网连接的Go通道？如何指定用于发送消息的协议？
&gt;
&gt; 	Go通道仅在单个程序内工作；通道不能用于与其他程序或计算机通信。可以查看Go的RPC包，它允许你通过互联网与其他Go程序通信：
&gt; 	https://golang.org/pkg/net/rpc/
&gt;
&gt; 6. 一些重要/有用的Go特定并发模式有哪些？
&gt;
&gt; 	这是一个关于该主题的幻灯片，由Go专家编写：
&gt; 	https://talks.golang.org/2012/concurrency.slide
&gt;
&gt; 7. 切片是如何实现的？
&gt;
&gt; 	切片是一个对象，包含指向数组的指针以及该数组的开始和结束索引。这种安排允许多个切片共享一个底层数组，每个切片可能暴露数组元素的不同范围。这里有一个更详细的讨论：
&gt; 	https://blog.golang.org/go-slices-usage-and-internals
&gt; 	Go切片比Go数组更灵活，因为数组的大小是其类型的一部分，而以切片作为参数的函数可以接受任何长度的切片。
&gt;
&gt; 8. 什么时候使用同步RPC调用，什么时候使用异步RPC调用？
&gt;
&gt; 	大多数代码需要在继续执行前获得RPC回复；在这种情况下，使用同步RPC是合理的。但有时客户端希望启动许多并发RPC；在这种情况下，异步可能更好。或者客户端希望在等待RPC完成时做其他工作，可能是因为服务器很远（所以光速时间很高）或因为服务器可能不可达，从而RPC经历长时间的超时。我(Robert)从未在Go中使用异步RPC。当我想发送RPC但不必等待结果时，我创建一个goroutine，并让这个goroutine进行同步Call()。
&gt;
&gt; 9. 开发人员在开始使用Go时常见的问题有哪些？
&gt;
&gt; 	以下是一些常见问题：
&gt;
&gt; 	- 未在并发访问时使用锁保护映射。使用Go的竞态检测器！
&gt; 	- 使用通道时的死锁。
&gt; 	- 在创建goroutine时未捕获变量。
&gt; 	- 泄漏的goroutines。
&gt;
&gt; 10. Go是否支持继承？（像Java/C&#43;&#43;那样的“扩展”方式？）
&gt;
&gt; 	Go不支持C&#43;&#43;风格的继承，但有接口和嵌入结构体，可以完成在C&#43;&#43;中使用继承的许多事情。这是Go设计中备受争议的部分；可以搜索“golang generics”。
&gt;
&gt; 11. 我对选择值接收器或指针接收器仍有些困惑。能否提供一些具体的实际例子说明我们应该选择哪一个？
&gt;
&gt; 	当你想修改接收器的状态时，必须使用指针接收器。如果结构体非常大，你可能想使用指针接收器，因为值接收器操作的是一个副本。如果两者都不适用，可以使用值接收器。然而，要小心使用值接收器；例如，如果结构体中有一个互斥锁，你不能将其作为值接收器，因为互斥锁会被复制，从而失去其作用。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/02.%E4%BD%BF%E7%94%A8go%E8%BF%9B%E8%A1%8C%E7%BA%BF%E7%A8%8B%E5%92%8Crpc%E7%BC%96%E7%A8%8B/  

