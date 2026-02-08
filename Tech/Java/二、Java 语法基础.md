### 2.1 Java 程序基本规则
&emsp;&emsp;与C++相比，Java是完全的面向对象的语言，因此Java与C++在程序规则上有所区别：
&emsp; &emsp;&emsp; ① Java程序必须以`class`的形式存在，`class`是java的最小程序单位。<font color=red>在Java程序中，不允许可执行语句、方法独立存在。</font>
&emsp; &emsp;&emsp;  <font color=red>② 如果一个Java程序中定义了一个`public`类，则该程序文件的文件名必须和该`public`类名相同。因此，一个Java程序文件最多只能有一个`public`类。</font>
&emsp; &emsp;&emsp;  ③ Java中编译器通过 Main() 方法作为程序的入口，且 Main() 的修饰符必须为：`public static void main(String [] args){}`
### 2.2 Java 语法基础
#### 2.2.1 Java关键字
&emsp; &emsp; Java中一共有48种关键字，如下表所示。关键字可以分为**程序逻辑控制关键字，系统控制(线程同步)关键字，数据类型关键字，类/对象关键字，包/方法管理关键字**。
![[../picture/Pasted image 20240105215809.png#pic_center|750]]

##### 1. volatile 类型
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
&emsp; &emsp;② 定义为 **volatile** 类型变量会<font color=red>**禁止指令重排序优化**</font>。而普通变量仅会保证变量在方法中能够得到正确执行结果，而不保证变量赋值操作顺序与程序代码中逻辑顺序是一致的。( 指令重排序是指CPU采用了允许将多条指令不按程序顺序，分开发送给相应电路单元进行处理，但指令重排不是任意的，CPU需要根据指令依赖情况进行重排，以保证得到正确的执行结果 )
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
##### 2. synchronized 类型
&emsp; &emsp; Java中互斥同步方式就是 synchronized 关键字。synchronized 主要有三种用法：**修饰实例方法，修饰静态方法，修饰代码块**。
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
&emsp; &emsp; **② 修饰静态方法**:  **给当前类加锁，会作用于类的所有对象实例 ，其作用的范围是整个静态方法，进入同步代码前要获得当前 class 的锁**。<font color=red>如果一个线程A调用一个实例对象的 non-static-synchronized 方法，而线程B需要调用这个实例对象所属类的 static-synchronized 方法，不会发生互斥情况，因为访问 static-synchronized 方法占用的锁是当前类的锁，而访问 non-static-synchronized 方法占用的锁是当前实例对象锁。</font>`synchronized void staic method(){ 业务代码 }`
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
#### 2.2.2 Package 与 Import  机制
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
#### 2.3.1  Class类 / 对象 - 类的元数据
##### 1. Class类概述
&emsp; &emsp; 在 Java 中，一切皆对象。Java分为两种对象：<font color=red>**Java 实例对象 和 Java Class 对象(字节码文件描述对象)**</font>。每个类的运行时的类型信息就是用 *Class* 对象表示的。它包含了与类有关的信息。每一个类有且只有一个 Class 对象，Class 对象对应 `java.lang.Class` 类，是对类的抽象和集合，是类的字节码文件描述对象。Class 类的特点如下：
&emsp; &emsp;&emsp; ● 自定义的类在编译后会<font color=green>**生成一个唯一的 Class 对象**，***Class* 对象保存在与自定义类同名的 .class 文件中**</font>。
&emsp;&emsp;&emsp;  ●<font color=green> **无论创建多少个自定义类的对象，有且只有一个 Class 对象**</font>，表示自定义类的类型信息。
&emsp; &emsp;&emsp; ● Class 类没有公共的构造方法，<font color=green>**仅在类加载的过程中，由 jvm 自动构造，因此不能显式的声明一个 Class 对象**</font>。
![[../picture/Pasted image 20240106220329.png#pic_center|500]]
><font color=SlateBlue>  <u>**Q1. Class类的作用 ？**</u></font>
&emsp;&emsp;&emsp; 在C++中有个重要的概念：<font color=red>**运行时类型识别(`RTTI`)**</font>，其作用是在运行时识别一个对象的类型和类的信息。java中同样存在`RTTI`，java中的`RTTI`实现有两种方式：
&emsp;&emsp; &emsp; ● <font color=green>在编译期已确定其所有对象的类型，这种方式需要在写程序的时候将对象通过 new 创建出来</font>。
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
&emsp; &emsp; &emsp;  ● **Constructor** 类：代表某个类中的一个构造方法 
&emsp; &emsp; &emsp; ● **Method** 类：代表某个类中的一个成员方法 
&emsp; &emsp;  &emsp; ● **Field** 类：代表某个类中的一个成员变量
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
&emsp; &emsp;&emsp; ② **线程本地存储**：如果我们把共享数据的处理代码能够保证在一个线程中执行，即<font color=red>**把共享数据的可见范围限制在同一个线程之中，共享变量数据在每个线程中都有一个副本，每个线程仅操作副本当中的共享变量数据，那么这样也可以避免共享数据在不同线程操作所导致的线程安全问题**</font>。在Java中可以通过 ***ThreadLocal*** 类实现线程的本地存储功能。每个 Thead 线程对象中都有一个 *ThreadLocalMap* 对象，*ThreadLocalMap* 中 存储了多个以  threadLocalHashCode 为 Key，本地变量为值的为 Value 的数据。TheadLocal 对象就是 ThreadLocalMap 的访问入口。
&emsp; &emsp;&emsp; 线程安全的本质是为了保证线程数据的安全，在Java中可以使用 ThreadLocal  维护变量，从而可以不再使用锁，同步器等实现线程的同步。

![[../picture/Pasted image 20240121163427.png#pic_center|650]]

##### 1. ThreadLocal - 任务实体中的"共享/全局"变量
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

##### 2. InheritableThreadLocal ( ITL )
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
>
![[../picture/Pasted image 20240121165120.png#pic_center|650]]

##### <font color=Sienna>3. TransmittableThreadLocal (TTL)</font>
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

### 2.7 Java 线程锁机制
&emsp;&emsp; Java 内存模型在线程并发过程中存在三大特性：<font color=red>**原子性、可见性、有序性**</font>。 这个三大特性仅在多线程并发时会体现。在线程并发的过程中，首先要保证并发的正确性，在正确性的基础上实现高效并发。
&emsp;&emsp;&emsp; ① **原子性**：是指在一个操作中就是 CPU 不可以在中途暂停然后再调度，即不被中断操作，要不全部执行完成，要不都不执行。在程序中原子性指的是最小的操作单元，比如自增操作，它本身并不是原子性操作，分了3步，包括 (1).读取变量的原始值、(2).进行加1操作、(3).写入工作内存。所以在多线程中，有可能一个线程还没自增完，可能才执行到第二步，另一个线程就已经读取了值，导致结果错误。
&emsp;&emsp;&emsp; ② **可见性**：当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。
&emsp;&emsp;&emsp; ③ **有序性**：程序执行的顺序按照代码的先后顺序执行。

&emsp;&emsp;由于线程创建和运行时，必须通过 Runnable 来定义需要执行的任务，当在 `Runnable -> run(...)` 方法中定义了一个线程的局部变量，且有多个 ( >2个 ) Thread 启动并运行该 Runnable 任务时，会出现线程安全问题。线程安全问题的本质原因是<font color=red>由于一个共享数据在多线程并发过程中，由于逻辑运算的非原子性 ( 逻辑运算的非原子性是指一个线程在执行过程中，其CPU的执行时间片会进行重新调度，导致当前线程执行过程中断 )，导致并发计算的结果与代码逻辑结果不一致的问题，即出现线程安全问题</font>。为了解决这个问题，Java提供了同步器来保证线程运行时顺序和同步。
#### 2.7.1 线程安全程度分类
&emsp;&emsp;按照线程安全的安全程度由强至弱进行排序，可以将 Java 中各种操作共享的数据分为5类：**不可变数据、绝对线程安全数据、相对线程安全数据、线程兼容数据、线程对立数据**。
![[../picture/Pasted image 20250914213903.png#pic_center|670]]
 &emsp;&emsp;**① 不可变数据**：不可变数据的对象一定是线程安全的，无论是对象方法实现还是方法调用者，都不需要采取任何的线程安全保障措施。在Java中，如果共享数据是基本数据类型，只要在定义时使用 final 修饰该数据变量就可以保证该数据变量是不可变的。如果共享数据是一个对象，需要保证对象的行为不会对其状态 ( 原有变量值 ) 产生任何影响 ( 如 String 对象，调用其 `replace()` , `substring()` 方法不会影响其原有的值 )。
&emsp;&emsp;&emsp;**② 绝对线程安全数据**：绝对线程安全是指不管运行时环境如何排列，调用者和线程都不需要任何额外的同步措施，该类的对象被多个线程访问时仍然有效。要实现绝对线程安全，其所需的代价是很大的。
&emsp; &emsp;&emsp;**③ 相对线程安全数据**：相对线程安全，即通常意义上说的线程安全类。它保证了对某个对象的单独调用和操作是线程安全的，但是对于一些特定顺序的连续调用，需要在调用时通过额外的同步方式来保证调用的正确性。
```java
1. 虽然Vector本身是线程安全的，但对于下面的操作，如果没有synchronized同步块，可能会出现thread2线程调用了thread1线程刚刚删除的元素，导致访问vector时抛出ArrayIndexOutOfBoundsException错误。
2. 通过添加synchronized同步块，保证了thread1和thread2两个线程同一时刻只能有一个线程可以获得访问vector数据的同步锁，这样就避免了线程安全问题。
3. 由于在操作vector时，由于其操作顺序问题，需要增加额外的同步措施，因此vector是相对线程安全数据。
public class Main {
    private static Vector<Integer> vector = new Vector<>();
    public static void main(String[] args) throws InterruptedException {
        while(true){
            for(int i=0;i<30;++i){  vector.add(i); }
            Thread thread1 = new Thread(() -> {
                synchronized (vector){
                    for(int i=0;i<vector.size();++i){
                        vector.remove(i);
                    }
                }
            });

            Thread thread2 = new Thread(() -> {
                synchronized (vector){
                    for(int i=0;i< vector.size();++i){
                        System.out.println(vector.get(i));
                    }
                }
            });
            thread1.start();
            thread2.start();

            while(Thread.activeCount() > 20);
        }
    }
}
```
&emsp; &emsp;**④ 线程兼容**：线程兼容是指对象本身并不是线程安全的，但可以通过对象调用端使用同步措施来保证对象在并发环境中是线程安全的。Java API 中大部分的类都是线程兼容的。
&emsp; &emsp;&emsp; **⑤ 线程对立**：线程对立是指无论对象调用端采取何种同步措施，都无法在多线程环境中并发使用该对象。如 Thread 类的 `suspend()`  和 `resume()` 方法，如果有两个线程同时持有一个线程对象，一个尝试中断线程，另一个尝试恢复线程，如果并发进行的话，无论调用是否进行了同步，都存在死锁风险。

#### 2.7.2 Java 锁分类
&emsp;&emsp;要解决线程安全问题，就是要避免在多线程并发过程中，多个线程对同一个共享变量数据进行逻辑运算操作。Java提供了种类丰富的锁，每种锁因其特性的不同，在适当的场景下能够展现出非常高的效率。

![[../picture/Pasted image 20251130172711.png#pic_center|800]]

##### 1. 悲观锁 vs 乐观锁
&emsp;&emsp;悲观锁与乐观锁是从线程同步的不同角度来看待。在应用层面会造成线程阻塞的是悲观锁，而不会造成线程阻塞的是乐观锁。

 **▧ 悲观锁**
 
&emsp;&emsp;悲观锁是一种基于悲观态度的数据并发控制机制，用于防止数据冲突。对于同一个数据的并发操作，悲观锁认为自己在使用数据的时候一定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改。在悲观锁的机制下，当一个使用者要修改某个数据时，首先会尝试获取该数据的锁。如果锁已经被其他使用者持有，则当前使用者会被阻塞，直到对应的锁被释放。在Java中，常见的悲观锁实现是使用 `synchronized` 关键字或 `ReentrantLock` 类。这些锁能够确保同一时刻只有一个线程可以访问被锁定的代码块或资源，其他线程必须等待锁释放后才能继续执行。
- **悲观锁的优缺点如下**：
	- **优点**：
    - **数据一致性高**：悲观锁认为冲突一定会发生，因此在数据处理前会先加锁，这样可以确保数据在任一时刻只被一个事务访问和修改，从而避免数据的不一致性和脏读。
    - **简单易用**：悲观锁的实现相对简单，只需要在操作数据前获取锁即可。
	- **缺点**：
		- **性能开销大**：悲观锁在操作数据前需要获取锁，如果有大量的并发操作，可能会导致性能问题，因为其他事务需要等待锁释放。
		- **容易造成死锁**：如果多个事务相互等待对方释放锁，可能会导致死锁的发生，影响系统的稳定性和可用性。
		- **可能导致资源浪费**：如果获取锁后长时间不释放，可能会导致其他事务无法操作数据，从而造成资源浪费。


![[../picture/Pasted image 20251130223626.png#pic_center|950]]

&emsp;&emsp;悲观锁适用的场景：
&emsp;&emsp;&emsp; ① **高并发且数据竞争激烈的场景**：当多个事务需要同时访问和修改同一份数据时，使用悲观锁可以确保数据在任一时刻只被一个事务访问和修改，从而避免数据的不一致性和脏读。
&emsp;&emsp;&emsp; ② **数据一致性要求极高的场景**：在金融、医疗等行业中，对数据的一致性要求非常高，不允许出现任何的数据不一致或脏读现象。在这些场景中，使用悲观锁可以确保数据在任一时刻只被一个事务访问和修改，从而满足数据一致性的要求。
&emsp;&emsp;&emsp; ③ **写操作频繁的场景**：如果系统中写操作 ( 如更新、删除等 ) 远多于读操作，那么使用悲观锁可以更有效地保护数据，避免在写操作时被其他事务干扰。
&emsp;&emsp;&emsp; ④  **事务执行时间较长的场景**：当事务的执行时间较长时，使用悲观锁可以确保在该事务执行期间，数据不会被其他事务修改，从而避免数据的不一致性和脏读。

 **▧ 乐观锁**
 
&emsp;&emsp;&emsp;乐观锁是基于冲突检测 ( 版本控制 )的乐观并发机制，认为自己在使用数据时不会有别的线程修改数据，所以不会添加锁，只是在更新数据的时候会进行版本比对，判断之前有没有别的线程更新了这个数据。如果这个数据没有被更新，当前线程将自己修改的数据成功写入。如果数据已经被其他线程更新，则根据不同的实现方式执行不同的操作( 如报错或重试 )。乐观锁不需要将线程阻塞，从而不需要在用户态与内核态之间切换，提高系统效率。在 Java 中，常见的乐观锁实现是使用`Atomic`类，例如`AtomicInteger`、`AtomicLong`、`AtomicReference`等原子类。这些类提供了原子操作，可以确保对共享资源的更新操作是原子性的，从而避免了锁的开销和线程等待。
- **乐观锁的优缺点如下**：
	- **优点**：
	    - **高并发高吞吐**：乐观锁不会阻塞其他事务的读取操作，只在提交时检查数据是否被修改，因此可以提供更好的并发性能。
	    - **无锁操作**：乐观锁不需要显式地获取和释放锁，减少了锁竞争和上下文切换的开销。
	    - **无死锁风险**：由于乐观锁不会阻塞其他事务的访问，因此不会出现死锁的情况。
	- **缺点**：
		- **冲突处理复杂**：由于乐观锁不会阻塞其他事务，因此在提交时需要检查数据是否被其他事务修改，如果发现冲突，需要回滚事务或重新尝试操作，这增加了冲突处理的复杂性。
		- **数据一致性风险**：乐观锁假设并发冲突较少，因此可能存在数据一致性的风险。如果多个事务同时对同一数据进行修改，可能会导致数据不一致的情况。
		- **需要额外字段**：为了实现乐观锁，通常需要在数据表中添加额外的版本号或时间戳字段，这增加了存储空间的需求。
		- **处理不当造成死循环风险**：在大多数业务中乐观锁更新失败都会进行自旋，如果没有控制好自旋退出逻辑可能会造成递归死循环问题。

![[../picture/Pasted image 20251206151821.png#pic_center|750]]

&emsp;&emsp;乐观锁的适用场景：
&emsp;&emsp;&emsp;① **写操作较少**：在这种场景下，多个事务或线程大部分时间都在读取数据。乐观锁能够减少锁的持有时间，允许多个事务或线程同时读取数据，而不会相互阻塞。
&emsp;&emsp;&emsp;② **数据冲突较少**：如果数据更新操作之间的冲突较少，即多个事务或线程同时更新同一份数据的概率较低，那么乐观锁能够发挥很好的性能。因为即使偶尔出现冲突，也只是在更新数据时才会被检测到，而不需要在整个数据处理过程中都锁定资源。
&emsp;&emsp;&emsp;③ **重试成本较低**：乐观锁在检测到冲突时会回滚事务或提示冲突，需要客户端重新尝试更新操作。因此，如果重试的成本较低（例如，重试不会导致大量计算或I/O操作），那么使用乐观锁是合适的。
&emsp;&emsp;&emsp;④ **系统能够容忍一定程度的失败**：由于乐观锁在更新数据时可能会因为版本冲突而失败，因此系统需要能够处理这种失败情况。如果系统能够容忍一定程度的失败（例如，通过重试或其他补偿机制来恢复），那么使用乐观锁是可行的。

##### 2. 自旋锁 vs 适应性自旋锁 (线程获取锁失败的状态)
 **▧ 自旋锁**
 
&emsp;&emsp; 在互斥同步过程中，对共享变量数据的锁定和解锁操作会使得线程阻塞，挂起，而这些操作需要转入到内核态中完成，这种状态转换需要耗费处理器时间。如果同步代码块中的内容过于简单，这就导致操作系统内核状态的切换时间大于共享数据逻辑运算的时间，给并发性能带来了很大的开销。因此提出了自旋锁，在多线程并发过程中，让请求锁的线程请求失败后”自旋“等待一段时间，而不是马上进入线程阻塞状态，从而避免切换线程的开销。

![[../picture/Pasted image 20251207220215.png#pic_center|730]]

&emsp;&emsp; 自旋锁虽然避免了线程切换的开销，但是自旋锁在”自旋“过程中是要占用处理器时间的。如果锁被占用的时间很长，那么自旋的线程会白白消耗处理器的资源，会带来性能上的浪费，所以，自旋等待的时间必须要有一定的限度，如果自旋超过了限定次数没有成功获得锁，就应当挂起线程 ( 默认是10次，可以使用 `-XX:PreBlockSpin` 来更改 )。自旋锁在JDK1.4.2中引入，使用 `-XX:+UseSpinning` 来开启。
```java
//实现一个可重入的自旋锁
public class ReentrantSpinLock  {
    private AtomicReference<Thread> owner = new AtomicReference<>();  //通过原子引用避免线程的竞争
    private int count = 0;					//锁重复次数
    public void lock() {
        Thread t = Thread.currentThread(); //获取当前线程
        if (t == owner.get()) {				//由于通过原子引用，避免了线程的竞争，因此只有线程持有者才能加锁
            ++count;
            return;
        }
        while (!owner.compareAndSet(null, t)) {    //自旋获取锁
            System.out.println("自旋中...ß");
        }
    }
    public void unlock() {
        Thread t = Thread.currentThread();
        if (t == owner.get()) {		//只有持有锁的线程才能解锁
            if (count > 0) {
                --count;
            } else {
                owner.set(null);  //释放锁，此处无需CAS操作，因为没有竞争，只有线程持有者才能解锁
            }
        }
    }
 
    public static void main(String[] args) {
        ReentrantSpinLock spinLock = new ReentrantSpinLock();
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + "开始尝试获取自旋锁");
                spinLock.lock();
                try {
                    System.out.println(Thread.currentThread().getName() + "获取到了自旋锁");
                    Thread.sleep(4000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    spinLock.unlock();
                    System.out.println(Thread.currentThread().getName() + "释放了了自旋锁");
                }
            }
        };
        Thread thread1 = new Thread(runnable);
        Thread thread2 = new Thread(runnable);
        thread1.start();
        thread2.start();
    }
}
```

 **▧ 适应性自旋锁**
 
&emsp; &emsp;为了避免自旋锁自旋时间过长导致CPU资源的浪费，引入了如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也是很有可能再次成功，进而它将允许自旋等待持续相对更长的时间。如果对于某个锁，自旋很少成功获得过，那在以后尝试获取这个锁时将可能省略掉自旋过程，直接阻塞线程，避免浪费处理器资源。。自适应意味着自旋的时间 ( 次数 ) 不再固定，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。
&emsp; &emsp; &emsp; ● 如果在同一锁对象上，上一”阻塞“的线程在自旋等待后成功获得过锁，且当前持有锁的线程正在运行中，则虚拟机会认当前"阻塞"的线程通过自旋等待能够再次成功获得锁。
&emsp; &emsp; &emsp;  ● 如果线程自旋很少成功获得锁，则虚拟机为了避免自旋浪费 CPU 资源会省略掉自旋过程，直接将线程阻塞挂起。

##### 3. 公平锁 vs 非公平锁 (线程获取锁的顺序)
 **▧ 公平锁**
 
&emsp;&emsp;公平锁是指多个线程按照申请锁的顺序来获取锁，线程直接进入等待队列中排队，只有队列中的第一个线程才能获得锁。公平锁的优点是等待锁的线程不会饿死，不会产生死锁，缺点是整体吞吐效率相对非公平锁要低，等待队列中除第一个线程以外的所有线程都会阻塞，CPU唤醒阻塞线程的次数和开销比非公平锁大。

![[../picture/Pasted image 20251207221705.png#pic_center|550]]

 **▧ 非公平锁**

&emsp;&emsp;非公平锁是多个线程加锁时首先尝试获取锁，获取不到才会到等待队列的队尾等待。如果线程在获取锁的同时，锁刚好可用，那么这个线程可以无需阻塞直接获取到锁，所以非公平锁有可能出现后申请锁的线程先获取锁的场景。非公平锁的优点是可以减少唤起线程的开销，整体的吞吐效率高，因为线程有几率不阻塞直接获得锁，CPU 不必唤醒所有线程。缺点是处于等待队列中的线程可能会饿死 ( 由于竞争导致线程死锁 )，或者等很久才会获得锁。

![[../picture/Pasted image 20251207222839.png#pic_center|750]]

##### 4. 可重入锁 vs 非可重入锁 (同一资源,同一线程)
**▧ 可重入锁**

&emsp;&emsp;可重入锁又名递归锁，是指以线程为单位，在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁 ( 锁对象必须是同一个对象或同一个Class )，不会因为外层方法已经获取过还没释放而阻塞。Java中 ReentrantLock 和 synchronized 都是可重入锁，<font color=green>**可重入锁可一定程度避免死锁**</font>。
&emsp;&emsp;&emsp;● **加锁**: 可重入锁中都维护了一个同步状态 status 来计数重入次数，status 初始值为0。当线程尝试获取锁时，可重入锁先尝试获取并更新 status 值，如果 `status = 0` 表示没有其他线程在执行同步代码，则把 status 置为1，当前线程开始执行。如果 `status != 0`，则判断当前线程是否是获取到这个锁的线程，如果是的话执行 `status+1`，且当前线程可以再次获取锁。
&emsp;&emsp;&emsp;● **解锁**: 释放锁时，可重入锁同样先获取当前 status 的值，在当前线程是持有锁的线程的前提下。如果 `status - 1 = 0`，则表示当前线程所有重复获取锁的操作都已经执行完毕，然后该线程才会真正释放锁。

![[../picture/Pasted image 20251213151729.png#pic_center|680]]

```java
public class Widget {
    //因为synchronized是可重入的，所以同一个线程在调用doOthers()时可以直接获得当前对象的锁，进入doOthers()进行操作。
  	//如果是一个不可重入锁，那么当前线程在调用doOthers()之前需要将执行doSomething()时获取当前对象的锁释放掉，实际上该对象锁已被当前线程所持有，且无法释放。所以此时会出现死锁。
    public synchronized void doSomething() {
        System.out.println("方法1执行...");
        doOthers();
    }
    public synchronized void doOthers() {
        System.out.println("方法2执行...");
    }
}
```
**▧ 非可重入锁**

&emsp;&emsp; 不可重入锁是一种不支持重进入的锁机制。当一个线程获得了不可重入锁之后，如果该线程再次尝试获取锁，就会被阻塞，直到当前持有锁的线程释放锁。不可重入锁在 Java 中没有内置的实现，需要通过自定义实现或基于 AQS 等基础类来构建。不可重入锁可能会导致死锁问题，因为如果一个线程在持有锁的情况下又尝试获取同一个锁，就会导致自己无限等待。
&emsp; &emsp;&emsp;  ● **加锁**: 非可重入锁中也维护了一个同步状态 status ，但非可重入锁是直接去获取并尝试更新当前 status 的值，如果 `status != 0` 的话会导致其获取锁失败，当前线程阻塞。
&emsp; &emsp;&emsp;  ● **解锁**: 非可重入锁在确定当前线程是持有锁的线程之后，直接将 status 置为0，将锁释放。

![[../picture/Pasted image 20251213164227.png#pic_center|680]]

##### 5. 独占锁 vs 共享锁 (同一资源,不同线程)
&emsp;&emsp; 独占锁和共享锁通常是对于共享资源的获取方式的描述，是对于不同线程，同时获取同一共享资源的获取方式的区分。

**▧ 独占锁**

&emsp;&emsp; 独占锁也叫排他锁、互斥锁、独享锁，是指锁在同一时刻只能被一个线程所持有。一个线程加锁后，任何其他试图再次加锁的线程都会被阻塞，直到持有锁线程解锁。通俗来说，就是共享资源某一时刻只能有一个线程访问，其余线程阻塞等待。如果是公平地独占锁，在持有锁线程解锁时，如果有一个以上的线程在阻塞等待，那么最先抢锁的线程被唤醒变为就绪状态去执行加锁操作，其他的线程仍然阻塞等待。在 Java 中的 Synchronized 内置锁、ReentrantLock 显式锁、ReentrantReadWriteLock 中的写锁都是独占锁。**独占锁适用于需要对数据进行修改的场景**。

![[../picture/Pasted image 20251213174800.png#pic_center|500]]

**▧ 共享锁**

&emsp;&emsp; 共享锁是指该锁可被多个线程所持有。如果线程T对数据A加上共享锁后，则其他线程只能对A再加共享锁，不能加排他锁。获得共享锁的线程只能读数据，不能修改数据。在 Java 中典型的共享锁是 ReentrantReadWriteLock 中的读锁。**共享锁适用于只需要读取数据的场景。**

![[../picture/Pasted image 20251213175611.png#pic_center|500]]

#### 2.7.3 Java 锁类型
&emsp; &emsp;Java 的锁分为两类：
&emsp;&emsp;&emsp; **① 第一类是 synchronized 同步关键字**，该关键字属于隐式锁，是 JVM 层面实现，使用的时候看不见(对开发者来说是“透明”的)；
&emsp;&emsp;&emsp;  **② 第二类是在 JDK5 后增加的 Lock 接口以及对应的各种实现类**，这属于显式的锁，就是我们能在代码层面看到锁这个对象，而这些个对象的方法实现，大都是直接依赖 CPU 指令的，无关 JVM 的实现。显式锁最常用的是 ReentrantLock。

![[../picture/Pasted image 20250927161504.png#pic_center|500]]

##### 1. synchronized 隐式锁
&emsp; &emsp; Java 中最基本的互斥同步方式就是 synchronized 关键字。 synchronized 同步块对同一个线程来说是可重入的，不会出现自己锁自己的问题。synchronized 关键字无论是修饰代码块，还是修饰实例方法和静态方法，<font color=red>**本质上都是作用于对象上**</font>。<font color=green>synchronized 同步块对于已进入的线程执行完成之前，会阻塞后面其他线程的进入。</font>在阻塞或唤醒一个线程时，需要进入操作系统的内核态，状态的转换需要消耗很多的CPU处理时间。在 JDK 1.6 之后，在使用 synchronized 时对其进行了优化，如在通知操作系统阻塞线程之前，加入一段自旋等待过程，避免频繁的进入内核态。

###### (1). synchronized 原理
&emsp; &emsp;synchronized 是悲观锁，在操作同步资源之前需要给同步资源先加锁，而这把锁就是存在<font color=red> Java 对象头</font>里的。在 Java 的对象模型中，<font color=green>所有对象的对象头都有锁状态标记</font>。偏向锁、轻量锁、重量锁都在 Mark Word 中都有锁标记或锁的地址，即<font color=red>每个Java对象都可以关联一个 Monitor 对象</font>。当使用 synchronized 给对象上锁（重量级）之后，该对象头的 Mark Word 就被设置指向 Monitor 对象的指针。在 synchronized 在编译之后，会在所指定的对象引用的字节码前后分别加上 monitorenter 和 monitorexit 字节码指令，用于启动和关闭 Monitor。

![[../picture/Pasted image 20231225215727.png#pic_center|1080]]

![[../picture/Pasted image 20251019104418.png#pic_center|700]]

- Monitor的基本工作机制如下：
	- **Step 1**： 当多个线程同时访问一段同步代码 `synchroniezd(obj)` 时，线程会进入 \_EntryList 队列中。
	- **Step 2**： 当 T2 线程获取到对象的 Monitor 后进入临界区域，并把 Monitor 中的 \_owner 变量设置为当前线程，同时 Monitor 中的计数器 \_count 加1，即获得对象锁。此时共享对象的对象头 MarkWord 的锁标志位被设置为01，当别的线程执行 `synchroniezd(obj)` 这段代码时，它会先判断 obj 对象是否有关联 Monitor 锁，如已经关联 Monitor 锁，再判断这把锁是否 \_owner != Null，Monitor 中只能有一个 \_owner。如 \_owner != Null，此时将 \_EntryList 中的线程都会阻塞等待锁释放。
	- **Step 3**： 若持有 Monitor 的线程调用 wait() 方法，将释放当前持有的 Monitor，\_owner变量恢复为 Null，计数器 \_count自减1，同时该线程进入 \_WaitSet 集合中等待被唤醒。
	- **Step 4**： 如果是对象锁的 \_owner 拥有者线程再次获取锁时，由于 synchronized 锁是可重入的，此时进行 count++，而不是在 \_EntryList 队列中等待锁；每个加锁代码块运行完成或因发生异常退出时 count-- ，当 count=0 时，对象锁的拥有者线程释放锁。
	- **Step 5**： 在 \_WaitSet 集合中的线程会被再次放到 \_EntryList 队列中，重新竞争获取锁。
	
	- **Step 6**： 若当前线程执行完毕，且 count == 0 时，会释放 Monitor 并复位变量的值 \_owner = Null，以便其他线程进入获取锁。

###### (2). synchronized 锁升级机制
&emsp;&emsp; 在 JDK1.6 之前，synchronized 实现线程同步都依赖于操作系统的互斥量 (Mutex) 实现，这种重量级锁会导致线程在用户态和内核态之间频繁切换，性能开销巨大。为了解决这个问题，JDK 1.6 为了提高锁的获取与释放效率对 synchronized 进行了优化，引入了**锁升级机制**，通过偏向锁、轻量级锁等优化手段，显著提升了并发性能。锁升级的核心思想是：**根据竞争情况动态调整锁的粒度，从无锁状态逐步升级到重量级锁**，既保证了线程安全，又最大化了性能。
&emsp;&emsp;&emsp;优化之后的 synchronized 的锁一共有4种状态，锁的级别从低到高依次是：<font color=red>**无锁、偏向锁、轻量级锁和重量级锁</font>**。四种锁的状态会随着竞争的情况逐渐升级，锁状态只能升级不能降级。

![[../picture/Pasted image 20251026192036.png#pic_center|780]]

▨ **无锁 - `| hashcode:25 | age:4 | biased_lock:0 | 01 |`**
&emsp;&emsp;&emsp;无锁状态是对象初始化后的默认锁状态，此时对象头中的 Mark Word 不包含任何锁信息，此时 Mark Word 状态为：偏向锁标志为0，锁标志位为01。在这种状态下，多个线程可以同时访问对象，不需要任何同步机制。

![[../picture/Pasted image 20251025224555.png#pic_center|700]]

▨ **偏向锁 - `| threadID:23 | epoch:2 | age:4 | biased_lock:1 | 01 |`**
&emsp;&emsp;&emsp; 偏向锁是为了解决在同一个线程连续多次获取同一锁的情况，降低不必要的同步操作开销。当一个线程首次获取锁时，JVM会将对象头中的 Mark Word 设置为偏向模式 (biased_lock = 1)，并记录当前线程ID。在偏向锁状态下，只有持有偏向锁的线程才能再次获取这个锁，且不会引起竞争。如果其他线程尝试获取这个锁，偏向锁就会升级为轻量级锁。
&emsp;&emsp;&emsp;**在偏向锁生效期间，除非有其他线程尝试获取该锁，否则持有偏向锁的线程不会主动释放锁**。当出现锁竞争时，原有的偏向锁持有线程会经历锁撤销过程。锁撤销过程发生在全局安全点，即在所有线程均停止执行字节码的时刻，JVM会暂停当前持有偏向锁的线程，检查锁对象的状态。如果发现持有偏向锁的线程不再活动或者锁确实处于被争夺状态，则会释放偏向锁。锁释放后会根据共享对象的被线程锁定的状态来决定撤销偏向锁后的共享对象状态：
&emsp; &emsp;&emsp; ● 如果对象未被线程锁定 ( 存在多线程竞争，但共享数据未被锁定 )，则共享对象状态恢复到未锁定状态 ( 标志位为01 )；
&emsp; &emsp;&emsp; ● 如果对象已经被某个线程锁定，则共享对象状态恢复到轻量级锁状态 ( 标志位为00 )。

![[../picture/Pasted image 20251026154440.png#pic_center|880]]

- **偏向锁优缺点：**
	- **优点**：对于没有或很少发生锁竞争的场景，偏向锁可以显著减少锁的获取和释放所带来的性能损耗。
	- **缺点**:
		- **额外存储空间**：偏向锁会在对象头中存储一个偏向线程ID等相关信息，这部分额外的空间开销虽然较小，但在大规模并发场景下，累积起来也可能成为可观的成本。

		- **锁升级开销**：当一个偏向锁的对象被其他线程访问时，需要进行撤销（revoke）操作，将偏向锁升级为轻量级锁，甚至在更高竞争情况下升级为重量级锁。这个升级过程涉及到CAS操作以及可能的线程挂起和唤醒，会带来一定的性能开销。

		- **适用场景有限**：偏向锁最适合于绝大部分时间只有一个线程访问对象的场景，这样的情况下，偏向锁的开销可以降到最低，有利于提高程序性能。但如果并发程度较高，或者线程切换频繁，偏向锁就可能不如轻量级锁或重量级锁高效。
		

▨ **轻量级锁 - `| ptr_to_lock_record:30 | 00 |`**
&emsp; &emsp;&emsp;当锁是偏向锁的时候，如果锁被另外的线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋 (不会阻塞线程) 的形式尝试获取锁。如果在这段时间内锁被释放了，那么这个线程就可以成功获取锁。如果自旋结束后锁仍然被持有，那么这个线程就会尝试将锁升级为重量级锁。
&emsp; &emsp;&emsp;**轻量级锁基于CAS操作，主要目的是在没有大量多线程竞争的前提下，减少重量级锁使用操作系统互斥量所产生的性能消耗**。

![[../picture/Pasted image 20251026170151.png#pic_center|950]]
- **轻量级锁的加锁 & 解锁流程：**
	- **加锁过程**：
		- **Step 1**：线程在执行同步代码块之前，JVM 会先在当前线程的栈帧中创建用于存储锁记录 Lock Record 的空间，并将对象头中的 Mark Word 复制到锁记录中，称为 Displaced Mark Word。

		-  **Step 2**：当前线程尝试使用CAS将对象头中的 Mark Word 替换为指向 Lock Record 的指针，并将 Lock Record 里的 owner 指针指向对象的 Mark Word。
			- 如果成功，当前线程获得锁，并将锁对象的 Mark Word 的锁标志位置为 "00"。
	
			- 如果失败，虚拟机首先会检查对象的 Mark Word 是否指向当前线程的栈帧，如果是就说明当前线程已经拥有了这个对象的锁，那就可以直接进入同步块继续执行，否则说明多个线程竞争锁。若当前只有一个等待线程，则该线程通过自旋进行等待。但是当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁升级为重量级锁，并将 Mark Word 的锁状态置为"10"，后续等待锁的线程也进入阻塞状态。
	- **解锁过程**：解锁时使用原子CAS操作将 Displaced Mark Word 替换回到对象头中。
		- 如果成功，则表示没有竞争发生。

		- 如果失败，表示当前锁存在竞争，在释放锁的同时，唤醒被阻塞的线程 (此时由于锁存在竞争，轻量级锁已经膨胀为重量级锁，所有等待锁的线程进入了阻塞状态)

 - **轻量级锁优缺点：**
	 - **优点**：
		 - **低开销**：轻量级锁通过CAS操作尝试获取锁，避免了重量级锁中涉及的线程挂起和恢复等高昂开销。

		 - **快速响应**：在无锁竞争或者锁竞争不激烈的情况下，轻量级锁使得线程可以迅速获取锁并执行同步代码块。
	 - **缺点**：
		 - **自旋消耗**：当锁竞争激烈时，线程可能会长时间自旋等待锁，这会消耗CPU资源，导致性能下降。

		 - **升级开销**：如果自旋等待超过一定阈值或者锁竞争加剧，轻量级锁会升级为重量级锁，这个升级过程本身也有一定的开销。

▨ **重量级锁 - `| ptr_to_heavyweight_monitor:30 | 10 |`**
&emsp; &emsp;&emsp;当轻量级锁的自旋尝试达到一定阈值，或者检测到多个线程竞争激烈时，JVM会将轻量级锁升级为重量级锁。升级过程中，会取消当前线程的自旋操作，并在对象头中设置重量级锁标志。在重量级锁状态下，线程在获取锁失败时会被操作系统挂起，放入到该对象关联的监视器 Monitor 的等待队列中，由操作系统进行线程调度，当锁被持有锁的线程释放时，操作系统会选择合适的线程将其唤醒并授予锁。

 - **重量级锁优缺点：**
	 - **优点**：
		 - **强一致性**：重量级锁提供了最强的线程安全性，确保在多线程环境下数据的完整性和一致性。

		- **简单易用**：`synchronized`关键字的使用简洁明了，不易出错。
	- **缺点**：
		- **性能开销大**：获取和释放重量级锁时需要操作系统介入，可能涉及线程的挂起和唤醒，造成上下文切换，这对于频繁锁竞争的场景来说性能代价较高。

		- **延迟较高**：线程获取不到锁时会被阻塞，导致等待时间增加，进而影响系统响应速度。

##### 2. Lock 显式锁 ( synchronized -> ReentrantLock )
&emsp;&emsp;在 Java 早期 JVM 只提供了 synchronized 这一种内置锁。synchronized 虽然用起来简单，但在某些场景下 synchronized 无法响应中断请求、无法尝试获取锁、不支持线程公平性选择、通知机制基于单一等待队列，难以实现等待线程的精准唤醒。因此在 JDK 1.5 后，JUC 新增显式锁 Lock。
&emsp;&emsp;&emsp;  Lock 虽然缺少了 synchronized 同步块隐式获取和释放锁的便捷性，但可以进行非阻塞的、可轮询的、定时的、可中断的锁获取和释放操作等 synchronized 没有的同步特性。Lock 是基于 Java 实现的锁，因此所有加锁和解锁的方法都是通过显式调用的方式来实现。<font color=green>Lock 锁的底层是由 CAS 理论实现的，CAS 本身是自旋锁，由于自旋锁锁不需要像操作系统申请锁资源，所以线程不会进入阻塞状态，所以 Lock 锁的效率要比 Synchronized 高很多。</font>

###### (1).Lock 理论 - CAS 无锁算法
&emsp;&emsp; Lock 锁的底层是由 CAS 实现。CAS 算法是一种原子操作机制，它是许多并发工具和数据结构的底层实现基础。CAS 算法全称为 Compare And Swap，即比较并交换，是一种乐观锁机制，其核心思想是 "先比较，后操作"。与传统的互斥锁（悲观锁）不同，<font color=green>CAS 不需要阻塞线程，而是通过不断重试的方式实现原子操作，从而提高并发性能。</font>
&emsp;&emsp;&emsp; CAS 算法公式为 `CAS(V,A,B) = V`，涉及到3个操作符：变量的内存地址 (或偏移量 valueOffset ) V 、预期值 A (内存地址 V 中存放的数值) 、更新值 B。当 CAS 指令执行时：当且仅当对象偏移量 V 上的值和预期值 A 相等时，才会用更新值 B 更新 V 地址内存上的值，否则不执行更新。但无论是否更新了 V 内存上的值，最终都会返回 V 内存上的旧值。对于 JVM 来说，CAS 算法的本质是一条原子的CPU指令 `Atomic::cmpxchg(x, addr, e) == e`。

![[../picture/Pasted image 20251102203614.png#pic_center|700]]

```java
// AtomicInteger类
public final int incrementAndGet() {
  return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
// Unsafe.java
public final int getAndAddInt(Object o, long offset, int delta) {
   int v;
   //循环获取给定对象o中的偏移量处的值v，然后判断内存值是否等于v。如果相等则将内存值设置为 v + delta，否则返回false，继续循环进行重试，直到设置成功才能退出循环，并且将旧值返回。
   do {
       v = getIntVolatile(o, offset);
   } while (!compareAndSwapInt(o, offset, v, v + delta));
   return v;
}
```

![[../picture/Pasted image 20251101225220.png#pic_center|680]]

&emsp;&emsp; CAS虽然很高效，但是它也存在三大问题：
&emsp;&emsp;&emsp;  ① **ABA问题**：CAS需要在操作值的时候检查内存值是否发生变化，没有发生变化才会更新内存值。但是如果内存值原来是A，后来变成了B，然后又变成了A，那么CAS进行检查时会发现值没有发生变化，但是实际上是有变化的。ABA 问题的解决思路就是在变量前面添加版本号，每次变量更新的时候都把版本号加一，这样变化过程就从“A－B－A”变成了“1A－2B－3A”。除此之外，JDK 从1.5开始提供了 AtomicStampedReference 类来解决 ABA 问题，具体操作封装在`compareAndSet()` 中。`compareAndSet()` 首先检查当前引用和当前标志与预期引用和预期标志是否相等，如果都相等，则以原子方式将引用值和标志的值设置为给定的更新值。
&emsp; &emsp; &emsp; ② **循环时间长开销大** : CAS操作如果长时间不成功，会导致其一直自旋，给CPU带来非常大的开销。
&emsp; &emsp; &emsp;  ③ **只能保证一个共享变量的原子操作**：对一个共享变量执行操作时，CAS能够保证原子操作，但是对多个共享变量操作时，CAS是无法保证操作的原子性的。JDK 从 1.5 开始提供了 AtomicReference 类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行CAS操作。

###### (2).Lock 基础 - AQS
&emsp; &emsp; AQS是 AbstractQueuedSynchronizer 的简称，AQS是一种基于模板方法模式，并提供了原子式管理同步状态、阻塞和唤醒线程功能以及队列模型的框架。AQS 提供了以下核心功能：
&emsp;&emsp;&emsp;  ① **资源状态管理 - state**：通过 `volatile int state` 一个整数表示同步状态，子类可以定义该状态的意义和操作方式。
&emsp;&emsp;&emsp;  ② **线程排队管理 - CLH 队列**：通过 FIFO 等待队列管理多个线程的竞争和等待。
&emsp;&emsp;&emsp;  ③ **线程的唤醒机制 - ConditionObject**：通过条件变量 Condition 来实现等待唤醒机制，并且支持多个条件变量。
&emsp;&emsp;&emsp;  ④ **提供模板方法**：AQS 提供了一系列模板方法，子类可以通过实现这些方法来定义具体的同步机制。

- AQS 包括三大核心组件：<font color=red>**同步状态 state**、**CLH 等待队列** 和 **ConditionObject 基于条件变量的线程等待/唤醒机制**。</font>
	  ![[../picture/Pasted image 20251115222350.png#pic_center|700]]
	  
	-  ① **同步状态 state**：AQS使用一个 voliate 修饰的 int 类型作为共享资源的同步状态 status，voliate 可以保证可见性但不保证原子性，因此需要使用 CAS 对该同步状态进行原子操作实现对其值的修改。
		- 对于独占锁，state 表示锁的持有状态 ( 0 表示未持有，1 表示持有 )。
		- 对于共享锁，state 表示当前持有的读锁数量。
	- ② **CLH 等待队列**：CLH 是 AQS 内部维护的基于链表数据结构的 FIFO 双端双向队列，用来管理处于等待状态的线程。
	  &emsp;&emsp;&emsp;当被请求的共享资源被占用时，就会将等待资源的线程封装成一个 Node 节点，每个 Node 节点都有两个指针，分别指向直接的后继节点和直接前驱节点，并通过 CAS 原子操作将 Node 节点插入队列尾部。CLH 队列是一个虚拟的双向队列 ( 虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系 )。
	  
	  ```java
	static final class Node {
		static final Node SHARED = new Node(); //当前节点处于共享模式的标记
		static final Node EXCLUSIVE = null;    //当前节点处于独占模式的标记
		static final int CANCELLED =  1;   //线程被取消
		static final int SIGNAL    = -1;   //head持有锁线程释放资源后需唤醒后继节点
		static final int CONDITION = -2;   //等待condition唤醒
		static final int PROPAGATE = -3;   //工作于共享锁状态，需要向后传播，
		volatile int waitStatus;   //等待状态，有1,0,-1,-2,-3五个值。分别对应上面的值
		volatile Node prev;      //前驱节点
		volatile Node next;      //后继节点
		volatile Thread thread;  //当前线程 - 等待锁的线程
		Node nextWaiter; //存储在condition队列中的后继节点
		//是否为共享锁
		final boolean isShared() { 
			return nextWaiter == SHARED;
		}	
			
		final Node predecessor() throws NullPointerException {
			Node p = prev;
			if (p == null)		
				throw new NullPointerException();
			else
				return p;
		}
		
		//将线程构造成一个Node，添加到等待队列
		Node(Thread thread, Node mode) {     // Used by addWaiter
			this.nextWaiter = mode;
			this.thread = thread;
		}
		//这个方法会在Condition队列使用
		Node(Thread thread, int waitStatus) { // Used by Condition
			this.waitStatus = waitStatus;
			this.thread = thread;
		}
	}	
	  ```
	  
	- ③ **ConditionObject  基于条件变量的线程等待/唤醒机制**：
	  &emsp;&emsp;&emsp;AQS 的 ConditionObject 条件变量通过 `await()` 和 `signal()` 两类函数，模拟线程等待 wait 和唤醒 notify 的功能，实现了线程间同步协作。不同于 Synchronized 锁，一个 AQS 可以以对应多个条件变量，而 Synchronized 只有一个。ConditionObject 内部维护着一个单向条件队列。<font color=orange>当某个线程执行了 ConditionObject 的 `await()` 函数，会阻塞当前线程，线程会被封装成 Node 节点添加到条件队列的末端。当线程执行 ConditionObject 的 `signal()` 函数，会将条件队列头部线程节点转移到 CLH 等待队列参与竞争资源。</font>因此，条件队列只入队执行 await 的线程节点，并且加入条件队列的节点，不能在 CLH 队列， 条件队列出队的节点，会入队到 CLH 队列。


&emsp;&emsp;AQS同步器的设计是基于<font color=orange>模板⽅法模式</font>的，通过模板方法重写，可以实现⾃定义同步器。AQS 的类关系图如下图所示：

![[../picture/Pasted image 20251109223128.png#pic_center|890]]

&emsp;&emsp;AQS定义了两种资源共享方式：**Exclusive** ( 独占资源，只有一个线程能执行，如 ReentrantLock ) 和 **Share** ( 共享资源，多个线程可同时执行，如 Semaphore/CountDownLatch )。AQS的所有子类中，要么使用了它的独占方式，要么使用了共享方式，但不会同时使用独占和共享。实现 Lock 接口的类有很多，以下为几个常见的锁实现：
&emsp; &emsp; &emsp;  ① **ReentrantLock**：表示重入锁，它是唯一一个实现了 Lock 接口的类。重入锁指的是线程在获得锁之后，再次获取该锁不需要阻塞，而是直接关联一次计数器增加重入次数。
&emsp; &emsp; &emsp; ② **ReentrantReadWriteLock**：重入读写锁，它实现了 ReadWriteLock 接口，在这个类中维护了两个锁，一个是 ReadLock，一个是 WriteLock，它们都分别实现了 Lock 接口。读写锁是一种适合读多写少的场景下解决线程安全问题的工具，基本原则是： 读和读不互斥、读和写互斥、写和写互斥。也就是说涉及到影响数据变化的操作都会存在互斥。
&emsp; &emsp;&emsp;  ③ **StampedLock**： StampedLock 是 JDK8 引入的新的锁机制，是读写锁的一个改进版本，读写锁虽然通过分离读和写的功能使得读和读之间可以完全并发，但是读和写是有冲突的，如果大量的读线程存在，可能会引起写线程的饥饿。StampedLock 是一种乐观的读策略，使得乐观锁完全不会阻塞写线程。

###### (3).Lock 实现 - ReentrantLock
&emsp;&emsp;ReentrantLock 锁基于 AQS 实现的可重入锁，默认采用非公平锁。与 Sychronized 相比，ReentrantLock 有以下特点：
&emsp; &emsp;&emsp;  **① 可中断**：synchronized 只能等待同步代码块执行结束，不可以中断，而 ReentrantLock 可以调用线程的 interrupt 方法来中断尝试获取锁的线程，继续执行主线程代码。主要用于处理执行时间很长的同步块。
&emsp; &emsp;&emsp;  **② 可以设置超时时间**：调用 `lock.trylock()`，如果没有设置等待时间的话，没获取到锁，将返回 false。
&emsp; &emsp;&emsp;  **③ 可以设置为公平锁**：公平锁是指多个线程等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。公平锁是为了解决饥饿问题 (当一个线程由于优先级太低的时候，就可能一直获取不到时间片）。Synchronize 与 ReentrantLock 的锁都是非公平的，但 ReentrantLock 可以通过带布尔值的构造函数要求使用公平锁。
&emsp; &emsp;&emsp;  **④ 可以支持绑定多个条件变量**：类似于调用 wait 方法时，不满足条件的线程进入 waitset 队列等待CPU随机调度，支持多个变量表示支持多个自定义 waitset 队列 ，这样就可以指定对象来唤醒了。而 Synchronized 同步块对象的 `wait()`、`notify()`、`notifyAll()` 方法仅可以实现一个隐含条件，如果多个条件关联时，需要额外的添加锁。

![[../picture/Pasted image 20251122230224.png#pic_center|630]]

▨ **ReentrantLock 加锁与解锁过程**

&emsp; &emsp;  在加锁过程中，对比 FairSync 的 `tryAcquire()` 方法和 `nonfairTryAcquire()` 方法，可以看出唯一的区别就是 `nonfairTryAcquire()` 方法少了一个 `hasQueuedPredecessors()` 的判断，判断CLH队列中有没有线程在等待获取资源。而非公平锁就不管等待队列是否有等待线程，直接去CAS尝试获取资源。

![[../picture/Pasted image 20251122231259.png#pic_center|720]]

```java
ReentrantLock 类的相关方法：ReentrantLock 中的内部类 Sync、NonfairSync、FairSync 继承 AbstractQueuedSynchronizer 并重写指定的⽅法。
	isHeldExclusively()  //该线程是否正在独占资源。只有⽤到condition才需要去实现它。
	tryAcquire(int)			 //独占⽅式。尝试获取资源，成功则返回true，失败则返回false。
	tryRelease(int)      //独占⽅式。尝试释放资源，成功则返回true，失败则返回false。
	tryAcquireShared(int)//共享⽅式。尝试获取资源。负数为失败；0表示成功，但没有剩余资源；正数表示成功，且有剩余资源。
	tryReleaseShared(int)//共享⽅式。尝试释放资源，成功则返回true，失败则返回false。  
```

---

### 2.8 Java 代理模式
&emsp;&emsp; 代理是一种常用的设计模式，其目的就是为其他对象提供一个代理以控制对某个对象的访问。代理模式可以在不修改被代理对象的基础上，通过扩展代理类，进行一些功能的附加与增强。**在Java中，存在三种代理模式：<font color=red>静态代理(设计模式中介绍)，JDK 动态代理 (代理类是由JDK在运行时动态生成)，Cglib 代理。</font>**
#### 2.8.1 动态代理
&emsp;&emsp; 动态代理中的所谓 "动态"，是相对于使用 Java 代码实际编写静态代理而言，"动态" 的优势并不是省去了编写代理类的工作量，而是<font color=green>实现了可以在原始接口和原始类未知的时候，就确定了代理类的代理行为</font>，即动态代理类不再是针对某一特定的已知的类接口进行代理，当动态代理类与原始类"脱离直接联系"时，可以复用于其他不同的场景中。动态代理的作用如下：
&emsp;&emsp;&emsp; ● **增强对象的功能**：通过动态代理，可以在不修改原始对象的情况下，对其方法进行增强或添加额外的行为。可以在方法执行前后进行一些操作，比如日志记录、性能监测、事务管理等。
&emsp;&emsp;&emsp; ●  **解耦和业务逻辑分离**：动态代理可以将对象的特定操作从业务逻辑中解耦，使得代码更加模块化和可维护。代理对象可以负责处理一些通用的横切关注点，而业务对象可以专注于核心业务逻辑。
&emsp;&emsp;&emsp; ● **实现懒加载**：通过动态代理，可以延迟加载对象，只有在真正需要使用对象时才会进行创建和初始化，从而提高性能和资源利用效率。
&emsp;&emsp;&emsp; ● **实现AOP编程**：动态代理是实现面向切面编程(AOP)的基础。通过代理对象，可以将横切关注点（如日志、事务、安全性）与业务逻辑进行解耦，提供更高层次的模块化和可重用性。

&emsp;&emsp; 动态代理分为 **JDK 动态代理** (基于接口代理) 和 **Cglib 动态代理** (基于继承代理) 两种方式

><font color=SlateBlue><u>**Q1. 动态代理与静态代理的区别 ？**</u></font>
&emsp;&emsp;&emsp;① 加载被代理类的时机不同: 静态代理在编译时就已经实现，编译完成后代理类是一个实际的 *class* 文件。动态代理是在运行时动态生成的，即编译完成后没有实际的 *class* 文件，而是在**运行时动态生成类字节码，并加载到 JVM中**。
&emsp;&emsp;&emsp;② 静态代理代理对象要实现与目标(被代理)对象一致的接口，而<font color=green>**动态代理对象不需要实现接口，需要实现 InvocationHandler 接口，但目标(被代理) 对象必须实现接口**</font>，否则不能使用动态代理。
>
><font color=SlateBlue><u>**Q2. JDK 动态代理与 CGLIB 代理的区别 ？**</u></font>
>&emsp;&emsp;&emsp;**① JDK 动态代理**：适用于接口代理，且只能代理接口，不能代理普通类。JDK动态代理创建代理实例速度比CGLIB快，但由于方法调用是通过反射实现的，因此方法调用的性能相对较低。<font color=orance>**适合代理接口的方法调用频率不高的场景。**</font>
>&emsp;&emsp;&emsp;**② Cglib 动态代理**：Cglib 基于底层的字节码操作，通过生成目标类的子类，并在子类中拦截方法调用来实现代理。因此，Cglib 可以代理没有接口的普通类。由于 Cglib 在创建代理类时需要进行字节码操作，因此性能开销较大。但在代理类生成之后，代理类的方法调用性能比 JDK 代理类快，<font color=orance>**适合方法调用频率较高的场景**</font>。除此之外，Cglib局限性在于它是通过继承的方式来实现代理的，但是在日常开发中，会存在 final 修饰的情况，方法不能修改、变量不能修改、类不能被继承，也就意味着无法通过 Cglib 进行增强，故不支持final 修饰的场景。
>
>![[../picture/Pasted image 20250608223306.png#pic_center|380]]

##### 1. JDK动态代理 - 基于反射 & 接口方式

&emsp; &emsp; JDK 动态代理依赖于 Java 反射机制，通过`java.lang.reflect.Proxy`类和`java.lang.reflect.InvocationHandler`接口实现。JDK  动态代理只能代理实现了接口的类。在运行时，JDK动态代理会生成一个代理类，该代理类实现了目标接口，并将方法调用委托给`InvocationHandler`的`invoke`方法。每当对代理对象执行方法调用时，调用的方法不会直接执行，而是转发到实现了`InvocationHandler` 的 `invoke` 方法上。在这个 `invoke` 方法内部，我们可以定义拦截逻辑、调用原始对象的方法、修改返回值等操作。

![[../picture/Pasted image 20250607182037.png#pic_center|560]]

###### (1).JDK动态代理的实现原理
&emsp; &emsp;Java JDK 动态代理通过 Proxy 的静态方法 <font color=red>newProxyInstance()</font> 动态创建代理。其代理过程如下图所示：

```java
public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,InvocationHandler h)
// loader - 类加载器，用于加载被代理类到jvm
// interfaces - 要用来代理的接口,即被代理的类的接口
// InvocationHandler - 每个代理的实例都有一个与之关联的 InvocationHandler 实现类，如果代理的方法被调用，那么代理便会通知和转发给内部的 InvocationHandler 实现类，由它决定处理。
```

![[../picture/Pasted image 20240602173259.png]] 

&emsp; &emsp; ① `Proxy.newProxyInstance()` 方法是动态代理的入口，在第一次创建代理对象时，`Proxy.newProxyInstance()` 会从 proxyClassCache 缓存中检查是否有已经生成的代理类。
&emsp; &emsp; &emsp; ② 如果缓存中存在，则会使用缓存中的代理类。
&emsp; &emsp; &emsp; ③ 如果缓存中不存在，则最终调用 `ProxyGenerator.generateProxyClass()`方法来动态生成一个实现了所有给定接口的新类。这个新生成的类通常命名为 \$Proxy0，\$Proxy1 等，每个代理类对应一个唯一的代理实例。一旦生成，这个代理类就会被缓存，后续的相同接口的代理对象创建将直接复用这个类，而不再重新生成。

><font color=SlateBlue><u>**Q1. JDK 动态代理缓存为什么使用 WeakCache ？**</u></font>
>
&emsp; &emsp;WeakCache 是一个具有二级缓存的弱引用类，key 为一级缓存, value 为二级缓存。<font color=green>一级缓存的 key 是弱引用，二级缓存是强引用</font>。
>
&emsp;&emsp; &emsp; &emsp;`private final ConcurrentMap<Object, ConcurrentMap<Object, Supplier<V>>>  map = new ConcurrentHashMap<>();`
>
&emsp;&emsp;一级缓存的 key 为 ClassLoader 是根据入参直接传入的。由于不同 ClassLoader 加载的类是不同的，因此可以根据 ClassLoader 对缓存进行筛选一遍。二级缓存的 key 是用接口数组来生成，因为大部分类都是实现了一个或两个接口，所以二级缓存key分为 key0 ~ keyX。二级缓存的值是一个 Factory 实例，用来存放生成代理类的 Factory工厂 ，最终代理类的值是通过 Factory 这个工厂来获得的。
>
&emsp;&emsp;由于<font color=green> **JDK 动态代理生成的代理类占用内存较大，为了不影响GC对内存的回收**，JDK 动态代理使用 WeakCache 作为生成的动态代理类的缓存。</font>当弱引用被GC回收后，二级缓存强引用会被以惰性 ( lazily ) 方式被删除。

&emsp; &emsp; JDK 动态代理的使用案例如下：
```java
public static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces, InvocationHandler h)
  
// ============================================
// 1.Trains接口，抽象主题角色
public interface Trains {
    public void run();
}
// ============================================
// 2.Trains接口实现，被代理的角色
public class SlowTrains implements Trains {
    @Override
    public void run() {
        System.out.println("火车开车了");
    }
}
// ============================================
// 3.动态代理类，代理了被代理类，对被代理类进行了增强，通过实现 InvocationHandler 接口创建自己的动态代理器
public class Shop implements InvocationHandler {
    private Trains trains;  //引入要代理的对象
    public Shop(Trains trains){
        this.trains = trains;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("本商店代售火车票");
        method.invoke(trains,args);
        System.out.println("结束");
        return null;
    }
}
// ============================================
// 4. 主函数，动态代理的使用
	Trains trains = new SlowTrains();
	Trains trains1 = (Trains) Proxy.newProxyInstance(
    				SlowTrains.class.getClassLoader(), SlowTrains.class.getInterfaces(),new Shop(trains));
  trains1.run();
```

##### 2. Cglib 代理 - 基于字节码 & 继承方式
&emsp; &emsp; cglib 是一个强大的、高性能的**代码生成库**。cglib 代理代理为控制要访问的目标对象提供了一种途径，通过对**字节码**进行操作，当访问对象时，它引入了一个间接的层(代理层)，以控制对象的访问。**<font color=red>Cglib 的应用本质是为那些</font><font color=green>没有接口的类创建一个代理对象</font>**，从而实现对原有代码的增强，拦截等操作。Cglib 主要应用场景如下:
&emsp; &emsp; &emsp;  ① 广泛的应用于 AOP 的框架使用，例如：Spring AOP，为他们提供方法的 intercep (拦截器策略)。 
&emsp; &emsp; &emsp; ② Hibernate (ORM 持久层框架) 使用 cglib 来代理单端 single-ended (多对一、一对一)关联。 
&emsp; &emsp; &emsp; ③ EasyMock 和 jMock 是通过使用模仿(moke) 对象来测试 java 代码的包。

![[../picture/Pasted image 20240602231211.png#pic_center|380]]

###### (1).Cglib 的使用
&emsp;&emsp;▧ Cglib 动态代理主要包含以下几个类：
&emsp; &emsp; &emsp; ① **Enhancer类**：Cglib 的核心类之一，它可以动态地生成一个子类，并且拦截父类的所有方法。
&emsp; &emsp; &emsp; ② **Callback接口**：Cglib 的回调接口，在使用 Cglib 动态代理时，我们需要实现Callback 接口或其子接口，并将其设置到 Enhancer 对象中，以实现不同的拦截逻辑。
&emsp; &emsp; &emsp; ③ **MethodInterceptor接口**：Cglib 中的拦截器接口，是 Callback 接口的子接口。主要用于拦截目标对象的方法调用，并在调用前后进行一些处理。MethodInterceptor 接口中有一个`intercept()`方法，用于实现拦截逻辑。
```java
public Object intercept(Object obj, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {...}
// obj - 表示被代理来的目标对象，
// method - 表示被代理类的目标方法
// args - 表示被代理类的目标方法的参数
// proxy - 表示方法的代理对象
```
&emsp; &emsp; ④ **MethodProxy类**：为了实现对被代理类中方法的代理和拦截而设计的。 Cglib 通过生成代理类的方式来实现对被代理类中方法的代理和拦截，从而避免使用反射机制，提高了程序的性能。在代理类中，Cglib 会为每个方法生成对应的MethodProxy 对象，用于代理被代理类中的方法。通过调用 MethodProxy 的 `invoke()`方法，可以实现对被代理类中方法的调用。  另外，MethodProxy 还提供了一些辅助方法，用于获取方法信息和判断方法类型，方便实现具体的拦截逻辑。

&emsp;&emsp;▧ Cglib 的使用步骤如下：
&emsp; &emsp; &emsp; ① Cglib 通过`Enhancer`类创建一个代理对象，同时绑定方法拦截器。该代理对象是目标类的子类，因此可以继承目标类的所有方法和属性。
&emsp; &emsp; &emsp; ② 通过实现 MethodInterceptor 接口，定义一个方法拦截器 (又称为方法回调)，并在拦截器中实现代理逻辑。当代理对象调用目标方法时，Cglib 会首先调用拦截器的`intercept()`方法，然后在该方法中调用目标方法。
&emsp; &emsp; &emsp; ③ 最后，Cglib 会将增强后的方法返回给代理对象，从而实现对目标方法的动态代理。
```java
// ① 被代理类
public class Dog {
    public String call() {
        System.out.println("wang wang wang");
        return "Dog ..";
    }
 }

 // ② 通过实现 MethodInterceptor 接口，定义一个方法拦截器，并在拦截器中实现代理逻辑。不用依赖被代理业务类的引用
   /**
     * @param object      代表Cglib 生成的动态代理类 对象本身
     * @param method      代理类中被拦截的接口方法 Method 实例
     * @param args        接口方法参数
     * @param methodProxy 用于调用父类真正的业务类方法。可以直接调用被代理类接口方法
     * @return 被代理类方法执行返回值
     */
public class CglibMethodInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object object, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("before");
        Object result = methodProxy.invokeSuper(object, args);
        System.out.println("after");
        return result;
    }
}

// ③ 主方法，创建一个 Enhancer 对象，用于设置目标类并生成代理类。
public class TestMain {
    public static void main(String[] args) throws ClassNotFoundException {
        Enhancer enhancer = new Enhancer(); // 创建加强器，用来创建动态代理类
        enhancer.setSuperclass(Dog.class);  // 为代理类指定需要代理的类，也即是父类
        enhancer.setCallback(new CglibMethodInterceptor()); // 设置方法拦截器回调引用，对于代理类上所有方法的调用，都会调用CallBack，而Callback则需要实现intercept()方法进行拦截
        Dog poolDog = (Dog) enhancer.create(); // 创建cglib 代理类
        poolDog.call();   //调用代理类方法
    }
}
```
###### (2).Cglib 的实现原理
![[../picture/Pasted image 20250607215244.png#pic_center|500]]

&emsp; &emsp; Cglib 利用 ASM 技术基于继承子类实现动态代理，当需要代理一个类时，Cglib 会创建一个该类的子类，并在子类中重写需要代理的方法。Cglib 代理类的创建流程：

### 2.9 Java 集合容器
&emsp;&emsp;在Java中，数组不是面向对象的，存在明显的缺陷。集合弥补了数组的缺点，比数组更灵活更实用，而且不同的集合框架类可适用不同场合。Java集合类存放在 java.util 包中，是一个用来存放对象的容器。集合的特性主要有以下几点：
&emsp; &emsp;&emsp; ① 集合只能存放对象，如果存入一个基本数据类型，其会自动转换成包装类。 
&emsp; &emsp; &emsp;② 集合存放的都是对象的引用，而非对象本身，对象本身还是在堆内存当中。
><font color=SlateBlue><u>**Q1. 数组与集合有什么区别与相似之处</u></font>**
>&emsp;&emsp;&emsp; ① **内容区别**：数组能存放基本数据类型和对象，而集合类存放的都是对象，集合类不能存放基本数据类型。数组和集合存放的对象皆为对象的引用地址。
&emsp;&emsp;&emsp; ② **长度区别**：数组的长度在初始化时被固定而无法动态改变，集合类的容量是动态改变的。
&emsp;&emsp;&emsp; ③ **存在形式区别**：集合以类的形式存在，具有封装、继承、多态等类的特性，提高了软件的开发效率
#### 2.9.1 Java集合框架
![[../picture/Pasted image 20260102105047.png#pic_center|1200]]

#### 2.9.2 Collection 集合及其子类
&emsp; &emsp;Collection 接口是 java 集合类的顶级接口之一，包含了集合的基本操作和属性。 Collection 接口有3种子类型集合:  **List 、Set 和 Queue**。
```java
// Collection接口定义的基本方法如下：
1、添加方法
 boolean add(Object obj)	  	// 添加一个对象
 boolean addAll(Collection c) // 添加一个集合的对象
2、删除方法
 void clear() 								// 移除所有对象
 boolean remove(Object)				// 移除一个对象
 boolean removeAll(Collection c) // 移除一个集合的对象，只要有一个对象移除了，就返回true
3、判断方法
 boolean contains(Object o) 		   // 判断集合是否包含该对象
 boolean containsAll(Collection c) // 判断集合中是否包含指定的集合对象，只有包含所有的对象，才返回 true。
 boolean isEmpty() 								 // 判断集合是否为空。
4、获取方法
 Iterator<E> iterator() 					 // 迭代器
5、长度功能
 int size() 											 //	对象个数
6.交集功能
 boolean retainAll(Collection c) 	 // 移除此Collection中未包含在指定Collection中的所有对象
```

##### 1. List 集合
&emsp;&emsp;List\<T> 接口继承于 Collection 接口，它定义一个允许重复的有序集合。List\<T> 接口所代表的是有序 Collection，它用某种特定的插入顺序来维护元素顺序。可以对列表中每个元素的插入位置进行精确地控制，同时可以根据元素的整数索引(在列表中的位置)访问元素，并搜索列表中的元素。实现 List\<T> 接口的集合主要有: **ArrayList**、**LinkedList**、**Vector**、**Stack**。

###### (1). ArrayList
&emsp;&emsp;ArrayList 底层是通过 Object 对象数组的数据结构来实现的。<font color=green>ArrayList 初始化时，如果指定了容器大小，则按照指定的大小进行容器初始化，如果没有指定容器的大小，则会首先创建一个空的容器，当该容器第一次添加数据时，会设置容器的大小 Capacity = 10。</font>每当执行 `add()`,`insert()`等添加元素的方法，都会检查内部数组的容量是否足够。<font color=red>当容量不足时，它就会以当前容量的 1.5 倍来重新构建一个数组，将旧元素 Copy 到新数组中，然后丢弃旧数组</font>。ArrayList 的数组扩容是影响其效率的重要因素之一。例如一个有150个元素的数据动态添加到一个以默认10个元素大小创建的ArrayList中，将会经过10次的扩容才会满足最终的要求，那么如果一开始就以 `ArrayList List = new ArrayList(160);`的方式创建 ArrayList，不仅会减少10次数组创建和 Copy 的操作，还会减少内存使用。
![[../picture/Pasted image 20260102110847.png#pic_center|450]]

&emsp;&emsp; <font color=red>注：**ArrayList 实现不是线程安全的**</font>。如果多个线程同时访问一个 ArrayList 实例，而其中至少一个线程修改了列表，那么该列表必须保持外部同步。为了保证同步，最好的办法是在列表创建时完成，以防止意外对列表进行不同步的访问。`List list = Collections.synchronizedList(new ArrayList());`

```java
// ArrayList的遍历方式
ArrayList支持的4种遍历方式:
1. 通过迭代器遍历
Integer value = null;
Iterator iter = list.iterator();
while (iter.hasNext()) {
    value = (Integer)iter.next();
}

2. 随机访问，通过索引值遍历
for (int i=0; i<list.size(); i++) {
    value = (Integer)list.get(i);        
}

3. 通过for循环遍历
for (Integer integ:list) {
    value = integ;
}

4. 通过forEach + lambda 循环遍历,由于通过forEach进行遍历时，item为临时变量，因此不能对item进行修改操作
	 forEach只有在"只读方式"时才适用
list.forEach(item -> {
  item.hashCode();
});

// ArrayList的删除数据方式
虽然ArrayList有四种遍历方式，但是能够正确删除数据的方式只有两种：①通过迭代器进行删除；② 倒序循环删除
1. 通过迭代器删除数据
Iterator<String> iter = list.iterator();
while (iter.hasNext()) {
    iter.next().hashCode();
    iter.remove();
}

2. 倒序循环删除数据
for(int i = list.size()-1;i>=0;i--){
   list.remove(i);
}
```

###### (2). LinkedList
&emsp;&emsp; LinkedList 是一个继承于 AbstractSequentialList 的<font color=red>**双向链表**</font>，因此<font color=red>其顺序访问会非常高效，而随机访问效率比较低。</font>它也可以被当作堆栈、队列或双端队列进行操作。LinkedList  的数据结构如下图所示:
![[../picture/Pasted image 20260102111934.png#pic_center|550]]

&emsp;&emsp;虽然 LinkedList 是双向链表，但在 LinkedList 底层通过计数索引值建立了 "索引值与双向链表的关系"，使得 LinkedList 也可以像 ArrayList 一样根据索引来进行数据操作。当 LinkedList 查找索引对应的节点时，会首先根据入参的索引值与双向链表长度的1/2进行对比，小于1/2时，从头结点开始遍历，大于1/2时，从尾节点开始遍历。

```java
// LinkedList作为“栈”使用时的操作：
addFirst(e)，removeFirst()，peekFirst()
// LinkedList作为“队列”使用时的操作：
addLast(e)，offerLast(e)，removeFirst()，pollFirst()，getFirst()，peekFirst()
  
//LinkedList支持多种遍历方式：
1. 通过迭代器遍历。即通过Iterator去遍历。
for(Iterator iter = list.iterator(); iter.hasNext();)
    iter.next();

2. 通过快速随机访问遍历LinkedList
for (int i=0; i<list.size(); i++) {
    list.get(i);        
}

3. 通过forEach循环来遍历LinkedList
for (Integer item : list){}

4. 通过pollFirst()来遍历LinkedList
while(list.pollFirst() != null){}

5. 通过pollLast()来遍历LinkedList
while(list.pollFirst() != null){}

6. 通过removeFirst()来遍历LinkedList
try {
    while(list.removeFirst() != null) {}
} catch (NoSuchElementException e) {}

7. 通过removeLast()来遍历LinkedList
try {
    while(list.removeLast() != null) {}
} catch (NoSuchElementException e) {}
```

###### (3). Vector
&emsp;&emsp;Vector 底层是用数组实现的，其容量与 ArrayList 一样是可以动态扩展的，不同的是<font color=red> Vector 支持线程的同步，即某一时刻只有一个线程能够写 Vector，避免多线程同时写而引起的不一致性，所以 Vector 是线程安全的</font>。因为 Vector 类中每个方法中都添加了 synchronized 的关键字来保证同步，使得它的效率大大的降低了，比 ArrayList 的效率要慢，因此一般情况下都不使用 Vector 对象，而会选择使用 ArrayList。

##### 2. Set 集合
&emsp;&emsp; Set 继承于 Collection 接口，是一个不允许出现重复元素，并且无序的集合，主要有 **HashSet** (无序集合) 和 **TreeSet** (有序集合) 两大实现类。

###### (1). HashSet
&emsp;&emsp; HashSet 继承于 AbstractSet\<E> 抽象类，是一个没有重复元素的集合。<font color=red>HashSet 的底层数据结构是哈希表 HashMap，不保证元素的顺序，而且 HashSet 允许使用 null 元素。</font>HashSet 按 Hash 算法来存储集合中的元素，因此具有很好的存取和查找性能。HashSet的特点如下： 
&emsp; &emsp; ● HashSet 不能保证元素的排列顺序，顺序可能与添加顺序不同。
&emsp; &emsp; ● HashSet不是同步的，当多个线程同时访问一个HashSet必须保证外部同步，否则会存在线程安全问题。 
&emsp; &emsp; ● 集合元素值可以是null。
![[../picture/Pasted image 20260102174424.png#pic_center|480]]

&emsp; &emsp; 在 HashSet 插入对象的过程如下：当调用`add(object)`方法往集合里添加元素时，本质上调用的是 HashMap 的`put(Key,Value)`方法，将插入对象作为 Key，Value 为静态的Object 对象，HashMap 中所有节点的 value 均指向静态的 Object 对象，因此也不存在空间浪费的问题。

###### (2). TreeSet
&emsp; &emsp; TreeSet 是一种有序集合，它实现了 Set 接口，因此具有不允许重复元素的特性。与 HashSet 不同，TreeSet使用红黑树数据结构来存储元素，这使得元素在集合中保持有序。

### 2.10 Java 异常
 &emsp;&emsp;在程序设计中，进行异常处理是非常关键和重要的一部分。异常在程序编译期间和运行期间经常发生，在编译期间出现的异常有编译器在编译期间进行"拦截"。而运行期间的错误往往是难以预料的，假若程序在运行期间出现了错误，会使程序终止或导致系统崩溃。<font color=green>异常处理机制能让程序在异常发生时，按照代码的预先设定的异常处理逻辑，通过异常机制，我们可以更好地提升程序的健壮性。</font>Java异常机制的框架如下图所示:
![[../picture/Pasted image 20251227212946.png#pic_center|690]]

 &emsp;&emsp; 在Java中异常被当做<font color=red>对象</font>来处理，根类是`java.lang.Throwable`类，异常类分为两大类：Error 和 Exception： 
&emsp;&emsp;&emsp;● **Error** :  Error 指错误，对于所有的编译时期的错误以及系统错误都是通过 Error 抛出的。Error 表示故障发生于虚拟机自身、或者发生在虚拟机试图执行应用时。**Error 错误是不可查的**，因为它们在应用程序的控制和处理能力之外，且绝大多数是程序运行时不允许出现的状况。如Java虚拟机运行错误( VirtualMachineError )，类定义错误 ( NoClassDefFoundError )等。
&emsp;&emsp;&emsp;● **Exception** : Exception 指程序本身可以捕获并且可以处理的异常。**Exception 类的异常都是在运行期间发生的**。 Exception 类的异常包括  `Checked Exception`和 `Unchecked Exception(Runtime Exception)`。"检查"与"非检查"的是针对编译器来说的，检查异常是指编译器会对异常进行检查，非检查异常是指编译器不会对异常进行检查。
&emsp; &emsp; &emsp; ① Checked Exception : 检查异常 (非运行时异常)。对于非运行时异常，编译器要求必须进行异常捕获处理，如果不进行捕获或者抛出声明处理，编译不会通过。如 IOExeption 和 SQLException。
&emsp; &emsp; &emsp; ② Unchecked Exception: 非检查异常 (运行时异常)。对于运行时异常，不要求必须进行异常捕获处理或者抛出声明，由程序员自行决定。如  NullPointerException 和 IndexOutOfBoundsException。

#### 2.10.1 异常的底层实现
&emsp;&emsp; 在编译生成的字节码的过程中，会为每个方法中的 try-catch 语句生成一个异常表 ( Exception Table )，存放在 .class 字节码的最后。异常表中的每一条记录，都代表了一个异常处理器。异常表记录了该方法内每个异常发生的起止指令和处理指令。 
![[../picture/Pasted image 20251227220219.png#pic_center|1000]]

&emsp;&emsp; ● from :可能发生异常的起始点。 
&emsp;&emsp;&emsp; ● to :可能发生异常的结束点。
&emsp;&emsp;&emsp; ● target : 上述 from 和 to 之前发生异常后的异常处理者的位置。 
&emsp;&emsp;&emsp; ● type :异常处理者处理的异常的类信息。
![[../picture/Pasted image 20251227220345.png#pic_center|1100]]

#### 2.10.2 异常的抛出与捕获
##### 1. 异常的抛出
&emsp; &emsp; 异常的抛出分为两种: **显示抛出**和**隐式抛出**。
&emsp; &emsp; &emsp; ● **显示抛出**: 显示抛出异常的主体是应用程序，程序中使用 throw 关键字，手动抛出异常，显式抛出异常强制程序对异常进行处理，否则程序在编译阶段就会发生错误，无法通过编译。 
&emsp; &emsp; &emsp; ● **隐式抛出**: 隐式异常的主体是Java虚拟机，它指的是JVM在执行过程中，碰到无法继续执行的异常状态，自动抛出的异常。

><font color=SlateBlue><u>**Q1. throw 与 throws 的区别 ？**</u></font>
>&emsp; &emsp; ● **throws** 用来声明一个方法可能产生的所有异常，表示抛出异常，由该方法的调用者来对异常进行处理，而声明该异常的方法不做任何处理。
>&emsp; &emsp; ● **throw** 用来抛出一个具体的异常类型，用在方法体内。throw 抛出异常有两种方式: ①自己捕获异常 try...catch 代码块，② 抛出一个异常( throws 异常) 
>```java
> public class TestThrow{
> 	public static void main(String[] args) {
> 	     try{
> 		       throwChecked(-3); //调用带throws声明的方法，必须通过try..catch{}显式捕获该异常，否则，必须在main方法中再次声明抛出
> 		  }catch (Exception e){
> 		       System.out.println(e.getMessage());
> 		  }
> 		  throwRuntime(3);  //调用抛出Runtime异常的方法既可以显式捕获该异常，也可不捕获该异常
> 	 }
>      public static void throwChecked(int a)throws Exception{	
>           if (a > 0){
>                throw new Exception("a的值大于0，不符合要求"); //抛出Exception异常，该代码必须处于try块里，或处于带throws声明的方法中
>           }
>       }
>       public static void throwRuntime(int a){
> 	      if (a > 0){
> 	         throw new RuntimeException("a的值大于0，不符合要求"); //抛出RuntimeException异常，既可以显式捕获该异常,也可不捕获该异常	  
> 	      }    
>       }
>   }
>  
>  --Output--
>  // 异常抛出后，从上到下为异常的追踪路径，是从调用方法栈的栈顶向栈低进行回溯。
>  Exception in thread "main" java.lang.RuntimeException: a的值大于0，不符合要求   //异常的线程，异常类型，异常原因
>  	at test.Main.throwRuntime(Main.java:221) 	// 异常的抛出点
>  	at test.Main.main(Main.java:212)
>```
>

##### 2. 异常的捕获
&emsp;&emsp;异常的捕获分为三个部分: <font color=red>try，catch，finally</font>。
&emsp; &emsp;&emsp; ● try: 用来标记需要进行异常监控的代码。
&emsp; &emsp;&emsp; ● catch: 用来捕获在 try 代码块中触发的某种指定类型的异常。在Java中，try 代码块后面可以跟着多个 catch 代码块，来捕获不同类型的异常。Java 虚拟机会从上至下匹配异常处理器。因此，不要把上层类的异常放在最前面的catch块。
&emsp; &emsp;&emsp; ● finally: 用来声明一段必定运行的代码，finally 块无论在什么情况下都会执行。它的设计初衷是为了避免跳过某些关键的清理代码，如关闭已打开的系统资源。
```java
try{
    // try块中放可能发生异常的代码。     
  	// 如果执行完try且不发生异常，则接着去执行finally块和finally后面的代码（如果有的话）。     
  	// 如果发生异常，则尝试去匹配catch块。
}catch(SQLException SQLexception){
    // 每一个catch块用于捕获并处理一个特定的异常，或者这异常类型的子类。Java7中可以将多个异常声明在一个catch中。
    // catch后面的括号定义了异常类型和异常参数。如果异常与之匹配且是最先匹配到的，则虚拟机将使用这个catch块来处理异常。
    // 在catch块中可以使用这个块的异常参数来获取异常的相关信息。异常参数是这个catch块中的局部变量，其它块不能访问。
    // 如果当前try块中发生的异常在后续的所有catch中都没捕获到，则先去执行finally，然后到这个函数的外部caller中去匹配异常处理器。    
  	// 如果try中没有发生异常，则所有的catch块将被忽略。
}catch(Exception exception){
    //...
}finally{
    // finally块通常是可选的。
  	// 无论异常是否发生，异常是否匹配被处理，finally都会执行。
   	// 一个try至少要有一个catch块，否则， 至少要有1个finally块。但是finally不是用来处理异常的，finally不会捕获异常。
  	// finally主要做一些清理工作，如流的关闭，数据库连接的关闭等。 
}
```
><font color=SlateBlue><u>**Q1. 异常的"屏蔽"问题(链化)  ？**</u></font>
>&emsp; &emsp; 在以下的场景中，存在着异常屏蔽的情况 :
>&emsp; &emsp; &emsp; ① 如果 catch 代码块中捕获了异常，但触发了新的异常，那么 catch 捕获并且重抛的异常会是后一个异常，这样原本的异常就会被忽略掉。
>&emsp; &emsp; &emsp; ② 如果 finally 代码块中也抛出了异常，那么这个异常向上传递，try 中的异常也就被“屏蔽”了。
>```java
>public class ExceptionShield {
>	public static void main(String[] args) {
> 	   testExceptionShield();
> 	}
> 	private static void testExceptionShield() {
> 	    try {
> 	        double a = 1 / 0;
> 	        System.out.println(a);   //java.lang.ArithmeticException: / by zero 异常
> 	    } catch (Exception e) {
> 	        int[] a = {1, 2};
> 	        System.out.println(a[2]);  //java.lang.ArrayIndexOutOfBoundsException: 2 异常
> 	    } finally {
> 	        System.out.println("finally");
> 	    }
>	}
>}
>
>--Output--
>// try内的原始错误应该为 java.lang.ArithmeticException: / by zero 异常，但却被catch内的新异常给“屏蔽”了
>finally
>Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: 2
>	at com.xzz.exception.ExceptionShield.testExceptionShield(ExceptionShield.java:15)
>	at com.xzz.exception.ExceptionShield.main(ExceptionShield.java:6)

>```

### 2.12 Java 数据库 - JDBC
&emsp;&emsp; JDBC的全称是Java数据库连接 ( Java Database Connect )，它是一套用于执行SQL语句的 Java API。应用程序可通过这套API连接到关系数据库，并使用SQL语句来完成对数据库中数据的查询、更新和删除等操作。