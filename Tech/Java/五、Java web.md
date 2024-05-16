### 5.1 软件架构设计分层模型
&emsp; &emsp; 软件分层架构设计模型是一种常用的软件设计模式，它将软件系统按照功能或责任进行分层，每一层都有明确定义的职责和接口。这种架构模型具有许多优点，包括代码重用性、可维护性、可扩展性和灵活性。BS 和CS 架构是两种常见的软件架构设计模式。
&emsp; &emsp; &emsp;● BS架构（Browser/Server Architecture）是基于浏览器和服务器之间的通信，将应用程序的逻辑和数据存储在服务器上，而客户端 (浏览器) 只是通过网络请求数据和交互操作。
&emsp; &emsp; &emsp;● CS架构（Client/Server Architecture）是基于客户端和服务器之间的通信，将应用程序的逻辑和数据存储在服务器上，而客户端（终端设备）会运行一部分程序代码来处理数据和交互操作。
### 5.2 Servlet 与 Tomcat
&emsp; &emsp; Servlet 全称为Server Applet，称为小服务程序或服务连接器，用Java编写的服务器端程序，具有独立于平台和协议的特性，主要功能在于<font color=red>**交互式地浏览和生成数据，生成动态Web内容**</font>。
&emsp; &emsp; &emsp; 在B/S架构发展的过程中，浏览器(B)通过 Http 协议来访问服务器(S)，此时就需要对 Http 的请求和响应进行处理和封装。通过URL访问一个Web服务器通常分为三个过程：**接收请求、处理请求、响应请求**。接收请求和响应请求是共性功能，将这两个功能抽取出来组成 “Web 服务器”。**对于处理请求，不同的业务处理逻辑是不同，则这一部分抽取出来组成了 Servlet (Server Applet)。**

#### 5.2.1 Servlet 规范
##### 1. Servlet - 服务处理
###### (1).Servlet 框架与工作流程
&emsp;&emsp; *Servlet* 是一系列 Java 接口，是 JavaEE 规范的一种，主要是为了扩展 Java 作为Web服务，<font color=red>**主要功能是交互式地浏览和修改数据，生成动态的Web内容**</font>。Java Servlet **由 <font color=green> Servlet 容器</font>管理并产生动态的内容**。Servlet 与客户端通过 Servlet 容器实现的请求/响应模型进行交互。Servlet 容器也叫做 Servlet 引擎，是Web服务器或应用程序服务器的一部分。**<font color=red>*Servlet* 没有 `main()` 方法，不能独立运行，它必须被部署到 Servlet 容器中</font>**，由容器来实例化和调用 Servlet 的方法 ( 如 `doGet()` 和 `doPost()` )，Servlet 容器在 Servlet 的生命周期内包容和管理 Servlet。

![[../picture/Pasted image 20240219133117.png#pic_center|400]]

><font color=SlateBlue>  <u>**Q1. Servlet(类) 和 Servlet容器 的区别 ？**</u></font>
&emsp;&emsp;&emsp;● Servlet 容器：**Servlet 容器也叫做Servlet 引擎**，用于在发送的请求和响应之上提供网络服务，解码基于 MIME 的请求，格式化基于MIME (常用的MIME类型：text/html，application/pdf，video/quicktime，application /java，image/jpeg，application/jar，application/octet-stream，application/x- zip) 的响应。 Servlet容器在Servlet的生命周期内包容和管理Servlet类。Servlet容器将Servlet类载入内存，并产生Servlet实例和调用它具体的方法。**<font color=red>在一个应用程序中，每种Servlet类型只能有一个实例（以单例模式存在）</font>**。
&emsp;&emsp;&emsp;● Servlet (类)：Servlet 类是指任何实现了这个 Servlet 接口的自定义类， Servlet没有 `main()` 方法，不能独立运行，它必须被部署到 Servlet 容器中，由容器来实例化和调用 Servlet 的方法。
>
><font color=SlateBlue>  <u>**Q2. Servlet线程安全问题 ？**</u></font>
&emsp;&emsp; Servlet 是单实例多线程，当多个客户端并发访问同一个 Servlet 时，web服务器会为每一个客户端的访问请求创建一个线程，并在这个线程上调用 service 方法，因此，service 方法内如果访问了同一个资源的话，就有可能引发线程安全问题。为了避免出现线程安全问题，使用 Servlet 最好保证 Servlet 是无状态的，也就是没有可以修改的成员变量。

&emsp;&emsp;*Servlet* 接口框架主要包括: *ServlerContext*、*ServlerConfig*、*Servlet*、*ServletRequest*、*ServletResponse* 五个部分组成。

![[../picture/Pasted image 20240225100853.png#pic_center]]

&emsp;&emsp; Servlet 只有放在容器中才能执行，最常见的容器为Tomcat，Servlet工作流程如下图：
&emsp;&emsp;&emsp; ① 浏览器向服务器发送GET请求，请求服务器ServletA；
&emsp;&emsp;&emsp; ② 服务器上的 Servlet 容器接收到该URL，根据该URL判断为Servlet请求，此时Servlet 容器将产生两个对象：请求对象(HttpServletRequest)和响应对象(HttpServletResponse)；
&emsp;&emsp;&emsp; ③ Servlet 容器对请求的 URL 进行解析并根据web.xml配置文件找到处理该请求的Servlet（ServletA），并创建一个线程A；
&emsp;&emsp;&emsp; ④ Servlet 容器将刚才创建的请求对象和响应对象传递给线程A；
&emsp;&emsp;&emsp; ⑤ Servlet 容器调用Servlet的service()方法；
&emsp;&emsp;&emsp; ⑥ service() 方法根据请求类型调用 doGet() 或doPost()方法；
&emsp;&emsp;&emsp; ⑦ doGet() 执行完后，将结果返回给Servlet 容器
&emsp;&emsp;&emsp; ⑧ 线程A被销毁或被放在线程池中；
![[../picture/Pasted image 20240225103624.png#pic_center|400]]

###### (2).Servlet 生命周期
&emsp;&emsp; Servlet 是运行在Servlet容器中的，由 **Servlet 容器**来负责 Servlet 实例的查找、创建以及整个生命周期的管理。Servlet整个生命周期可以分为四个阶段：
&emsp;&emsp;&emsp; **① 类装载以及实例创建阶段**：默认情况下，Servlet实例是在接收到第一个请求时进行创建，并且在以后的请求中对这个实例进行复用。
&emsp;&emsp;&emsp; **② 实例初始化阶段**：一旦 Servlet 实例被创建，将会调用Servlet中的 init(ServletConfig arg) 方法，传入ServletConfig，即 Servlet 的相关配置信息，init() 方法在整个Servlet的生命周期中只会被调用一次。
&emsp;&emsp;&emsp; **③ 服务阶段**：实例初始化后，一旦由客户端请求，Servlet 就会调用 service(ServletRequest req, ServletRespose res) 方法处理数据并响应数据。
&emsp;&emsp;&emsp; **④ 实例销毁阶段**：当Servlet容器决定销毁某个Servlet时，将会调用 Servlet 实例中的 destory() 方法，在 destory() 方法中进行资源释放。一旦Servlet实例的 destory() 方法被调用，Servlet 容器将不会发任何请求给这个Servlet实例，若 Servlet 容器需要再次使用这个 Servlet，需要重新实例化该 Servlet 实例。
![[../picture/Pasted image 20240225104542.png#pic_center|345]]

###### (3).Servlet 简单使用
&emsp;&emsp;*Servlet* 在使用时，通常通过自定义类来继承 *HttpServlet* 类，并重写 `init()`、`doPost()`、`doPut()`等方法。同时在Web.xml文件中添加Servlet的配置信息。[IDEA 中创建一个 Servlet 服务器](https://blog.csdn.net/weixin_44107140/article/details/119618734?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3-119618734-blog-110453620.235%5Ev43%5Epc_blog_bottom_relevance_base3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3-119618734-blog-110453620.235%5Ev43%5Epc_blog_bottom_relevance_base3&utm_relevant_index=6)
![[../picture/Pasted image 20240303223242.png#pic_center|460]]
```html
1. 静态资源
//Local.html登录界面，部署在web目录下，可以通过http://localhost:8080/JavaDemoProject_Servlet/Login.html路径访问到该静态资源，其中<form>表单的动作映射指向LoginServlet。
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>登录</title>
</head>
<body>
<form action="http://localhost:8088/JavaDemoProject_Servlet/LoginServlet" method="post">
    用户：<input type="text" name="username" /><br/>
    密码：<input type="password" name="password" /><br/>
    <input type="submit" value="登录" />
</form>
</body>
</html>
```
```Java
2. Servlet实例
// LoginServlet 继承 HttpServlet 并重写了doGet(),doPost()方法
public class LoginServlet extends HttpServlet {
    //重写doGet方法
    public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String username = request.getParameter("username");
        //设置编码格式
        response.setContentType("text/html;charset=GB18030");
        response.getWriter().println("欢迎【" + username + "】用户登录成功！！！");
    }
    //重写doPost方法
    public void doPost(HttpServletRequest request, HttpServletResponse response) throws IOException {
        doGet(request, response);
    }
}
```
```java
3. Filter过滤器实例
public class MyFilter implements Filter {
    private FilterConfig filterConfig;
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        this.filterConfig = filterConfig;
    }
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                         FilterChain chain) throws IOException, ServletException {
        if(request.getParameter("password").equals("")){	 //校验输入的密码参数是否为空
            response.setContentType("text/html;charset=GB18030");
            response.getWriter().println("请输入密码");
            return;		//return之后，过滤器链不会将请求传递到下一级的过滤器
        }
        if(request.getParameter("username").equals("null")){	//校验输入的用户名是否为null
            response.setContentType("text/html;charset=GB18030");
            response.getWriter().println("登录失败");
            return;	//return之后，过滤器链不会将请求传递到下一级的过滤器
        }
        chain.doFilter(request,response);  //过滤器链将请求传递到下一级的过滤器
    }
    @Override
    public void destroy() {
        this.filterConfig = null;
    }
}
```
```java
4. Servlet配置文件
<!-- 
	① 首先浏览器通过http://localhost:8080/JavaDemoProject_Servlet/LoginServlet来找到web.xml <servlet-mapping>中的<url-pattern>，其他JavaDemoProject_Servlet为Tomcat容器的访问路径。
	② 匹配到了<url-pattern>后，就会找对应的Servlet的名字<servlet-name> - MyServlet;
	③ 知道了名字，就可以通过<servlet-name>找到<servlet-class>，也就能够知道Servlet的位置了。然后到其中找到对应的处理方式进行处理。
-->
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         version="4.0">
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>WEB-INF/applicationContext.xml</param-value>
    </context-param>
  
  	<!-- listener监听器配置信息 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
  
    <!-- filter过滤器配置信息 -->
    <filter>
        <!-- filter-name: 过滤器名称 -->
        <filter-name>MyFilter</filter-name>
				<!-- filter-class: filter全限定类名，也就是filter的位置 -->
        <filter-class>com.xzz.main.MyFilter</filter-class>
    </filter>
  	<!-- filer实例,过滤路径映射配置信息 -->
    <filter-mapping>
        <filter-name>MyFilter</filter-name>
        <url-pattern>/LoginServlet</url-pattern>
    </filter-mapping>
     
  	<!-- servlet实例配置信息 -->
    <servlet>   
      	<!-- servlet-name: Servlet名字 -->
        <servlet-name>MyServlet</servlet-name>        	         
      	<!-- servlet-class: Servlet全限定类名，也就是Servlet的位置 -->
        <servlet-class>com.xzz.main.LoginServlet</servlet-class>
    </servlet>
  	<!-- servlet实例路径映射配置信息 -->
    <servlet-mapping>
        <servlet-name>MyServlet</servlet-name>		<!-- Servlet名字，与上面的名字相同 -->
        <url-pattern>/LoginServlet</url-pattern>  <!-- 浏览器通过该URL路径可以找到该Servlet实例 -->
    </servlet-mapping>
</web-app>
```

##### 2. Filter - 过滤器
&emsp;&emsp; Filter 的作用是对 Servlet 容器传给Web资源的 request 对象和 response 对象进行检查和修改，实现用户在访问某个目标资源之前，对访问的请求和响应进行拦截，一般常用于实现URL级别的权限访问控制、过滤敏感词汇、压缩响应信息等功能。<font color=green>过滤器并不是必须要将请求传递到下一个过滤器或目标资源，它可以自行对请求进行处理，并发送响应给客户端，也可以将请求转发或重定向到其他的Web资源。</font>Filter 使用的是责任链设计模式。
![[../picture/Pasted image 20240303110026.png#pic_center|420]]
###### (1). *Filter* 生命周期与工作过程
&emsp;&emsp; Filter 的生命周期分为三个阶段: **初始化阶段、拦截和过滤阶段、销毁阶段**。
&emsp; &emsp; ● **初始化阶段**：Servlet 容器负责加载和实例化 Filter 。容器启动时，读取 web.xml  的配置信息对所有的过滤器进行加载和实例化。加载和实例化完成后，Servlet 容器调用 `init()` 方法初始化 Filter 实例。
&emsp; &emsp; ● **拦截和过滤阶段**：当客户端请求访问 Web资源时，Servlet 容器会根据 web.xml 的过滤规则进行检查。当客户端请求的 URL 与过滤器映射匹配时，容器将该请求的 request 对象、response 对象以及 FilterChain 对象以参数的形式传递给 Filter 的 <font color=red>**`doFilter()` 方法**</font>，对请求/响应进行拦截和过滤。 
&emsp; &emsp; ● **销毁阶段**：Filter 对象创建后会驻留在内存中，直到容器关闭或应用被移除时销毁。销毁 Filter 对象之前，容器会先调用 `destory()` 方法，释放过滤器占用的资源。在 Filter 的生命周期内，`destory()` 只执行一次。
![[../picture/Pasted image 20240303110805.png#pic_center|550]]

##### 3. Listener - 监听器 
&emsp;&emsp; *Listener* 用于监听Java对象的方法调用或属性改变，当被监听对象发生上述事件后，监听器某个方法立即被执行。监听器 *Listener* 按照监听的事件分为3类：
&emsp; &emsp;&emsp; ① 监听对象创建和销毁的监听器：*Servlet* 规范定义了监听 *ServletContext*、*HttpSession*、*HttpServletRequest* 这三个对象创建和销毁事件的监听器。
&emsp; &emsp;&emsp; ② 监听对象中属性变更的监听器：*Servlet* 规范定义了监听 *ServletContext*、*HttpSession*、*HttpServletRequest* 这三个对象中的属性变更事件的监听器。
&emsp; &emsp;&emsp; ③ 监听 *HttpSession* 中对象状态改变的监听器：*Session* 中的对象可以有多种状态，绑定到 *Session* 中、从 *Session* 中解除绑定、随 *Session* 对象持久化到存储设备中(钝化)、随 *Session* 对象从存储设备中恢复(活化)
![[../picture/Pasted image 20240303224859.png#pic_center|450]]
![[../picture/Pasted image 20240308224434.png#pic_center|800]]
&emsp;&emsp;注册 *Servlet* 监听器有2种方式，分别是：① 在 *web.xml* 中注册监听器；② 使用注解 `@WebListener` 注册监听器。
```java
1. 在web.xml中使用 <listener> 标签配置监听器，Web容器会自动把监听器注册到事件源中
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
    version="4.0">
    <listener>
        <listener-class>net.biancheng.www.listener.MySessionListener</listener-class>
    </listener>
</web-app>
          
2. 在监听器类上使用@WebListener注解，可以将该Java类注册为一个监听器类
@WebListener
public class MyListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {  }
    @Override
    public void contextDestroyed(ServletContextEvent sce) {   }
}
```
#### 5.2.2 Web 容器 - Tomcat
&emsp;&emsp; Servlet 与 Tomcat 在Java Web开发中密切相关，  <font color=red>Servlet 没有 `main()` 方法，不能独立运行，它必须部署到 Web 容器中</font>，由容器来实例化和调用 Servlet 的方法 ( 如 `doGet()`、`doPost()` )，Web 容器在 Servlet 的生命周期内包容和管理 Servlet。
&emsp;&emsp;&emsp; Tomcat是一个开源的、支持Java Servlet 规范的 Web 容器，它实现了 Servlet 和JSP ( JavaServer Pages ) 技术。<font color=green>Tomcat 负责管理Servlet 的生命周期、处理与客户端之间的网络通信以及其他与 Servlet 相关的任务</font>。因此 Servlet 和Tomcat之间存在一种包含关系：Tomcat提供了环境和基础设施来运行和管理 Servlet。当开发一个基于 Servlet 技术的Web应用时，将自己的 Servlet 并将其部署到 Tomcat 中，然后Tomcat 负责加载、初始化和调度 Servlet 对象以响应客户端请求。除了Tomcat之外，还有其他类似功能的Web容器可供选择，比如 Jetty、GlassFish 等。
##### 1. Tomcat 整体框架
&emsp;&emsp;Tomcat的顶层容器是 Server，它代表着整个服务器。一个 Server 可以包含至少一个 Service，用于具体提供服务。Service 主要由两个部分组成：<font color=red>**Connector 和 Container**</font>。一个Tomcat 中只有一个Server，一个 Server 可以包含多个 Service，一个 Service 只有一个 Container，但可以有多个 Connectors，因为一个服务可以有多个连接，如同时提供 Http 和 Https 链接，也可以提供向相同协议不同端口的连接。
&emsp;&emsp;&emsp; ① **Connector** 用于处理连接相关的事情，并提供 Socket 与 Request 和 Response 相关的转化; 
&emsp;&emsp;&emsp; ② **Container** 用于封装和管理Servlet，以及具体处理Request请求；

![[../picture/Pasted image 20240309224241.png#pic_center]]

&emsp;&emsp;一个请求发送到 Tomcat 之后，首先经过 Service 然后会交给 Connector，Connector 用于接收请求并将接收的请求封装为 Request 和 Response 来具体处理，Request 和 Response 封装完之后再交由 Container 进行处理，Container 处理完请求之后再返回给 Connector，最后在由 Connector 通过 Socket 将处理的结果返回给客户端。

##### 2. Server - 服务实例
&emsp;&emsp;  Server 组件**用于描述一个启动的 Tomcat 实例**，可以看做 Tomcat 实例的抽象。一个 Tocmat 被启动，在操作系统中占用一个进程号，提供web服务的功能，这整个服务用 Server 来表示。Tomcat提供的所有功能，都由 Server 组件中的子组件实现。**Server 组件对应关联的配置文件为 `server.xml`**。Server 服务的启动流程如下图所示：

![[../picture/Pasted image 20240309224606.png#pic_center|750]]
&emsp;&emsp; Tomcat 的 `main()` 线程在启动完所有的组件后，会开启一个 Socket 服务端，在指定的端口上进行监听，直到有 shutdown 命令发送过来就退出 Socket 的等待，开始执行关闭方法 Server 服务的关闭。Server 的关闭放在Tomcat的生命周期中的 `stop()` 方法和 `destroy()` 方法中进行处理。Server 服务关闭的流程如下图所示：
![[../picture/Pasted image 20240309225941.png]]

&emsp;&emsp; Server 组件默认有五个监听器( Tomcat 8之后 )，分别为：
&emsp;&emsp;&emsp;   ① **VersionLoggerListener** : 通过 *StringManager* 对象，监听并且记录Tomcat启动的时候记录版本的信息。
&emsp;&emsp;&emsp;   ② **AprLifecycleListener** : 初始化APR以及在Tomcat销毁之后的清理APR工作。初始化之前尝试初始化APR库，成功则使用APR接受处理客户端请求；Tomcat销毁之后，该监听器会做APR的清理工作。
&emsp;&emsp;&emsp;   ③ **JreMemoryLeakPreventionListener** : 该监听器为了处理由于**上下文加载器而导致的内存泄漏问题**，Tomcat 在重加载一个Web应用时会实例化一个新的类加载器，而旧的类加载器无法被垃圾回收器回收，导致内存泄漏。除此之外，还解决了**锁文件问题**。锁文件的情景主要由 *URLConnection* 默认的缓存机制导致，当使用 *URLConnection* 的方式读取本地Jar包里面的资源时，它会将资源内存缓存起来，这就导致了该Jar包被锁。此时如果进行重新部署将会失败，因为被锁的文件无法删除。
&emsp;&emsp;&emsp;   ④ **GlobalResourcesLifecycleListener** : 该监听器用于监听 Tomcat 容器的启动、销毁，Tomcat启动时 *GlobalResourcesLifecycleListener* 实例化 JNDI 资源的 *MBean*，Tomcat停止时销毁 *MBean*。
&emsp;&emsp;&emsp;   ⑤ **ThreadLocalLeakPreventionListener** : 该监听器用于监听Tomcat容器启动后、停止前、停止后，目的是为了防止 *ThreadLocal* 对象带来的内存泄漏问题。Tomcat 内部接收请求都是通过线程池的方式处理，线程池中线程生命周期一般都长，如某个Web应用 Web-A，使用 *ThreadLocal* 保存 Web-A 中信息，Web-A 又是由 Web应用的 *WebappClassLoader* 加载的，如果部署新的 Web 应用，实例化了新的 *WebappClassLoader*，线程池中线程一直在运行或等待着，但是旧的*WebappClassLoader* 由于 Web-A 保留着引用无法被回收，这样就导致了内存泄露。因此，当新的Web应用部署时，该监听器会将所有的线程池内所有线程销毁并且重新创建新的线程。

##### 2. Server - 服务实例
&emsp;&emsp;  Service 是对外提供服务的。一个 Server 可以有多个 Service。Service 包含了两个组件 **Container** 和 **Connector**。Container 是一个容器，负责处理请求的，Connector 是负责接受请求，并且将请求提交给 Container。一个 Service 包含**多个 Connector 和一个 Container** ，两者的关联关系使用 Mapper 来做映射。
###### (1). Connector - 连接器
**▨  Connector 框架**

&emsp;&emsp; Connector 是客户端连接到 Tomcat 容器的服务点。每个 Connector 都将指定一个端口进行监听，Connector 用于接受请求并将请求封装成 Request 和 Response，然后交给 Container 进行处理，Container 处理完之后在交给 Connector 返回给客户端。连接器主要包括以下核心组件：

![[../picture/Pasted image 20240316000828.png#pic_center]]

&emsp;&emsp;以 **Http11Protocol** 协议为例，其请求数据处理流程如下图所示：
&emsp;&emsp;&emsp; ① Tomcat 基于 `server.xml` 的安全协议配置，通过 ServerSocketFactory 创建对应的套接字对象。
&emsp;&emsp;&emsp;  ② Tomcat 通过 JioEndPoint 建立套接字连接，并通过 *LimitLatch* 进行套接字连接数限制(流量限制)
&emsp;&emsp;&emsp;  ③ Tomcat通过操作系统的 Socket 套接字获取到请求的字节流（Http字节流），在字节流信息传输过程中，为了提高发送和接收效率，在套接字的发送端与接收端都引入了缓冲区 InternalInputBuffer 。在接收到字节流之后，会首先创建 ByteChunk 字节块（字节数组），将字节流写入到 ByteChunk 缓冲区当中，当缓冲区中的字符数量到达缓冲数组最大值时，将字符数据输出到指定目标中（Acceptor）。
&emsp;&emsp;&emsp;  ④ Acceptor 接收到套接字请求时，会将创建 SocketProcess 线程任务，并丢进 Executor 线程池中。SocketProcess 会读取套接字字节轮流，并对Http报文进行解析，组成，并生成 Request 对象。
&emsp;&emsp;&emsp;  ⑤ Request 请求对象通过 Adapter 适配器，将请求传递给 Engine 容器。

![[../picture/Pasted image 20240318230731.png#pic_center|550]]

**▨  Connector 组件**

&emsp;&emsp; **● ProtocolHandler 协议处理器**：传输协议的抽象，将不同通信协议的处理进行了封装常见的通信协议如: Http、AJP。不同的 ProtocolHandler 代表不同的连接类型。Connector使用哪种 Protocol，可以通过 `<connector>` 元素中的 protocol 属性进行指定，也可以使用默认值。指定的 protocol 取值及对应的协议如下：
&emsp;&emsp;&emsp; ① HTTP/1.1：默认值，使用的协议与Tomcat版本有关
&emsp;&emsp;&emsp; ② org.apache.coyote.http11.Http11Protocol：BIO，使用的是普通Socket来连接的。
&emsp;&emsp;&emsp; ③ org.apache.coyote.http11.Http11NioProtocol：NIO， 使用的是 NioSocket 来连接的。
&emsp;&emsp;&emsp; ④ org.apache.coyote.http11.Http11Nio2Protocol：NIO2
&emsp;&emsp;&emsp; ⑤ org.apache.coyote.http11.Http11AprProtocol：APR

![[../picture/Pasted image 20240314231725.png#pic_center|600]]

&emsp;&emsp; 目前大多数<font color=red>**HTTP请求使用的是长连接**</font> ( HTTP/1.1默认keep-alive为true )，一个TCP的 socket 在当前请求结束后，如果没有新的请求到来，socket不会立马释放，而是等timeout后再释放。如果使用BIO，在socket等待下一个请求或等待释放的过程中，处理这个socket的工作线程会一直被占用，无法释放；因此Tomcat 性能受到了极大限制。而使用NIO，socket是非阻塞的，当socket在等待下一个请求或等待释放时，并不会占用工作线程，因此Tomcat可以同时处理的socket数目远大于最大线程数，并发性能大大提高。

&emsp;&emsp; **● EndPoint 通信端点**：EndPoint 是请求接收端的抽象，用于监听通信接口，接收和响应网络请求，处理系统底层 Socket 的网络连接。因此 **Endpoint 是用来实现TCP/IP协议的**。同时由于系统底层 Socket 存在不同的I/O模式，因此会对应存在多种类型的 EndPoint，如 BIO 模式下的 JIoEndPoint，NIO 模式下的 NioEndPoint，本地库I/O模式下的 AprEndpoint。
&emsp;&emsp;&emsp; ① LimitLatch：用来限流，可以控制最大连接个数，类似J.U.C中的Semaphore。
&emsp;&emsp;&emsp; ② Acceptor：只负责接收新的socket连接。
&emsp;&emsp;&emsp; ③ Poller ：只负责监听socket channel是否有可读的I/O事件，默认线程数量为1，采用的是 NIO 多路复用。一旦存在可读I/O事件，就封装一个任务对象 sockerProcessor 提交给 Executor线程池进行处理。

&emsp;&emsp; **● 线程池 Executor**：Executor 线程池的作用是负责处理请求。线程池的 run() 方法会调用 Processor 解析应用层协议，将Socket连接与请求数据转换为 Http 协议数据，生成 Tomcat request。
![[../picture/Pasted image 20240315232433.png#pic_center|500]]
&emsp;&emsp;&emsp;Tomcat 的线程池并没有直接使用 j.u.c ( java.util.concurrent - java核心包之一 ) 里面的线程池，而是扩展了 JDK 的线程池 ThreadPoolExecutor，而且缓存队列用的是自己的task queue，因此其策略与 JDK 线程池的有所不同。
><font color=SlateBlue>  <u>**Q1. Tomcat executor 线程池与 JDK ThreadPoolExecutor 线程池的区别 ？**</u></font>
>&emsp;&emsp;● JDK 线程池在运行时：
>&emsp;&emsp;&emsp; ① 如果当前运行的线程，少于corePoolSize，则创建一个新的线程来执行任务。 如果运行的线程等于或多于 corePoolSize，将任务加入 BlockingQueue。
>&emsp;&emsp;&emsp; ② 如果 BlockingQueue 内的任务超过上限，则创建新的线程来处理任务。
>&emsp;&emsp;&emsp; ③ 如果创建的线程超出 maximumPoolSize，任务将被拒绝策略拒绝。
>
>&emsp;● Tomcat 线程池在运行时：
>&emsp;&emsp;&emsp; ① 任务执行失败时不会直接抛出错误，而是装回队列里再次尝试执行；
>&emsp;&emsp;&emsp; ② 当线程池没有达到最大执行线程的时候，会优先开线程再使用任务队列；
>&emsp;&emsp;&emsp; ③ 扩展计数用于追踪任务的执行情况；

**▨  Tomcat Connector相关配置参数**
```
1. Endpoint 配置参数：
	● port：指定Tomcat监听的端口。
	● protocol：Connector设置使用什么协议处理入口流量，默认值 HTTP/1.1。
	● maxConnections：Tomcat在任意时间接收和处理的最大连接数。当连接数达到 maxConnections 时，仍可基于acceptCount的配置接受连接，但并不会处理，直到Tomcat接收的连接数小于maxConnections。即 maxConnections = 线程池阻塞队列的大小 + maxThreads。
	● connectionTimeout：网络连接超时时间，默认60000ms。设为0表示永不超时。
	● acceptCount：TCP的最大链接数。当最大线程数 (maxThreads)被使用完时，可以放入请求队列的队列长度，默认100。一旦队列满了，就会返回connection refused。
	● enableLookup：是否启用DNS查找功能。如果设为true，会用request.getRemoteHost()执行DNS lookup，返回远程客户端的主机名。设为false则跳过DNS lookup,并以字符串形式返回IP地址，从而提高性能，默认false，生产环境建议保持关闭。
	● compression：是否开启GZIP压缩。off(禁用)、on(打开，压缩文本数据)，force(强制压缩所有格式)、min-response-size(数据量达到该值就GZIP传输)。
	
2. Connector executor配置参数：
	● maxThreads：能创建的最大线程数，超过该线程数，请求会被放入队列中等待执行。默认值是200。
	● maxSpareThreads：线程池中保持的最小线程数，也是Tomcat启动时的初始化的线程数。
	● maxIdleTime：线程空闲的最大时间，当空闲超过该值时关闭线程，直到保持maxSpareThreads的最小线程数，默认值60000ms。
	● maxQueueSize：缓存队列大小，在拒绝执行之前可以排队等待执行的任务数量
```
><font color=SlateBlue>  <u>**Q1. maxConnections、maxThreads、acceptCount之间的关系 ？**</u></font>
![[../picture/Pasted image 20240317090439.png#pic_center|700]]
###### (2). Container - 容器
&emsp;&emsp;  Container 负责对内处理业务逻辑，内部由**四个容器**组成，各容器之间的调用都会通过一个通道 **Pipeline-Valve** ( 责任链模式 )。每个 Service 会包含一个 Container 容器。一个 Container 容器由一个引擎可以管理多个 Host 虚拟主机。每个 Host 虚拟主机可以管理多个 Web 应用。每个 Web 应用会有多个 Wrapper 包装器。 Container 容器的请求处理过程就是在 **Engine、Host、Context 和 Wrapper** 这四个容器之间层层调用，最后在 Servlet 中执行对应的业务逻辑。
&emsp;&emsp;&emsp;● **Engine - 引擎**：表示可运行的Catalina的servlet引擎实例，包含了servlet容器的核心功能。Engine 用来管理多个站点，<font color=green>主要功能是将传入请求委托给适当的虚拟主机处理。一个Service 最多只能有一个Engine。</font>在Tomcat启动过程中，解析 `server.xml` 时碰到一个 Engine 节点就会提交一个异步线程，并在异步线程中执行当前容器及所有子容器的 `backgroundProcess()` 方法，从而实现 Engine 容器中各个子容器的加载。

![[../picture/Pasted image 20240318234134.png#pic_center|290]]
&emsp;&emsp;&emsp;● **Host - 虚拟主机**：代表一个站点。Host 负责安装和展开这些应用 ( **解析web.xml** )，并且标识这个应用以便能够区分它们。一个虚拟主机下都可以部署一个或者多个Web App，每个Web App对应于一个 Context ，当 Host 获得一个请求时，将把该请求匹配到某个 Context 上，然后把该请求交给该 Context 来处理。如 `http://tomcat.apache.org/index.html`，tomcat.apache.org 被抽象为一个主机。
&emsp;&emsp;&emsp;● **Context - 应用**：代表一个应用程序，表示Web应用程序本身。Context 最重要的功能就是管理它里面的 Servlet 实例，一个 Context 对应于一个 Web Application，一个 Web Application 由一个或者多个Servlet实例组成。拥有了 Context 就代表 Tomcat 具备了 Servlet 运行的基本环境。
&emsp;&emsp;&emsp;● **Wrapper - Servlet**：Wrapper 是最底层的容器，每个Wrapper 封装着一个servlet。Wrapper 负责管理一个 Servlet，包括的 Servlet 的装载、初始化、执行以及资源回收。

![[../picture/Pasted image 20240318231635.png#pic_center|650]]

###### (3). Pipeline - 容器管道
###### (4). Mapper -  URL 映射器

