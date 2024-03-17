### 2.1 Java 程序基本规则
&emsp;&emsp;与C++相比，Java是完全的面向对象的语言，因此Java与C++在程序规则上有所区别：
&emsp; &emsp;&emsp; ① Java程序必须以`class`的形式存在，`class`是java的最小程序单位。<font color=red>在Java程序中，不允许可执行语句、方法独立存在。</font>
&emsp; &emsp;&emsp;  <font color=red>② 如果一个Java程序中定义了一个`public`类，则该程序文件的文件名必须和该`public`类名相同。因此，一个Java程序文件最多只能有一个`public`类。</font>
&emsp; &emsp;&emsp;  ③ Java中编译器通过 Main() 方法作为程序的入口，且 Main() 的修饰符必须为：`public static void main(String [] args){}`
### 2.2 Java 语法基础
#### 2.2.1 Java关键字
&emsp; &emsp; Java中一共有48种关键字，如下表所示。关键字可以分为**程序逻辑控制关键字，系统控制(线程同步)关键字，数据类型关键字，类/对象关键字，包/方法管理关键字**。
![[../picture/Pasted image 20240105215809.png#pic_center|750]]

##### 1. *volatile* 类型
&emsp; &emsp;当一个变量定义为 ***volatile*** 类型后，该变量将具备两种特性：
&emsp; &emsp; &emsp; ① <font color=green>**定义为 *volatile* 类型变量会对所有线程保持可见性，即当一个线程修改了这个变量的值，修改后的值对于其他线程来说是可以被立刻获取到**</font>。而普通变量的值在线程间传递均需要通过主内存完成。注意：虽然定义为 *** volatile*** 类型的变量在被执行时可以保证数据一致性，但 Java运算并不是原子操作，所以<font color=red>**定义为 *volatile* 类型的变量在并发下也是线程不安全的**</font>。因此除了以下两种运算场景，其余运算均需要加锁来保证原子性。
&emsp; &emsp; &emsp; ● 运算结果不依赖变量的当前值，或者能够确保只有单一线程来修改变量的值。
&emsp; &emsp; &emsp; ● 变量不需要与其他的状态变量共同参与不变条件约束。
```java
public class Main {
    public static volatile int race = 0;
    /* 由于volatile类型仅保证了获取变量时的数据一致性，并不会保证数据运算的原子性，从而导致线程运算不安全。对于"race++"操作，分为三个运算过程：
    ① 复制race变量的当前值到临时变量temp中；
    ② 将临时变量自增；
    ③ 将临时变量复制到race变量中； */
    public static void increase(){
        race++;  ，
    }
    public static void main(String[] args) throws InterruptedException {
        Thread threads[] = new Thread[20];
        for(int i=0;i<20;++i){
            threads[i] = new Thread(new Runnable() {
                @Override
                public void run() {
                    for(int i=0;i<10000;i++){
                        increase();
                    }
                }
            });
            threads[i].start();
        }
        System.out.println(race);  //由于线程不安全，导致最终结果是小于20*10000的
    }
}
```
&emsp; &emsp;② 定义为 ***volatile*** 类型变量会<font color=red>**禁止指令重排序优化**</font>。而普通变量仅会保证变量在方法中能够得到正确执行结果，而不保证变量赋值操作顺序与程序代码中逻辑顺序是一致的。( 指令重排序是指CPU采用了允许将多条指令不按程序顺序，分开发送给相应电路单元进行处理，但指令重排不是任意的，CPU需要根据指令依赖情况进行重排，以保证得到正确的执行结果 )
```java
Map<String,String> config;
char [] configText;

// 如果inited变量没有定义为volatile类型，则代码可能会被指令重排序优化，
// 导致线程A中的inited = true;语句被提前执行,但此时配置信息并没有被加载，导致线程B执行出错。
volatile boolean inited = false;  

//线程A中执行，读取配置信息，读取完成后将inited置为true，通知其他线程。
config = new HashMap();
configText = readConfig();
inited = true;

//线程B中执行，等待inited为true，并执行初始化操作
while(!inited){
	 sleep();
}
doSomethingWithConfig();
```
##### 2. *synchronized* 类型
&emsp; &emsp; Java中互斥同步方式就是 *synchronized* 关键字。*synchronized* 主要有三种用法：**修饰实例方法，修饰静态方法，修饰代码块**。
&emsp; &emsp;&emsp;  **① 修饰实例方法**: **被修饰的方法称为同步方法，其作用范围是整个方法，作用对象是调用这个方法的对象**，进入同步代码前要获得当前对象实例的锁。
```java
//共享对象数据
public class HasSelfPrivateNum {
    private int num = 0;
    // 用synchronized修饰实例方法
    public synchronized void addI(String name){
        try{
            if(name.equals("a")){
                num = 100;
                System.out.println("a set over");
                Thread.sleep(1000);
            }else{
                num = 200;
                System.out.println("b set over");
            }
            System.out.println("num = " +  num);
        }catch (Exception ex){
            ex.printStackTrace();
        }
    }
}
// 线程1 - Thread1
public class Thread1 extends Thread{
    private HasSelfPrivateNum hasSelfPrivateNum1;
    public Thread1(HasSelfPrivateNum hasSelfPrivateNum){
        super();
        this.hasSelfPrivateNum1 = hasSelfPrivateNum;
    }
    @Override
    public void run(){
        super.run();
        hasSelfPrivateNum1.addI("a");
    }
}
// 线程2 - Thread2
public class Thread2 extends Thread{
    public HasSelfPrivateNum hasSelfPrivateNum2;
    public Thread2(HasSelfPrivateNum hasSelfPrivateNum){
        super();
        this.hasSelfPrivateNum2 = hasSelfPrivateNum;
    }
    @Override
    public void run(){
        super.run();
        hasSelfPrivateNum2.addI("b");
    }
}
//主函数
public static void main(String[] args) throws InterruptedException {
    HasSelfPrivateNum hasSelfPrivateNum = new HasSelfPrivateNum();
    Thread1 thread1 = new Thread1(hasSelfPrivateNum);
    thread1.start();
    Thread2 thread2 = new Thread2(hasSelfPrivateNum);
    thread2.start();
}
-- Output -- 
1. 如果hasSelfPrivateNum.addI()方法没有加synchronized同步，则输出结果会出现非线程安全问题，结果如下：
 a set over
 b set over
 num = 200
2. 加synchronized同步后，避免了非线程安全问题，结果如下：
 a set over
 num = 100
 b set over
 num = 200
```
&emsp; &emsp; **② 修饰静态方法**:  **给当前类加锁，会作用于类的所有对象实例 ，其作用的范围是整个静态方法，进入同步代码前要获得当前 *class* 的锁**。<font color=red>如果一个线程A调用一个实例对象的 *non-static-synchronized* 方法，而线程B需要调用这个实例对象所属类的 *static-synchronized* 方法，不会发生互斥情况，因为访问 *static-synchronized* 方法占用的锁是当前类的锁，而访问 *non-static-synchronized* 方法占用的锁是当前实例对象锁。</font>`synchronized void staic method(){ 业务代码 }`
&emsp; &emsp;&emsp;  **③ 修饰代码块**: 需要指定加锁对象，对给定对象/类加锁。**当多个线程持有的对象监听器为同一个对象时，线程是同步的，同一时间只有一个线程可以访问同步块**，**但是如果是同一个类的不同实例，同步块的执行就是异步的**。 `synchronized(object)` / `synchronized(类.class)` `synchronized(this) { 业务代码 }`
```java
//共享对象数据
public class HasSelfPrivateNum {
   private String anyString = new String()
   // 方法A
   public void aFunction(String name){
       try{
           synchronized (anyString) 
              System.out.println("aFunction begin ");
              Thread.sleep(1000);
              System.out.println("aFunction end");
           }
       }catch (Exception ex){
           ex.printStackTrace();
       }
   }

 	// 方法B
   synchronized public void b() {
      System.out.println("bFunction begin");
      System.out.println("aFunction end");
   }
}
// 线程1 - Thread1 - 执行HasSelfPrivateNum.a()方法
public class Thread1 extends Thread{
   private HasSelfPrivateNum hasSelfPrivateNum1;
   public Thread1(HasSelfPrivateNum hasSelfPrivateNum){
       super();
       this.hasSelfPrivateNum1 = hasSelfPrivateNum;
   }
   @Override
   public void run(){
       hasSelfPrivateNum1.a();
   }
}
// 线程2 - Thread2 - 执行HasSelfPrivateNum.b()方法
public class Thread2 extends Thread{
   public HasSelfPrivateNum hasSelfPrivateNum2;
   public Thread2(HasSelfPrivateNum hasSelfPrivateNum){
       super();
       this.hasSelfPrivateNum2 = hasSelfPrivateNum;
   }
   @Override
   public void run(){
       hasSelfPrivateNum2.b();
   }
}
//主函数
public static void main(String[] args) throws InterruptedException {
    HasSelfPrivateNum hasSelfPrivateNum = new HasSelfPrivateNum();
    Thread1 thread1 = new Thread1(hasSelfPrivateNum);
    thread1.start();
    Thread2 thread2 = new Thread2(hasSelfPrivateNum);
    thread2.start();
}
-- Output --
同一个类的不同实例，在执行同一个共享数据类时，不同的同步块是异步的
 A begin
 B begin
 B end
 A end
```
#### 2.2.2 *Package* 与 *Import*  机制
&emsp; &emsp; Java引入了包机制，用于解决类名冲突与类文件管理等问题(类似于C++中的`namespace`)。Java允许将一组功能相关的类放在同一个`Package`下，构成类库单元。当Java程序文件中使用了`Package`语句时，则该程序文件中定义的所有类都在这个`Package`下。同时，在父包下可以创建子包，<font color=green>虽然父包和子包存在某种联系，但在用法上没有任何关系，如在父包类中需要使用子包中的类时，必须使用子包的全名，而不能省略父包部分</font>。 同时Java引入了`import`机制，可以向某个Java文件中指定包层次下的某个类或全部类。
```java
//t1Class.java
package Test      		//父包
import Test.A.t2Class;  //在父包导入子包类时，必须使用子包的全名，不能省略父包部分
public class t1Class{}

//t2Class.java
package Test.A			//子包
public class t2Class()
```
### 2.3 Java 类与对象
#### 2.3.1  *Class*类 / 对象 - 类的元数据
##### 1. *Class*类概述
&emsp; &emsp; 在 Java 中，一切皆对象。Java分为两种对象：<font color=red>**Java 实例对象 和 Java Class 对象(字节码文件描述对象)**</font>。每个类的运行时的类型信息就是用 *Class* 对象表示的。它包含了与类有关的信息。每一个类有且只有一个 *Class* 对象，*Class* 对象对应 `java.lang.Class` 类，是对类的抽象和集合，是类的字节码文件描述对象。*Class* 类的特点如下：
&emsp; &emsp;&emsp; ● 自定义的类在编译后会<font color=green>**生成一个唯一的 *Class* 对象**，***Class* 对象保存在与自定义类同名的 *.class* 文件中**</font>。
&emsp;&emsp;&emsp;  ●<font color=green> **无论创建多少个自定义类的对象，有且只有一个 *Class* 对象**</font>，表示自定义类的类型信息。
&emsp; &emsp;&emsp; ● Class 类没有公共的构造方法，<font color=green>**仅在类加载的过程中，由 *jvm* 自动构造，因此不能显式的声明一个 *Class* 对象**</font>。
![[../picture/Pasted image 20240106220329.png#pic_center|500]]
><font color=SlateBlue>  <u>**Q1. Class类的作用 ？**</u></font>
&emsp;&emsp;&emsp; 在C++中有个重要的概念：<font color=red>**运行时类型识别(`RTTI`)**</font>，其作用是在运行时识别一个对象的类型和类的信息。java中同样存在`RTTI`，java中的`RTTI`实现有两种方式：
&emsp;&emsp; &emsp; ● <font color=green>在编译期已确定其所有对象的类型，这种方式需要在写程序的时候将对象通过 *new* 创建出来</font>。
&emsp; &emsp; &emsp; ● 通过反射机制，在运行时发现和使用类型的信息。在java中用来表示运行时类型信息的对应类就是Class类。
```java
1. Class类常用的方法:
  1）public static Class<?> forName(String className):参数是一个类的全限定名(package path)，返回该类的Class对象引用。
  2）public T newInstance():创建此 Class 对象所表示的类的一个新实例。  
  3）public native Class getSuperclass()：获取类的父类
  4）public ClassLoader getClassLoader() ：获得类的类加载器。
  5）public String getName() ：获取类或接口的名字。enum为类，annotation为接口。
  6）public Constructor<?>[] getConstructors() ：获得所有的构造函数。
  7）public Field[] getFields() :获得域成员数组。    
  8）public Method[] getMethods() ：获得方法。
2. 三种方法获得Class对象：
    1）获取类的静态成员变量class
          Class c = Test.class;
    2）调用对象的getClass()方法，返回该对象对应的一个Class对象
          Class c = test.getClass();
    3) 调用Class类的静态方法forName();
          Class c =Class.forName("testpackage.test");   
```

##### 2. Class类包含的信息
&emsp; &emsp; Class类由类加载器从 *.class* 文件中加载，并在 *jvm* 中生成该类的 Class 对象，每一个Class对象都关联着定义它的那个类加载器。每个Class类中包含的信息有 *field* (字段)，*method* (方法)、*constructor* (构造函数)，将这些信息的共有特性分别封装成一个类，就分别对应 *Field* 类， *Method* 类、 *Constructor* 类。
![[../picture/Pasted image 20240106220701.png#pic_center|500]]
&emsp; &emsp; &emsp;  ● ***Constructor*** 类：代表某个类中的一个构造方法 
&emsp; &emsp; &emsp; ● ***Method*** 类：代表某个类中的一个成员方法 
&emsp; &emsp;  &emsp; ● ***Field*** 类：代表某个类中的一个成员变量
```java
package test
public class ReflectClass {
    private int nums;
    public ReflectClass(int i){
        this.nums = i;
    }
    public int getNum(){
        return this.nums;
    }
    public void setNum(int n){
      	this.nums = n;
    }
}
// ============================================
// 1. Constructor类
// 从Class类中获取所有构造器类
Class clazz = Class.forName("test.ReflectClass");  //根据全限定类名，将对应的.class字节码文件加载到内存，并生成Class对象
Constructor [] constructors= clazz.getConstructors();  //获取Class类中的所有构造器
// 根据构造函数的参数类型，从Class类中，获取对应的构造器类
Constructor constructor1 = clazz.getConstructors(null);      //获取默认构造函数
Constructor constructor2 = clazz.getConstructors(int.class); //根据构造参数的类型，获取对应的构造器
ReflectClass reflectClassObj = (ReflectClass) constructor.newInstance(num);  //通过构造器创建实例对象，并输入构造函数参数

// ============================================
// 2. Method类
// 从Class类中获取对应的方法,仅能获取public定义的方法
reflectClassObj.getClass().getMethod("setNum",int.class).invoke(test,100);   //通过实例对象的Class类获取对应的方法，并执行
System.out.println(reflectClassObj.getClass().getMethod("getNum").invoke(test)); //通过实例对象的Class类获取对应的方法，并执行

// ============================================
// 3. Field类
// 从Class类中获取对应的方法，仅能获取public定义的成员变量
System.out.println(reflectClassObj.getClass().getField("nums").get(test));
```

#### 2.3.2 类引用
##### 1. 类的引用
&emsp; &emsp; 所谓引用，是指创建一个”对象标识符“ (句柄)，通过操纵”标识符“ (句柄) 使其指向一个对象。这个”标识符“称为引用变量(通俗来说，引用 = 引用变量 = 常说的变量)。<font color=green>**一个引用变量可以指向多个对象，但仅保留最后一次引用。一个对象也可以被多个引用变量所指**。</font>
```java
// 1.这里 new Person是一个对象，p是一个”标识符“，指向Person对象的引用。
Person p = new Person("A");   
int a = 2;     //2是一个Intger的对象，a是一个指向Intger的引用

// 2.一个引用可以指向多个对象，但仅保留最后一次引用
int a=2;
a=3;   

// 3.一个对象也可以被多个引用所指
Person p1 = new Person("B");
Person p2=p1;
```
&emsp; &emsp; 为了更加灵活的控制对象的声明周期，Java将对象的引用分为4个等级，<font color=red>从高到低依次为：**强引用，软引用，弱引用，虚引用**。</font>
&emsp; &emsp; &emsp; ●   **强引用 *FinalReference***：强引用是指创建一个对象并把这个对象赋给一个引用变量，强引用是使用最普遍的引用。<font color=green>**如果一个对象具有强引用，当内存空间不足时，JVM 宁愿抛出 *OutOfMemoryError* 错误，使程序异常终止，也不会回收具有强引用的对象来解决内存不足的问题**。</font>因此，<font color=darkorange>**当强引用对象不使用时，需要将其指向 *null*，使其可以被GC回收**。</font>
&emsp; &emsp; &emsp; ●   **软引用 *SoftReference***：软引用用来描述一些还**有用但非必需**的对象。如果一个对象**只**具有软引用，则内存空间充足时，垃圾回收器就不会回收它；如果内存空间不足了，jvm 会将软引用中的对象引用置为 *null*，然后通知垃圾回收器GC进行内存回收。</font>只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来**实现内存敏感的高速缓存**，如图片缓存，浏览器后端页面缓存。 
&emsp; &emsp; &emsp; ●   **弱引用 *WeakReference***：弱引用用来描述非必需对象。只具有弱引用的对象拥有更短暂的生命周期，当垃圾回收器GC扫描到只具有弱引用的对象，不管当前内存空间是否足够，都会回收内存。
&emsp; &emsp; &emsp; ●   **虚引用 *PhantomReference***：虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。使用虚引用的目的就是为了得知对象被GC的时机，可以利用虚引用来进行销毁前的一些操作，比如说资源释放等。
![[../picture/Pasted image 20240106221255.png#pic_center|650]]
```java
class Superclass{
    public void test(){
        System.out.println("test");
    }
    public static String DEFINE_TYPE = "hello";
    static{
        System.out.println("super");
    }
}

// 1.强引用 - 正常的对象引用
	String str =  new String("abc"); 
	Superclass super =  new Superclass();    
	super = null; // 当强引用不使用时，置为null，以便GC回收

// 2.软引用 - SoftReference
	Superclass superclass =  new Superclass(); 
	SoftReference<Superclass> softReference=new SoftReference<Superclass>(superclass);
	if(softReference.get()!=null){	//内存充足，还没有被回收器回收，直接通过get()方法获取对象
	   Superclass soft=(Superclass)softReference.get();  
	   soft.test();
	}else{
	    softReference = new SoftReference(superclass); // 内存不足，软引用的对象已经回收，重新构建软引用
	}
	// 软引用可以和引用队列联合使用，软引用对象被垃圾回收，JVM会把软引用加入到与之关联的引用队列中。
	ReferenceQueue<String> referenceQueue = new ReferenceQueue<>();  //引用队列
	String str = new String("abc");
	SoftReference<String> softReference = new SoftReference<>(str, referenceQueue);//软引用关联引用队列

// 3.弱引用 - WeakReference
	String str = new String("abc");
	WeakReference<String> weakReference = new WeakReference<>(str);
	str = null;
	// 弱引用可以和引用队列联合使用，弱引用对象被垃圾回收，JVM会把弱引用加入到与之关联的引用队列中。
	ReferenceQueue<String> referenceQueue = new ReferenceQueue<>();  //引用队列
	String str = new String("abc");
	WeakReference<String> weakReference = new WeakReference<>(str, referenceQueue);//弱引用关联引用队列

// 4.虚引用 - PhantomReference
	String str = new String("abc");
	ReferenceQueue queue = new ReferenceQueue();
	// 创建虚引用，要求必须与一个引用队列关联
	PhantomReference pr = new PhantomReference(str, queue);
```

&emsp; &emsp;<font color=Sienna>**2. Java 类加载过程中的引用关系**</font>
&emsp; &emsp;&emsp; 在类的全生命周期中，从类的加载器，生成类的 Class 对象，到类的实例对象的使用有着密不可分的引用关系： 
&emsp; &emsp; &emsp; ● 类加载器与 Class 对象：类的Class和加载它的加载器之间是<font color=orange>**双向关联**</font>关系。即<font color=green>一个Class对象总是会引用他的类加载器，调用Class对象的`getClassLoader`方法就可以获得它的类加载器。</font>
&emsp; &emsp; &emsp; ● 类，类的 Class 对象，类的实例对象：<font color=green>一个类的实例对象总是引用该类的Class对象，在Object类中定义该类`getClass`方法，会返回对象所属类的Class对象的引用。</font>
```java
package test;
class Superclass{
    public void test(){
        System.out.println("test");
    }
    public static final String DEFINE_TYPE = "hello";
    static{
        System.out.println("super");
    }
}
public class test {
    public static void main(String[] args) throws IllegalAccessException, InstantiationException, ClassNotFoundException {
        ClassLoader classLoader = Superclass.class.getClassLoader();  //Superclass的类加载器
        Class objClass = classLoader.loadClass("test.Superclass");   //Superclass Class对象
        Object obj=objClass.newInstance();                     //Superclass的Object父类
        Superclass superclass = (Superclass)obj;        //Superclass的实例对象
        superclass.test();
    }
}
```
![[../picture/Pasted image 20240106222811.png#pic_center|400]]

#### 2.3.3 类反射 - Class对象的应用
&emsp; &emsp; Java程序在运行之前需要先编译。程序中的对象初始化，对象的调用在编译时期就已经确定并加载到JVM中。当程序在运行时需要 "**动态加载**"某些类时，由于这些类并没有加载到JVM当中，无法直接获取。但是<font color=green>通过Java的反射机制，可以在运行时动态地创建对象并调用其属性和方法，不需要提前在程序中(编译时期) 进行对象的初始化和调用。</font><font color=red>Java反射机制的本质是JVM得到Class对象之后，再通过Class对象进行反编译，从而获取对象的各种信息。</font>
![[../picture/Pasted image 20240106225201.png#pic_center|850]]

><font color=SlateBlue>  <u>**Q1. 反射有什么用途 ？**</u></font>
&emsp; &emsp;&emsp; ① 通过Java反射机制，访问Java未初始化对象的属性和方法。
&emsp;&emsp;&emsp; ②  反射最重要的用途就是开发各种通用框架。如: `Spring`都是通过`xml`文件配置化的，为了保证框架的通用性，大多数框架需要根据配置文件加载不同的类或者对象，调用不同的方法，这时框架就必须使用反射在程序运行时动态加载需要使用的对象。
&emsp;&emsp;&emsp; ③ 当使用IDE编程时，IDE会自动列举某一对象所包含的方法和属性，这是通过反射实现的。
>
<font color=SlateBlue>  <u>**Q2. 反射技术的优缺点 ？**</u></font>
&emsp;&emsp;&emsp;① 优点：反射提高了Java程序的灵活性和扩展性，降低耦合性，提高自适应能力。
&emsp;&emsp;&emsp;② 缺点：
&emsp;  &emsp;  &emsp; ● 性能问题：反射技术是通过.class文件的一种解释操作，反射用于方法和字段时其性能要慢于代码中直接定义。因此Java反射机制一般用于灵活性和扩展性要求很高的系统框架上，普通程序不建议使用。
&emsp;  &emsp; &emsp;  ● 代码维护问题：反射技术绕过了源代码，使用反射会模糊程序内部逻辑，因而会带来维护问题。

#### 2.3.4 继承与接口

##### 1. 继承 *extends*

&emsp; &emsp; 继承是 java 面向对象的特征之一，Java的继承通过关键字 ***extends*** 来实现，<font color=green>与C++相比，*java* 摒弃了多继承，只保留了单继承，即**每一个类最多只有一个直接父类**。</font>在子类创建某个类对象时，系统会隐式的创建该类的父类对象，且可以通过 ***super*** 来该子类的父类对象。如果一个 java类没有显式指定直接父类，则默认其直接父类为 `Java.lang.Object`。
```java
class parent{
	public int a;
}
class children extends parent{
    void fun(){
       super.a=10;  //父类的成员变量
    }
} 
```
&emsp;&emsp; 由于**继承会破坏父类的封装性，使子类与父类之间的耦合**，因此子类与父类之间应当遵循如下规则：
&emsp; &emsp; &emsp;  <font color=green>① 将父类的**所有属性**设置为` private`，不让子类直接访问父类属性。</font>
&emsp; &emsp;  &emsp; <font color=red>② 不要让子类随意修改、访问父类的方法。父类中的工具方法要设置为 ***private*** ; 父类中需要被外部调用但不希望子类重写该方法的要设置为 ***final*** ;
如果父类的方法能够被子类重写，但不希望被其他类访问，要设置为 ***protected***。</font>
&emsp; &emsp; &emsp; <font color=green>③ 不要在父类构造器中调用被子类重写的方法。</font>

##### 2. 抽象类与接口
&emsp; &emsp; 与C++相比，java的抽象与接口定义更加明确。在 java中，通过 ***abstract*** 定义抽象类，与C++相同，<font color=green>**因为抽象类中含有无具体实现的方法，所以不能用抽象类创建对象(无法进行实例化)，仅能通过子类的继承对抽象方法进行实例化**。</font>除此之外，java还引入了一种更加纯粹的抽象类 - 接口( ***interface*** )，<font color=green>在 ***interface*** 中，所有的方法都是抽象方法，同时引入了 ***implement*** 来实现接口。</font>
><font color=SlateBlue>  <u>**Q1. 抽象类与接口的区别 ？**</u></font>
&emsp; &emsp; ● 语法层面：
&emsp; &emsp; &emsp;  ①  抽象类可以提供成员方法的**实现细节**，而接口中只能存在 `public abstract` 方法；
&emsp; &emsp; &emsp;  ②  抽象类中的成员变量可以是各种类型的，而接口中的成员变量只能是 `public static final` 类型；
&emsp; &emsp;&emsp;   ③  接口中不能含有静态代码块以及静态方法，而抽象类可以有静态代码块和静态方法；
&emsp; &emsp;&emsp;   ④  一个类只能继承一个抽象类，而一个类却可以实现多个接口。
&emsp;&emsp;  ● 设计层面：
&emsp; &emsp;&emsp;   ①  抽象类是对类本质的抽象，避免在子类开发重复的代码，表达的是 ***is-a*** 的关系，包含的是实现子类的通用特性，将子类存在差异化的特性进行抽象，交由子类去实现。
&emsp; &emsp;&emsp;   ②  接口是对行为的抽象，表达的是 ***like-a*** 的关系，接口的核心是定义行为，即实现类可以做什么，至于实现类是谁，如何实现的，接口并不关心。  
```java
// ifaceA.java
package A.package1;
public interface ifaceA{  //定义接口
    void fun1();
    void fun2();
}
//ifaceB.java
package A.package1;
public interface ifaceB{  //定义接口
    void fun3();
    void fun4();
}

// Iimplement.java
package A;
import A.package1.iface;
public class Iimplement implements A.package1.ifaceA,A.package1.infaceB{  //实现接口
	@Override
	public void fun1(){
		System.out.print("A");
	}
	@Override
	public void fun2(){
		System.out.print("B");
	}
	@Override
	public void fun3(){
		System.out.print("C");
	}
	@Override
	public void fun4(){
		System.out.print("D");
	}
} 
```

#### 2.3.5 内部类
&emsp; &emsp;内部类是指定义在一个类中的类，内部类主要有以下三个作用：
&emsp; &emsp; &emsp; ① 内部类可以访问该类定义所在作用域中的数据，包括私有数据
&emsp; &emsp; &emsp; ② 内部类可以对同一个包中的其他类是隐藏，即同一个包的其他类是无法调用该类的内部类的。
&emsp; &emsp; &emsp; ③ 当要定义一个回调函数时，可以使用匿名内部类的方式。

### 2.4 Java 注解 Annotation - 类的标签

&emsp; &emsp; 注解是 Java 提供的一种途径和方法，可以使源程序中的元素关联到代码中的元数据 ( ***metadata*** )。  <font color=green>注解是附加在代码中的一些源信息或标签信息，用于在一些工具在编译、类加载、运行时进行解释和使用，起到**说明**，**配置**的功能，**可以将注解理解为标签**。</font>注解为一种修饰符，应用于类、方法、参数、变量的声明语句中。注解不会也不能影响代码本身的业务逻辑，仅仅只能起到辅助性的作用。**在定义注解时，通过 *@interface* 进行修饰，使用注解时要在被修饰的类或变量之前调用定义的注解。**
><font color=SlateBlue>  <u>**Q1. 什么是元数据 (Metadata)？**</u></font>
&emsp; &emsp;&emsp;  要真正了解注解的工作方式和原理，就需要先了解什么是元数据。
&emsp; &emsp; &emsp;元数据是一种很抽象的定义，<font color=red> 元数据是一系列关于数据的数据，是一系列用来描述数据的数据。</font>元数据可以为数据说明其元素或属性(如：名称、大小、数据类型等)，或其结构 (如：长度、字段等)，或其相关数据(如：位于何处、数据拥有者等)
![[../picture/Pasted image 20240113231441.png#pic_center|600]]
<font color=SlateBlue>  <u>**Q2. @interface 和 interface 的区别 ？**</u></font>
&emsp;&emsp;&emsp; ●  ***@interface*** : 是用来修饰 ***Annotation*** 的,表示实现了`java.lang.annotation.Annotation`接口
&emsp; &emsp;&emsp;●  ***interface***: 声明一个java的接口
#### 2.4.1 注解基本概念
##### <font color=Sienna>**1. 注解的属性**</font>
&emsp;&emsp;注解的属性也叫做成员变量。<font color=red>**注解只有成员变量，没有方法**</font>。注解的成员变量在注解的定义中以“无形参的方法”形式来声明，其**方法名=该成员变量的名字，其返回值 = 该成员变量的类型，若注解中属性存在默认值，默认值需要用 default 关键字指定。**对注解的属性赋值的方式是在注解的括号内以value=”” 形式，多个属性之前用","隔开。
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
    int id() default 0;    	//等同于成员变量 => int id = 0;
    String msg(); 					//等同于成员变量 => string msg;
}

@TestAnnotation(id=3,msg="hello")  //对注解的变量进行赋值
public class Test {
}
```
##### <font color=Sienna>**2. JDK内置 *Annotation* **</font>
&emsp;&emsp; 在Java中存在三个基本的Annotation：
&emsp;&emsp;&emsp;  ● **@Override** - 限定重写父类方法：强制一个子类覆盖父类的方法。
&emsp;&emsp;&emsp; ● **@Deprecated** - 标示已过时：表示某个程序元素(类，方法，成员变量)已过时，当使用过时方法时，编译器会发出警告。
&emsp;&emsp;&emsp; ● **@SuppressWarnings** - 抑制编译器警告：通常情况下，程序中使用没有泛型限制的集合会引起编译器警告，为了避免警告，可以通过`@SuppressWarning`抑制编译器警告。
```java
public class Base{
	void fun(){}
	@Deprecated
	void old(){}   //该方法已过时，不可再调用
}
class child extends Base{
	@Override   //重写父类方法
	@SuppressWarnings(value = "uncheck")  //抑制编译器警告
	void fun(){
		List<String>list=new ArrayList();
	}
}
```

##### <font color=Sienna>**3. 元注解**</font>
&emsp;&emsp;元注解是 Java 提供的**用于修饰注解的注解**，是基本注解。`Annotation`提供了四种元注解，分别是：
&emsp;&emsp; &emsp;**● @Target：**注解所作用的目标，指明这个注解最终是用来修饰方法，还是修饰类，还是修饰属性；
&emsp;&emsp; &emsp;**● @Retention：**用于**指明当前注解的生命周期**，生命周期包含三个阶段：
&emsp;&emsp; &emsp;&emsp;  **`SOURCE` ：注解仅保留在源级别，在编译阶段由编译器丢弃忽略**。
&emsp; &emsp; &emsp;&emsp;  **`CLASS` ： 注解在编译时由编译器保留，但在JVM类加载时被丢弃**。
&emsp;&emsp; &emsp;&emsp;   **`RUNTIME` ：注解始终不会丢弃，运行期也保留该注解，可以使用反射机制读取该注解的信息**。
![[../picture/Pasted image 20240113231940.png#pic_center|580]]
&emsp;&emsp; &emsp;● **@Documented**：注解修饰的注解，当执行 JavaDoc 文档打包时会被保存进 doc 文档，反之将在打包时丢弃。
&emsp;&emsp; &emsp;● **@Inherited**：注解修饰的注解是具有可继承性的，也就说我们的注解修饰了一个类，而该类的子类将自动继承父类的该注解。
```java
public @interface TestAnnotation {    //通过 @interface 来定义注解 
}

@Inherited	//元注解，用于对注解的修饰，相当于对@TestAnnotation注解打上了一个标签
						//注解@TestAnnotation被@Inherited修饰，之后类Test被@TestAnnotation注解，类TestB继承A,类TestB也拥有@TestAnnotation注解。
@TestAnnotation   //对类使用注解，相当于对Test类打上了一个标签
public class Test {
}
```

#### 2.4.2 自定义注解 (注解的应用)
&emsp;&emsp;定义新的Annotation类型需要使用`@interface` 关键字。<font color=green>通俗来说，可以将注解理解为”标签“，在类和方法中使用注解，相当于对类和方法打上了”标签“，后续只需要判断是否存在这个”标签“，并对”标签“的”内容(定义的变量)“进行处理即可</font>。<font color=red>在注解应用的过程中，通常包括三个步骤：**注解声明，使用注解元素，配置注解处理器**</font>。自定义Annotation 的规则如下： 
&emsp;&emsp;&emsp;① Annotation 型定义为 @interface，所有的 Annotation 会自动继承` java.lang.Annotation`这一接口,并且不能再去继承别的类或是接口。
&emsp;&emsp;&emsp;② 注解的参数成员(变量、方法)只能用 `public` 或默认(`default`) 这两个访问权修饰。
&emsp;&emsp;&emsp;③ 注解的参数成员只能用基本类型 byte、short、char、int、long、float、double、boolean 八种基本数据类型和String、Enum、Class、annotations等数据类型，以及这一些类型的数组。
&emsp;&emsp;&emsp;<font color = orange>④ 要获取类方法和字段的注解信息，必须通过 java的反射技术来获取 `Annotation` 对象。</font>
![[../picture/Pasted image 20240113232446.png#pic_center|700]]

#### 2.4.3 注解的底层原理 (从字节码到注解实例)
&emsp;&emsp; <font color=red> `Annotion`(注解)是一个接口，程序可以通过反射来获取指定程序元素的`Annotion`对象，然后通过`Annotion`对象来获取注解里面的元数据。</font><font color=green>注解不支持继承，因此**不能使用关键字`extends`来继承某个`@interface`**，但注解在编译后，编译器会自动继承`java.lang.annotation.Annotation`接口。因此，注解的本质是一个继承了 `Annotation` 接口的接口。</font>注解作为一个特殊的接口，其实现类是在代码运行时生成的动态代理类，而之后底层代码通过**反射**的方式获取到注解。
![[../picture/Pasted image 20240114172651.png#pic_center|600]]
&emsp;&emsp;  下图以`@Override`为例，说明注解的本质：
![[../picture/Pasted image 20240114172749.png#pic_center|700]]
&emsp;&emsp;&emsp;注解的解析流程如下所示：
&emsp; &emsp;&emsp; ① `getAnnotation() `作为获取注解的主要入口，*Class*类，*Field*类，Method类调用 `annotationData() `方法创建 ***Class.AnnotationData*** 类实例，***Class.AnnotationData*** 是一个注解缓存类，用于缓存该类的注解信息，其中包含两个Map，分别用于存储当前类的注解信息和继承的注解信息。
&emsp; &emsp;&emsp; ② ***Class.AnnotationData***  是由 ***AnnotationParser* 类对.class字节码进行解析**，并通过 ***AnnotationInvocationHandler*** 生成注解的**动态代理对象 *Annotation***，并将注解对象加入到 `LinkedHashMap (key=Class类型，value=注解对象)`中。
![[../picture/Pasted image 20240114172857.png#pic_center|780]]

### 2.5 Java 线程与并发
&emsp; &emsp;线程，一个执行实体，正在执行的程序，担当分配系统资源（CPU、内存）的实体。线程的底层实现原理在[操作系统](a)中具体介绍。这里介绍Java线程的实现方式以及Java线程如何使用。
#### 2.5.1 线程的实现

##### 1. 线程的创建与运行
&emsp;&emsp;Java的线程是通过 *java.lang.Thread* 类来实现的，在Java中有**三种方法来实现**线程：
&emsp;&emsp;&emsp;① <font color=green>**通过创建继承 *Thread* 类的实例来创建新的线程**。</font>每个线程都是通过对应的方法 `run()` 来描述该线程需要执行的操作。通过调用 *Thread* 类的`start()` 方法来启动一个线程。
&emsp;&emsp;&emsp;② Java中只支持单继承，如果一个类继承了某个父类，就无法再继承 *Thread* 类。因此 *Thread* 类提供了一个 *Runnable* 接口，<font color=green>**通过重写 *Runnable* 接口中的`run()`方法，也可以实现线程的启动**，</font><font color=orange>**因此 *Runnable* 一个线程操作的方法体，是用户定义的需要完成的具体任务，并通过 `Thread.start()` 去启动线程**。</font>
&emsp;&emsp;&emsp;③ 对于某些场景，需要在线程执行完成后将任务执行的结果返回，或当线程在执行时抛出异常，因此可以通过实现 *Callable* 接口中的  `call()` 方法来完成结果的返回。<font color=green> ***Callable* 接口通常与 *FutureTask* 一起使用，通过 *FutureTask* 来异步的执行线程，并保存线程结果**。</font><font color=red>注：通过`new Thread()` 创建线程时，只能通过 *Runnable* ，不能通过 *Callable* 。</font>
&emsp;&emsp;&emsp; <font color=red>**Notice**：当一个线程运行结束后，无法通过 `start()` 方法再次启动。即**每个线程只能被启动一次**。</font>
```java
public class test {
    public static void main(String[] args) {
    // 1. 通过继承Thread类实现多线程
        MyThreadA myThread=new MyThreadA();
        
	  // 2.通过实现Runnable接口实现多线程
        MyThreadB myThreadB = new MyThreadB();
        Thread thread = new Thread(myThreadB);
        
   // 3. 通过Callable接口实现线程结果返回
   		CallableExample callable = new CallableExample();
   		FutureTask futureTask=new FutureTask(callable);
   		futureTask.run();
   		futureTask.get();  // 获取线程返回的运行结果
      //线程启动
      myThread.start();
      thread.start();
    }
}
// 1. 通过继承Thread类实现多线程
class MyThreadA extends Thread{
    @Override
    public void run() {
        for(int i=0;i<10;++i)
            System.out.println("A "+i);
    }
}
// 2.通过实现Runnable接口实现多线程
class MyThreadB implements Runnable{
    @Override
    public void run() {
        for(int i=0;i<10;++i)
            System.out.println("B "+i);
    }
}

// 3. 通过实现Callable接口，实现线程结果的返回
class CallableExample implements Callable { 
	@Override
    public Object call() throws Exception { 
        int a=10;
        return a; 
    } 
} 
```
><font color=SlateBlue>  <u>**Q1. `run()` 和 `start()` 的区别 ？**</u></font>
&emsp;&emsp;  ● `run()` 方法是一个普通的成员方法，当线程调用了`start()`方法后，该线程会去调用这个`run()`方法，运行该线程需要执行的操作。因此，**如果直接调用 `run()` 方法，只会在原有线程上运行，不会创建一个新的线程**。
&emsp;&emsp;  ● `start()` 方法用来启动线程。当线程创建成功时，线程处于 `NEW(创建)` 状态，调用 `start()` 后，线程会变为`READY(就绪)`状态，在等待CPU调度后，线程才可以运行，进入`RUNNING(运行)`状态。

##### 2. 获取线程运行结果
&emsp; &emsp;  在Java中，为了充分利用计算机CPU资源，一般开启多个线程来执行异步任务。但不管是继承 *Thread* 类还是实现 *Runnable* 接口，都无法获取任务执行的结果。JDK 5中引入了 *Callable* 和 *Future*，主要用于Java多线程计算过程的异步结果获取。

#### 2.5.2 线程状态切换
&emsp;&emsp;在线程的生命周期中，线程共有5种状态：<font color=red>**创建、就绪、运行、死亡、阻塞**</font>。在任意一个时间点，一个线程只能有且只有其中一种状态。
&emsp;&emsp;&emsp;① **创建态 ( *New* )**：创建后尚未启动的线程处于这种状态。 
&emsp;&emsp;&emsp;② **运行态 ( *Runable* )**：处于此状态的线程有可能正在执行，也有可能正在等待CPU分配执行时间。
&emsp;&emsp;&emsp;③ 阻塞态：一个正在运行的线程因某些原因不能继续运行时，它就进入阻塞状态。阻塞态根据阻塞方式的不同又可以分为三种：
&emsp; &emsp; &emsp; ● **等待阻塞 ( *Waiting* )**：处于这个状态的线程不会被分配CPU时间，只能等待被其他线程显式的唤醒。以下方式会使线程进入等待阻塞状态：【没有设置*Timeout* 参数的 `Object.wait()` 方法】、【没有设置 *Timeout* 参数的 `Thread.join()` 方法】、【 `LockSupport.park()` 方法】。
&emsp; &emsp; &emsp; ● **同步阻塞 ( *Blocked* )**：在线程等待进入同步区域时，线程会处于此状态。线程会等待获取一个排他锁，从而进入同步区域。
&emsp; &emsp; &emsp; ● **限期阻塞 ( *Timed Waiting* )**：处于此状态的线程不会被分配CPU时间，但此状态的线程不需要其他线程的显式的唤醒，在一段时间后，线程会由系统自动唤醒。以下方式会使线程进入等待阻塞状态：【`Thread.sleep()` 方法】、【设置了 *Timeout* 的 `Object.wait()` 方法】、【设置了 *Timeout* 的 `Thread.join()` 方法】、【`LockSupport.parkNanos()` 方法】、【`LockSupport.parkUntil()` 方法】
&emsp;&emsp;&emsp; ④ **死亡态 ( *Terminated* )**：已终止线程的线程状态，线程已经结束执行。

![[../picture/Pasted image 20240121162847.png#pic_center|750]]
><font color=SlateBlue>  <u>**Q1. `sleep()`、`wait()` 和 `notify()` 的区别 ？**</u></font>
&emsp;&emsp;&emsp;Java中的`sleep()`和`wait()`函数都可以挂起当前线程，使线程休眠，但实现方式和用法不同：
&emsp;&emsp;&emsp;  ● <font color =green > `sleep()`是  *Thread* 类的方法**静态方法**，需要通过 *Thread* 类调用 `Thread.sleep()`。而 `wait()` 和 `notify()` 是 *Object* 类中的实例方法，因为Java所有类都继承于 *Object* 类，所有类中都可以使用。</font>
&emsp;&emsp;&emsp;  ● <font color = red> `wait()`、`notify()`必须用在 ***synchronized*** 代码块中调用。</font><font color=green>当调用`wait()` 方法后，当前获得 *synchronized* 同步块对象锁的线程进入”等待阻塞“状态，同时**释放当前线程的对象锁**。此时**其他线程可以获得该 *synchronized* 同步块的对象锁**。被阻塞的线程需要通过 `notify()` 方法来唤醒。</font>
&emsp;&emsp;&emsp;  ● 当在 *synchronized* 同步块中使用 `sleep()`，该线程会被挂起，但**不会释放对象锁**，所以如果有其他线程等待执行该 synchronized 代码块，一直会被阻塞，等待该线程被 `notify()`唤醒释放对象锁。
![[../picture/Pasted image 20240121163022.png#pic_center|680]]

#### 2.5.3 线程的调度
&emsp;&emsp;线程的调度是指系统为线程分配CPU使用权的过程，线程的调度分为两种方式: 协同式线程调度、抢占式线程调度。 
&emsp;&emsp;&emsp;**① 协同式线程调度**：使用协同式线程调度的多线程系统，线程的执行时间是由线程本身控制的。线程把自己的任务执行完成之后，会主动通知系统切换到另一个线程上。
&emsp;&emsp;&emsp; ● 优点：由于线程执行完当前任务才会切换新线程，因此不存在线程同步问题。
&emsp;&emsp;&emsp; ● 缺点：由于每个线程的执行时间是不可控的，如果线程出现死循环，则会导致程序被阻塞。
&emsp;&emsp;&emsp;**② 抢占式线程调度**：使用抢占式线程调度的多线程系统，每个线程都是由系统来分配执行时间，线程的切换不由线程本身决定。由于线程的执行时间是可控的，即使某一线程出现问题，不会导致整个进程被阻塞。同时，通过给线程设置优先级，可以给不同的线程分配不同的执行时间。

#### 2.5.4 线程共享变量
&emsp;&emsp;要保证线程安全，不一定必须要进行同步，同步只是保证共享数据在多线程竞争时保持顺序性的措施。如果一个方法中不存在共享数据，则该方法一定是线程安全的。这种不存在共享数据的方法(代码)分为以下两类： 
&emsp; &emsp;&emsp; ① **可重入代码**：可重入代码也称为纯代码，这类<font color=red>**代码不依赖存储在堆上的数据和公共的系统资源，用到的状态量都是有参数传入，且代码中不会调用非可重入方法**</font>。因此，所有的可重入代码都是线程安全的，可重入代码在其执行期间的任何时刻发生中断，其计算结果都不会发生任何错误，
&emsp; &emsp;&emsp; ② **线程本地存储**：如果我们把共享数据的处理代码能够保证在一个线程中执行，即<font color=red>**把共享数据的可见范围限制在同一个线程之中，共享变量数据在每个线程中都有一个副本，每个线程仅操作副本当中的共享变量数据，那么这样也可以避免共享数据在不同线程操作所导致的线程安全问题**</font>。在Java中可以通过 ***ThreadLocal*** 类实现线程的本地存储功能。每个 Thead 线程对象中都有一个 *ThreadLocalMap* 对象，*ThreadLocalMap* 中 存储了多个以  *threadLocalHashCode* 为 Key，本地变量为值的为 Value 的数据。*TheadLocal* 对象就是 *ThreadLocalMap* 的访问入口。
&emsp; &emsp;&emsp; 线程安全的本质是为了保证线程数据的安全，在Java中可以使用 *ThreadLocal*  维护变量，从而可以不再使用锁，同步器等工具实现线程的同步。
![[../picture/Pasted image 20240121163427.png#pic_center|580]]

##### 1. *ThreadLocal* - 任务实体中的"共享/全局"变量
&emsp; &emsp; 为了保证线程的数据安全，在Java中可以使用 ThreadLocal 维护变量，ThreadLocal为每个使用该变量的线程提供<font color=green>**独立的局部变量副本**</font>，每一个线程都可以独立地改变自己的副本，通过 set() 和 get() 来对这个局部变量进行操作，但不会和其他线程的局部变量进行冲突，实现了线程的数据隔离。
![[../picture/Pasted image 20240121163529.png#pic_center|580]]
><font color=SlateBlue>  <u>**Q1.ThreadLocal 是如何实现线程 (数据) 隔离的 ？**</u></font>
&emsp;&emsp;&emsp;Thread 类中有两个变量`ThreadLocalMap threadLocals`和`ThreadLocalMap inheritableThreadLocals`。在每个 Thread 线程对象中，都维护了一个*ThreadLocalMap*。即一个 Thread 线程对象，最多只有一个 *ThreadLocalMap*，而 *ThreadLocalMap* 底层是一个 **Entry 数组**，但是一个 Thread 可以有多个 ThreadLocal，一个 *ThreadLocal* 对应一个变量数据，变量数据将 *ThreadLocal* 作为Key，Object 作为 value，封装成 Entry 存到 *ThreadLocalMap* 中 Entry[] 数组中。Thread 与 ThreadLocal 之间的关系如下图所示:
![[../picture/Pasted image 20240121164021.png#pic_center|750]]
<font color=SlateBlue>  <u>**Q2.ThreadLocalMap 是如何解决Hash冲突的 ？**</u></font>
&emsp;&emsp;&emsp;每个ThreadLocal都有一个对应的`threadLocalHashCode`，通过`threadLocalHashCode & (len-1)`可以算出 ThreadLocal 变量对应的 Entry[] 数组的下标(即Key)。当Key发生Hash冲突时，ThreadLocalMap 采用<font color=red>**线性探测方法**</font>，循环查找下一位(索引)是否冲突，直到找到不存在冲突的索引 ( Entry[] 数组下标)。
>
<font color=SlateBlue>  <u>**Q3.ThreadLocalMap的Entry中，对 ThreadLocal 的引用为什么要设置成弱引用 ？**</u></font>
&emsp;&emsp;&emsp;当代码中将 ThreadLocal 的强引用置为null后，这时候 Entry 中的 ThreadLocal 应该被回收了，但是如果 Entry 的 key 被设置成强引用则该 ThreadLocal 就不能被回收，从而会导致内存泄露。 
![[../picture/Pasted image 20240121164412.png#pic_center|680]]
>
<font color=SlateBlue>  <u>**Q4. ThreadLocal的内存泄露问题？**</u></font>
&emsp;&emsp;&emsp;●   <font color=green>虽然 Entry 对象中的 ThreadLocal 引用为弱引用，但这个弱引用只是针对key的。当把 Threadlocal 实例置为null以后，没有任何强引用指向 Threadlocal 实例，此时 Threadlocal 将会被gc回收。</font><font color=orange>虽然 ThreadLocal 被回收了，但是 Entry 对象中的value却不能回收，因为存在一条从`Current Thread`连接过来的强引用。只有当前Thread结束以后, `Current thread`就不会存在栈中，连接value的强引用断开。此时Current Thread, ThreadLocalMap, Entry-value将全部被GC回收。</font>
&emsp;&emsp;&emsp;●  根据上述 ThreadLocal 内存回收的过程可以看出，<font color=red>只要当前的线程对象被GC回收，ThreadLocal 就不会出现内存泄露的情况。</font>但如果是在<font color=red>使用线程池</font>的时候，线程结束是不会销毁的，会再次使用的。就可能出现内存泄露。<font color=red>因此，当使用完 ThreadLocal 之后，调用`Threadlocal`的`remove()`方法把当前`ThreadLocal`从当前线程的`ThreadLocalMap`中移除。</font>

##### 2. *InheritableThreadLocal* ( ITL )
&emsp; &emsp; 虽然 *ThreadLocal* 为每个使用该变量的线程提供<font color=green>**独立的局部变量副本**</font>，使当前线程变量不会和其他线程的局部变量进行冲突。但是在父线程中创建的本地变量是无法传递给子线程的，因此Java提供了<font color=red>`InheritableThreadLocal (ITL)`来解决线程在继承过程中变量的传递问题。</font>
```java
public static void main(String[] args) throws InterruptedException {
	ThreadLocal<String> tL=new ThreadLocal<>(); 						//主线程(父线程)创建本地变量
	ThreadLocal<String> itL=new InheritableThreadLocal<>(); //主线程(父线程)创建本地变量
	tL.set("Threadlocal"); 
	itL.set("Threadlocal"); 				
	System.out.println(tL.get());			//输出 “Threadlocal”
	new Thread(()->{                	//在父线程中创建子线程，并在子线程中输出父线程的本地变量
	   System.out.println(tL.get()); 	//输出null，因为ThreadLocal变量不能通过父子线程进行继承
	   System.out.println(itL.get()); //输出Threadlocal，InheritableThreadLocal变量可以通过父子线程进行继承
	}).start();
}
```
><font color=SlateBlue>  <u>**Q1. InheritableThreadLocal 是如何实现线程本地变量继承的 ？**</u></font>
&emsp;&emsp;&emsp;`InheritableThreadLocal`是 ThreadLocal 的子类。在线程在创建并初始化时，会检查其父线程是否存在 inheritableThreadLocals，如果存在则会在父线程的 inheritableThreadLocals 的基础上创建子线程。
![[../picture/Pasted image 20240121165120.png#pic_center|650]]

##### <font color=Sienna>**3. *TransmittableThreadLocal* ( TTL )**</font>
&emsp; &emsp;TL 解决了不同线程之间使用同一本地变量时的冲突问题，ITL解决了在线程继承中，本地变量从父线程传递(继承)到子线程的问题。但在ITL中仅解决了线程继承这一瞬间的变量传递问题，如果创建子线程一直被池化复用 ( 如线程池中的子线程)，则父线程与子线程之间的变量无法进行同步，则会导致数据问题。针对该问题，AliBaba 在ITL的基础上提出了TTL，<font color=green>**用来解决子线程池化复用时的变量数据（此时的变量数据可以看做是业务逻辑的上下文）传递问题**。</font>
&emsp; &emsp;&emsp; TTL 为了能够在子线程池化复用的过程中保持变量数据的一致性，TTL 对原有的 `Runnable` 进行了改造，实现了 `TtlRunnable` 。通过**CRR模式 `(capture[抓取]，replay[回放]，restore[恢复])`** 对上下文的数据进行同步。
![[../picture/Pasted image 20240121165342.png#pic_center|600]]

### 2.6 Java 线程池
#### 2.6.1 线程池描述
&emsp;&emsp;线程池 (Thread Pool)是一种基于池化思想管理线程的工具。通常一个线程池包含4个基本组成部分：
&emsp; &emsp;&emsp; ① 线程池管理器 ：用于创建并管理线程池，包括创建线程池，销毁线程池，添加新任务； 
&emsp; &emsp;&emsp; ② 工作线程：线程池中线程，在没有任务时处于等待状态；
&emsp; &emsp;&emsp; ③ 任务接口：为工作线程提供任务； 
&emsp; &emsp;&emsp; ④ 任务队列：用于存放没有处理的任务。

### 2.12 Java 数据库 - JDBC
&emsp;&emsp; JDBC的全称是Java数据库连接 ( Java Database Connect )，它是一套用于执行SQL语句的 Java API。应用程序可通过这套API连接到关系数据库，并使用SQL语句来完成对数据库中数据的查询、更新和删除等操作。