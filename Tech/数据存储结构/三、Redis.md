&emsp;&emsp; Redis是一个由C语言编写的，<font color=red>**基于内存，单线程且支持持久化**</font>的 key-value 的NoSQL数据库，其中每个key和value都是使用对象表示的。Redis数据库具有以下特性：
 &emsp; &emsp; ● 基于内存运行，性能高；
 &emsp; &emsp; ● 支持分布式，理论上可以无限扩展
 &emsp; &emsp; ● 采用Key-Value存储方式，且可持久化。
&emsp;&emsp;&emsp;  由于 Redis 是基于内存的，性能高，因此 Redis 通常用作数据的高并发读写 (数据访问频率高)，配合MySQL等关系型数据库做高速缓存，缓存高频率访问的数据，从而降低数据库的IO。

![[../picture/Pasted image 20241117155740.png#pic_center|530]]

&emsp;&emsp;Redis的整体组成架构如下图所示：
![[../picture/Pasted image 20241201084743.png#pic_center|580]]

><font color=SlateBlue> <u>**Q1. Redis 有多快？**</u></font>
>&emsp;&emsp;&emsp; 通常来说，在一台普通硬件配置的 Linux 机器上跑单个 Redis 实例，处理简单命令 (时间复杂度 O(N) 或者 O(log(N) )，QPS 可以达到 8w+，而如果使用 pipeline 批处理功能，则 QPS 至高能达到 100w。
>
><font color=SlateBlue> <u>**Q2. Redis 为什么快？**</u></font>
>&emsp;&emsp;&emsp; Redis 的高性能得益于以下几个基础：
>&emsp;&emsp;&emsp;&emsp; ① **C 语言实现**：虽然 C 对 Redis 的性能有助力，但语言并不是最核心因素。
>&emsp;&emsp;&emsp;&emsp; ② **纯内存 I/O**：相较于其他基于磁盘的 DB，具有天然的性能优势。
>&emsp;&emsp;&emsp;&emsp; ③ <font color=red>**I/O 多路复用**</font>：基于 epoll/select/kqueue 等 I/O 多路复用技术，实现高吞吐的网络 I/O。
>&emsp;&emsp;&emsp;&emsp; ④ <font color=red>**单线程模型**</font>：单线程避免了多线程频繁上下文切换，以及同步机制带来的开销。
>
><font color=SlateBlue> <u>**Q3. Redis 的单线程与多线程之争？**</u></font>
>&emsp;&emsp;&emsp; "Redis是单线程的" 这个我们耳熟能详，但它有一定的前提。 <font color=green>如果仅仅是 Redis 的核心业务部分 (命令行处理)，那一定是单线程的。</font><font color=orange>如果是整个 Redis，那么 Redis 就是多线程。</font>在Redis版本迭代过程中，在两个重要的时间节点上引入了多线程的支持：
>&emsp;&emsp;&emsp; ● Redis v4.0：引入多线程异步任务，处理一些耗时较久的任务，例如异步删除命令unlink。
>&emsp;&emsp;&emsp; ● Redis v6.0：在核心网络模型中实现 I/O 多线程，提高对于多核CPU的利用率，比如命令回复处理器中对数据的写出以及命令的解析都是使用的多线程。
>
><font color=SlateBlue> <u>**Q4. Redis 为何选择单线程？**</u></font>
>&emsp;&emsp;&emsp;对于一个 DB 来说，CPU 通常不会是瓶颈，因为大多数请求不会是 CPU 密集型的，而是 I/O 密集型，特别是对于Redis的内存存储，如果不考虑 RDB/AOF 等持久化方案，Redis 是完全的纯内存操作，执行速度非常快。Redis 真正的性能瓶颈在于网络 I/O，也就是客户端和服务端之间的网络传输延迟，因此 Redis 选择了单线程的 I/O 多路复用来实现它的核心网络模型。具体的选择单线程的原因可以归纳如下：
>&emsp;&emsp;&emsp; ① **避免过多的上下文切换开销**：多线程调度过程中必然需要在 CPU 之间切换线程上下文 context。单线程则可以规避进程内频繁的线程切换开销。
>&emsp;&emsp;&emsp; ② **避免同步机制的开销**：多线程模型会引入某些同步机制，而且不同的数据结构对同步访问的加锁粒度又不尽相同，可能会导致在操作数据过程中带来很多加锁解锁的开销。增加程序复杂度的同时还会降低性能。
>&emsp;&emsp;&emsp; ③ **简单可维护**：单线程可以保持代码逻辑上的串行性，而多线程代码执行的顺序将变成不可预测的，可能出现各种并发编程的问题，而且所有的底层数据结构都必须实现成线程安全的，这无疑又使得 Redis 的实现变得更加复杂。
>

### 3.1 Redis 对象类型与数据结构
&emsp; &emsp; 在 Redis 中 Value 包含五大数据结构：<font color=red>String (字符串)，Hash (哈希)，List (列表)，Set (集合)，Sorted Set (有序集合)。</font>不同的数据结构使用的命令不同，但所有的 Redis 命令都是<font color=red>原子操作</font> ( 注意:对命令是原子的，而不是对字段是原子的 )，避免了多个客户端的操作命令在执行过程中的竞争，但不能避免多个客户端对字段执行操作时的竞争。Redis 存储数据所涉及到的数据结构如下图所示：
![[../picture/Pasted image 20241117215152.png]]
&emsp;&emsp;● redisDb：表示 Redis 数据库的结构，结构体里存放了指向 dict 结构的指针。
&emsp;&emsp;● dict：存放了2个哈希表，通常只使用 Hash1。只有在哈希冲突率过高，触发 rehash 扩容时才用 Hash2。
&emsp;&emsp;● dictht：表示哈希表的结构，结构里存放了哈希表数组，数组中的每个元素都指向一个 dictEntry 的指针。 
&emsp;&emsp;● dictEntry：哈希表节点的结构，存放了 key 和 value 指针，

#### 3.1.1 String 类型

&emsp;&emsp;字符串类型是 Redis 最基础的数据结构，可以是字符串，包括 json串，可以是数字，可以是图片二进制等，存储最大不超过512M。Redis 构建了一种简单动态字符串 ( Simple Dynamic String，SDS ) 的抽象类型作为 Redis 的字符串类型。
![[../picture/Pasted image 20241117215711.png#pic_center|780]]

&emsp;&emsp;SDS 有以下几个优点：
&emsp; &emsp; &emsp;① 由于SDS结构中记录了字符串的长度，因此获取字符串长度的复杂度为O(1)，不需要遍历所有元素。
&emsp; &emsp; &emsp;② 不会造成缓冲区的溢出，在执行字符串拼接时，SDS会判断剩余的 free 空间是否能存下需要拼接的字符串。防止出现C语言中 `strcat()` 造成的字符串溢出问题。 
&emsp; &emsp; &emsp;③ 字符串的二进制安全。String类型是根据 len 属性来判断当前字符串是否结束，而不是以 '\0' 作为判断字符串结束的标准，避免了C语言中 '\0'  字符被认为是字符串结尾的问题。因此字符串输出=字符串输入。
&emsp; &emsp; &emsp;④ 减少修改字符串带来的内存重分配次数：
&emsp; &emsp; &emsp;  ● 在对SDS字符串进行修改后，若SDS的长度小于1MB时，则SDS会分配一个和 len 属性相同大小的未使用空间。即 free=len。若SDS的长度大于1MB时，则SDS会分配1MB的未使用空间。即 free=1MB。
&emsp; &emsp; &emsp; ● 当SDS字符串进行删除时，不会进行内存的重新分配，多余的空间也不会被回收，会留着为字符串再次添加做准备，减少了内存重新分配的次数。 

#### 3.1.2 List 类型

&emsp; &emsp; List 类型是一个简单的字符串列表，一个 Key 对应一个 List，按照插入顺序排序。List 基于双链表实现，既可以从头部插入数据，也可以从尾部插入数据。一个列表最多可以包含2的23次方-1 个元素。List 使用两种数据结构实现：压缩列表 ZipList 和双向链表 LinkedList。
![[../picture/Pasted image 20241117220603.png#pic_center|440]]

▧  **压缩列表 ZipList**：

&emsp;&emsp; ZipList 是一个经过特殊编码的双向链表，旨在节约内存，提高内存效率。ZipList 会对数据进行压缩，对于不同的数据类型有着不同的编码方式。当元素个数较少时，Redis 用 Ziplist 来存储数据，当元素个数超过某个值时，链表键 ( Value ) 中会把 ZipList  转化为 LinkedList，字典键 ( Key ) 中会把 ZipList  转化为 HashTable。

![[../picture/Pasted image 20241124100013.png]]

&emsp;&emsp;<font color=orange>由于每个操作都会进行数据压缩，每个操作都需要重新分配内存，因此当一个节点的数据长度发生变化时，可能会连锁更新问题，导致后续所有节点的长度字段都要重新编码。随着存储的 Value 数据量级的增加，ZipList 所付出的代价也随之增加，从而造成极差的性能。 (在7.0 版本之后 ZipList 被 listpack 代替) </font>
><font color=SlateBlue> <u>**Q1. 为什么 ZipList 会发生连锁更新的情况 ？**</u></font>
>&emsp;&emsp;&emsp;由于 ZipList 每个节点的 prelen 会记录上一个节点的数据大小，如果前一节点的长度小于254字节，则用1个字节来保存这个长度值，如果前一节点的长度大于等于254字节，则采用5个字节来保存这个长度值。如果有N个连续的、长度为250~253字节之间的 entry 数据节点，这时新增、删除数据可能会导致 prelen 由1字节变为5字节，从而产生连续多次空间扩展操作，导致连锁更新的情况发生。

▧  **双向列表 LinkedList**：

&emsp;&emsp; 双向列表的每个节点 ListNode 可以通过 prev 和 next 指针分布指向前一个节点和后一个节点组成双端链表，同时每个链表还会有一个 List 结构为链表提供表头指针 head、表尾指针 tail、以及链表长度计数器 len。
  ![[../picture/Pasted image 20241117221015.png#pic_center|680]]
  
#### 3.1.3 Hash 类型
&emsp; &emsp; Hash 类型是一个键值对集合，该类型是由字段 field 和关联的 value 组成的 map 。一个 Key 可以对应多个map。其中，field 和 value 都是 String 类型的。Hash 类型适合用于存储对象信息，类似于Java中的 Map<String,Object>。以用户ID为 Key，存储的 Value 为用户对象 ( 包括姓名，年龄，生日等信息 )为例对比 String 和 Hash 两种存储方式：
![[../picture/Pasted image 20241117221158.png#pic_center|680]]


&emsp;&emsp;Hash 使用两种数据结构实现：压缩列表 ZipList (7.0 版本之后被 listpack 代替) 和哈希表 Dict ( 字典 )。

▧  **压缩列表 ZipList**

&emsp;&emsp;当存储的数据量较少的时，Hash 采用 ZipList 作为底层存储结构，此时要求符合以下两个条件，当无法满足条件时 Hash 会采用哈希表 Dict 来存储数据：
&emsp; &emsp; &emsp;  ● 哈希对象保存的所有键值对 (键 field 和值 value ）的字符串长度总和小于 64 个字节。
&emsp; &emsp; &emsp;  ● 哈希对象保存的键值对数量要小于 512 个。

▧  **字典 Dict**

&emsp;&emsp; Redis字典使用散列表最为底层实现，一个散列表里面有多个散列表节点，每个散列表节点就保存了字典中的一个键值对。Dict 是一个无序的字典，其查找性能非常高效，其时间复杂度为 O(1)。Dict的结构如下图所示：

![[../picture/Pasted image 20241124144658.png#pic_center|630]]

&emsp;&emsp;  随着操作的进行，散列表中保存的键值对会也会不断地增加或减少，为了保证负载因子维持在一个合理的范围，当散列表内的键值对过多或过少时，内存需要定期进行 rehash，以提升性能或节省内存。rehash 操作需要满足以下条件:
&emsp;&emsp;&emsp; ① 服务器目前没有执行`BGSAVE`(rdb持久化)命令或者`BGREWRITEAOF`(AOF文件重写)命令，并且散列表的负载因子大于等于1。
&emsp;&emsp;&emsp; ② 服务器目前正在执行`BGSAVE`命令或者BGREWRITEAOF命令，并且负载因子大于等于5。
&emsp;&emsp;&emsp; ③ 当负载因子小于0.1时，程序自动开始执行收缩操作。

&emsp;&emsp;**Redis的 rehash 的步骤如下**:
&emsp;&emsp;&emsp; ① 为字典的ht\[1]散列表分配空间，空间的大小取决于ht\[0]当前包含的键值对数量(即:ht\[0].used的值)。
&emsp; &emsp; &emsp; ● 扩展操作：ht\[1]的大小为第一个大于等于$ht[0].used×2$的2的n次方幂的值。如:ht\[0].used=3则ht\[1]的大小为8 (3\*2=6，第一个大于等于6的2的n次方幂的值是8)，ht\[0].used=4则ht\[1]的大小为8。
&emsp; &emsp; &emsp; ● 收缩操作: ht[1]的大小为 第一个大于等于ht[0].used的2的n次方幂。

![[../picture/Pasted image 20241124225105.png]]
&emsp;&emsp; ② 将保存在ht\[0]中的键值对重新计算键的散列值和索引值，然后放到ht\[1]指定的位置上。将ht\[0]包含的所有键值对都迁移到了ht\[1]之后，释放h\t[0]，将ht\[1]设置为ht\[0],并创建一个新的ht\[1]哈希表为下一次 rehash 做准备。
![[../picture/Pasted image 20241124225125.png]]

&emsp;&emsp;在 rehash 的过程中，如果散列表当前大小过大 (如大小为1GB，要想扩容为原来的两倍大小，那就需要对 1GB 的数据重新计算哈希值，并且从原来的散列表搬移到新的散列表），就会造成一次性扩容耗时过多的情况。为了解决这个问题，提出了渐进式 rehash。渐进式 rehash 是将扩容操作穿插在插入操作的过程中，分批完成。当负载因子触达阈值之后，只申请新空间，但并不将老的数据搬移到新散列表中。当有新数据要插入时，将新数据插入新散列表中，并且从老的散列表中拿出一个数据放入到新散列表。每次插入一个数据到散列表。经过多次插入操作之后，老的散列表中的数据就一点一点全部搬移到新散列表中。
&emsp;&emsp;&emsp;在进行渐进式 rehash 的过程中，字典会同时使用 ht\[0] 和 ht\[1] 两个哈希表，所以**在渐进式 rehash 进行期间，字典的删除 (delete)、查找(find)、更新(update)等操作会在两个哈希表上进行**。在渐进式 rehash 执行期间，新添加到字典的键值对一律会被保存到 ht\[1] 里面，而 ht\[0] 则不再进行任何添加操作。

&emsp;&emsp;**Redis 渐进式 rehash 的步骤如下**:
&emsp;&emsp;&emsp; ① 为 ht[1] 分配空间， 让字典同时持有 ht[0] 和 ht[1] 两个哈希表。
&emsp;&emsp;&emsp; ② 在字典中维持一个索引计数器变量  rehashidx， 并将它的值设置为 0 ，表示 rehash 工作正式开始。
&emsp;&emsp;&emsp; ③ 在 rehash 进行期间， 每次对字典执行添加、删除、查找或者更新操作时， 程序除了执行指定的操作以外， 还会顺带将 ht\[0] 哈希表在 rehashidx 索引上的所有键值对 rehash 到 ht\[1] ， 当 rehash 工作完成之后， 程序将 rehashidx 属性值加一。
&emsp;&emsp;&emsp; ④ 随着字典操作的不断执行， 最终 ht\[0] 的所有键值对都会被 rehash 至 ht\[1] ， 这时将 rehashidx 属性的值设为 -1 ， 表示 rehash 操作已完成。

#### 3.1.4 Set 类型
#### 3.1.5 Sorted Set 类型

### 3.2 Redis 网络模型与IO
&emsp;&emsp; Redis 是CS架构，网络通信主要分为两步：① 客户端 client 向服务端 server发送一条命令；② 服务端解析并执行命令，返回响应结果给客户端。
&emsp;&emsp;&emsp; Redis 最初选择单线程网络模型，是因为在当时 CPU 通常不会成为性能瓶颈，瓶颈往往是**内存**和**网络**，由于当时的网络流量较少，因此单线程足够了。随着互联网的飞速发展，互联网业务系统所要处理的线上流量越来越大，Redis 的单线程模式会导致系统消耗很多 CPU 时间在网络 I/O 上从而降低吞吐量。因此从 Redis v6.0开始，在核心网络模型中实现 I/O 多线程。
&emsp;&emsp;&emsp;  <font color=green>I/O 多线程仅仅是把读取客户端请求命令和回写响应数据的逻辑异步化</font>了，交给 I/O 线程去完成。需要特别注意的一点是：<font color=red>**I/O 线程仅仅是读取和解析客户端命令而不会真正去执行命令，客户端命令的执行最终还是要在主线程上完成**。</font>Redis 的I/O 多线程网络模型如下图所示：

![[../picture/Pasted image 20241207165947.png]]

&emsp;&emsp; 当客户端想要去连接服务器，会去先到I/O多路复用模型去进行排队，会有一个连接应答处理器去接受读请求，并把读请求注册到I/O多路复用模型中。当客户端请求处理器去进行执行命令时，I/O多路复用模型去把数据读取出来，然后把数据放入到client中， clinet去解析当前的命令转化为redis命令，并处理这些命令，操作对应的数据。当数据操作完成后，会去找到命令回复处理器并将数据写出。具体的处理流程如下：
&emsp;&emsp;&emsp; ① Redis 服务器启动后，开启主线程事件循环 (Event Loop)，注册 **tcpAcceptHandler** 连接应答处理器(时间)到用户配置的监听端口对应的文件描述符，等待新连接到来；
&emsp;&emsp;&emsp; ② 客户端和服务端建立网络连接，**tcpAcceptHandler** 被调用，主线程 将 **readQueryFromClient** 命令读取处理器绑定到新连接对应的文件描述符上，并初始化一个 client 绑定这个客户端连接；
&emsp;&emsp;&emsp; ③ 客户端发送请求命令，触发读就绪事件，服务端主线程不会通过 socket 去读取客户端的请求命令，而是先将 client 放入一个 LIFO 队列 clients_pending_read；
&emsp;&emsp;&emsp; ④ 在事件循环 Event Loop中，主线程利用 Round-Robin 轮询负载均衡策略，把 clients_pending_read 队列中的连接均匀地分配给 I/O 线程各自的本地 FIFO 任务队列 io_threads_list[id]，I/O 线程通过 socket 读取客户端的请求命令，存入 `client.querybuf` 并解析第一个命令，**但不执行命令**，主线程忙轮询，等待所有 I/O 线程完成读取任务；
&emsp;&emsp;&emsp; ⑤ 主线程和所有 I/O 线程都完成了读取任务，主线程结束忙轮询，遍历 clients_pending_read 队列，执行所有客户端连接的请求命令；
&emsp;&emsp;&emsp; ⑥ 根据请求命令的类型 (SET, GET, DEL, EXEC 等），分配相应的命令执行器去执行，最后将响应数据写入到对应 client 的写出缓冲区：`client.buf` 或者 `client.reply`。`client.buf` 是首选的写出缓冲区，固定大小 16KB。但是如果客户端在时间窗口内需要响应的数据非常大，那么则会自动切换到 `client.reply` 链表上去，使用链表理论上能够保存无限大的数据 (受限于机器的物理内存)。最后把 client 添加进一个 LIFO 队列 clients_pending_write;
&emsp;&emsp;&emsp; ⑦ 在事件循环 (Event Loop) 中主线程执行利用 Round-Robin 轮询负载均衡策略，把 clients_pending_write 队列中的连接均匀地分配给 I/O 线程各自的本地 FIFO 任务队列 io_threads_list[id]，I/O 线程把 client 的写出缓冲区里的数据回写到客户端，主线程忙轮询，等待所有 I/O 线程完成写出任务；
&emsp;&emsp;&emsp; ⑧ 主线程和所有 I/O 线程都完成了写出任务， 主线程结束忙轮询，遍历 clients_pending_write 队列，如果 client 的写出缓冲区还有数据遗留，则注册 **sendReplyToClient** 命令回复处理器到该连接的写就绪事件，等待客户端可写时在事件循环中再继续回写残余的响应数据。

### 3.3 Redis 高可用
&emsp;&emsp;Redis高可用的方案包括<font color=red>**持久化**、**主从复制（及读写分离）**、**哨兵**和**集群**。</font>
&emsp;&emsp; &emsp; ● **持久化**：持久化侧重解决的是 Redis 数据的单机备份问题 (从内存到硬盘的备份)；
&emsp;&emsp; &emsp; ● **主从复制**：主从复制是Redis高可用的基础，侧重解决的是数据多机热备和读写分离，同时实现了读操作的负载均衡和故障恢复。但也存在缺陷：故障恢复无法自动化、写操作无法负载均衡、存储能力受到单机的限制。
&emsp;&emsp; &emsp; ● **哨兵**：哨兵模式是在主从模式的基础上，实现了主从节点的故障自动转移的功能。缺陷：写操作无法负载均衡。
&emsp;&emsp; &emsp; ● **集群**：集群模式是一种分布式的解决方案，多个Redis节点协同工作，解决了写操作无法负载均衡，以及存储能力受到单机限制的问题。

#### 3.3.1 Redis 持久化 - 数据单机备份
&emsp;&emsp; 由于 Redis 运行在内存当中，当服务器关闭或Redis重启后，其存储的数据就会丢失，为了是Redis重启后能够恢复数据，需要将Redis持久化。持久化是指将数据保存到能够长期存储的存储介质上(如磁盘等)，以便在需要时可以重新加载和使用。Redis有两种方式来实现持久化： 
&emsp;&emsp;&emsp;● **RDB 持久化 - Redis DataBase**：根据指定的规则，定时将内存中当前数据状态进行保存 (类似于**快照形式**)，**只存储数据结果**，存储格式简单，**关注点在于数据**。
&emsp;&emsp;&emsp;● **AOF 持久化 - Append Only File**：将数据的操作过程进行保存 (类似于**日志形式**），**存储操作过程**，存储格式复杂，**关注点在于数据操作**。

##### 1. RDB 持久化
&emsp;&emsp; **RDB 是 Redis 默认的持久化方式**，RDB 持久化通过创建数据集的快照 snapshot 来工作，在指定的时间间隔内将 Redis 在某一时刻的数据状态保存到磁盘的一个非常紧凑 RDB 二进制快照文件中。其特点如下：
&emsp;&emsp; &emsp; ① **具有周期性**；
&emsp;&emsp; &emsp; ② **不影响数据写入**：RDB会启动子进程，备份所有数据。当前进程，继续提供数据的读写。当备份完成，才替换老的备份文件。
&emsp;&emsp; &emsp; ③ **高效**：RDB可以一次性还原所有数据。
&emsp;&emsp; &emsp; ④ <font color=orange>**完整性较差,数据丢失风险大**</font>：如果出现故障只能恢复上次备份的数据，从故障点到上一次备份之间的数据无法恢复。

▧  ** RDB 持久化触发**

&emsp;&emsp;RDB 持久化触发方式有以下几种：
&emsp;&emsp;&emsp; ●<font color=orange> **手动 save**</font>：在客户端中执行 `save` 命令，就会触发 Redis 的持久化。`save` 命令会阻塞 Redis 服务器进程，直到 RDB 文件创建完毕为止，在服务器进程阻塞期间，服务器不能处理任何命令请求。

![[../picture/Pasted image 20241208092315.png#pic_center|450]]

&emsp;&emsp; ●<font color=orange> **手动 bgsave**</font>：即background save，当执行 `bgsave` 命令时，redis 会 fork 出一个子进程，专门用于写入 RDB 文件，避免了主线程的阻塞，这也是 Redis RDB 文件生成的默认配置。
![[../picture/Pasted image 20241208092753.png#pic_center|580]]

&emsp;&emsp;`bgsave` 子进程是由主线程 fork 生成的，可以共享主线程的所有内存数据。`bgsave` 子进程运行后，开始读取主线程的内存数据，并把它们写入 RDB 文件。此时，如果主线程对这些数据也都是读操作，那么主线程和 bgsave 子进程相互不影响。但是如果主线程要修改一块数据，那么这块数据就会被复制一份，生成该数据的副本。然后 `bgsave` 子进程会把这个副本数据写入 RDB 文件，在这个过程中主线程仍然可以直接修改原来的数据。
![[../picture/Pasted image 20241208102640.png#pic_center|540]]

&emsp;&emsp; ●<font color=orange> **手动 flushall**</font>：执行`flushall`后，Redis 会清除所有数据。当自动快照条件不为空时，Redis会执行一次快照操作。

&emsp;&emsp;● <font color=orange> **自动触发**</font>：自动方式是在配置文件中设置，满足条件时自动触发。如在 Redis 主从复制中，当从节点执行全量复制操作时，主节点会执行 `bgsave` 命令，并将 RDB 文件发送给从节点，该过程会自动触发 Redis 持久化。

▧  ** RDB 相关配置**

&emsp;&emsp;● `save`：指定 RDB 持久化操作的条件，当 Redis 的数据发生变化，并且经过指定的时间 seconds 和变化次数 changes 后，Redis 会自动执行一次 RDB 操作。如：save 3600 10000 表示如果 Redis 的数据在一个小时内发生了至少 10000 次修改，那么 Redis 将执行一次 RDB 操作。
&emsp;&emsp;&emsp;● `stop-writes-on-bgsave-error`：在 RDB 持久化过程中如果出现错误是否停止写入操作。
&emsp;&emsp;&emsp;● `rdbcompression`：是否对 RDB 文件进行压缩。
&emsp;&emsp;&emsp;● `rdbchecksum`：是否对 RDB 文件进行校验和计算。
&emsp;&emsp;&emsp;● `replica-serve-stale-data`：指定复制节点在与主节点断开连接后是否继续向客户端提旧数据。
&emsp;&emsp;&emsp;● `repl-diskless-sync`：指定复制节点在进行初次全量同步 (即从主节点获取全部数据) 时是否采用无盘同步方式。当设置为 yes 时，复制节点将通过网络直接获取主节点的数据，并且不会将数据存储到本地磁盘中；当设置为 no 时，复制节点将先将主节点的数据保存到本地磁盘中，然后再进行同步操作。采用无盘同步方式可以避免磁盘 IO 操作对系统性能的影响，但同时也会增加网络负载和内存占用。

##### 2. AOF 持久化
&emsp;&emsp;AOF **是一种基于日志的持久化方式**，以日志的形式来记录每个**写操作**，将 Redis 执行过的所有写指令记录下来(读操作不记录)，**按照 Redis 的写命令顺序将写命令追加到磁盘文件的末尾**，只许追加文件但不可以改写文件。Redis 启动时可以通过重新执行 AOF 中的命令来恢复数据状态。其特点如下：
&emsp;&emsp; &emsp; ① 实时性
&emsp;&emsp; &emsp; ② 完整性较好，不会丢失数据，同时也可以恢复误操作。
&emsp;&emsp; &emsp; ③ 存储体积大

▧  **AOF 日志格式与日志写入**

&emsp;&emsp; AOF 里记录的是 Redis 收到的每一条写入操作命令，这些命令是以文本形式保存的。以`set k1 v1`命令为例，AOF的日志格式如下：
![[../picture/Pasted image 20241208212320.png#pic_center|430]]

&emsp;&emsp; AOF 日志是写后日志，Redis 是先执行命令，把数据写入内存，然后才记录日志。(不同于MySql，MySql 是写前日志，在实际写数据前先把修改的数据记到日志文件中)。当启用 AOF 时，Redis 发生写命令时其实并不是直接写入到AOF 文件，而是将写命令追加到AOF缓冲区的末尾，之后 AOF缓存区再同步至 AOF文件中。为了避免额外的检查开销，Redis 在向 AOF 里面记录日志的时候，并不会先去对这些命令进行语法检查，这是因为 Redis 已经执行过该命令，可以确保命令的的正确性。·

![[../picture/Pasted image 20241208160955.png#pic_center|430]]

▧  **AOF 耐久性**

&emsp;&emsp; 由于AOF是先写入缓存区，再刷到磁盘上，在这个过程中就存在两个风险：
&emsp;&emsp; &emsp; ① 如果刚执行完一个命令，还没有来得及记日志就宕机了，那么这个命令和相应的数据就有丢失的风险。
&emsp;&emsp; &emsp; ② AOF 虽然避免了对当前命令的阻塞，但可能会给下一个操作带来阻塞风险。由于 AOF 日志是在主线程中执行的，如果在把日志文件写入磁盘时，磁盘写压力大，就会导致写盘很慢，从而导致后续的操作的阻塞。

&emsp;&emsp;针对上面的两个风险，Redis 提供了三种写回策略(也叫 AOF 耐久性)，用来控制一个写命令执行完后 AOF 日志写回磁盘的时机：
&emsp;&emsp; &emsp;● **同步写回 - appendfsync always**：每个写命令执行完，立马同步地将日志写回磁盘；
&emsp;&emsp; &emsp;● **每秒写回 - appendfsync everysec**：每个写命令执行完，只是先把日志写到 AOF 文件的内存缓冲区，每隔一秒把缓冲区中的内容写入磁盘；
&emsp;&emsp; &emsp;● **操作系统控制的写回 - appendfsync no**：每个写命令执行完，只是先把日志写到 AOF 文件的内存缓冲区，由操作系统决定何时将缓冲区内容写回磁盘。对大多数 Linux 操作系统，每 30 秒进行一次 fsync，将缓冲区中的数据写到磁盘上。

![[../picture/Pasted image 20241208223138.png#pic_center|650]]

▧  **AOF 重写**

&emsp;&emsp; 由于 AOF 采用文件追加方式，会导致文件会越来越大。过大的 AOF 文件会对 Redis 服务器甚至宿主机造成影响，并且 AOF 越大，使用 AOF 来进行数据恢复所需的时间也就越多。为了解决 AOF 文件体积膨胀的问题，Redis 提供了 AOF 文件重写 (rewrite) 机制。<font color=red>**Redis的 AOF 重写机制指的是将 AOF 文件中的冗余命令删除，以减小 AOF 文件的大小并提高读写性能的过程。**</font><font color=orage>虽然叫做AOF重写，但实际上 AOF 文件重写并不需要对现有的AOF 文件进行任何读取、分析或者写入操作，而是通过读取服务器当前的数据库状态来实现的。</font>AOF 重写的过程如下：

![[../picture/Pasted image 20241214202507.png#pic_center|550]]
&emsp;&emsp;&emsp;①  Redis 执行 fork() 创建子进程 ，现在同时拥有父进程和子进程；
&emsp;&emsp;&emsp;② 子进程开始将新 AOF 文件的内容写入到临时文件；
&emsp;&emsp;&emsp;③ 对于所有新执行的写入命令，父进程一边将命令写入内存缓存中，一边将改动追加到现有 AOF 文件的末尾，防止在重写的中途发生停机，导致 AOF 文件数据异常；
&emsp;&emsp;&emsp;④ 当子进程完成重写工作时，给父进程发送一个信号，父进程在接收到信号之后将内存缓存中的所有数据追加到新 AOF 文件的末尾。
&emsp;&emsp;&emsp;⑤ 最后将 Redis 原子地用新文件替换旧文件，之后所有命令都会直接追加到新 AOF 文件的末尾。

▧  **AOF 相关配置**

&emsp;&emsp;● `appendonly`：用于开启或关闭 AOF，默认为关闭。
&emsp;&emsp;&emsp;● `appendfilename`：设置 AOF 文件名，默认为 appendonly.aof。
&emsp;&emsp;&emsp;● `appendfsync`：该配置项用于设置 AOF 的同步机制。有三种可选值：
&emsp;&emsp;&emsp;&emsp; □ always：每个写命令都要同步到磁盘，安全性最高，但是性能较差。
&emsp;&emsp;&emsp;&emsp; □  everysec：每秒同步一次，是默认选项，既能保证数据安全，又具有较好的性能。
&emsp;&emsp;&emsp;&emsp; □ no：不进行同步，由操作系统决定何时将缓冲区中的数据同步到磁盘上，性能最好但安全性较低。
&emsp;&emsp;&emsp;● `auto-aof-rewrite-min-size` 与 `auto-aof-rewrite-percentage`：用于设置 AOF 重写规则，当 AOF 文件大小超过 `auto-aof-rewrite-min-size` 设置的值时，并且 AOF 文件增长率达到 `auto-aof-rewrite-percentage` 所定义的百分比时，Redis 会启动 AOF 重写操作。
&emsp;&emsp;&emsp;● `aof-use-rdb-preamble`：混合持久化。AOF重写期间是否开启增量式同步。该配置项在AOF重写期间是否使用RDB文件内容。默认是no，如果设置为yes，在AOF文件头加入一个RDB文件的内容，可以尽可能的减小AOF文件大小，同时也方便恢复数据。

##### 3. RDB 与 AOF 混合持久化
&emsp;&emsp; Redis 4.0 中提出了混合使用 AOF 日志和 RDB 内存快照的方法。<font color=orage>**RDB 内存快照以一定的频率执行，在两次快照之间，使用 AOF 日志记录这期间的所有命令操作。**</font>也就是将 RDB 文件的内容和增量的 AOF 日志文件存在一起。这里的 AOF 日志不再是全量的日志，而是自持久化开始到持久化结束的这段时间发生的增量 AOF 日志。混合持久化结合了 RDB 和 AOF 的优点，快速加载同时避免丢失过多的数据。

![[../picture/Pasted image 20241214212136.png#pic_center|580]]

#### 3.3.2 Redis 主从复制 - 数据多机热备
&emsp;&emsp; Redis 的主从复制是一种数据备份和读写分离的模式，是指将一台 Redis 服务器的数据，复制到其他的Redis服务器。前者称为主节点(master)，后者称为从节点(slave)。数据的复制是单向的，只能由主节点到从节点。所有的写操作都在主节点上进行，而读操作可以在主节点和从节点上进行。从节点会复制主节点的数据，实现数据的备份。主从复制的作用主要包括：
&emsp; &emsp; &emsp;  **① 数据冗余**：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
&emsp; &emsp; &emsp;  **② 故障恢复**：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复，是一种服务的冗余。
&emsp; &emsp; &emsp;  **③ 负载均衡**：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务 (即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点)，分担服务器负载。尤其在写少读多的场景下，通过多个从节点分担读负载，可以大大提高 Redis 服务器的并发量。

![[../picture/Pasted image 20241215110859.png#pic_center|450]]

##### 1.主从复制的数据同步过程
&emsp; &emsp;主从复制的数据同步过程大体可以分为3个阶段：连接建立阶段（即准备阶段）、数据同步阶段、命令传播阶段；

![[../picture/Pasted image 20241215172815.png#pic_center|450]]

- **1.连接建立阶段** 
	- **步骤1：保存主节点信息**：从节点服务器内部维护了两个字段，即 `masterhost` 和`masterport` 字段，用于存储主节点的ip和port信息。从节点完成主节点ip和port的保存后，向主节点异步发送 `slaveof` 命令，主节点向客户端直接返回OK。
	- **步骤2：建立socket连接**：从节点每秒1次调用复制定时函数 `replicationCron()`，如果发现了有主节点可以连接，便会根据主节点的ip和port，创建socket连接。如果连接成功，则：
	  &emsp; &emsp; □ 从节点：为该socket建立一个专门处理复制工作的文件事件处理器，负责后续的复制工作，如接收RDB文件、接收命令传播等。
	  &emsp; &emsp; □ 主节点：接收到从节点的socket连接后（即accept之后），为该socket创建相应的客户端状态，**并将从节点看做是连接到主节点的一个客户端，后面的步骤会以从节点向主节点发送命令请求的形式来进行。**
	-  **步骤3：发送ping命令**：从节点成为主节点的客户端之后，发送ping命令进行首次请求，目的是检查socket连接是否可用，以及主节点当前是否能够处理请求。
	-  **步骤4：身份验证**：如果从节点中设置了 masterauth 选项，则从节点需要向主节点进行身份验证，没有设置该选项，则不需要验证。
	-  **步骤5：发送从节点端口信息**：身份验证之后，从节点会向主节点发送其监听的端口号，主节点将该信息保存到该从节点对应的客户端的 slave_listening_port 字段中。
- 2.**数据同步阶段**
&emsp; &emsp;&emsp;主从节点之间的连接建立以后，便可以开始进行数据同步。从节点向主节点发送 psync 命令 (Redis2.8以前是sync命令），开始同步。根据主从节点当前状态的不同，可以分为<font color=red>**全量复制和部分复制**</font>。
- 3.**命令传播阶段**
  &emsp; &emsp;&emsp;数据同步阶段完成后，主从节点进入命令传播阶段。在这个阶段主节点将自己执行的写命令发送给从节点，从节点接收命令并执行，从而保证主从节点数据的一致性。在命令传播阶段，除了发送写命令，主从节点还维持着心跳机制：PING 和 REPLCONF ACK。<font color=orage>**命令传播是异步的过程，即主节点发送写命令后并不会等待从节点的回复，因此实际上主从节点之间很难保持实时的一致性，延迟在所难免。**</font>
	-  **主->从 PING**：每隔指定的时间，主节点会向从节点发送 PING 命令，主要作用是为了让从节点进行超时判断。PING 发送的频率由 `repl-ping-slave-period` 控制，单位是秒，默认值是10s。
	-  **从->主 REPLCONF ACK**：从节点会向主节点发送 REPLCONF ACK命令，频率是每秒1次，格式为：REPLCONF ACK {offset}，其中 offset 指从节点保存的复制偏移量。REPLCONF ACK命令的作用包括：
	   &emsp; &emsp;&emsp;**① 实时监测主从节点网络状态**：该命令会被主节点用于复制超时的判断。
	   &emsp; &emsp;&emsp;**② 检测命令丢失**：从节点发送了自身的offset，主节点会与自己的offset对比，如果从节点数据缺失 (如网络丢包），主节点会利用复制积压缓冲区推送缺失的数据。
	   &emsp; &emsp;&emsp;**③ 辅助保证从节点的数量和延迟**：Redis主节点中使用 `min-slaves-to-write`(从节点数量) 和 `min-slaves-max-lag` (从节点的延迟值)，来保证主节点在从节点数量太少，或延迟过高的情况下不会执行写命令。

##### 2.主从复制的数据同步方式
&emsp;&emsp;在Redis2.8以前，从节点向主节点发送sync命令请求同步数据，此时的同步方式是全量复制；在Redis2.8及以后，从节点可以发送psync命令请求同步数据，此时根据主从节点当前状态的不同，同步方式可能是全量复制或部分复制。
&emsp; &emsp; &emsp; ● **全量复制**：用于初次复制或其他无法进行部分复制的情况，将主节点中的所有数据都发送给从节点。
&emsp; &emsp; &emsp; ● **部分复制**：用于网络中断等情况后的复制，只将中断期间主节点执行的写命令发送给从节点，与全量复制相比更加高效。

▧  **全量复制过程**

&emsp;&emsp; 当从节点判断无法进行部分复制时，会向主节点发送全量复制的请求。或者从节点发送部分复制的请求，但主节点判断无法进行部分复制时会执行全量复制。Redis 通过 psync 命令进行全量复制的过程如下：

![[../picture/Pasted image 20241224232110.png#pic_center|660]]

&emsp; &emsp; ① 主节点收到全量复制的命令后，执行 `bgsave`，在后台生成RDB文件，并使用复制缓冲区记录从现在开始执行的所有写命令。该过程非常消耗CPU、内存(页表复制)、硬盘IO等资源。
&emsp; &emsp; &emsp; ② 主节点的 `bgsave` 执行完成后，将 RDB 文件发送给从节点；**从节点首先清除自己的旧数据，然后载入接收的RDB文件**，将数据库状态更新至主节点执行bgsave时的数据库状态。在主节点将RDB文件发送给从节点的过程中，会对主从节点的带宽都会带来很大的消耗，同时从节点清空老数据、载入新RDB文件的过程是阻塞的，此时从节点无法响应客户端的命令。
&emsp; &emsp; &emsp; ③ 主节点将复制缓冲区中的所有写命令发送给从节点，从节点执行这些写命令，将数据库状态更新至主节点的最新状态。
&emsp; &emsp; &emsp; ④ 如果从节点开启了AOF，则会触发 `bgrewriteaof` 的执行，从而保证AOF文件更新至主节点的最新状态。

▧  **部分复制过程**

&emsp;&emsp; 由于全量复制在主节点数据量较大时效率太低，因此 Redis2.8 开始提供部分复制，用于处理网络中断时的数据同步。部分复制的实现，依赖于三个重要的概念：
&emsp;&emsp;&emsp;① **复制偏移量**：主节点和从节点分别维护一个复制偏移量（offset），代表的是**主节点向从节点传递的字节数**。主节点向从节点传播N个字节数据时，主节点的 Offset 增加N，从节点每次收到主节点传来的N个字节数据时，从节点的 Offset 增加N。如果主从二者 Offset 相同，则主从数据状态一致，如果 Offset 不同，则不一致，此时可以根据两个 Offset 找出从节点缺少的那部分数据。
&emsp;&emsp;&emsp;② **复制积压缓冲区**：复制积压缓冲区是由主节点维护的、固定长度的、先进先出(FIFO)队列，默认大小1MB；当主节点开始有从节点时创建，其作用是备份主节点最近发送给从节点的数据。**无论主节点有一个还是多个从节点，都只需要一个复制积压缓冲区**。在命令传播阶段，主节点除了将写命令发送给从节点，还会发送一份给复制积压缓冲区，作为写命令的备份；除了存储写命令，复制积压缓冲区中还存储了其中的每个字节对应的复制偏移量（offset）。由于复制积压缓冲区定长且是先进先出，所以它保存的是主节点最近执行的写命令，时间较早的写命令会被挤出缓冲区。因此，当主从节点offset的差距过大超过缓冲区长度时，将无法执行部分复制，只能执行全量复制。
&emsp;&emsp;&emsp;③ **服务器运行ID (runid)**：每个Redis节点(无论主从)，在启动时都会自动生成一个随机ID - runid，用来唯一识别一个Redis节点。主从节点初次复制时，主节点将自己的runid发送给从节点，从节点将这个runid保存起来；当断线重连时，从节点会将这个runid发送给主节点；主节点根据runid判断能否进行部分复制：
&emsp; &emsp; &emsp; ●  如果从节点保存的runid与主节点现在的runid相同，说明主从节点之前同步过，主节点会尝试使用部分复制 (能不能部分复制还要看offset和复制积压缓冲区的情况)。
&emsp; &emsp; &emsp; ● 如果从节点保存的runid与主节点现在的runid不同，说明从节点在断线前同步的Redis节点并不是当前的主节点，只能进行全量复制。

![[../picture/Pasted image 20241225232112.png#pic_center|880]]

#### 3.3.3 Redis 哨兵 - 主从节点的故障自动转移
&emsp;&emsp; Redis 哨兵 (Redis Sentinel) 在Redis 2.8版本开始引入。哨兵的主要包括以下几个功能：
&emsp; &emsp; &emsp; ● **监控 Monitoring**：哨兵会不断地检查主节点和从节点是否正常工作。
&emsp; &emsp; &emsp; ● **配置提供者 Configuration Provider**：客户端在初始化时，通过连接哨兵来获得当前 Redis 服务的主节点地址。
&emsp; &emsp; &emsp; ● **通知 Notification**：哨兵可以将故障转移的结果发送给客户端。
&emsp; &emsp; &emsp; ● **自动故障转移 Automatic Failover**：当主节点不能正常工作时，哨兵会开始自动故障转移操作，它会将失效主节点的其中一个从节点升级为新的主节点，并让其他从节点改为复制新的主节点。

&emsp;&emsp; Redis 为哨兵模式提供了专属的命令及配置文件，是以独立进程运行，不存储任何数据。哨兵不只是监控 Redis 节点，各哨兵节点之间也会互相监控，保证哨兵节点的高可用。哨兵架构如下图所示：

![[../picture/Pasted image 20241228143307.png#pic_center|450]]

##### 1.哨兵机制的工作原理
&emsp;&emsp;在哨兵模式下，哨兵节点会定期检查主节点和从节点的运行状态。如果发现主节点发生故障，哨兵节点会在从节点中选举出一个新的主节点，并通知其他的从节点和哨兵节点。此外，哨兵节点还可以接收客户端的查询请求，返回当前的主节点信息，从而实现客户端的透明切换。哨兵机制在进行主从节点的故障转移过程中会经历三个阶段：**监控，通知(提醒)，故障转移**。
&emsp; &emsp; &emsp; ● **监控**：哨兵会不断地检查你的 Master 节点和 Slave 节点是否运作正常。
&emsp; &emsp; &emsp; ● **提醒(通知)**：当被监控的某个 Redis 节点出现问题时，哨兵可以通过 API 向管理员或者其他应用程序发送通知。
&emsp; &emsp; &emsp; ● **自动故障转移**：当 Master 异常时，哨兵选举其中一个 Slave 升级为新的 Master，并让失效 Master 的其他Slave改为复制新的 Master。

▧  **哨兵定时监控**

&emsp;&emsp;每个哨兵节点维护了3个定时监控任务。定时监控任务的功能分别如下：
&emsp;&emsp; &emsp; **任务1**：每个哨兵节点每 10 秒会向主节点和从节点发送 Info命令，获取最新的主从拓扑结构。

![[../picture/Pasted image 20241228160936.png#pic_center|320]]

&emsp; &emsp; **任务2**：每个哨兵节点每隔2秒向 Redis 数据节点的指定频道上发送改该哨兵节点对于主节点的判断，以及当前哨兵节点的信息，同时每个哨兵节点会订阅该频道，来了解其它哨兵节点的信息及对主节点的判断。
&emsp;&emsp; &emsp; **任务3**：每隔 1 秒每个哨兵会向主节点、从节点及其余哨兵节点发送一次 ping 命令进心跳检测，用来判断节点是否正常。

![[../picture/Pasted image 20241228161412.png#pic_center|395]]

▧  **哨兵提醒通知**

&emsp; &emsp; 当一个节点距离最后一次有效回复 ping 命令的时间超过 `down-after-milliseconds` 所指定的值，哨兵节点则认为该节点存在异常或下线。这种下线操作称为**主观下线**。主观下线并不代表这个 Master 节点真的不能用(有可能网络波动导致该哨兵与服务连接异常)，所以主观下线是不可靠的，可能存在误判。
&emsp; &emsp;&emsp;  如果主观下线的节点是主节点时，此时该哨兵节点会通过指令`sentinelis-masterdown-by-addr` 寻求其它哨兵节点对主节点的判断，当超过 `quorum` (法定人数) 个数，此时哨兵节点则认为该主节点确实有问题，大部分哨兵节点都同意下线操作。这种下线操作称为**客观下线**。<font color=orange>客观下线是主节点才有的概念，如果从节点和哨兵节点发生故障，被哨兵主观下线后，不会再有后续的客观下线和故障转移操作</font>。

![[../picture/Pasted image 20241228163139.png#pic_center|590]]

▧  **哨兵选举和故障转移**

&emsp;&emsp; 当主节点发生故障时，由哨兵节点之间基于 **Raft** 算法进行投票选举，按照谁发现主节点故障谁去处理的原则，选举出一个领导者哨兵节点 ( Leader Sentinel )。这个 Leader 哨兵节点负责进行故障转移操作。通常哨兵选择的过程很快，谁先完成客观下线，一般就能成为 Leader 哨兵。

![[../picture/Pasted image 20241228232539.png#pic_center|780]]

&emsp;&emsp; Leader 哨兵选举完成后，由 Leader 哨兵开始进行故障转移操作，故障转移的操作过程如下：
&emsp; &emsp; &emsp; **① 在从节点中选择最优的从节点作为新的主节点**，选择的原则是：首先过滤掉不健康的从节点，然后选择优先级最高的从节点(由slave-priority指定)。如果优先级无法区分，则选择复制偏移量最大的从节点。如果仍无法区分，则选择 runid 最小的从节点。
&emsp; &emsp; &emsp; **② 更新主从状态**：通过 `slaveof no one` 命令，让选出来的从节点成为主节点，并通过 `slaveof` 命令让其他节点成为其从节点。
&emsp; &emsp; &emsp; **③ 原主节点变为从节点**：将已经下线的主节点设置为新的主节点的从节点，当原主节点重新上线后，它会成为新的主节点的从节点。

##### 2.哨兵配置文件
```javascript
# 哨兵sentinel实例运行的端口，默认26379  
port 26379

# 哨兵sentinel的工作目录
dir ./

# 哨兵sentinel监控的redis主节点的 
## ip：主机ip地址
## port：哨兵端口号
## master-name：可以自己命名的主节点名字（只能由字母A-z、数字0-9 、这三个字符".-_"组成。）
## quorum：当这些quorum个数sentinel哨兵认为master主节点失联 那么这时 客观上认为主节点失联了  
# sentinel monitor <master-name> <ip> <redis-port> <quorum>  
sentinel monitor mymaster 127.0.0.1 6379 2

# 当在Redis实例中开启了requirepass <foobared>，所有连接Redis实例的客户端都要提供密码。
# sentinel auth-pass <master-name> <password>  
sentinel auth-pass mymaster 123456  

# 指定主节点应答哨兵sentinel的最大时间间隔，超过这个时间，哨兵主观上认为主节点下线，默认30秒  
# sentinel down-after-milliseconds <master-name> <milliseconds>
sentinel down-after-milliseconds mymaster 30000  

# 指定了在发生failover主备切换时，最多可以有多少个slave同时对新的master进行同步。这个数字越小，完成failover所需的时间就越长；反之，但是如果这个数字越大，就意味着越多的slave因为replication而不可用。可以通过将这个值设为1，来保证每次只有一个slave，处于不能处理命令请求的状态。
# sentinel parallel-syncs <master-name> <numslaves>
sentinel parallel-syncs mymaster 1  

# 故障转移的超时时间failover-timeout，默认三分钟，可以用在以下这些方面：
# 1. 同一个sentinel对同一个master两次failover之间的间隔时间。  
# 2. 当一个slave从一个错误的master那里同步数据时开始，直到slave被纠正为从正确的master那里同步数据时结束。  
# 3. 当想要取消一个正在进行的failover时所需要的时间。
# 4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来同步数据了
# sentinel failover-timeout <master-name> <milliseconds>  
sentinel failover-timeout mymaster 180000

# 当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本。一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。
# 对于脚本的运行结果有以下规则：  
# 1. 若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10。
# 2. 若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。  
# 3. 如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
# sentinel notification-script <master-name> <script-path>  
sentinel notification-script mymaster /var/redis/notify.sh

# 这个脚本应该是通用的，能被多次调用，不是针对性的。
# sentinel client-reconfig-script <master-name> <script-path>
sentinel client-reconfig-script mymaster /var/redis/reconfig.sh
```

#### 3.3.4 Redis 集群
&emsp;&emsp; Redis 单实例的架构，从最开始的一主N从，到读写分离，再到 Sentinel 哨兵机制，单实例的 Redis 缓存足以应对大多数的使用场景，也能实现主从故障迁移。但是，在某些场景下，单实例存Redis缓存会存在的几个问题：
&emsp; &emsp; &emsp; ● **写并发**：Redis 单实例读写分离可以解决读操作的负载均衡，但对于写操作，仍然是全部落在了master节点上面，在海量数据高并发场景，一个节点写数据容易出现瓶颈，造成master节点的压力上升。
&emsp; &emsp; &emsp; ● **海量数据的存储压力**：单实例 Redis 只有一台 Master 作为存储，如果面对海量数据的存储，一台 Redis 的服务器就应付不过来了，而且数据量太大意味着持久化成本高，严重时可能会阻塞服务器，造成服务请求成功率下降，降低服务的稳定性。

&emsp;&emsp; 基于上述的问题，Redis 3.0开始引入的分布式存储方案，它允许多个 Redis 节点 (服务器) 协同工作。<font color=red>**Redis 集群会对数据进行分片，将不同的数据存储在不同的 master 节点上，每个节点负责一部分数据的读写**</font>。Redis集群采用去中心化的思想，没有中心节点的说法。对于客户端，整个集群可以看成一个整体，可以连接任意一个节点进行操作，不需要任何代理中间件，当客户端操作的key没有分配到该 node 上时，Redis会返回转向指令，指向正确的 node。**PS：集群模式主要解决水平拓展问题和整体的高可用 (局部节点故障不影响其他节点的数据)。如果要保证所有数据的高可用还需要配合主从模式。**

![[../picture/Pasted image 20241229111622.png#pic_center|650]]

![[../picture/Pasted image 20250104191908.png#pic_center|880]]

##### 1.集群的工作原理
###### (1).集群的分片
&emsp; &emsp; Redis 集群会对数据进行分片，将数据进行拆分到不同的Redis服务器上。Redis集群采用**哈希槽分区算法 (一致性哈希算法)**，将整个数据库分为 16384 个槽位 slot，所有key-value数据都存储在这些 slot 中的某一个上。一个 slot 槽位可以存放多个数据，而每个 Master 主节点负责一部分哈希槽，每个 Master 节点中存在一个 clusterNode 数据结构，其slots属性记录了当前 Master 节点负责处理那些槽。当一个键需要被存储时，Redis 会根据key 的值计算出一个哈希值，然后根据哈希值决定将这个键存储在哪个节点上。 key 的槽位计算公式为：`slot_number=crc16(key)%16384`，其中 crc16 为 16 位的循环冗余校验和函数。
&emsp; &emsp;&emsp;由于Redis集群无中心节点，请求会随机发给任意主节点 (通常情况下客户端有哈希槽的本地映射缓存，只有在第一次请求或Redis集群迁移时会导致缓存失败)，而主节点只会处理自己负责槽位的命令请求，对于其它槽位的命令请求，该主节点会返回客户端一个 ASK 转向错误，客户端根据 ASK 错误中包含的地址和端口重新向正确的负责的主节点发起命令请求。

![[../picture/Pasted image 20241229162508.png#pic_center|650]]

###### (2).节点之间的通信机制
&emsp; &emsp; 由于 Redis 集群是去中心化的架构，当集群的状态发生变化时，比如新节点加入、slot迁移、节点宕机、slave提升为新Master等，这些变化需要尽快被其他节点发现。<font color=orage>**为了维护节点之间的元数据信息状态的一致性，在Redis集群中，不同的节点之间采用 Gossip 协议进行通信**</font>。Gossip 协议，利用一种随机、带有传染性的方式，将信息传播到整个网络中，并在一定时间内，使得系统内的所有节点数据一致。Gossip 协议的方法论是**在一个处于有界网络的集群里，如果每个节点都随机与其他节点交换特定信息，经过足够长的时间后，集群各个节点对该份信息的认知终将收敛到一致**。由于是节点直接相互交换信息，即使集群节点的数量增加，每个节点的负载也不会增加很多，几乎是恒定的。这就允许 Redis 集群管理的节点规模能横向扩展到数千个。

![[../picture/Pasted image 20241229211043.png#pic_center|440]]

&emsp; &emsp; 在 Redis 集群中的每个节点都**维护一份自己视角下的当前整个集群的状态**，主要包括：当前集群状态、集群中各节点所负责的哈希槽 slots 信息、集群中各节点的master-slave状态、集群中各节点的存活状态及怀疑Fail状态。Redis 集群的节点之间会相互发送多种消息，较为重要的有以下几个消息：
- **① 定时 Ping / Pong 消息**
  &emsp;&emsp;节点按照配置的时间间隔向集群中其他节点发送 ping 消息，消息中带有自己的状态，还有自己维护的集群元数据，和部分其他节点的元数据。当其他节点收到 Ping 消息后会这些消息会回复一个 Pong 消息。经过 Ping 和 Pong 通信，两个节点就可以交换各自的节点状态信息，检查各个节点状态，包括在线状态、疑似下线状态 PFAIL 和已下线状态 FAIL。
- **② 新节点加入 Meet 消息**
	- Redis 集群加入新节点时，客户端需要执行 `Cluster Meet` 命令。
	- 收到 `Cluster Meet` 命令的节点A会为新节点创建一个 clusterNode 数据，并将其添加到自己维护的 clusterState 的 nodes 字典中。
	- 随后根据`Cluster Meet` 命令中的 IP 地址和端口号，向新节点发送一条 Meet 消息。新节点收到消息后也会为节点创建一个 clusterNode 结构，并将该结构添加到自己维护的 clusterState 的 nodes 字典中，并返回一条 Pong 消息。
	- 节点A还会向新节点发送一条 Ping 消息。新节点接收到该条 Ping 消息后，可以知道节点A已经成功的接收到了自己返回的 Pong 消息，从而完成了新节点接入的握手操作。![[../picture/Pasted image 20250102233441.png#pic_center|500]]
- **③ 节点疑似下线和真正下线**
	- **疑似下线 - PFail 状态**：Redis 集群中的节点会定期检查已经发送 PING 消息的接收方节点A是否在规定时间 (cluster-node-timeout) 内返回了 PONG 消息，如果没有则会将其标记为疑似下线状态。
	- **真正下线 - Fail 状态**：随着时间的推移，当半数以上的节点认为节点A处于疑似下线状态时，会将节点A标记为已下线 FAIL 状态。

![[../picture/Pasted image 20250102231417.png#pic_center|640]]

### 3.4 Redis 内存
#### 3.4.1 Redis 内存模型与管理
&emsp;&emsp; 内存是计算机系统中最重要的存储资源之一。Redis 将所有数据存储在内存中，这对内存管理产生巨大的挑战。如何高效的分配和释放内存？如何在内存有限的情况下处理大量数据？这些都是 Redis 内存模型需要解决的问题。Redis作为一个内存数据库，在内存中存储的内容主要是数据（键值对）。但是，除了数据以外，Redis还有其他部分也会占用内存。Redis的内存占用主要可以划分为以下几个部分：
![[../picture/Pasted image 20250105155532.png#pic_center|480]]
&emsp; &emsp; &emsp;① **数据内存**：作为数据库，数据是最主要的部分。这部分占用的内存会统计在 `used_memory`中。Redis使用键值对存储数据，其中的值(对象)包括5种类型，即字符串、哈希、列表、集合和有序集合。此外，Redis在存储对象时，并不是直接将数据放入内存，而是会将对象进行各种包装：如redisObject、SDS等。
&emsp; &emsp; &emsp;② **进程本身运行需要的内存**：Redis主进程本身运行需要占用内存，如代码、常量池等等。这部分内存不会统计在`used_memory`中，除了主进程外，Redis创建的子进程运行也会占用内存，如 Redis执行AOF、RDB重写时创建的子进程。
&emsp; &emsp; &emsp;③ **缓冲内存**：缓冲内存包括客户端缓冲区、复制积压缓冲区、AOF缓冲区等。其中，客户端缓冲区存储客户端连接的输入输出缓冲；复制积压缓冲区用于部分复制功能；AOF缓冲区用于在进行AOF重写时，保存最近的写入命令。这部分内存由jemalloc分配，会被统计在`used_memory`中。
&emsp; &emsp; &emsp;④ **内存碎片**：内存碎片是 Redis 在分配、回收物理内存过程中产生的。如果对数据更改频繁，而且数据之间的大小相差很大，可能导致 Redis 释放的空间在物理内存中并没有释放，但 Redis 又无法有效利用，就形成了内存碎片。如果Redis 服务器中的内存碎片已经很大，可以通过安全重启的方式减小内存碎片。重启之后，Redis 重新从备份文件中读取数据，在内存中进行重排，为每个数据重新选择合适的内存单元，减小内存碎片。

&emsp;&emsp; 要想了解 Redis 内存模型，就要知道如何统计Redis使用内存的情况。在 Redis 客户端通过 `info memory` 命令可以查看内存使用情况，常用的内存参数如下：
&emsp; &emsp; &emsp;  ① `used_memory`：Redis分配器分配的内存总量，单位为字节，包括使用的虚拟内存。
&emsp; &emsp; &emsp;  ② `used_memory_rss`：Redis进程占据操作系统的内存，单位是字节，包括进程运行本身的内存、内存碎片等，但不包括虚拟内存。
&emsp; &emsp; &emsp;  ③ `mem_fragmentation_ratio`：内存碎片比率，该值是`used_memory_rss / used_memory`的比值。`mem_fragmentation_ratio` 一般大于1，且该值越大，内存碎片比例越大。如果 `mem_fragmentation_ratio ＜ 1`， 说明 Redis 使用了虚拟内存，由于虚拟内存的媒介是磁盘，比内存速度要慢很多，当这种情况出现时，应该及时排查是否是内存不足，并及时处理，如增加Redis节点、增加Redis服务器的内存、优化应用等。
&emsp; &emsp; &emsp;  ④ `mem_allocator`：Redis 使用的内存分配器，在编译时指定。可以是 libc 、jemalloc或者tcmalloc，默认是jemalloc。
&emsp; &emsp; &emsp;  ⑤ `used_memory_overhead`：Redis 进程开销所占用内存大小，包括缓冲区、连接和其他元数据。
&emsp; &emsp; &emsp;  ⑥ `used_memory_dataset`：Redis 数据集占用内存大小，是实际存储的数据的内存占用。
&emsp; &emsp; &emsp; ⑦ `mem_clients_normal`：Redis 客户端连接输入输出缓冲区占用的内存大小。
``` java
# Memory 这是一个存在故障的 Redis 的内存信息
# 具体的故障表现是：Redis的内存使用率到达100%，但是实际的业务流量没有增长
used_memory:1072693248
used_memory_human:1023.99M  // 内存的实际使用情况
maxmemory:1073741824
maxmemory_human:1.00G       // Redis的最大内存限制
maxmemory_policy:noeviction // Redis的内存淘汰策略

used_memory_rss:1090519040  // Redis进程占据操作系统的内存
used_memory_rss_human:1.02G
used_memory_peak:1072693248
used_memory_peak_human:1023.99M
used_memory_peak_perc:100.00%
used_memory_overhead:1048576000  // Redis进程开销所占用内存大小，包括缓冲区、连接和其他元数据
used_memory_startup:1024000
used_memory_dataset:23929848     // Redis数据集占用内存大小，是实际存储的数据的内存占用。
used_memory_dataset_perc:2.23%   // Redis数据集占用内存的占比
allocator_allocated:1072693248
allocator_active:1090519040
allocator_resident:1090519040
total_system_memory:16777216000
total_system_memory_human:16.00G
used_memory_lua:37888
used_memory_lua_human:37.89K
used_memory_scripts:1024000
used_memory_scripts_human:1.00M
allocator_frag_ratio:1.02
allocator_frag_bytes:17825792
allocator_rss_ratio:1.00
allocator_rss_bytes:0
rss_overhead_ratio:1.00
rss_overhead_bytes:0
mem_fragmentation_ratio:1.02   // 内存碎片比率，碎片率不高，表明内存被合理使用但被缓冲区占用过多。
mem_fragmentation_bytes:17825792
mem_not_counted_for_evict:0
mem_replication_backlog:0
mem_clients_slaves:0
mem_clients_normal:1048576000(1.00GB)  // Redis客户端连接输入输出缓冲区占用的内存大小，意味着当前Redis内存被输出缓存区占满了
mem_aof_buffer:0
mem_allocator:jemalloc-5.1.0
active_defrag_running:0
lazyfree_pending_objects:0
```

##### 1.数据内存回收策略
&emsp;&emsp;Redis 的内存回收机制仅能处理数据集合所占用的内存，主要体现在两个方面上：
&emsp; &emsp; &emsp;**① 对过期数据的处理，删除达到过期时间的键值对象。**
&emsp; &emsp; &emsp;**② 当内存使用情况达到 maxmemory 时触发内存淘汰策略，强制删除选择出来的键值对象。**

▧  **过期数据的删除**

&emsp;&emsp;过期数据的删除有两种方式：**惰性删除**和**定时删除**。
&emsp;&emsp;&emsp; ● **惰性删除**：当一个键值对过期之后，不会立即从内存中删除，Redis 会等到下次访问这个键值对时，才会将其删除。这样做的目的主要是为了节省 CPU 成本考虑，不需要单独维护 TTL 链表来处理过期键的删除。但如果当前的键值永远不再被访问时，那么该过期的键值对就无法被删除，就会造成内存泄漏的问题。
&emsp;&emsp;&emsp; ● **定时删除**：为了避免出现惰性删除导致的内存泄漏问题，Redis内部维护了一个定时任务，每隔100ms就会**随机选择**一些键值对，将过期的键值对删除。如果超过 25% 的 key 过期了，则任务会一直循环执行删除的过程，直到过期 key 的比例降至 25% 以下。<font color=orange>需要注意的是：**删除操作是阻塞的，循环执行删除过程会引起其他键值操作的延迟增加，导致 Redis 响应就会变慢**。</font><font color=orage>如果频繁使用带有相同时间参数的 EXPIREAT 命令设置过期 key，且key的过期时间相同，则在同一秒内有大量的 key 同时过期就会触发该循环，因此通常会将带有过期Key的时间参数用固定值 + 随机数的方式打散过期时间。</font>

![[../picture/Pasted image 20250111220403.png#pic_center|600]]

▧  **内存淘汰策略**

&emsp;&emsp; 虽然 Redis 的内存管理很高效，但是当内存达到上限`maxmemory`后，就会根据`maxmemory-policy`设置的相关策略进行对应的操作，Redis 支持以下六种策略：
![[../picture/Pasted image 20250111190424.png#pic_center|860]]

##### 2. 输入输出缓冲区
&emsp;&emsp; Redis 是典型的 client-server 架构，所有的操作命令都需要通过客户端发送给服务器端，请求结果再由服务端返回给客户端。<font color=orage>为了避免客户端和服务器端的请求发送和处理速度不匹配，服务器端给每个连接的客户端都设置了一个输入缓冲区和输出缓冲区</font>。输入缓冲区会先把客户端发送过来的命令暂存起来，Redis 主线程再从输入缓冲区中读取命令进行处理。当 Redis 主线程处理完数据后，会把结果写入到输出缓冲区，再通过输出缓冲区返回给客户端，如下图：
![[../picture/Pasted image 20250112180937.png#pic_center|420]]
-  缓冲区的大小总是有上限的，当其中的数据积压太多就会发生缓冲区溢出的情况。
	- **输入缓冲区溢出**：输入缓冲区大小阈值设定为1GB，没有参数可以调整这个阈值。当超过这个大小时，会导致输入缓冲区溢出。
		- <font color=orange>【**现象**】</font>：输入缓冲区发生溢出后，对应的客户端连接会被 Redis 关闭，导致客户端连接受影响。当多个客户端连接占用的内存总量超过 `maxmemory` 阈值时会触发 redis 数据淘汰导致数据异常。
		- <font color=orage>【**原因**】</font>：当写入了 bigkey 时，会导致输入缓冲区被大量占用。或者 redis 主线程出现间歇性阻塞，请求处理速度变慢，导致缓冲区中堆积数据越来越多。
	- **输出缓冲区溢出**：
		- <font color=orange>【**现象**】</font>：输出缓冲区溢出后对应的客户端连接会被 Redis 关闭，导致业务受影响。当输出缓冲区占用的内存总量超过 `maxmemory` 阈值时会触发 redis 数据淘汰导致数据异常。
		- <font color=orage>【**原因**】</font>：输出缓冲区溢出的原因有以下三种：
		  &emsp;&emsp; ① 当写入了 bigValue 时，服务器端返回 bigkey 的大量结果，会导致输出缓冲区被大量占用。
		  &emsp;&emsp; ② 执行了 MONITOR 命令。MONITOR 命令是用来监测 Redis 执行的，执行这个命令之后，会持续输出监测到的各个命令操作，从而会持续占用输出缓冲区
		  &emsp;&emsp; ③ 缓冲区大小设置得不合理，可以通过 `client- output-buffer-limit` 配置项来设置缓冲区的大小。
		
### 3.5 Redis 问题与使用规范
#### 3.5.1 Redis 可能产生的问题
##### 1. bigKey 问题
&emsp;&emsp; 大Key问题通常是指值 Value 的长度非常大，而实际上键 Key 长度很大也是大Key问题的一种。通常来说，键本身不会很长，占用的内存较少，因此判断一个键是否为bigKey主要看它对应的值的大小。因此，**大Key分为两种情况：**① 键 Key 非常大、② 值 Value 非常大。

###### (1).键 Key 非常大
&emsp;&emsp; 虽然 Redis 的键可以存储任意字符串 (最大限制为512M)，但通常情况下，键的长度都比较小。**过长的键会带来以下问题：**
&emsp;&emsp; &emsp; **① 内存占用增加**：键的长度直接影响内存使用。如果键的长度过大且大量存在时，会增加内存消耗。
&emsp;&emsp; &emsp; **② 性能下降**：Redis的操作 (如查找、删除、更新等) 都需要对键进行哈希计算或字符串比较。如果键的长度过长，这些操作的时间复杂度会增加，导致性能下降。
&emsp;&emsp; &emsp; **③ 网络传输开销**：在客户端与Redis服务器之间传输数据时，过长的键会增加网络带宽的使用。

###### (2).值 Value 非常大
&emsp;&emsp; Redis 的值可以是多种类型的数据结构，包括字符串、列表、集合、哈希表、有序集合等。当值的大小非常大时，也会对 Redis 的性能和内存使用产生负面影响。Redis 的大Key没有固定的判别标准，通常认为<font color=orage>字符串类型的key对应的value值占用空间大于1M，或者集合类型的k元素数量超过1万个，就算是大key。</font>在设计与运用 Redis 时，要依据业务需求与性能指标来确立合理的大key阈值。

▧  **大key的影响**

&emsp;&emsp; 当出现大key时，会带来以下几方面的影响：
&emsp;&emsp; &emsp; ① **内存占用过高**：大Key占用过多的内存空间，可能导致可用内存不足，从而触发内存淘汰策略。在极端情况下，可能导致内存耗尽，Redis 实例崩溃，影响系统的稳定性。
&emsp;&emsp; &emsp; ② **性能下降**：大Key会占用大量内存空间，导致内存碎片增加，进而影响Redis的性能。对于大Key的操作，如读取、写入、删除等，都会消耗更多的CPU时间和内存资源，进一步降低系统性能。
&emsp;&emsp; &emsp; ③ **阻塞其他操作**：某些对大Key的操作可能会导致 Redis 实例阻塞。例如，使用DEL命令删除一个大Key时，可能会导致 Redis 实例在一段时间内无法响应其他客户端请求，从而影响系统的响应时间和吞吐量。
&emsp;&emsp; &emsp; ④ **网络拥塞**：每次获取大key产生的网络流量较大，可能造成机器或局域网的带宽被打满，导致其他服务无法正常响应。例如：一个大key占用空间是1MB，每秒访问1000次，就有1000MB的流量。
&emsp;&emsp; &emsp; ⑤ **主从同步延迟**：当 Redis 实例配置了主从同步时，大Key可能导致主从同步延迟。由于大Key占用较多内存，同步过程中需要传输大量数据，这会导致主从之间的网络传输延迟增加，进而影响数据一致性。
&emsp;&emsp; &emsp; ⑥ **数据倾斜**：在Redis集群模式中，某个数据分片的内存使用率远超其他数据分片，无法使数据分片的内存资源达到均衡。另外也可能造成 Redis 内存达到maxmemory参数定义的上限导致重要的key被逐出，甚至引发内存溢出。

▧  **大key的处理**

&emsp;&emsp; 当出现大Key问题时，就需要对大Key进行处理，通常有以下几个处理方法：
&emsp;&emsp; &emsp; ① 拆分成多个小key，降低单key的大小，读取可以用mget批量读取。
&emsp;&emsp; &emsp; ② 数据压缩，使用String类型的时候，使用压缩算法减少value大小。
&emsp;&emsp; &emsp; ③ 设置合理的过期时间，为每个key设置过期时间，并设置合理的过期时间，以便在数据失效后自动清理，避免长时间累积的大Key问题。
&emsp;&emsp; &emsp; ④ 启用内存淘汰策略，例如 LRU，以便在内存不足时自动淘汰最近最少使用的数据，防止大Key长时间占用内存。
&emsp;&emsp; &emsp; ⑤ 使用UNLINK命令删除大key，UNLINK 命令是 DEL 命令的异步版本，它可以在后台删除Key，避免阻塞 Redis 实例。

##### 2. 热点 Key (缓存击穿) 问题
&emsp;&emsp; Redis 集群中数据都是均匀分配到每个节点，请求也会均匀的分布到每个分片上。但在一些特殊场景中，比如外部爬虫攻击、热点商品、热点新闻、热点评论等，这种短时间内某些 key 访问量过于大，对于这种相同的 key 会请求到同一台数据分片上，导致该分片负载较高成为瓶颈问题，还可能导致缓存击穿，使得请求直接涌向数据库，导致服务雪崩等一系列问题。
&emsp;&emsp;&emsp;  Redis单节点的查询性能一般在2万QPS，因此对单个固定Key的查询不能超过这个数值。在服务端读取数据并进行分片切分时，如果对某个Key的访问量超过了该节点 Server 的承受极限，热点Key问题就会出现。

▧  **热点key的影响**

&emsp;&emsp; 当出现热点 key 时，会带来以下几方面的影响：
&emsp;&emsp; &emsp; ① **流量集中超过网卡带宽上限**：热点Key请求过多，超过单个分片的主机网卡流量上限，影响当前分片的服务稳定性。
&emsp;&emsp; &emsp; ② **打垮热点Key缓存分片**：热点Key查询超阈值会占用大量CPU资源，降低整体性能，导致热点Key对应的缓存分片服务崩溃。
&emsp;&emsp; &emsp; ③ **集群访问倾斜**：在集群架构下，某个数据分片被大量访问，其他分片空闲，可能导致该分片连接数耗尽，新连接请求被拒。
&emsp;&emsp; &emsp; ④ **缓存击穿与业务雪崩**：热点Key请求超 Redis 承受能力致缓存击穿，缓存失效时大量请求DB层，引发雪崩，影响业务。

▧  **热点key的处理**

&emsp;&emsp; 当出现热点 Key 问题时，就需要对热点Key进行处理，通常有以下几个处理方法：
&emsp;&emsp; &emsp; ① **多级缓存策略**：通过在客户端浏览器、就近CDN、Redis等缓存框架以及服务器本地进行缓存，形成二级、三级等多级缓存，目的是尽量缩短用户访问链路长度。
&emsp;&emsp; &emsp; ② **本地缓存**：使用本地缓存，如利用Ehcache、Caffeine等，甚至是一个HashMap都可以。在发现热点Key以后，把热点Key加载到JVM中，针对这种热点Key请求，直接从本地存中取，而不会直接请求 Redis。
&emsp;&emsp; &emsp; ③ **热 Key 备份**：在多个Redis节点上备份热 Key，避免固定Key总是访问同一节点。通过在初始化时为 Key 拼接0-2N之间的随机尾缀，便生成的备份 Key 分散在各个节点上。在有热Key请求时，随机选取一个备份 Key所在的节点进行访问取值，这样读写操作就不会集中于单个节点，从而有效减轻了单个Redis节点的负担，提升系统应对热 Key 问题的能力。
&emsp;&emsp; &emsp; ④ **热 Key 拆分**：将热 Key 拆分成多个带后缀名的 Key，分散存储到多个实例中。客户端请求时按规则算出固定 Key，使多次请求分散到不同节点。如以“某热搜”为例，拆分成多个带编号后缀的 Key 存储在不同节点，用户查询时根据用户ID算出下标访问对应节点。
&emsp;&emsp; &emsp; ⑤ **热 key 隔离**：为避免热 Key 问题波及核心业务，将存在热 Key 的Redis集群与核心业务隔离开，保障核心业务不受热 Key 问题影响。

##### 3. 缓存穿透问题
&emsp;&emsp; 缓存穿透是指客户端请求的数据在缓存中和数据库中都不存在，这样缓存永远不会生效，这些请求都会打到数据库。当有无数个请求同时请求时，则数据库大概率会崩溃。目前常见的解决方案有**返回缓存空对象**和**布隆过滤**两种方法：

▧  **返回缓存空对象**

&emsp;&emsp; 当一个请求直接打到数据库的时候，数据库返回为空(null) 时，直接在Redis中缓存一个空对象(null)，并设置一定的过期时间，当多次相同的请求时，直接命中 Redis 并返回结果。该方法实现简单，维护方便。但可能造成短期的 Redis 与 DB 数据不一致。

![[../picture/Pasted image 20250206231720.png#pic_center|320]]

▧  **布隆过滤**

&emsp;&emsp; 布隆过滤器采用的是哈希思想来解决这个问题，通过一个庞大的二进制数组，判断当前这个要查询的这个数据是否存在，如果布隆过滤器判断存在，则这个请求会去访问 Redis。如果布隆过滤器判断这个数据不存在，则直接返回。

![[../picture/Pasted image 20250209212605.png#pic_center|450]]

##### 4. 缓存雪崩问题
&emsp;&emsp; 缓存雪崩是指在同一时段大量的缓存 key 同时失效或者 Redis 服务宕机，导致大量请求到达数据库，带来巨大压力。目前常见的解决方案有以下几种：
&emsp;&emsp; &emsp; ① 设置不同的过期时间：给缓存数据设置不同的过期时间，让它们在一段时间内均匀过期。这样可以避免大量缓存同时失效。
&emsp;&emsp; &emsp; ② 给缓存业务添加降级限流策略：当请求量超过一定阈值时，拒绝部分请求，保护数据库。

#### 3.5.2 Redis 使用规范
![[../picture/Pasted image 20250111235733.png#pic_center|700]]