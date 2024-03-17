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
#### 4.3.1 *Feign* 功能架构
&emsp;&emsp; *Feign* 是一个 _HTTP_ 请求调用的轻量级框架，可以以 Java 接口**注解的方式**调用HTTP请求。_Fegin_ 主要包含了三大功能：**简化远程调用**、**负载均衡  *Ribbon*、服务容错与熔断机制 *Hystrix***。    
&emsp;&emsp;&emsp;  ◎ 简化远程调用： _Feign_ 通过处理注解，将请求模板化，当实际调用的时候，传入参数，根据参数再应用到请求上，进而转化成真正的请求，封装了 _HTTP_ 调用流程。_Fegin_ 的灵感来自于 _Retrofit_、_JAXRS-2.0_ 和 _WebSocket_。    
&emsp;&emsp;&emsp;  ◎ 负载均衡 *Ribbon*： 利用负载均衡策略选定目标机器。    
&emsp;&emsp;&emsp;  ◎ 服务容错与熔断机制 *Hystrix*： 根据熔断器的开启状态，决定是否发起此次调用。
![[../picture/Pasted image 20231125222845.png#pic_center|650]]
&emsp;&emsp;&emsp; *Feign* 远程调用，核心就是通过一系列的封装和处理，将以 Java 注解的方式定义的远程调用API接口，最终转换成 Http 的请求形式，然后将 Http 的请求的响应结果，解码成 Java Bean，返回给调用者。*Feign* 远程调用的基本流程如下图所示：
![[../picture/Pasted image 20231125231834.png#pic_center|400]]
#### 4.3.2 Feign 底层原理
##### 1. *Feign* 的装载与调用
&emsp;&emsp;在微服务启动时，*Feign* 会进行包扫描，对加 *@FeignClient* 注解的接口，按照注解的规则，创建远程接口的本地 *JDK Proxy* 代理实例。然后，将这些本地Proxy代理实例，注入到 *Spring IOC* 容器中。当远程接口的方法被调用，由 *Proxy* 代理实例去完成真正的远程访问，并且返回结果。Fegin 的装载与调用过程主要分为三个步骤：**(1). *Fegin* 相关 *Bean* 的注册；(2). *Fegin* 相关 *Bean* 的依赖注入**；**(3).** 
###### (1). _Feign_ 相关 *Bean* 的注册
**▧  *Feign* 启动注解：*@EnableFeignClients* -> *@Import(FeignClientsRegistrar.class)***

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

**▧  *FeignClientsRegistrar* 注册到 IoC 容器**

&emsp;&emsp;_FeignClientsRegistrar_ 类继承 Spring 中的 _ImportBeanDefinitionRegistrar_ 接口，并在 `registerBeanDefinitions(...)` 实现方法中向 *Spring* 容器注册 *Bean*，以达到自动注入第三方功能的目的。
&emsp; &emsp;&emsp;&emsp;   ①  在 `registerBeanDefinitions(...)` 方法中首先调用 `registerDefaultConfiguration(...)`方法从 _@EnableFeignClients_ 注解中提取 _defaultConfiguration_ 属性对应的 _Value_，并封装为 _**FeignClientSpecification**_ 作为默认的 _Feign_ 配置注册到 Spring 容器中。    
&emsp; &emsp;&emsp;&emsp; ② 然后调用 `registerFeignClients(...)` 查找指定路径 _basePackage_ 的所有带有 _@FeignClients_ 注解的类、接口，将带有 _@FeignClients_ 注解的类、接口包装成 <font color=red>_**FeignClientFactoryBean**_ </font>注册到 Spring 容器。_FeignClientFactoryBean_ 类实现了 _FactoryBean<T>_，可以通过 `getObject()` 方法来获取并注入实例化对象。
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