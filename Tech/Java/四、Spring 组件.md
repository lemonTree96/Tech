&emsp;&emsp;大量的应用使用Java进行开发，其原因之一是Java拥有非常丰富和框架与基础组件，这些框架与组件使得Java的开发十分高效和便捷。
![[../picture/Pasted image 20231125221604.png#pic_center|550]]

---
### 4.1 Maven - 项目管理
---
### 4.2 Nacos - 配置中心
---
### 4.3 Feign - RPC
&emsp;&emsp; *Feign* 由 _Netflix_ 公司开发的一个轻量级 _RESTful_ 的 HTTP 服务客户端**框架**，其主要作用是**简化 _RPC_ 远程调用流程**。_Fegin_ 本身是不支持 _Spring MVC_ 的注解，随着 _Spring MVC_ 框架的流行，_Spring Cloud_ 对 _Feign_ 进行了封装为 _OpenFegin_，使其支持了 _Spring MVC_ 标准注解和 _HttpMessageConverters_。因此，我们常用的 _Fegin_ 是指的 _OpenFegin_。
![[../picture/Pasted image 20231125221750.png#pic_center|580]]
&emsp;&emsp;&emsp;在Java中常见的 _HTTP_ 客户端如下所示：     
&emsp;&emsp;&emsp;  ◎ ***HttpURLConnection***：_HttpURLConnection_ 是在JDK的 _java.net_ 包中提供的用于 _HTTP_ 协议访问的基本功能的类。_HttpURLConnection_ 是基于HTTP协议的，其底层通过 _socket_ 通信实现。    
&emsp;&emsp;&emsp;  ◎ ***HttpClient***：是 _Apache Jakarta Common_ 下的子项目，用来提供高效的、最新的、功能丰富的支持HTTP协议的客户端编程工具包。    
&emsp;&emsp;&emsp;  ◎<font color=red>***OkHttp***</font>：是由 Square 公司研发并开源的，处理 HTTP 网络请求的依赖库。单例模式下，_HttpClient_ 的响应速度要更快一些，非单例模式下，_OkHttp_ 的性能更好。
&emsp;&emsp;&emsp; *Feign* 是一个 HTTP 客户端框架，它并没有去做跟 _HttpClient_ 或 _OkHttp_ 一样重复的事情，而是开发了一个框架，用于**集成 _HttpURLConnection_，_Apache Http Client_，_OkHttp_ 实现具体的 HTTP 请求**，并提供了更加丰富实用的功能。在 _Feign_ 的实现下，我们只需要创建一个接口并使用注解的方式来配置它，即可完成对服务提供方的接口绑定。

![[../picture/Pasted image 20231125222416.png#pic_center|250]]

#### 4.3.1 Feign 功能架构
&emsp;&emsp; *Feign* 是一个 _HTTP_ 请求调用的轻量级框架，可以以 Java 接口**注解的方式**调用HTTP请求。_Fegin_ 主要包含了三大功能：**简化远程调用**、**负载均衡  *Ribbon*、服务容错与熔断机制 *Hystrix***。    
&emsp;&emsp;&emsp;  ◎ 简化远程调用： _Feign_ 通过处理注解，将请求模板化，当实际调用的时候，传入参数，根据参数再应用到请求上，进而转化成真正的请求，封装了 _HTTP_ 调用流程。_Fegin_ 的灵感来自于 _Retrofit_、_JAXRS-2.0_ 和 _WebSocket_。    
&emsp;&emsp;&emsp;  ◎ 负载均衡 *Ribbon*： 利用负载均衡策略选定目标机器。    
&emsp;&emsp;&emsp;  ◎ 服务容错与熔断机制 *Hystrix*： 根据熔断器的开启状态，决定是否发起此次调用。
![[../picture/Pasted image 20231125222845.png#pic_center|650]]
&emsp;&emsp;&emsp; *Feign* 远程调用，核心就是通过一系列的封装和处理，将以 Java 注解的方式定义的远程调用API接口，最终转换成 Http 的请求形式，然后将 Http 的请求的响应结果，解码成 Java Bean，返回给调用者。*Feign* 远程调用的基本流程如下图所示：
![[../picture/Pasted image 20231125231834.png#pic_center|400]]
#### 4.3.2 Feign 底层原理
##### 1. Feign 的装载与调用
&emsp;&emsp;在微服务启动时，Feign 会进行包扫描，对加 *@FeignClient* 注解的接口，按照注解的规则，创建远程接口的本地 *JDK Proxy* 代理实例。然后，将这些本地Proxy代理实例，注入到 *Spring IOC* 容器中。当远程接口的方法被调用，由 *Proxy* 代理实例去完成真正的远程访问，并且返回结果。Fegin 的装载与调用过程主要分为三个步骤：**(1). Fegin 相关 Bean 的注册；(2). Fegin 相关 *Bean* 的依赖注入**
###### (1). Feign相关Bean的注册
**▧  Feign 启动注解：@EnableFeignClients -> @Import(FeignClientsRegistrar.class)**

&emsp;&emsp; 在 Spring 项目启动阶段，Spring 会去扫描启动类是否存在 _@EnableFeignClients_ 注解，_@EnableFeignClients_ 注解是开启 _Fegin_ 功能的关键。在 _@EnableFeignClients_ 注解中，`@Import(FeignClientsRegistrar.class)` 会导入自动装载注册器 _FeignClientsRegistrar_ 类，对 _Feign_ 接口进行加载。 
```java
//1、Spring Application启动类
@EnableFeignClients
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
//2、@EnableFeignClients 注解
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(FeignClientsRegistrar.class)
public @interface EnableFeignClients {...}
```

**▧  FeignClientsRegistrar 注册到 IoC 容器**

&emsp;&emsp; FeignClientsRegistrar 类继承 Spring 中的 ImportBeanDefinitionRegistrar 接口，并在 `registerBeanDefinitions(...)` 实现方法中向 *Spring* 容器注册 *Bean*，以达到自动注入第三方功能的目的。
&emsp; &emsp;&emsp;&emsp;   ①  在 `registerBeanDefinitions(...)` 方法中首先调用 `registerDefaultConfiguration(...)`方法从 @EnableFeignClients 注解中提取 defaultConfiguration 属性对应的 Value，并封装为 **FeignClientSpecification** 作为默认的 Feign 配置注册到 Spring 容器中。    
&emsp; &emsp;&emsp;&emsp; ② 然后调用 `registerFeignClients(...)` 查找指定路径 basePackage 的所有带有 @FeignClients 注解的类、接口，将带有 @FeignClients 注解的类、接口包装成 <font color=red>**FeignClientFactoryBean** </font>注册到 Spring 容器。FeignClientFactoryBean 类实现了 FactoryBean\<T>，可以通过 `getObject()` 方法来获取并注入实例化对象。
![[../picture/Pasted image 20231209215925.png#pic_center|520]]

![[../picture/Pasted image 20231125224133.png#pic_center|570]]

```java
private ResourceLoader resourceLoader;  // 资源加载器，可以加载 classpath 下的所有文件
private Environment environment;   // 上下文，可通过该环境获取当前应用配置属性等

@Override
public void setEnvironment(Environment environment) {
    this.environment = environment;
}

@Override
public void setResourceLoader(ResourceLoader resourceLoader) {
    this.resourceLoader = resourceLoader;
}

@Override
public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry){
    //注册＠EnableFeignClients提供的自定义配置类相关Bean实例
    registerDefaultConfiguration(metadata,registry);  
    registerFeignClients(metadata, registry);  //扫描package，将@FeignClient修饰的接口类注册为Bean
}
```

###### (2). *Feign* 相关 *Bean* 的依赖注入
&emsp;&emsp; 当代码中通过 _@Resource_ 或 _@Autowired_ 注解注入 _FeignClient_ 客户端时，会**通过 _FeignClientFactoryBean_ 的 `getObject()` 方法来获取到动态代理对象**。动态代理对象的生成是通过 _Feign.Builder_ 的 `target()` 方法中调用 `build()` 方法生成 _ReflectiveFeign_ 的实例，然后通过 `newInstance()` 方法创建最终的 RPC 动态代理的实例。
![[../picture/Pasted image 20231125224607.png#pic_center|580]]
```java
<T> T getTarget() {
    // 从 IOC 容器获取 FeignContext
    FeignContext context = applicationContext.getBean(FeignContext.class);
    // 通过 context 创建 Feign 构造器
    Feign.Builder builder = feign(context);
  ...
}
```

---
### 4.4 MyBatis - 数据持久层管理
&emsp;&emsp;所谓持久化就是将程序数据在**持久状态**和**瞬时状态**间转换的机制。持久化的主要应用是将内存中的对象存储在数据库中，或者存储在磁盘文件中、XML数据文件中等等。JDBC 就是一种持久化机制，文件IO也是一种持久化机制。由**完成持久化工作所对应的代码块** Date Access Object( Dao ) 称之为持久层。
&emsp;&emsp;&emsp;MyBatis 是一款优秀的持久层框架，它内部封装了 JDBC，支持自定义 SQL、存储过程以及高级映射。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口为数据库中的记录，因此 MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集的过程，减少了代码的冗余，减少程序员的操作。

##### 1. Mybatis 框架

&emsp;&emsp;&emsp;Mybatis 通过 **xml 或注解**的方式将要执行的各种statement (statement、preparedStatemnt) 配置起来，并通过 Java 对象和Statement 中的 sql 进行映射生成最终执行的sql语句，最后由 Mybatis 框架执行sql并将结果映射成 Java 对象并返回。

![[../picture/Pasted image 20240601144342.png#pic_center|480]]

&emsp; &emsp; ① **接口层**：主要是和数据库交互，提供给外部使用的接口API，开发人员通过这些本地API来操纵数据库，接口层一接收到调用请求就会调用数据处理层来完成具体的数据处理。
&emsp; &emsp;&emsp; 以使用 Mapper 接口为例，将配置文件中的每一个 节点抽象为一个 Mapper 接口，这个接口中声明的方法和跟 Mapper.xml 中的节点项对应。MyBatis 会根据接口声明的方法信息，通过**动态代理机制生成一个Mapper 实例**，当调用接口方法时，根据这个方法的方法名和参数类型，确定Statement Id，底层还是通过 `SqlSession.select/update( “statementId”, parameter)`等来实现对数据库的操作。

&emsp; &emsp; ② **数据处理层**：MyBatis 的核心，负责具体的SQL查找、SQL解析、SQL执行和执行结果映射处理等，它主要的目的是根据调用的请求完成一次数据库操作。它要完成两个功能： **通过传入参数构建动态SQL语句、SQL语句的执行以及封装查询结果集**。参数映射指的是对于 java 数据类型和 jdbc 数据类型之间的转换，这里有包括两个过程：
&emsp; &emsp; &emsp; ● 查询阶段：将 Java 类型的数据，转换成 JDBC 类型的数据，经过 `preparedStatement.setXXX()` 来设值。
&emsp; &emsp; &emsp; ● 查询结果集转换阶段：对 resultset 查询结果集的 JdbcType 数据转换成 Java 数据类型。

&emsp; &emsp; ②**基础支撑层**：基础支撑层是整个MyBatis框架的地基，负责最基础的功能支撑，**包括连接管理、事务管理、配置加载和缓存处理**，这些都是共用的东西，为上层的数据处理层提供最基础的支撑。

##### 2. Mybatis 执行流程

![[../picture/Pasted image 20240604090428.png]]

- **Step 1**：加载 mybatis 全局配置文件 config.xml、SQL Mapper 映射文件 mapper.xml，解析配置文件。
	- 基于  config.xml 配置文件生成 Configuration
	- 基于  mapper.xml 配置文件生成多个 MappedStatement (包括了参数映射配置、动态SQL语句、结果映射配置)。mapper.xml 中的 select | update | delete | insert 标签分别对应一个 MappedStatement 对象。标签的id就是Statement ID 。
- **Step 2**：SqlSessionFactoryBuilder 通过 Configuration 对象生成 SqlSessionFactory，SqlSessionFactory 一旦被创建在应用的运行期间会一直存在。SqlSessionFactory 用来生成 SqlSession 对象并调用 `openSession()` 方法开启 SqlSession。SqlSession 是单线程对象，它是<font color=green>**非线程安全的**</font>，所以 SqlSession 对象的作用域需限制方法内，<font color=green>**在每一次操作完数据库后都要调用 close 对其进行关闭**</font>。
-  **Step 3**：SqlSession 对象完成和数据库的交互：
	- ① 用户程序调用mybatis接口层api (即Mapper接口中的方法)。
	- ② SqlSession 通过调用api的 Statement ID 找到对应的 MappedStatement 对象。
	- ③ 通过 Executor (负责动态SQL的生成和查询缓存的维护) 将 MappedStatement 对象进行解析，sql参数转化、动态 SQL 拼接，生成 JDBC Statement 对象。
	- ④ JDBC 执行sql，借助 MappedStatement 中的结果映射关系，将返回结果转化成 HashMap、JavaBean 等存储结构并返回。

##### 3. Mybatis 缓存机制
&emsp; &emsp; 为了提高查询效率，减少数据库的查询次数，Mybatis 提供了缓存机制来提高性能。MyBatis 中默认定义了两级缓存：**一级缓存**和**二级缓存**。
&emsp; &emsp; &emsp; ● 一级缓存：一级缓存，也叫本地缓存，是 SqlSession 级别的缓存，不同的SqlSession的缓存是不同的。当 Session flush 或 close 后, 该Session中的一级缓存将被清空。一级缓存是 MyBatis 维护的，并且默认开启，且不能被关闭，但可以调用clearCache()来清空缓存，或者改变缓存的作用域。
&emsp; &emsp; &emsp; ● 二级缓存：二级缓存是 Mapper 级别的缓存，多个 Session 去操作同一个 Mapper 映射的SQL语句时，多个 Session 可以共用二级缓存，二级缓存是跨 SqlSession 的。当二级缓存和一级缓存同时存在时，先查询二级缓存，再查询一级缓存。

![[../picture/Pasted image 20240604234320.png#pic_center|400]]

---
### 4.4 Redis 客户端
&emsp;&emsp;在实际的业务场景中，Redis 客户端主要有以下 3 种，具体如下所示：
&emsp;&emsp;&emsp; ①  **Jedis** ：作为一款 Redis 的 Java 实现客户端，其提供了比较全面的 Redis 命令的支持。其基于阻塞 I/O，且其方法调用为同步，程序流需要等到 Sockets 处理完 I/O 才能执行，不支持异步。Jedis 客户端实例不是线程安全的，所以需要通过连接池来使用 Jedis 。
&emsp;&emsp;&emsp; ② **Lettuce** ：一款高级的 Redis 客户端，用于线程安全同步，异步和响应使用，支持集群，Sentinel，管道和编码器等。其基于 Netty 框架的事件驱动的通信层，其方法调用为异步。Lettuce 的 API 是线程安全的，所以可以操作单个 Lettuce 连接来完成各种操作。Lettuce 需要 Java 8 及以上版本运行平台，其能够支持 Redis Version 4 以实现与 Redis 服务端进行同步和异步的通信。
&emsp;&emsp;&emsp; ② **Redisson** ：一款基于实现分布式和可扩展的 Java 数据结构，促使开发人员对 Redis 的关注分离，提供很多分布式相关操作服务，例如，分布式锁，分布式集合，可通过 Redis 支持延迟队列。其基于 Netty 框架的事件驱动的通信层，其方法调用是异步的。Redisson 的 API 是线程安全的，所以可以操作单个 Redisson 连接来完成各种操作。



&emsp; &emsp; Jedis是一个Java语言的Redis客户端，用于在Java程序中连接和操作Redis服务器。Jedis提供了简单而强大的API，可以实现对Redis的各种操作。Jedis 为了适配Spring框架，通过 Spring-Data-Redis 对 Jedis 进行了高度封装。通过以下方式可以对Jedis 和 Spring-Data-Redis 进行引入。
```xml
<!--Jedis-->
<dependency> 
	<groupId>redis.clients</groupId>
	<artifactId>jedis</artifactId> 
	<version>3.0.1</version> <scope>compile</scope>
</dependency>

<!--Spring-Data-Redis-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
![[../picture/Pasted image 20250216150030.png#pic_center|360]]

#### 4.4.1 Jedis
&emsp; &emsp; Jedis 是一款基于 BIO 实现的 Redis 的 Java 客户端。以微服务体系为例，其主要应用于 Spring Boot 1.x 中，在 Spring Boot 2.0 后，其默认已被 Lettuce 所取代。当然在 Spring Boot 2.x 中，Jedis 也可以继续使用。Jedis 作为 Redis 的客户端，对应 Redis 的四种工作模型：Redis Standalone (单节点模式)、Redis Cluster (集群模式) 、Redis Sentinel (哨兵模式) 以及 Redis Sharding (分片模式)。

![[../picture/Pasted image 20250222221039.png#pic_center|660]]

&emsp; &emsp;Jedis 实例通过 Socket 建立客户端与服务端的长连接，往 OutputStream 发送命令，从 InputStream 读取回复。其主要包括 3 种调用模式，具体如下：
&emsp; &emsp; &emsp; ● **Client 模式**：客户端发一个命令，阻塞等待服务端执行，然后读取返回结果。在该模式下每次处理都有结果，一旦发现返回结果中有 Error，就可以立即处理。
&emsp; &emsp; &emsp; ● **Pipeline 模式**：Pipeline 模式可以一次性发送多个命令，最后一次取回所有的返回结果。Pipeline 模式通过减少网络的往返时间和 IO 的读写次数，大幅度提高通信性能，但 Pipeline 不支持原子性，如果想保证原子性，可同时开启事务模式。
&emsp; &emsp; &emsp; ● **Transaction 模式**：Transaction 模式即开启 Redis 的事务管理，Pipeline 可以在事务中，也可以不在事务中。事务模式开启后，所有的命令 (除了 EXEC、DISCARD、MULTI 和 WATCH ) 到达服务端以后，不会立即执行，会进入一个等待队列，等收到上述四个命令时执行不同操作。

&emsp; &emsp;<font color=red>**Jedis 客户端是非线程安全的，Jedis 的请求流和响应流都是一个全局变量，如果同一个 Jedis 同时被多个线程使用的话，会出现数据错乱。**</font><font color=oragea>为了避免这个问题，同一个 Jedis 实例同一时刻只会被一个客户端线程使用即可。由于重复创建 Jedis 实例并建立连接是一个耗时操作，所以 Jedis 使用池化技术 - **JedisPool**。</font>

&emsp; &emsp;Jedis 包含以下几个核心类与服务端交互：Jedis、JedisCluster、ShardedJedis、JedisPool、JedisSentinelPool 以及 ShardedJedisPool。Jedis 由以下几个核心模块组成：
- ①  **Core: 核心模块，实现RESP协议**，包括：
	 - Jedis/BinaryJedis 入口类，封装Redis的各种命令。
	 -  Client/BinaryClient/Connection 与Redis进行具体的交互工作。
	 -  Protocol, RedisInputStream, RedisOutputStream 实现RESP协议。
 - ② **Sharding: 提供 Partitioning 支持**，为分片模式提供支持，包括：
	 - ShardedJedis/BinaryShardedJedis: 首先对传入的Key进行Hash计算(默认使用高性能、低碰撞率的MurmurHash算法），然后根据计算结果找到相应的 Jedis 实例，最后执行命令。
 - ③ **Pool: 提供连接池和 Sentinel 支持**，包括：
	 -  JedisPool: 基于Apache Commons Pool实现的连接池，通过JedisFactory获取Jedis实例。
	 -  JedisSentinelPool: 通过侦听"switch-master"事件，每当master切换时，调用 JedisFactory 重新初始化 master 连接信息。
	 -  ShardedJedisPool: 与 JedisPool 类似，通过 ShardedJedisFactory 获取 ShardedJedis 实例。
 - ④ **Pipeline: 提供 Pipelining 和 事务支持**，包括：
	 - Pipeline: 通过 Jedis#pipelined() 获取实例。以类型安全的方式获取执行结果，通过 BuilderFactory 将 Object 类型的 Response 转化为期望的结果类型。
		 -  非事务模式：构建 Response Queue，然后通过Client#getMany()批量获取结果。
		- 事务模式：通过 MultiResponseBuilder 缓存 Response，然后批量获取结果。
	-  Transaction: 通过 Jedis#multi() 获取实例。天然的事务属性，通过 Client#getMany() 批量获取结果，但无法获取单条命令的结果，且类型非安全。
	-  ShardedJedisPipeline: 通过 ShardedJedis#pipelined() 获取实例。不同于 Pipeline 和 Transaction，由于请求可能落到多个Client 上，只能通过 Client#getOne() 挨个获取结果，类型非安全。
- ⑤ **Cluster: 提供Cluster支持**，包括：
	-  JedisCluster/BinaryJedisCluster: 通过 JedisClusterConnectionHandler 获取 Jedis 实例，然后执行命令。
	- JedisClusterConnectionHandler & JedisClusterInfoCache: 通过 Collections#shuffle() 随机返回一个 Jedis 实例。使用ReentrantReadWriteLock 保证更新 Cluster 的 Jedis 实例列表时的线程安全性。
	- JedisClusterCommand: 通过retry机制获取有效的Jedis实例，然后再执行命令。

##### 1. Jedis Core
&emsp; &emsp;  Jedis是连接Redis的关键角色，它继承了 BinaryJedis。BinaryJedis 中保存了单个 Client 的实例，而 Client 最终继承了 Connection。Connection 中则包含了单个 Socket 的实例，以及与之对应的两个读写流。因此，每个 Jedis 实例都对应一个 Socket 连接。  

![[../picture/Pasted image 20250222223531.png#pic_center|760]]

![[../picture/Pasted image 20250222230504.png#pic_center|850]]

&emsp;&emsp;以 Jedis 执行 set命令为例，整个访问过程如下：
&emsp; &emsp; &emsp; ① Jedis的 set 操作是通过 Client 的 set 操作来实现的。
&emsp; &emsp; &emsp; ② Client 的 set操作是通过父类 Connection 的sendCommand(...) 来实现。
&emsp; &emsp; &emsp; ③ 在 sendCommand(...) 中进行 socket 连接，并按照 Redis 的协议发送命令。
```java
// ---------------- 1.Jedis ------------------
public class Jedis extends BinaryJedis implements JedisCommands, MultiKeyCommands,
    AdvancedJedisCommands, ScriptingCommands, BasicCommands, ClusterCommands, SentinelCommands, ModuleCommands {
  @Override
  public String set(final String key, final String value) {
    checkIsInMultiOrPipeline();
    // client执行set操作
    client.set(key, value);
    return client.getStatusCodeReply();
  }
}
 // ---------------- 2.Client ------------------
public class Client extends BinaryClient implements Commands {
  @Override
  public void set(final String key, final String value) {
    // 执行set命令
    set(SafeEncoder.encode(key), SafeEncoder.encode(value));
  }
}
// ---------------- 3.BinaryClient ------------------ 
public class BinaryClient extends Connection {
  public void set(final byte[] key, final byte[] value) {
    // 发送set指令
    sendCommand(SET, key, value);
  }
}
// ---------------- 4.Connectionas ------------------ 
public class Connection implements Closeable {
  public void sendCommand(final ProtocolCommand cmd, final byte[]... args) {
    try {
      // socket连接redis
      connect();
      // 按照redis的协议发送命令
      Protocol.sendCommand(outputStream, cmd, args);
    } catch (JedisConnectionException ex) {
    }
  }
}
```
##### 2. Jedis Pool
&emsp; &emsp; Jedis Core 采用直连方式连接 Redis，每次都会新建TCP连接，在使用后在断开连接。对于频繁访问 Redis 的场景显然不是高效的使用方式，因此一般使用连接池的方式对 Jedis 连接池进行管理。Jedis 提供了 JedisPool 作为 Jedis 的连接池，同时使用了Apache的通用对象池工具  [[六、Java 基础组件#6.1 Apache Commons Pool2 - 池化组件|common-pool]] 作为资源的管理工具，Jedis 对象预先放在 JedisPool 池子中，每次要连接 Redis，只需要在池子中借出对象，用完对象后再归还给池子 JedisPool。需要注意的是，虽然 Jedis Pool 使用的是 Common-pool2，但并不是完全使用，而是通过 Pool\<T> 进行了一次包装。

![[../picture/Pasted image 20250306224644.png#pic_center|500]]

![[../picture/Pasted image 20250225233614.png#pic_center|680]]

- ① 业务模块从资源池获取对象，会调用 ObjectPool#borrowObject()，最长会等待 maxWaitMillis 毫秒。如果资源池中没有空闲对象，则调用 PooledObjectFactory#makeObject() 创建对象，JedisFactory 是具体的实现类。
- ② 创建完对象放到资源池中，返回空闲对象 idleObject()  给业务模块使用。
- ③ 使用完对象会调用 ObjectPool#returnObject() ，其会校验一些条件是否满足。
	- 如果条件验证通过，对象归还给资源池。
	- 如果条件验证不通过，比如资源池已关闭、Jedis连接失效、已超出最大空闲资源数，则会调用 PooledObjectFactory#destoryObject() 从资源池中销毁对象。

##### 3. Jedis Sharding
&emsp; &emsp; ShardedJedis 主要是通过创建 ShardedJedisPool 对象来访问分片模式的多个Redis节点，主要用于在 Redis 没有集群功能的情况下实现数据的分布式存储，本质上是**客户端通过一致性哈希来实现数据分布式存储**。ShardedJedis 通过一致性哈希算法将不同的 key 分配到不同的 Redis服务器上，从而达到横向扩展。

![[../picture/Pasted image 20250302142717.png#pic_center|520]]

##### 4. Jedis Cluster
&emsp; &emsp; 在单服务器(非集群) 的模式下，Jedis 通过 JedisPool 来解决 Jedis 多线程下线程不安全的问题，此时一个 JedisPool 中的所有对象都对应到一个 Redis 服务器主节点。但是在 Cluster 集群模式下，就会有多个主节点，每个主节点还占据了一部分的槽位。Jedis 客户端需要确认到底是要用哪个 \<ip:port> 的 Jedis 客户端，因此 JedisCluster 的客户端至少需要维护一个主节点和 JedisPool 的一个 Map：`Map<String,JedisPool> nodes`，key 值就是 Cluster 每个主实例的 \<ip:port>。但客户端实际请求时不知道 Key 会到哪个主实例 \<ip:port> 上去执行的，仅能知道 Key 对应的 HashSlot，因此还需要一个Map `Map<Integer,JedisPool> slots` 来维护 Hash 槽(1-16384) 与 JedisPool 的关系。

![[../picture/Pasted image 20250306223251.png#pic_center|550]]
&emsp; &emsp; &emsp; Jedis Cluster 的类框架如下图所示。在初始化时，虽然只传递了一个主节点的信息，但由于 Redis cluster 是去中心化的( 每个Redis节点包含整个集群的所有信息 )，Jedis Cluster 通过 `initializeSlotCache()` 和 Redis 集群进行交互，通过一个command，拿到所有的主节点相关信息，以及每个主节点包含的槽位信息，并构造 `Map<String,JedisPool> nodes` 和 `Map<Integer,JedisPool> slots` 。

![[../picture/Pasted image 20250306225525.png]]

><font color=SlateBlue> <u>**Q1. 为什么 Cluster 模式下，客户端无法支持 Pipline 指令？**</u></font>
>&emsp; &emsp;&emsp;在 Redis Cluster 模式下，会有很多个 Redis 实例节点。而 Pipline 为了解决多次网络IO的问题，在 Pipline 的一系列指令中，必然包含了一系列的 key 值，这些 key 通过 `crc16(key)%16384` 计算的的槽位完全可能不在同一个节点上，所以在 Redis cluster 模式下不支持 `Pipline` 指令。
>
><font color=SlateBlue> <u>**Q2. 为什么 Cluster 模式，有时候又可以执行 mget，有时候不行？**</u></font>
>&emsp; &emsp;&emsp; mget() 与 Pipline 相似，多个 key 对应的槽位可能不在同一个实例节点上的。但当 key 中包含了 { } 这个标识符，那么在计算crc16的时候，就只会拿{}里面的内容进行计算，那么只要你保持{} 中的字符串是一样的，那么这些 key 就一定会落在同一个实例节点上，则 mget() 指令就可以在 Cluster 模式执行了。
>```java
>private static final String TOPIC_1  = "{%s}:topic:1"
>private static final String TOPIC_2  = "{%s}:topic:2"
>private static final String TOPIC_3  = "{%s}:topic:3"
>{%s}其实就是代表的一个唯一id (比如openid)，那么通过这种 hash_tag，就可以将一个用户的各类特征都存储在同一个 redis 实例中.
>```
>
#### 4.4.2 RedisTemplate
&emsp; &emsp; 为了简化对数据的操作性，Spring-Data-Redis 提供了统一的操作模板 RedisTemplate，RedisTemplate 封装了 Jedis、Lettuce 的 API 操作 ( Redisson 目前是由 Redisson-Spring-Data 封装的 )，提供了Redis交互的高级抽象，虽然RedisConnection 提供请求和响应的二进制值 (byte数组) 的低级别方法，但 RedisTemplate 模板负责序列化和连接管理，使客户端无需处理这些细节。

![[../picture/Pasted image 20250309213555.png#pic_center|500]]

- **Spring-Data-Redis RedisTemplate 针对Jedis提供了如下功能：**
	- ① **连接池自动管理，提供了一个高度封装的“RedisTemplate”类。**
	- ② **针对 Jedis 客户端中大量 API 进行了归类封装，将同一类型操作封装为 Operation 接口**。 
		- ValueOperations: 简单K-V操作
		- SetOperations: Set类型数据操作
		- ZSetOperations: ZSet类型数据操作
		- HashOperations: 针对map类型的数据操作
		- ListOperations: 针对list类型的数据操作.
	- ③ **提供了对 key 的“bound”(绑定)便捷化操作API，可以通过bound封装指定的key，然后进行一系列的操作而无须“显式”的再次指定Key**：BoundValueOperations、BoundSetOperations、BoundListOperations、BoundSetOperations、BoundHashOperations。
	- ④ **将事务操作封装，由Bean容器控制**。
	- ⑤ **针对数据的“序列化/反序列化”，提供了多种可选择策略 ( RedisSerializer )**。
		- `JdkSerializationRedisSerializer`：POJO对象的存取场景，使用JDK本身序列化机制，将pojo类通过ObjectInputStream/ObjectOutputStream进行序列化操作，最终redis-server中将存储字节序列。是目前最常用的序列化策略。
		- `StringRedisSerializer`：Key或者value为字符串的场景，根据指定的 charset 对数据的字节序列编码成 String，是`new String(bytes, charset)` 和 `string.getBytes(charset)` 的直接封装。是最轻量级和高效的策略。
		- `JacksonJsonRedisSerializer`：Jackson-Json工具提供了Javabean与Json之间的转换能力，可以将POJO实例序列化成Json格式存储在redis中，也可以将Json格式的数据转换成POJO实例。

##### 1. RedisTemplate 初始化与连接
&emsp; &emsp; 当我们引入 RedisTemplate 时，在对应的包内会有 `RedisAutoConfiguration` 类， 用于 RedisTemplate 的自动化配置。在这个类中，会引入 LettuceConnectionConfiguration 和 JedisConnectionConfiguration 两个配置类，分别对应 Lettuce 和 Jedis 两个客户端的配置类。在 Spring 进行 IoC 容器 Bean 加载的时候，在 RedisTemplate 的 bean 声明中注入了一个 `JedisConnectionFactory` 实例。通过 JedisConnectionFactory 在 Spring 启动的时候 Redis 服务器。

![[../picture/Pasted image 20250309222656.png#pic_center|580]]

##### 2. RedisTemplete 序列化
&emsp; &emsp; 从 Spring-Data-Redis 框架本身的角度看，存放到 Redis 的数据只是字节，虽然 Redis 本身支持各种类型，但大部分是指数据存储的方式，而不是它所代表的内容，由用户决定是否将字节转换为字符串或其他对象。用户自定义类型和原始数据之间的转换由 `org.springframework.data.redis.serializer` 包中的序列化器进行处理。

![[../picture/Pasted image 20250309224439.png#pic_center|580]]

- RedisTemplate 的序列化主要分为4类：**JDK 序列化方式、String 序列化方式、JSON 序列化方式、XML 序列化方式**。
	- **JDK 序列化方式 - JdkSerializationRedisSerializer**：在 RedisTemplate 未设置序列化的情况下，JDK 序列化是默认的序列化方式。JDK 序列化在存储内容时，除了属性的内容外还存了其它内容在里面，总长度长，且不容易直观阅读。因此通常情况下不会使用 JDK 序列化方式。
	- **String 序列化方式  - StringRedisSerializer**：用于字符串和二进制数组的**直接**转换。**大多数情况下，我们 KEY 和 VALUE 都会使用这种序列化方案**。而 VALUE 的序列化和反序列化，在业务逻辑中调用 JSON 方法去序列化。
	- **JSON 序列化方式**：使用 Jackson 实现 JSON 的序列化方式，JSON 序列化有四种序列化器：
		- ① **GenericJacksonJsonRedisSerializer**：通过使用传入对象的 `classPropertyTypeName` 属性对应的值，作为默认类型 (Default Typing) 进行序列化和反序列化为 Default Typing 对应的对象。
		- ② **GenericJackson2JsonRedisSerializer \<T>**：使用 Jackson 实现 JSON 的序列化方式，并且显示指定 `<T>` 类型，不再需要传入默认类型 (Default Typing) 。
		- ③ **GenericFastJsonRedisSerializer**：使用 FastJSON 实现 JSON 的序列化方式，和 GenericJacksonJsonRedisSerializer 一致。PS：GenericFastJsonRedisSerializer 不是 Spring-Data-Redis 内置实现，而是由于 FastJSON 自己实现。
		- ④ **FastJsonRedisSerializer \<T>**：使用 FastJSON 实现 JSON 的序列化方式，和 GenericJackson2JsonRedisSerializer \<T> 一致。PS：FastJsonRedisSerializer \<T> 不是 Spring-Data-Redis 内置实现，而是由于 FastJSON 自己实现。
	- **XML 序列化方式  - OxmSerializer**：使用 Spring OXM 实现将对象和 String 的转换，从而 String 和二进制数组的转换。


