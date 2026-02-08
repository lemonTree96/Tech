&emsp;  &emsp; Java 的基础组件是个人或公司在Java基础语法基础上开发公共组件，以方便其他开发者的使用。
### 6.1 Apache Commons Pool2 - 池化组件
&emsp;  &emsp;Apache Commons Pool2 是 Apache Commons下的一个开源项目，主要用于实现和管理对象池。通过复用来分摊线程对象的创建和销毁代价，从而优化资源利用和提高应用程序性能。Commons-Pool2 提供了一套用于实现对象池化的API，并内置了多种对象池实现。被广泛应用在各种数据库连接池、线程池以及请求分发池中。同时 Commons-Pool2 也提供了一些常用的实现类，如 `GenericObjectPool` 可以方便地进行配置和扩展。池化技术带来的优点如下：
&emsp;&emsp;&emsp;  ◎ **资源复用**：对象池通过复用对象实例，避免了频繁创建和销毁对象带来的开销。特别是对于创建和销毁成本较高(拥有复杂数据结构)的对象，如数据库连接等。
&emsp;&emsp;&emsp;  ◎ **性能提升**：对象池可以确保在需要时快速提供可用对象，减少了对象的创建和销毁次数，从而减少了对象的创建时间，等待时间。使整体性能得到提升。
&emsp;&emsp;&emsp;  ◎ **资源管理**：对象池封装了对象的创建、验证、销毁等复杂逻辑，减少了开发者对底层资源的管理。
&emsp;&emsp;&emsp;  ◎ **降低垃圾收集压力**：频繁的对象创建和销毁会增加垃圾收集器的工作负担，可能导致应用程序的停顿和延迟。对象池通过减少不必要的对象分配和释放，降低了垃圾收集的频率和强度，从而提高了应用程序的稳定性。

#### 6.1.1 commons-pool2的框架
&emsp;  &emsp; commons-pool2 工作的逻辑如下图所示：
![[../picture/Pasted image 20250301142049.png#pic_center|680]]

&emsp;&emsp;commons-pool2 的核心三元素主要有**工厂类、对象池、池化对象**三个核心角色。
&emsp; &emsp; &emsp; ①  **ObjectPool 接口 - 对象池**：负责管理对象生命周期，以及活跃和空闲对象的数据信息获取。`GenericObjectPool` 类是该接口的实现类。

&emsp; &emsp; ② **PooledObjectFactory 接口 - 对象工厂**：负责具体对象的创建、初始化，对象状态的销毁和验证。commons-pool2 框架提供了默认的抽象实现 `BasePooledObjectFactory`，业务方在使用的时候只需要继承该类，然后实现 `warp()` 和 `create()` 方法即可。

&emsp; &emsp; ③ **PooledObject 接口 - 池化对象**：基于包装类被维护在对象池中，并且维护一些附加信息用来跟踪，例如时间、状态。commons-pool2 提供了`DefaultPooledObject` 和 `PoolSoftedObject` 两种实现。其中 `PoolSoftedObject` 继承自 `DefaultPooledObject`，并使用 `SoftReference` 实现了对象的软引用，获取对象的时候使用也是通过 `SoftReference` 进行获取。

![[../picture/Pasted image 20250301170311.png#pic_center|500]]
##### 1. 池化工厂
&emsp; &emsp; 池化对象工厂是用于产生和控制池化对象的工厂类，是唯一一个需要完全由用户自行实现的组件。与对象池相对应，有负责普通 `ObjectPool` 的工厂 `PooledObjectFactory`，以及负责 `KeyedObjectPool` 的 `KeyedPooledObjectFactory`。`PooledObjectFactory` 接口方法如下：
```java
public interface PooledObjectFactory<T> {
  PooledObject<T> makeObject() throws Exception;          //创建池化对象
  void destroyObject(PooledObject<T> p) throws Exception; //销毁池化对象
  boolean validateObject(PooledObject<T> p);              //校验池化对象
  void activateObject(PooledObject<T> p) throws Exception;  //在借用时，激活池化对象
  void passivateObject(PooledObject<T> p) throws Exception; //在归还时，钝化池化对象
}
```


  
##### 2. 对象池
&emsp; &emsp; 在使用 commons-pool2 的时候，业务获取或释放对象的操作都是基于对象池进行的。对象工厂是 commons-pool2 框架中用于生成对象的核心环节，通过工厂模式，实现了对象池与对象的生成与实现过程细节的解耦。Commons Pool 2提供了3种 `ObjectPool` 的实现，分别是通用的 `GenericObjectPool`、基于软引用的 `SoftReferenceObjectPool`、基于动态代理的 `ProxiedObjectPool`。以 `GenericObjectPool` 为例，对象池的框架如下：

![[../picture/Pasted image 20250301154434.png#pic_center|500]]

```java 
// ============================================= GenericObjectPool 类 =================================================
public class GenericObjectPool<T> extends BaseGenericObjectPool<T>  
        implements ObjectPool<T>, GenericObjectPoolMXBean, UsageTracking<T> {
        
	//双端队列，用于存储对象
	private final LinkedBlockingDeque<PooledObject<T>> idleObjects;

	//构造函数
	public GenericObjectPool(final PooledObjectFactory<T> factory,final GenericObjectPoolConfig<T> config) { 
		idleObjects = new LinkedBlockingDeque<>(config.getFairness());
	} 
}
○ 在 GenericObjectPool 的构造中涉及到三个核心对象：工厂对象、配置对象、双端阻塞队列。
	● factory提供一个对象池创建工厂实例。
	● 在config中提供了一些简单的默认配置：例如 maxTotal、maxIdle、minIdle等，也可以扩展自定义配置；
	● 对象池中使用了双端队列 LinkedBlockingDeque 来存储对象。队列支持 FIFO 和 FILO 两种策略，并基于AQS来实现队列的操作的协同。

// ============================================== LinkedBlockingDeque 类 ==============================================
class LinkedBlockingDeque<E> extends AbstractQueue<E>  
        implements Deque<E>, Serializable {
        
    private transient Node<E> first; //第一个节点
	private transient Node<E> last;  //最后一个节点
	private transient int count;     //当前队列长度
	private final int capacity;      //队列最大容量
	private final InterruptibleReentrantLock lock;  //主锁
	private final Condition notEmpty;  //队列是否为空状态锁
	private final Condition notFull;   //队列是否满状态锁
}
○ 队列中所有的移入元素、移出、初始化构造元素都是基于主锁进行加锁操作。
○ 队列的offer和pull支持设置超时时间参数，主要是通过两个状态 Condition 来进行协调操作。
	● 如在进行offer操作的时候，如果操作不成功，则基于notFull状态对象进行等待;
	● 如进行pull操作的时候，如果操作不成功，则对notEmpty进行等待操作。
```

##### 3. 对象管理与对象状态
###### (1). 池化对象的状态流转
- 对象的管理主要针对池化对象的操作。在对象池中，池化对象共有以下几种状态：
	- **IDLE**：在空闲队列中,还未被使用。
	- **ALLOCATED**：对象使用中。
	- **RETURNING**：对象使用完毕，正在被归还到池中。
	- **ABANDONED**：遗弃态，针对活跃对象进行检测，长时间运行未被回收的活跃对象会判定为无效，并设置为该状态，最终被废弃回收。
	-  **INVALID**：无效状态(如驱逐测试或验证未通过)，对象将被销毁。
	- **EVICTION**：在空闲队列中，当前正在测试是否满足被驱逐的条件。
	-  **EVICTION_RETURN_TO_HEAD**：驱逐测试的中间态。由于当前对象正在测试是否可能被驱逐，在测试过程中会试图借用对象，并将其从空闲队列中删除。 待回收测试完成后，当前对象就会变为空间态，并被返回空闲队列的头部。

![[../picture/Pasted image 20250301214847.png#pic_center|680]]

###### (2). 池化对象的管理过程
- **池化对象的 browObject 过程**：browObject 过程会从对象池中借出可用对象。
	- Step1：首先从队列中获取对象，如果对象池中没有获取到可用对象，会调用工厂创建方法新的对象，并将新对象加入池化管理。
	- Step2：对象获取之后，调用 `allocate()`，将 `IDLE` 状态 变为`ALLOCATED` 状态。
	- Step3：调用工厂的 `activateObject()` 来初始化对象，如果发生错误，则调用`destroy()` 来销毁对象。
	- Step4：调用 TestFactory 的 `validateObject()` 进行基于 testOnBorrow 配置的对象可用性分析，如果不可用，则调用 `destroy()` 销毁对象。
-  **池化对象的 returnObject 过程**：returnObject过程将对象返回到对象池中。
	- Step1：调用 `markReturningState()` 将状态更改为 `RETURNING`。
	- Step2：基于 testOnReturn 配置，调用 PooledObjectFactory 的 `validateObject()` 进行可用性检查。如果检查失败，则调用 `destroy()` 销毁该对象，然后调用 `idle()` 以确保池中有 `IDLE` 状态对象可用，如果没有，则调用 `create()` 创建一个新对象。
	- Step3：调用 PooledObjectFactory 的 `passivateObject()` 进行反初始化操作。
	- Step4：调用 `deallocate()` 将状态更改为 `IDLE`。
	- Step5：检测是否已超过最大空闲对象数，如果超过，则销毁当前对象。
	- Step6：根据 LIFO (后进先出) 配置将对象放置在队列的开头或结尾。

###### (3). 对象池的自我保护机制
&emsp; &emsp; 在使用 commons-pool2 中获取对象的时候，会从双端队列中阻塞等待获取元素。但是如果是应用程序的异常，一直未调用 `returnObject()` 或者 `invalidObject()` 的时候，那就会导致对象池中正在使用对象一直上升，到达设置的上限后再去调用 `borrowObject()` 的时候就会出现一直等待或者是等待超时而无法获取对象的情况。commons-pool2 为了避免上述问题的出现，提供了两种自我保护机制：

&emsp; &emsp; ① **基于阈值的检测**：当从对象池中获取对象时，会校验当前对象池的活跃对象和空闲对象的数量占比，当空闲对象非常少，活跃对象非常多的时候，会触发空闲对象的回收。从所有活跃对象中根据对象最后一次借出时间与当前时间比对，然后基于 `removeAbandonedTimeout` (默认为300s) 配置的时间来判断是否是可回收的空闲对象。超过 `removeAbandonedTimeout` 设置的时间长度的，会被标记为 `ABANDONED` 状态，并调用 `invalidateObject` 方法来进行销毁。

&emsp; &emsp;  ② **异步调度线程检测**：当配置 `AbandonedConfig.setRemoveAbandonedOnMaintenance` 设置为 true 以后，就会异步维护一个任务来进行泄漏对象的清理，通过设置 `setTimeBetweenEvictionRunsMillis` 来设置维护任务执行的时间间隔。

##### 4. 对象池的相关配置项
&emsp; &emsp; 对象池提供了许多配置项，在使用的`GenericObjectPool`默认基础对象池中可以通过构造方法传参传入`GenericObjectPoolConfig`。具体包含如下配置:

![[../picture/Pasted image 20250309204223.png]]


