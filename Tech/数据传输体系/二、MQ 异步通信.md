&emsp;&emsp;   MQ全称是 *Message Queue* ( 消息队列 )，是基于 *FIFO* 队列数据结构的的一种消息容器，可以保存在传输过程中消息，是存储消息的一种中间件。常应用在分布式系统中进行通信的第三方中间件，一般用来解决**应用解耦，异步消息，流量削峰**等问题。如下图所示，发送方称为生产者，接收方称为消费者。
![[../picture/Pasted image 20240621231045.png#pic_center|550]]

&emsp;&emsp;MQ 的应用虽然能解决应用解耦，异步提速，削峰填谷等问题，但是引入MQ也会出现一些问题，MQ的劣势以及可能引发的问题如下：
&emsp;&emsp;&emsp;① **系统可用性降低**：系统引用的外部依赖越多，稳定性就越差。一旦MQ宕机，整个系统就会不可用，对业务造成影响**，如何保证MQ的高可用？**
&emsp;&emsp;&emsp;② **系统复杂度提高**：通过MQ的异步调用会存在以下问题：**如何保证消息没有被重复消费，怎么处理消息丢失情况？怎么保证消息传递的顺序性？**
&emsp;&emsp;&emsp;③ **数据一致性问题**：如果A系统处通过MQ给B、C系统发送消息数据，B处理成功，C处理失败，这种情况怎么处理**，即如何保证消息数据处理的一致性？**

&emsp;&emsp;目前常见的MQ及数据对比如下图所示：
![[../picture/Pasted image 20240621231419.png#pic_center|750]]

---
### 2.1 Kafka
&emsp;&emsp; Kafka 是一个发布和订阅数据流、存储和处理数据流的系统，同时也是一个支持分区的、多副本的、多订阅者的、基于 Zookeeper 协调的分布式消息中间件。Kafka 用于离线和在线消息的消费，将消息数据按顺序保存在磁盘上，并在集群内以副本的形式存储以防止数据丢失。Kafka 是一个分布式系统，由通过高性能TCP网络协议进行通信的**服务器和客户端**组成。**Kafka是在JVM之上构建**，它可以部署在本地和云环境中的裸机硬件、虚拟机和容器上。
&emsp;&emsp;&emsp;①  **服务器**：Kafka作为一个或多个服务器的集群运行，可以跨越多个服务器中心或云区域。其中一些服务器构成了存储层，称为代理。其他服务器运行 Kafka Connect 以持续导入和导出数据作为事件流，从而将Kafka与现有的系统 (如关系数据库以及其他Kafka集群) 集成。Kafka 集群具有高度可扩展性和容错性：如果其中任何一台服务器出现故障，其他服务器将接管其工作，以确保连续运行而不会丢失任何数据。
&emsp;&emsp;&emsp;② **客户端**：客户端是分布式应用程序和微服务，即使在网络问题或机器故障的情况下，也可以并行、大规模、容错地读取、写入和处理事件流。

&emsp;&emsp; Kafka 本质上也是一个消息队列 ( Message Queue )。Kafka 有两种数据传输模型：**点对点模型 和 发布/订阅模型**
&emsp; &emsp;&emsp; ● **点对点模型**：一对一，消费者主动拉取数据。消息生产者将消息发送到 MQ 中，然后消费者从 MQ 中取出并消费消息，消息被消费者消费以后，MQ 中不再存储已经被消费过的消息，所以消息消费者不可能重复消费已经被消费过的消息。MQ 支持存在多个消费者，即一个消费者组可以有多个消费者，但是消息只能被一个消费者消费。


![[../picture/Pasted image 20240622084720.png#pic_center|500]]

&emsp;  &emsp;● **发布/订阅模型**：发布/订阅模型与点对点模型多了一个 *Topic* 的概念，可以存在多个发布者向相同主题发送消息，而订阅者也可以存在多个，接收相同主题的消息。消费者消费数据之后不会清除消息数据。

![[../picture/Pasted image 20240622084754.png#pic_center|500]]

#### 2.1.1 Kafka 架构
&emsp;  &emsp;一个典型的 Kafka 系统架构会包括 Producer、Broker、Customer 等角色，以及一个 ZooKeeper 集群，其架构如下图所示：

![[../picture/Pasted image 20240622105544.png#pic_center|760]]

-  **Producer**：生产者，负责将客户端生成的消息发送到 Kafka 中，支持消息的异步发送和批量发送。
- **Broker - 服务节点**：服务代理节点。
  &emsp;&emsp;&emsp; Kafka 集群中，一台服务器就是一个 Broker，可以水平无限扩展，同一个Topic消息分布在多个 Broker 中。
- <font color=red>**Topic - 消息存储**：Topic 是一个**存储消息**的逻辑概念，可以理解为是一个消息的集合，Kafka 的数据就保存在 Topic 中。</font>
  &emsp;&emsp;&emsp;**Topic 中的事件在使用后不会删除，每个 Topic 通过配置来定义 Kafka 消息保留多长时间，消息过期之后旧事件将被丢弃**。在 Kafka 中消息是以 Topic 为单位进行归类的，Topic 在逻辑上可以被认为是一个队列，Producer 生产的每一条消息都必须指定一个 Topic，表明要将消息发送到哪个 Topic 上，然后 Consumer 会根据订阅的 Topic 到对应的 Broker 上去拉取消息。在存储方面，不同的 Topic 的消息是分开存储的。在每个 Broker 上都可以创建多个 Topic。
  &emsp;&emsp;&emsp; 为了使数据消息具有容错性和高可用性，每个 Topic 都可以复制，甚至可以跨地理区域或数据中心复制，因此会有多个 Broker 服务节点同时都拥有数据的副本，以防出现问题。
- <font color=red>**Partition**：Topic 的分区，是**物理存储**概念。</font>
  &emsp;&emsp;&emsp; 每个 Topic 可以有多个分区，分区的作用是做负载，提高了 Kafka 的并发，也解决了 Topic 中数据的负载均衡。同一个 Topic 在不同的分区的数据是不重复的，每个 Partition 在物理上对应一个文件夹，该文件夹里面存储了这个**分区的所有消息和索引文件**。生产者将消息发送到 Topic 时，消息会根据分区策略**追加到分区文件的末尾**，属于顺序写磁盘，因此效率非常高。每个消息在被添加到分区的时候，会被分配一个 Offset ( 偏移量 )，它是消息在当前分区中的唯一编号。<font color=orange>Kafka 通过 Offset 可以保证消息在分区中的顺序性，但是跨分区是无序的，即 **Kafka 只保证在同一个分区内的消息是有序的。**</font>

![[../picture/Pasted image 20240622105230.png#pic_center|660]]

-  **Offset**：分区偏移量。
  &emsp;&emsp;&emsp; Kafka 为每条在分区的消息分配一个偏移量 offset，它是消息在当前分区中的唯一编号，用来标识消费者当前消费的消息在分区 Partition 的位置。Kafka 的存储文件都是按照 offset.kafka 来命名。如果一个偏移量是5的消费者，表示已经消费了从0-4偏移量的消息，下一个要消费的消息的偏移量是5。
  ![[../picture/Pasted image 20240622153624.png#pic_center|440]]

-  **Leader 和 Follower**：提高容灾
   &emsp;&emsp;&emsp;一个 Partition 分区会有多个副本，副本之间是一主 Leader 多从 Follower 的关系，Leader 负责对外提供服务，而 Follower 只是被动地同步 Leader，不能与外界进行交互。<font color=green>同一分区 Parition 的不同副本 Follower 中保存的信息是相同的，但**副本 Follower 处于不同 Broker 中**，当 Leader 出现故障，会从 Follower 副本中重新选举新的 Leader 副本提供服务。</font>
-  **Consumer**：消费者，通过 Kafka 来接收消息，用于业务逻辑处理。
-  **Consumer Group**：消费者组
   &emsp;&emsp;&emsp;消费者组是 Kafka 提供的可扩展且具有容错性的消费者机制。<font color=red>多个消费者共同组成一个组来消费**同一个  Topic** 中的消息，同一个 Patition 分区的数据只能被消费者组中的某一个消费者消费。同一个消费者组的消费者可以消费同一个 Topic 的不同分区的数据</font>。
   
><font color=SlateBlue>  <u>**Q1. Kafka 为什么要分区 ？**</u></font>
&emsp;&emsp; ● **合理的使用存储资源**：把海量的数据按照**分区**切割成一小块的数据存储在多台 *Broker* 上，能够保证每台服务器的存储资源能够被充分利用，而且小块数据在寻址时间上更有优势，可以避免因为消息内容检索时间过长，导致 *Kafka* 性能降低。
&emsp;&emsp; ● **负载均衡**：数据生产或消费期间，生产者向分区的单位发送数据**，**消费者从分区的单位进行消费，各个分区之间的生产和消费数据互不影响。通过合理控制分区任务，提高任务的并行度，可以达到负载均衡的效果。
&emsp;&emsp; ●**实现系统的高伸缩性**：不同的 *Partition* 能够放置到不同节点机器上，每个 *Partition* 可以通过调整以适应它所在的机器，方便在集群中扩展。

#### 2.1.2 Kafka Producer - 消息生产
&emsp;&emsp; Producer 负责消息的生产，它的主要功能是将客户端的请求打包封装发送到 Kafka 集群的某个 Topic 的某个 Partition 分区上。Producer 的消息处理流程如下所示：

![[../picture/Pasted image 20240803213046.png#pic_center|750]]

&emsp;&emsp; Producer 其组件主要包括**元数据 Metadata、网络通信组件、相关配置、发送线程、消息拦截器**等组成。要想发送消息给 Broker，首先需要知道消息要发送到哪里去，所以就需要从 Broker 集群中获取发送消息需要的元数据。元数据中包括了 Topic 中的 Partitions 相关信息。

![[../picture/Pasted image 20240623152516.png#pic_center|700]]

-  **Step 1 - Producer 将数据封装为 ProducerRecord**：消息是 Kafka 中最基本的数据单元。在我们通过 `send (...)` 方法发送消息时，会将请求的数据创建一个 ProducerRecord 对象。ProducerRecord 对象中有两个必选参数：目标主题 (Topic) 和消息内容 (Value)，两个可选参数：分区 (Partition) 和键 (Key)。
  ![[../picture/Pasted image 20240719231911.png#pic_center|200]]
  
- **Step 2 - 消息的拦截、序列化、分区**：封装后消息通过拦截器，序列化之后，需要通过分区器来确定发送出去的消息需要存储在哪个分区中。<font color=red>分区使得每个节点能够实现独立的数据写入和读取，通过分区部署在多个 Broker 来实现负载均衡的效果</font>。在没有指定分区策略的情况下，Producer 会通过默认的分区策略指定当前消息应该存储在哪个分区下。分区器的 Partition 分配策略如下：
&emsp;  &emsp;  &emsp; <font color=green>● 若指定 Partition ID，则消息被发送至指定的 Partition。</font>
&emsp;  &emsp;  &emsp; <font color=green>● 若未指定 Partition ID，指定了 Key，消息会按照 Hash ( Key ) 作为 Partition ID 发送到 Partition。因此相同 Key 的消息会发到同一个 Partition ID 中。</font>
&emsp;  &emsp;  &emsp;  <font color=green>● 若既未指定 Partition ID 也没指定 Key，消息会按照 Round-Robin ( 顺序轮询 ) 算法发送到每个 Partition，Round-Robin 算法在第一次调用时随机生成一个整数 ( 后面每次调用在这个整数上自增 )，将这个值与 Topic 可用的 Partition 总数取余得到 Partition ID 。</font>
&emsp;  &emsp;  &emsp; <font color=green>● 若同时指定了 Partition ID 和 Key , 消息只会发送到指定的 Partition，此时 Key 不起作用。</font>
- **Step 3 - 消息的缓存**：确定好消息的分区后，消息会被放入待发送的缓存中 RecordAccumulator。缓存的默认大小为 32MB，通过 `buffer.memory` 参数可以指定 Producer 待发送消息缓冲区的内存大小。当 Producer 生产消息的速度超过了 Sender 线程发送消息的速度，并且缓冲区的消息数量超过 `buffer.memory` 指定的大小时，Producer 会抛出异常。除此之外，由于缓存的存在，会使消息的发送存在一定延时，通过 `linger.size` 参数可以设置被放进缓冲区 RecordAccumulator 中的消息是否立马被发送。

- **Step 4 - 消息的发送**：Kafka 的消息发送是将消息发送到 Broker 中，消息的发送分为3种方式：异步发送 ( Asynchronous-send )，同步发送 ( Synchronous-send )，发送即忘 ( Fire-and-forget )
&emsp;&emsp;&emsp;● **发送即忘 ( Fire-and-forget )**：Productor 在发送消息到 Kafka 集群时，消息先存储在缓冲区中，达到设定条件后批量发送，消息发送后并不等待服务器的响应，而是继续发送下一个消息。这样可以提高发送消息的吞吐量，但也是消息最不可靠的一种方式。
&emsp;&emsp;&emsp;● **同步发送 ( Synchronous send )**：Sender 线程调用 `send()` 方法会返回 Future 对象，通过调用 Future 对象的 `get()` 方法，等待直到结果返回，根据返回的结果判断是否发送成功。<font color=green>如果业务要求消息必须是按顺序发送的，那么可以使用同步的方式，并且消息只能在一个 Partation 上 ( 消息需指定某一 Partition 或 消息的 Key 相同 )。</font>
&emsp;&emsp;&emsp;● **异步发送 ( Asynchronous send )**：Sender 线程在调用 `send()` 方法的时候指定一个 Callback 函数，当接收到 Broker 的 Ack 返回通知时，该 Callback 函数会被触发执行。<font color=green>如果业务需要知道消息发送是否成功，并且对消息的顺序不关心，那么可以用异步+回调的方式来发送消息。</font>

![[../picture/Pasted image 20240623155003.png#pic_center|700]]

##### 1.Producer 的参数配置
&emsp;&emsp;●`acks`：Producer需要 Leader 确认的 Producer 请求的应答数。
&emsp;&emsp;&emsp;&emsp; ① acks = 0: 表示 Producer 请求立即返回，不需要等待 Leader 的任何确认。这种方案有最高的吞吐率，但是不保证消息是否真的发送成功。
&emsp;&emsp;&emsp;&emsp; ② acks = -1: 表示分区 Leader 必须等待消息被成功写入到所有的 ISR 副本中才认为 Producer 请求成功。这种方案提供最高的消息持久性保证，但是理论上吞吐率也是最差的。
&emsp;&emsp;&emsp;&emsp; ③ acks = 1: 表示 Leader 副本必须应答此 Producer 请求并写入消息到本地日志，之后 Producer 请求被认为成功。如果此时 Leader 副本应答请求之后挂掉了，消息会丢失。这个方案，提供了不错的持久性保证和吞吐。
&emsp;&emsp;&emsp;● `buffer.memory`：该参数用于指定Producer端用于缓存消息的缓冲区大小，单位为字节，默认值为32MB。
&emsp;&emsp;&emsp;● `compression.type`：压缩器，目前支持none (不压缩)，gzip，snappy和lz4。
&emsp;&emsp;&emsp;●`retries`：Produce r发送消息失败重试的次数。重试时 Producer 会重新发送之前由于瞬时原因出现失败的消息。瞬时失败的原因可能包括：元数据信息失效、副本数量不足、超时、位移越界或未知分区等。倘若设置了retries > 0，那么这些情况下Producer会尝试重新发送。
&emsp;&emsp;&emsp;●`batch.size`：默认值为16KB，Producer 按照 batch 进行发送，当batch 满了后，Producer 会把消息发送出去。
&emsp;&emsp;&emsp;●`linger.ms`：Producer 是按照 batch 进行发送的，但是还要看 linger.ms 的值，默认是0，表示不做停留。为了减少了网络IO，提升整体的性能。建议设置5-100ms。

#### 2.1.3 Kafka Broker - 消息数据的存放
&emsp;&emsp;  当 Producer 生产的消息推送到 Broker 后，**消息会存储到对应的 Partition 分区当中**。为了保证消息数据的<font color=red>**高可用和高持久**</font>，Kafka 的每个分区是多副本冗余的，如果一个副本数据丢失了，那么还可以从其他副本中获取分区数据。

![[../picture/Pasted image 20240803162205.png#pic_center|800]]

##### 1.Kafka 消息数据的存储 - 高持久
###### (1). 数据存储机制
&emsp;&emsp;Kafka是在 JVM 之上构建，Kafka 为了数据的“永久”存储，其消息并没有存储在 JVM 内存中，而是直接存储在硬盘上的。传统基于Jvm内存 (**用户内存**) 的数据存储方案存在以下问题：
&emsp;&emsp; &emsp; ● 对象的内存开销非常高，通常会让存储数据的大小加倍，因为对象存储的不只是数据还有结构以及其他信息。
&emsp;&emsp; &emsp; ● 随着堆内数据的增加，GC的速度越来越慢，不管是分代还是分区模型，数据量大起来后整理和清洁都比较繁琐和缓慢。

- 为了提升磁盘的效率，使其运行得和网络一样快，Kafka 对文件的存储进行了独特的设计：
	- <font color=orange>** 1.直接使用文件系统的页面缓存 pagecache 进行数据存储 - 高可用**</font>
		 &emsp;&emsp;① 通过 OS 的 pagecache 来有效利用主内存空间，避免内存开销过高，并且没有 GC 的压力。
		 &emsp;&emsp;② 通过存储紧凑的字节结构而不是单个对象，进一步提高可用缓存量
		 &emsp;&emsp;③ 当服务重新启动，文件系统和页面缓存 (内核内存) 中的数据仍然保持热状态，而内部缓存 (用户内存) 可能需要花费较长时间重新构建
		 &emsp;&emsp;④ 由操作系统维护缓存和文件系统之间的数据一致性，OS会自动将内核内存中的数据写入磁盘，从而简化了代码，提高效率和正确性。
		 &emsp;&emsp;⑤ 线性读取磁盘文件，并通过预读操作将有效地利用磁盘主内存 (内核内存) 缓存。
		> <font color=red>**注意**：</font> 在消息的刷盘过程中，为了提高性能，减少刷盘次数，Kafka 采用了批量刷盘的做法。即，按照一定的消息量，和时间间隔进行刷盘。数据存储到 Linux 操作系统中，会先存储到页缓存 Pagecache 中，如果系统挂掉，数据会丢失。理论上，要完全让 Kafka 保证单个 Broker 不丢失消息是做不到的，只能通过调整刷盘机制的参数缓解该情况。如，减少刷盘间隔，减少刷盘数据量大小。间隔时间越短，性能越差，可靠性越好。
	- <font color=orange>**2. 网络IO的字节零拷贝复制 -  高可用**</font>：
	  &emsp;&emsp;&emsp;生产者的消息生产和消费者的消息消费都是通过Web活动传递，会出现CPU不断地去网卡拉取和推送数据，都会涉及到字节复制以及小型I/O操作问题。针对这两个问题，Kafka进行了以下优化：
		- **大块数据读写**：将消息分组在一起，统一在一次网络传输中传递，生产者一次发送多个消息，消费者一次拉取多个消息。更大的网络数据包，更大的顺序磁盘操作、连续的内存块可以减少网络I/O的操作。
		- **零拷贝优化字节复制**：Kafka 通过零拷贝技术直接将磁盘读出的数据复制到 NIC 中，避免了数据由用户空间和内核空间的拷贝复制过程。
		  ![[../picture/Pasted image 20240706152845.png#pic_center|700]]
	-  <font color=orange>**3. 以恒定的时间复杂度进行磁盘访问 -  高持久**</font>：
	    &emsp;&emsp;一般的消息系统的持久化数据结构，通常是每个消费者使用队列和 BTree (用于磁盘访问的通用数据结构) 或一些通用的任意访问的数据结构来维护消息的元数据信息。但这些结构查询数据的效率一般最好情况下是O(logn)，因为B树查找数据要先找索引以及数据，磁盘就要多次寻道，随着数据量的增加，查询效率还会降低。
	     &emsp;&emsp; KafKa将数据写到磁盘，利用磁盘的**顺序读写**，因此其读取数据的时间复杂度为O(1)，即使是存储海量信息 (TB级) 性能和数据的大小关系也不大，只要磁盘空间足够大数据就可以一直追加存储。但同时如数组根据索引或者顺序遍历等时间复杂度为O(1)的操作一样，会**占据大量的连续空间**和**删除元素消耗性能**。

###### (2). 数据存储文件及格式
&emsp;&emsp;在 Kafka 集群中，每个broker 中有多个 topic。在每个 topic 中又有多个 partition，每个 partition 为一个分区，partition 的命名规则为 topic的名称+有序序号 ( 从0开始依次增加 )。<font color=red>**Partition 是以文件的形式存储在文件系统中**</font>，在每个 partition 由一个或多个Segment组成。Segment 作为Kafka中数据组织的基本单位，设计成固定 500M 大小，<font color=orange>**当  Segment 存储到 500M 时，Kafka 会自动创建一个新的 Segment 来继续存储数据**</font>，旧的 Segment 文件在满足一定条件 (如被消费且达到一定的保留期) 后会被删除，释放磁盘空间。
&emsp;&emsp;&emsp;每个 Segment 中对应三个 segment file 文件 (<font color=green>**二进制文件**</font>)，每个 segment file 有自己的命名规则，每个名字有20个字符，不够用0填充。每个名字从0开始命名，下一个segment file文件的名字就是上一个 segment file 中最后一条消息的索引值：
&emsp;&emsp; &emsp; ● **以 .log 结尾的<font color=red>数据文件</font>**：存储消息数据的地方，每个 Segment 有一个对应的.log文件。
&emsp;&emsp; &emsp; ● **以 .index 结尾的偏移量<font color=orange>索引文件</font>**：用于快速定位到 .log 文件中的具体消息。
&emsp;&emsp; &emsp; ● **以 .timeindex 结尾的时间戳<font color=orange>索引文件</font>**：如果启用了时间戳索引，则会有这个文件。它用于按时间戳高效检索消息。

![[../picture/Pasted image 20240706215919.png#pic_center|680]]

**▨  .index 文件 - 偏移量索引**

&emsp;&emsp; .index 文件是偏移量索引，存储着消息偏移量，其格式为 offset: 194 position: 5124。offset 表示当前消息的偏移量，position表示这个偏移量的消息所在的一批 (batch) 消息在 .log 文件中的起始二进制位置，只要打开 .log 文件并移动文件指针到这个 position 就可以读取对应的message了。**偏移量索引是稀疏结构，每隔一段记录一条消息的索引，但并不保证每个消息在索引文件中都有对应的索引项**。可以通过以下命令查看 .index 索引文件：`kafka-run-class kafka.tools.DumpLogSegments --files 00000000000000000000.index`

![[../picture/Pasted image 20240706234426.png#pic_center]]

**▨  .log 文件 - 数据文件**

&emsp;&emsp;.log 文件实际存储的是消息数据，.log 文件是采用顺序写的方式记录的。.log 文件主要包含消息体、消息大小、offset、压缩类型等：
&emsp;&emsp; &emsp; ● **offset**：offset是一个占8byte的有序id号，它可以唯一确定每条消息在 parition 内的位置。
&emsp;&emsp; &emsp; ● **消息大小**：消息大小占用4byte，用于描述消息的大小。
&emsp;&emsp; &emsp; ● **消息体**：消息体存放的是实际的消息数据（被压缩过）

&emsp;&emsp;通过以下命令查看 .log 文件：`kafka-run-class kafka.tools.DumpLogSegments --files 00000000000000000000.log --print-data-log`

![[../picture/Pasted image 20240707175444.png#pic_center]]

**▨  .timeindex 文件 - 时间戳索引**
&emsp;&emsp;.timeindex 文件是时间戳索引，存储了消息的时间戳与消息的offset偏移量之间的映射关系，**用于按照时间顺序进行消息的快速查找**。**时间戳索引也是稀疏索引**，当 Kafka 写入的消息长度超过一定量 (由参数指定) 或新消息的时间戳和上一个索引条目的时间戳超过一定时长 (由参数指定)，时间戳索引文件就会增加一个时间戳索引项。时间戳 (timestamp) 指这条消息的创建时间，offset 指这个消息的偏移量。可以通过以下命令查看 .index 索引文件：`kafka-run-class kafka.tools.DumpLogSegments --files 00000000000000000000.timeindex`

![[../picture/Pasted image 20240719231323.png#pic_center]]

###### (3). 数据文件清理策略
&emsp;&emsp; Kafka 中默认的日志保存时间为 7 天，可以通过调整如下参数修改日志的保存时间：
&emsp; &emsp; &emsp;  `log.retention.hours`：最低优先级小时，默认 7 天。
&emsp; &emsp; &emsp; `log.retention.minutes`：分钟。
&emsp; &emsp; &emsp; `log.retention.ms`：最高优先级毫秒。
&emsp; &emsp; &emsp; `log.retention.check.interval.ms`：负责设置检查周期，默认 5 分钟。

&emsp;&emsp;当日志的保留时间超过了设置的时间，则 Kafka 会对日志进行清理。Kafka 中提供的日志清理策略有 delete 和 compact 两种。
&emsp; &emsp; &emsp; ①  **delete 日志删除**：将过期数据删除。
&emsp; &emsp; &emsp; ② **compact日志压缩**：对于相同key的不同value值，只保留最后一个版本。压缩后的offset可能是不连续的。

![[../picture/Pasted image 20240720181524.png#pic_center|700]]
##### 2.Kafka 消息副本 Replicas - 高可用
&emsp; &emsp;  Kafka 中的副本分为两种类型：**领导者 ( Leader )副本**和**追随者 ( Follower ) 副本**。Leader 副本负责处理客户端的读写请求，而 Follower 副本则负责从 Leader 副本同步数据。<font color=green>每个 Partition 分区的副本数量称为副本因子 ( Replication Factor )，副本因子越高，系统的可靠性和容错能力越强</font>。但是，副本因子过高会导致更多的存储和网络开销。为了在可靠性和性能之间取得平衡，建议将副本因子设置为3，即一个 Partition 分区有一个 Leader 副本和两个 Follower 副本。
&emsp;&emsp;&emsp; ● 当 Producer 发送消息到某个 Partition 时，Leader 副本首先接收并处理这些消息，随后 Follower 副本从 Leader 副本同步数据。
&emsp;&emsp;&emsp; ● 当 Consumer 请求数据时，Leader 副本负责提供数据。

![[../picture/Pasted image 20240720182705.png#pic_center|530]]

###### (1). Follower 副本同步机制
&emsp;&emsp;  多副本之间有一个 Leader 副本，多个 Follower 副本， Follower 副本则负责从 Leader 副本同步数据。Kafka 使用<font color=red>副本同步队列 ISR ( In-Sync Replicas )机制来确保副本之间数据的一致性</font>。<font color=green>ISR 是一个动态维护的副本集合，包含了与 Leader 副本同步的 Follower 副本，即当前所有可用的副本</font>。ISR 队列主要有以下两个作用：
&emsp;&emsp;&emsp; ①  当 Follower 副本成功同步数据时，它会被添加到 ISR 中。如果 Follower 副本无法及时同步数据 ( 因为宕机或网络问题 )，则会从 ISR 中删除，同时将踢出 ISR 的 Follower 副本存入 OSR 队列。
&emsp;&emsp;&emsp; ② 当 Leader 副本发生故障时，Kafka 需要从 ISR 中选举出一个 Follower 作为新的 Leader 副本。由于在 ISR 中的 Follower 副本都可以确保与 Leader 副本具有最新的数据，从而避免了数据丢失。

![[../picture/Pasted image 20240720183044.png#pic_center|330]]

&emsp;&emsp; Kafka 在启动的时候，会启动一个副本管理器 ReplicaManager，这个副本管理器会启动两个用来处理 ISR 队列变更的定时任务：
&emsp; &emsp; &emsp; <font color=green>① ISR 过期定时任务 isr-expiration</font>：每隔 `replica.lag.time.max.ms` ( 默认值为 30s ) 执行一次。如果一个 Follower 副本落后 Leader 太多，但仍有活动会话，当滞后的 Follower 副本在 `replica.lag.time.max.ms`  配置设置的最长时间内没有赶上Leader 副本时，则该 Follower 副本会被Leader副本从 ISR 中删除。
&emsp;  &emsp;  &emsp; <font color=green>② ISR 变更的传播定时任务 isr-change-propagation</font>：ISR 的收缩和扩展，最终是修改 ISR 和内存,但是并没有通知到每个 Broker。只修改 Zookeeper 中的 `/brokers/topics/{Topic名称}/partitions/{分区号}/state` 节点,并不会通知集群，ISR已经变更了。Broker 是没有去监听每一个 state 节点的，state 节点太多了，因此需要通过定时任务定时去传播 ISR 的变更，每隔 2500ms 执行一次。

**▨ Follower 副本同步过程**

&emsp;&emsp;  副本同步过程由以下几个步骤组成：
&emsp; &emsp; &emsp; ① **Producer 将消息发送到 Leader 副本**：Producer 将消息发送到特定分区的 Leader 副本。Leader 副本负责处理数据写入请求，并将消息存储在内部日志 Log 中。
&emsp; &emsp; &emsp; ② **Leader 副本将消息发送给 Follower 副本**：Leader 副本将接收的消息发给分区内所有 Follower 副本。
&emsp; &emsp; &emsp; ③ **Follower 副本确认收到数据**：Follower 副本将接收到的消息写入它们的本地日志并确认已收到数据。在写入完成和确认之后，Follower 副本与 Leader 副本保持的日志和消息一致。
&emsp; &emsp; &emsp; ④ **Leader 副本收到确认**：Leader 副本会等待 ISR ( In-Sync Replicas ) 中的所有 Follower 副本确认。一旦收到 ISR 中所有副本的确认，Leader 副本会同步更新日志度量信息。只有在收到 ISR 中所有副本的确认后，Kafka 才将写入操作视为已成功完成。

![[../picture/Pasted image 20240720185147.png#pic_center|800]]

&emsp;&emsp;  Kafka 中 Topic 的每个 Partition 有一个预写式日志文件，每个 Partition 都由一系列有序的、不可变的消息组成，这些消息被连续的追加到 Partition 中，Partition 中的每个消息都有一个连续的序列号叫做 Offset，用来确定当前消息在 Partition 中的唯一位置。在 Leader 副本向 Follower 副本进行数据同步时，会依赖 Partition 的日志文件中的3个重要属性：
&emsp;  &emsp; &emsp;● **LEO ( Log And Offset )**：日志末端的位移，每个 Partition 的 log 的最后一条 message 的 Offset，用来标识当前日志文件中下一条待写入的消息的 Offset。
&emsp;  &emsp; &emsp;● **HW 高水位值 ( High Watermark )**：定义了消息可见性，标识了消息偏移量 Offset，Consumer 只能拉取到这个水位 Offset 之前的消息。每个副本都有 HW，Leader 副本和 Follower 副本各自负责更新自己的 HW 的状态。<font color=green>对于 Leader 副本新写入的消息，Consumer 不能立即消费，Leader 副本会等待该消息被所有 ISR 中的副本都同步后，才更新HW，此时消息才能被 Consumer 消费</font>。HW = 一个 Partition 对应的 ISR 中最小的 LEO。
&emsp;  &emsp; &emsp;● **Leader Epoch**：Leader 的任期，在每一个 Leader 副本时代分配一个标识符，由 Leader 将其添加到每个消息中，当 Follower 副本需要截断日志时，替代 HW 高位水作为其截断操作的参照数据。

![[../picture/Pasted image 20240720191248.png#pic_center|700]]

&emsp;&emsp; 副本常见的异常情况：
&emsp;  &emsp; &emsp;● **Follow 挂掉的情况**：当 Follower 挂掉后会被移除 ISR 队列。当挂掉的 Follow 再次恢复时，会把当前 Follow 本地记录的HW之后的数据全部截掉，重新同步现在leader的数据，并当同步到大于当前 HW (当前真实的HW) 时才会被重新加入到ISR中。

![[../picture/Pasted image 20240803180608.png#pic_center|560]]

&emsp;  &emsp;● **Leader 挂掉的情况**：当Leader挂了，会从 Follower 中选取 LEO 大的优先成为 leader (<font color=red>**此时新 Leader 和 旧 Leader中的没有同步的数据会被丢失**</font>)。除了新选举的 Leader，其他所有 Follower 副本都会把 HW 之后的数据删除，并从新Leader同步数据。当原 Leader 恢复后，也要截取 HW 之后的数据，并以 Follow 的身份从新Leader同步数据，当同步到大于当前 HW 时会被重新加入到ISR中。

![[../picture/Pasted image 20240803180530.png#pic_center|700]]

##### 3.Broker 的参数配置
&emsp;&emsp;●`replica.lag.time.max.ms`：ISR中，如果Follower长时间未向Leader发送通信请求或同步数据，则该Follower将被踢出ISR。该时间阈值，默认30s。
&emsp;&emsp;&emsp;●`auto.leader.rebalance.enable`：默认是true。自动 Leader Partition 平衡。
&emsp;&emsp;&emsp;●`leader.imbalance.per.broker.percentage`：默认是10%。每个 Broker 允许的不平衡的 Leader 的比率。如果每个 Broker 超过了这个值，控制器会触发 Leader 的平衡。
&emsp;&emsp;&emsp;●`leader.imbalance.check.interval.seconds`：默认值300秒。检查 Leader 负载是否平衡的间隔时间。
&emsp;&emsp;&emsp;●`log.segment.bytes`：Kafka中log日志是分成一块块存储的，此配置是指log日志划分成块的大小，默认值1G。
&emsp;&emsp;&emsp;●`log.index.interval.bytes`：默认4KB，Kafka每写入4KB的日志 (.log)，就往index文件里面记录一个索引。
&emsp;&emsp;&emsp;●`log.retention.hours`：Kafka中数据保存的时间，默认7天。
&emsp;&emsp;&emsp;●`log.retention.minutes`：Kafka中数据保存的时间，分钟级别，默认关闭。
&emsp;&emsp;&emsp;●`log.retention.ms`：Kafka中数据保存的时间，毫秒级别，默认关闭。
&emsp;&emsp;&emsp;●`log.retention.check.interval.ms`：检查数据是否保存超时的间隔，默认是5分钟。
&emsp;&emsp;&emsp;●`log.retention.bytes`：默认等于-1，表示无穷大。超过设置的所有日志总大小，删除最早的 segment。
&emsp;&emsp;&emsp;●`log.cleanup.policy`：默认是delete，表示所有数据启用删除策略；如果设置为compact，表示所有数据启用压缩策略。
&emsp;&emsp;&emsp;●`num.io.threads`：默认是8。负责写磁盘的线程数。整个参数值要占总核数的50%。
&emsp;&emsp;&emsp;●`num.replica.fetchers`：副本拉取线程数，这个参数占总核数的50%的1/3。
&emsp;&emsp;&emsp;●`num.network.threads`：默认是3。数据传输线程数，这个参数占总核数的50%的2/3 。
&emsp;&emsp;&emsp;●`log.flush.interval.messages`：强制页缓存刷写到磁盘的条数，默认为long的最大值。一般不修改，由系统自己管理。
&emsp;&emsp;&emsp;●`log.flush.interval.ms`：每隔多久，刷数据到磁盘，默认是null。一般不建议修改，交给系统自己管理。

#### 2.1.3 Kafka Consumer - 消息的消费
 &emsp;&emsp; 在 Kafka 中, 我们把消费消息的一方称为 Consumer 消费者, 它是 Kafka 的核心组件之一。它的主要功能是将 Producer 生产的消息进行消费处理，完成消费任务。
 
![[../picture/Pasted image 20240803224130.png#pic_center|660]]

##### 1.Consumer的消费方式
&emsp;&emsp; 消息队列的消费方式主要有两种：**Push** 和 **Pull**。Kafka Consumer 采用的是主动拉取 Broker 数据进行消费，即 Pull 模式。
&emsp;&emsp;&emsp;●  Push - 推模式 ( Broker 向 Consumer 推送消息 )：Push 模式最大缺点是 Broker 不清楚 Consumer 的消费速度，且推送速率是 Broker 进行控制的，这样很容易造成 Consumer 的消息堆积。如果 Consumer 消息比较耗时，可能会造成系统崩溃。
&emsp;&emsp;&emsp;●  Pull - 拉模式 ( Consumer 从 Broker 拉取消息 )：Pull 模式下 Consumer 根据自己的情况和状态来拉取数据, 也可以进行延迟处理。如果 Broker 中没有消息，Consumer 会被阻塞，等到 timeout 超时时间过后再去 Broker 拉取数据，直到 Broker 中存在数据，否则 Consumer 会在循环中不停阻塞，等待超时时间。

##### 2.Consumer Group机制
&emsp;&emsp;Consumer Group 是 Kafka 提供的可扩展且具有容错性的消费者机制。为了提高消费速率，提升 Kafka 的并发效率，Kafka将多个消费者 Consumer 共同组成一个消费者组 Consumer Group 来消费同一个 Topic 中的消息。Consumer Group 有以下特点：
&emsp; &emsp; &emsp; ① 每个 Consumer Group 有一个或者多个 Consumer。
&emsp; &emsp; &emsp; ② 每个 Consumer Group 拥有一个公共且唯一的 Group ID。
&emsp; &emsp; &emsp; ③ <font color=red>同一个 Partition 分区的数据只能由一个组内的一个消费者消费 (防止数据被重复消费)。</font>
&emsp; &emsp; &emsp; ④ 同一个 Consumer Group 的 Consumer 可以消费同一个 Topic 的不同分区的数据。
&emsp; &emsp; &emsp; ⑤ 如果消费者组的组员数量 > 分区数量，则就会有多余的消费者闲置。
![[../picture/Pasted image 20240804104116.png#pic_center|600]]

##### 3.Consumer的Partition分配策略
&emsp;&emsp; 一个 Consumer Group 中有多个 Consumer，一个 Topic 也有多个 Partition，所以必然会涉及到 Partition 的分配问题: 确定哪个 Partition 由哪个 Consumer 来消费的问题。Kafka 提供了4种分区分配策略：**RangeAssignor、RoundRobinAssignor、 StickyAssignor、CooperativeStick**。可以通过配置参数 `partition.assignment.strategy` ，修改分区的分配策略。默认策略是 Range + CooperativeStick。

###### (1).RangeAssignor - 以 Topic 维度进行分配：
&emsp;&emsp; RangeAssignor 是 Kafka 默认的分区分配算法，它是<font color=orange>**按照 Topic 的维度进行分配**</font>的，多个 Topic 之间的分配互不干扰。对于每个 Topic，首先对 Partition 按照分区ID进行排序，然后对订阅这个 Topic 的 Consumer Group 的 Consumer 再进行排序。最后，我们将 <font color=red>partition数/consumer数</font> ，从而确定要分配给每个 Consumer 的分区数。如果划分不均匀，则前几个消费者将多一个额外的分区。RangeAssignor 方式有个明显的问题：<font color=green>随着 Consumer 数量的增加，Partition 分配不均衡的问题会越来越严重。</font>

![[../picture/Pasted image 20240804232212.png#pic_center|700]]