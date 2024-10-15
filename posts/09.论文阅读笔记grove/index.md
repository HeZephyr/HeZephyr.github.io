# 【论文阅读笔记】Grove: A Separation-Logic Library for Verifying Distributed Systems (Extended Version)

## 介绍

大型应用在分布式系统中遭遇多重挑战，如并发控制、故障恢复、网络不稳定及服务器时钟异步等。形式化验证则是一种严格确立系统正确性的方法，帮助处理边缘情况。

租约是分布式系统中的关键技术，用于保证系统某方面在一定时间内不变，GFS、Chubby和DynamoDB都具有类似的机制。例如租约允许领导者高效执行只读查询，无需频繁验证自身领导权，然而，这一机制的有效性验证却是一项艰巨任务。

Grove，作为前沿的并发分离逻辑（Concurrent Separation Logic, CSL）库，首开先河地解决了基于时间的租约验证问题，包括其与系统重新配置、故障恢复、线程级并发以及不可靠网络通信之间的复杂交互。

CSL的应用精髓在于，通过将系统状态细分为独立资源，并借助同步原语转移资源所有权，从而实现模块化且精确的推理分析。

Grove的创新亮点可概括如下：

1. **时间有界不变性推理**：引入新颖的时间维度，有效解析租约的有效期及其对系统状态的影响。
2. **扩展Crash Hoare逻辑**：强化逻辑体系，使之能妥善应对分布式环境下的节点崩溃情形。
3. **抽象机制**：提供工具集，支持对仅附加日志及单调时钟计数器的精准推理，增强系统的时间一致性。

[Grove code](https://github.com/mit-pdos/perennial)

## 常见组件-Grove案例研究

### RPC库

RPC是分布式系统的重要构建模块，它允许客户端在远程服务器上调用过程。例如，客户端调用`rpcClient.Call(&#34;f&#34;, args)`将在与`rpcClient`相连的服务器上调用`f(args)`。RPC库提供的是不可靠的RPC，意味着客户端的一次调用可能导致服务器运行对应的函数一次、零次或多于一次。这是因为底层网络可能会丢弃、重排或复制数据包。应用程序通常不会直接调用RPC，而是使用各种代理（clerk），它们封装了RPC并附加额外的处理（如添加请求ID、重试等）。

### 复制状态机库

vRSM复制由应用程序提供的状态机，具体的接口如下图所示：

![image-20240830211745211](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/image-20240830211745211.png)

vRSM由多个组件实现，每个组件处理状态机复制的不同方面，例如持久性可以与复制协议分开实现。

1. **副本服务器：写入复制**

	- 副本组件管理被复制的状态机的副本。
	- 主要角色包括主服务器(Primary)和备份服务器(Backup)。主服务器处理来自客户端的写请求，备份服务器则处理读请求。
	- 主服务器在收到操作后，会先在本地应用，然后复制到所有备份服务器，最后回复客户端。复制操作时，主服务器会生成线程以并发方式向每个备份发送RPC，并使用Go WaitGroup等待所有线程完成，确保操作被所有副本服务器应用。

2. **使用configservice重新配置**

	利用`epochs`和`configservice`来管理服务器的添加或移除。系统通过epoch来跟踪不同的服务器配置。每个epoch对应一组特定的服务器配置，包括一个主服务器和多个备份服务器。时代分为活跃epoch和保留epoch，后者指未实际运行的配置。`configservice` 负责维护当前系统的最新epoch和配置信息。它允许客户端获取当前配置，并在重新配置过程中提供原子操作以更新配置。

	在重新配置期间，客户端可能向旧配置发送操作，这可能导致新配置中遗漏操作。为解决此问题，重新配置过程首先会封闭旧配置中的一个服务器，使其不再接受写操作，直到进入新的epoch。

	**重新配置步骤** 包括：

	1. 原子性地创建新epoch
	2. 从旧配置中获取状态
	3. 在新服务器上初始化状态
	4. 更新configservice中的配置信息
	5. 激活新主服务器。

	```go
	// Reserve a new epoch number for reconfiguration, and return the current configuration (set of servers).
	func (ck *Clerk) ReserveEpochAndGetConfig() (uint64, []Address)
	
	// Return current configuration, used by clients to determine what servers to talk to.
	func (ck *Clerk) GetConfig() []Address
	
	//  Set new configuration, making epoch live, as long as no higher-numbered epoch has been reserved.
	func (ck *Clerk) TryWriteConfig(epoch uint64, config []Address) Error
	
	// Get a lease for specified epoch, as long as it’s the current epoch, returning the new lease expiration time.
	func (ck *Clerk) GetLease(epoch uint64) (Error, uint64)
	
	func Reconfigure(newServers []Address) {
	    newEpoch, oldServers := configClerk.ReserveEpochAndGetConfig()
	
	    // get state from a server from old config
	    oldClerk := MakeClerk(oldServers[Rand() % len(oldServers)])
	    oldState := oldClerk.GetStateAndSeal(newEpoch)
	
	    // make clerks to all of the new servers
	    var newClerks = make([]Clerk, len(newServers))
	    for i := 0; i &lt; len(newServers); i&#43;&#43; {
	        newClerks[i] = MakeClerk(newServers[i])
	    }
	
	    // set state on all the new servers
	    wg := new(WaitGroup)
	    for i := 0; i &lt; len(newClerks); i&#43;&#43; {
	        wg.Add(1)
	        go func(i int) {
	            newClerks[i].SetNewEpochState(newEpoch, oldState)
	            wg.Done()
	        }(i)
	    }
	    wg.Wait()
	
	    // write new addresses to config service
	    err := configClerk.TryWriteConfig(newEpoch, newServers)
	    if err == nil {
	        // activate the new primary server
	        newClerks[0].BecomePrimary(newEpoch)
	    }
	}
	```

	在网络分区的情况下，configservice通过仅接受最高编号的新epoch来避免创建多个冲突的系统实例。

3. **副本服务器：基于租约的读取**

	副本服务器（主服务器和备份服务器）利用租约提供线性化读取服务，无需跨服务器通信。

	```go
	func (s *Server) ApplyReadonly(op) Result {
	    s.mutex.Lock()
	
	    if s.leaseExpiry &gt; GetTimeRange().latest {
	        e := s.epoch
	        idx, res := s.stateLogger.LocalRead(op)
	        s.mutex.Unlock()
	
	        // 如果在此期间发生重新配置，服务器会通知客户端重试
	        if s.waitForCommitted(e, op, idx) {
	            return res
	        } else {
	            return ErrRetry
	        }
	    } else {
	        s.mutex.Unlock()
	        return ErrRetry
	    }
	}
	```

	- **租约机制**：
		- 租约防止重新配置时返回过时数据，确保各服务器同步。
		- 后台线程定期更新租约，承诺配置不变，直至租约到期（如1秒后）。
	- **读取流程**：
		- 服务器收到只读请求且租约有效时，根据本地状态计算响应。本地状态包含所有已提交操作，可能含未提交的写操作。
		- 读取依赖的前序写操作需全部提交，方能向客户端发送结果。

	Grove使用类似TrueTime的`GetTimeRange()`API，提供当前时间的上下限，解决时钟偏移问题。

4. **存储库：状态日志器**

	副本服务器使用存储库管理持久状态，提供“状态日志器”用于在追加型文件中持久化新操作。状态日志器在内存中缓冲追加操作，后台线程异步追加并同步缓冲区到文件，以提升性能。存储库提供 `Wait()` 函数，允许等待直到文件的前缀部分被持久化。副本库在回复 RPC 之前使用 `Wait()` 确保变更被持久存储。

5. **基于Paxos的容错配置服务**

	**Paxos 库**用于处理配置服务自身的服务器故障，是一个简单的基于 Paxos 一致性算法的复制库。Paxos在固定服务器集上运行，仅需要多数服务器处理请求，使用 leader 协调操作，但在 leader 崩溃时允许更换。与主-备份复制的差异：

	- Paxos 自行选择新的epoch编号，因为服务器集不会在运行时改变。
	- 仅要求多数服务器提交操作，新 leader 必须从多数服务器获取最新状态。
	- Paxos 较为简单，不使用租约，每次更新都写入整个状态到磁盘，而非追加操作到日志。
	- 写操作性能较低，但对于配置服务是可接受的；提供快速但弱一致性的读取。

	Paxos提供的接口如下：

	```go
	// 返回当前Paxos实例中的复制状态。可能过时或未提交，GetConfig 使用 WeakRead 以确保快速响应
	func (p *Paxos) WeakRead() []byte
	// 开始一个新的提议过程。它返回当前的复制状态和一个commit回调函数。
	func (p *Paxos) Begin() (oldstate []byte, commit func(newstate []byte) error)
	// 尝试使当前节点成为领导者。
	func (p *Paxos) TryBecomingLeader()
	```

	要执行写操作，如下面这段代码所示，configserver使用`Begin()`方法。

	```go
	// 参数args在此处未使用，通常用于接收客户端请求的参数。
	// reply 是方法的输出，它将包含操作的结果状态以及预留的新epoch和配置信息。
	func (c *ConfigService) ReserveEpochAndGetConfig(args []byte, reply *[]byte) {
	    // 从Paxos实例开始一个新的提议，获取当前状态和提交函数。
	    // 应该在当前的领导者上调用此方法，否则提议无法成功提交。
	    oldstate, commit := c.paxos.Begin()
	
	    // 反序列化旧状态，以便修改。
	    st := unmarshal(oldstate)
	
	    // 更新状态中的预留epoch字段，准备进入下一个epoch。
	    st.reservedEpoch = st.reservedEpoch &#43; 1
	
	    // 序列化更新后的状态，准备提交。
	    newstate := marshal(st)
	
	    // 使用从Begin获得的提交函数尝试提交新状态。
	    // 此操作会与其他服务器通信，以确保新状态被复制到大多数服务器。
	    err := commit(newstate)
	
	    // 检查提交是否成功。
	    if err != nil {
	        // 如果提交失败，将错误状态编码并写入reply。
	        *reply = marshal.WriteInt(nil, STAT_ERROR)
	    } else {
	        // 如果提交成功，将OK状态编码并写入reply。
	        *reply = marshal.WriteInt(nil, STAT_OK)
	
	        // 编码并写入新的预留epoch到reply。
	        *reply = marshal.WriteInt(*reply, st.reservedEpoch)
	
	        // 编码并写入当前配置到reply。
	        *reply = marshal.WriteBytes(*reply, encode_cfg(st.config))
	    }
	}
	```

6. **版本化状态机API**

	开发者需实现如下所示的版本化状态机接口，以便在 vRSM 上构建应用。

	```go
	type VersionedStateMachine struct {
		Apply		func(op []byte, idx uint64) []byte
		Read		func(op []byte) (uint64, []byte)
		SetState 	func(snap []byte, idx uint64)
		GetState 	func() []byte
	}
	```

	- 操作执行：
		- `Apply()`：执行应用级别的读写操作。
		- `Read()`：对当前内存状态执行应用级别的读操作。
	- 状态管理：
		- `SetState()` 和 `GetState()`：允许序列化内存状态。
		- vRSM 库：负责状态的磁盘检查点和新副本的状态复制。

7. **vRSM客户端库（clerk）**

	vRSM 提供客户端库，简化了通过网络向 vRSM 发送请求的复杂性。Clerk 从配置服务获取并缓存副本服务器地址。

	操作执行：

	- `clerk.Apply(op)`：向主服务器发送读写操作。
	- `clerk.Read(op)`：向任何副本发送只读操作。

	若服务器不再为主服务器或副本服务器，Clerk 会请求新服务器信息并重试。由于重试，可能导致一个操作被应用许多次，需要更高级库处理操作去重。

8. **exactlyonce库**

	确保使用 vRSM 的应用操作仅执行一次。组件构成：

	- 新型 Clerk：包装 vRSM Clerk，通过添加唯一请求 ID 防止操作重复。
	- 状态机转换器：为应用级状态机添加回复表，追踪已应用请求及其回复。

	操作处理：

	- 新请求：调用状态机的 `Apply()` 并存储回复。
	- 重复请求：不调用状态机，直接返回先前回复。
	- 只读操作：忽略回复表，直接调用状态机的 `Read()`。

### vRSM上层应用

1. **vKV实现**

	- **vKV 架构**：基于 vRSM 和 exactlyonce 库实现。
	- **服务器端**：实现 vRSM 期望的状态机接口。
	- **客户端**：基于 exactlyonce clerk 实现的 clerk，提供简化的 API（`Put`、`CondPut`、`Get`）。
	- **实现细节**：vKV 实现简单，包括 (反)序列化方法和内存映射的读写函数。
	- **性能优化**：vKV 存储键到值的映射以及键的最后修改操作索引，利用 vRSM 的版本化状态机接口提升读取性能。

2. **基于租约的客户端缓存-cachekv**

	cachekv 库通过在 vKV 中存储数据和租约到期时间实现基于租约的客户端缓存。GetAndCache 函数如下所示：

	```go
	func (k *CacheKv) GetAndCache(key string, cachetime uint64) string {
	    for {
	        // first attempt to read from the local cache, and if not cached, call vKV&#39;s Get.
	        old := k.kv.Get(key)
	        new := old
	
	        newExpiration := max(GetTimeRange().latest&#43;cachetime, old.leaseExpiration)
	        new.leaseExpiration = newExpiration
	
	        // Try to update the lease expiration time on the backend
	        resp := k.kv.CondPut(key, old, new)
	
	        if resp == &#34;ok&#34; {
	            k.mu.Lock()
	            k.cache[key] = cacheValue{v: old.v, l: newLeaseExpiration}
	            k.mu.Unlock()
	            return old.v
	        }
	    }
	}
	```

	它返回指定键的值，并在内部缓存，使用 `CondPut` （确保仅在租约过期时更改值）原子增加租约持续时间，确保并发修改不会改变值。

3. **锁服务**

	**锁服务接口**基于 vKV 实现，使用 vKV 的 CondPut() 操作实现锁。每个锁对应一个键值对，提供 Acquire() 和 Release() 方法的规范，支持应用实现独占锁。锁服务的规范与传统的并发分离逻辑锁规范不同，简化了资源保护。

4. **银行事务**

	顶层应用，使用基于 vKV clerk 和锁服务接口构建的事务。

	- **账户状态存储**：使用 vKV 实例存储账户状态，每个账户余额用一对键值存储。
	- **并发访问控制**：使用锁服务处理账户的并发访问，每次转账操作获取两个锁，确保并发转账的安全执行。
	- **审计功能 (Audit)**：获取所有账户的锁，计算总余额，并释放锁。
	- **容错处理**：若银行节点崩溃，锁服务中的锁将保持锁定状态，恢复需要某种形式的撤销或重做日志，但原型中未实现。

## Grove性能评估

1. **实验目的**：

	- 证明 Grove 能够验证现实世界中高性能的分布式系统。

	- &lt;font color=&#34;red&#34;&gt;展示 vKV 原型通过 Grove 验证后能够实现高性能&lt;/font&gt;。

	- 特别强调租约在 vKV 中实现高性能读取的重要性。

2. **baseline性能对比**：将vKV与Redis进行比较，后者是高性能键值服务器，以C语言编写。为了使Redis与vKV在持久化保障上可比，Redis开启appendfsync always选项，而vKV运行于单核并禁用备份副本。结果显示，vKV吞吐量为Redis的67%-73%，请求延迟相当，多核下vKV吞吐量更高（例如，8核下YCSB 5%写入情况下，吞吐量提升5.1倍）。
3. **重新配置能力**：通过添加新服务器进行系统重配置，同时继续正确处理客户端请求的能力。实验中，主服务器在 10 秒时被杀掉，开始重配置过程。使用 YCSB 工作负载变体，100 个客户端持续写入，100 个客户端持续读取。重配置期间，写入操作会阻塞，但读取可以继续。实验结果显示 vKV 可以在重配置期间继续提供读取服务。
4. **租约对读取性能的影响**：**写入密集型工作负载**（50%或100%写入），增加副本会降低性能，因为写入在主服务器遇到更多开销，且其它副本处理的读取不足以抵消成本。对于**读取密集型工作负载**，增加副本可以提升性能，例如，YCSB 5%和0%写入情况下，3台服务器分别达到单服务器1.7倍和2.3倍的吞吐量。

## 总结

### GroveKV系统

GroveKV特点：

- 容错、线性化的键值（KV）服务。
- 操作（Put/Get）exactlyonce。
- 崩溃安全且可重配置。

要进行重配置，GroveKV使用configservice管理服务器更改。如果没有configservice，类似VMWare-FT的问题，即两个备份可能是网络的彼此分区并且都想成为主分区。这可以通过ZooKeeper之类的配置服务解决。

lab3也是一个容错KV服务，但GroveKV使用主/备份复制而不是Raft。**关键操作**如下：

1. 复制：Primary使用goroutines复制到其他服务器，基于RPC。执行操作时需要持有锁。
2. 重配置：封闭当前的服务器组，从中获取状态副本，安装到新服务器。使用epoch编号处理并发重配置。
3. 基于租约的读取：服务器不跟任何人协调回复`Get`请求。

### Grove核心

在正式验证领域，Grove 的核心在于证明代码在所有可能场景下均能表现得当，这一过程要求对代码执行的数学模型有深刻理解，以及对系统行为“正确”的明确定义。通过引入机械证明检查器，Grove 大幅降低了开发者在证明过程中犯错的可能性。

在 Grove 中，规范由前条件和后条件构成，用于描述操作前后的系统状态。以 GroveKV 为例，Put 和 Get 操作的规范不仅限定了操作的预期结果，还明确了数据所有权的转移。

Concurrent Separation Logic（CSL）是一种针对并发程序的形式化验证方法，Grove 对其进行了创新性的拓展，使之适用于分布式系统。CSL 强调基于资源所有权的代码分析，其中“堆指向”是一个典型的例子，它确保了数据的一致性不受并发访问的影响。

在 Grove 中，不变量指的是系统运行中必须始终保持为真的属性，而时间有界不变量（tinv）则进一步限制了特定资源的有效期。例如，`GetTimeRange` 函数允许在租约未到期的情况下，临时访问底层资源，这在处理基于租约的读取时尤为关键。

尽管正式验证能够显著减少某些类型的错误，但它并非万能药。验证不能保证所有实际中可能遇到的问题都被解决，特别是那些涉及系统活性性的问题，如死锁或饥饿。此外，编写高质量的证明和测试同样需要大量时间和精力，与开发代码无异，甚至有时还需经历重构的过程。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/09.%E8%AE%BA%E6%96%87%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0grove/  

