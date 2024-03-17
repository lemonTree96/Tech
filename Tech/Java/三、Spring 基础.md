&emsp;&emsp;广义的 _Spring_ 是指整个 _Spring_ 生态以及全部项目，包括_Spring Boot_、_Spring Cloud_等等，可以看作一个生态体系，而狭义的 _Spring_ 就是指 _Spring Framework_，其他的项目比如_Spring Boot_、_Sping Cloud_等等都是以 _Spring Framework_ ( 以下简称为 *Spring* ) 作为基础演变而来。
![[../picture/Pasted image 20231209220854.png#pic_center|530]]
>  **Q1. _Spring_ 的设计初衷 ？**
> &emsp;&emsp;_Spring_ 的设计初衷是降低企业级应用开发的复杂性，简化开发，基于这一初衷，Spring 采取了4个关键策略： 
> &emsp;&emsp; (1). 基于POJO的轻量级和最小侵入性编程 -> Bean；
> &emsp;&emsp; (2). 通过依赖注入和面向接口实现松耦合 -> DI；
> &emsp;&emsp; (3). 基于切面和惯性实现声明式编程 -> IoC； 
> &emsp;&emsp; (4). 通过切面和模板减少样板式代码 -> IoC；

&emsp;&emsp;*Spring* 的本质是针对 _Bean_ 的生命周期进行管理的轻量级容器。*Spring* 通过核心的 _Bean factory_ 实现了底层的类的实例化和生命周期的管理。在整个框架中，各类型的功能被抽象成一个个的_Bean_，这样就可以实现各种功能的管理，包括动态加载和切面编程。Spring 框架是一个分层架构，由7个定义良好的模块组成。Spring模块构建在核心容器之上，核心容器定义了创建、配置和管理 Bean 的方式。
&emsp;&emsp;&emsp;  ● **核心容器 - Core**：核心容器提供 Spring 框架的基本功能。核心容器的主要组件是 _BeanFactory_，它采用工厂模式实现。_BeanFactory_ 使用控制翻转 IoC 思想将应用程序的配置和依赖性规范与实际的应用程序代码分开。同时通过 _Spring Context_ 上下文向 Spring 框架提供上下文信息。 
&emsp;&emsp;&emsp;  ● **面向切面 - AOP**：通过配置管理特性，_Spring AOP_ 模块直接将面向切面的编程功能集成到了 Spring 框架中。所以，可以很容易地使 Spring 框架管理的任何对象支持AOP。Spring AOP 模块为基于 Spring 的应用程序中的对象提供了**事务管理服务**。通过使用 Spring AOP，不用依赖 EJB 组件，就可以将声明性事务管理集成到应用程序中。
&emsp;&emsp;&emsp; ● **Spring Web** ：_Web_ 上下文模块建立在应用程序上下文模块之上，为基于 _Web_ 的应用程序提供了上下文。所以，Spring 框架支持与 _Struts_ 的集成。
&emsp;&emsp;&emsp; ● **Spring ORM**：负责框架中对象关系映射，提供相关 _ORM_ 接入框架的关系对象管理工具。Spring 框架插入了若干个 _ORM_ 框架，从而提供了 _ORM_ 的对象关系工具，其中包括 _JDO_、_Hibernate_和 _iBatis_ 。所有这些都遵从 Spring 的通用事务和 DAO 异常层次结构。
![[../picture/Pasted image 20231209221252.png#pic_center|560]]
### 3.1 IoC 与 DI
&emsp;&emsp; IoC 不是什么技术，而是一种**设计思想**，指导如何设计出松耦合。传统应用程序都是由我们在类内部主动创建依赖对象，从而导致类与类之间高耦合；有了IoC容器后，把**创建和查找依赖对象的控制权交给了 IoC 容器**，由IoC容器进行注入组合对象，在开发过程中不在需要关注对象的创建和生命周期的管理，所以对象与对象之间是松散耦合。对于Spring生态来说，IoC是 *Spring* 框架的核心，**由 *Spring* 来负责控制对象的生命周期和对象间的关系，IoC容器负责实例化、定位、配置应用程序中的对象及建立这些对象间的依赖**。
![[../picture/Pasted image 20231209221813.png#oic_center|400]]
### 3.2 *Spring Bean*
&emsp;&emsp;在 Spring 中，由**IoC容器管理的组成应用程序的对象称为 _Spring Bean_**，_Spring Bean_就是由IoC容器初始化、装配及管理的对象。_Spring Bean_ 与IoC容器的关系如下图所示：
![[../picture/Pasted image 20231210164606.png#pic_center|580]]
>**Q1. _JavaBean_ 与 _Spring Bean_ 的区别 ？**
>&emsp;&emsp; (1). _JavaBean_ 是指一种遵循特定规则的Java类，符合这个规则的类都可以称为 _JavaBean_，也可以说 _JavaBean_ 是一种Java语言编写的可重用“组件”，即**一次编写，任何地方执行，任何地方重用**。一个类可以称为 _JavaBean_ 必须遵守以下几个规则：
>&emsp;&emsp;&emsp;● 类中的所有属性均为 _private_；
>&emsp;&emsp;&emsp;● 类中要提供默认构造方法;
>&emsp;&emsp;&emsp;● 对于类中的所有属性，要提供 _getter_ 方法和 _setter_ 方法；
>&emsp;&emsp;&emsp;● 该类要实现 _serializable_ 接口；
>&emsp;&emsp;(2). _SpringBean_ 是受 _Spring_ 管理的对象 所有能受 _Spring_ 容器管理的对象都可以成为 _SpringBean_. 
>&emsp;&emsp;(3). 两者**用处不同**：传统 _Javabean_ 更多地作为值传递参数，而 _Spring_ 中的 _Bean_ 用处几乎无处不在，任何组件都可以被称为 _Bean_。
>&emsp;&emsp;(4). 两者**写法不同**：传统 _Javabean_ 作为值对象，要求遵守 _JavaBean_ 的规则；但 _Spring_ 中的 Bean只需为接受设值注入的属性提供 _setter_ 方法。
>&emsp;&emsp;(5). 两者**生命周期不同**：传统 _Javabean_ 作为值对象传递，不接受任何容器管理其生命周期；_Spring_ 中的 _Bean_ 有 _Spring_ 管理其生命周期行为。

#### 3.2.1 _Bean_ 生命周期
&emsp;&emsp;_Spring Bean_ 的生命周期指的是从一个普通的 _Java_ 类变成 _Bean_ 对象的过程。_Spring bean_ 的生命周期非常重要 ，几乎所有跟 Spring 整合的框架，如Mybatis 、Dubbo 等框架基本上都是通过 _Bean_ 的生命周期来实现跟 Spring 的整合。
![[../picture/Pasted image 20231210165913.png#pic_center|430]]
&emsp;&emsp;&emsp; 在Spring中，从 _Bean_ 的生成到注册到 IoC容器中一共分为以下几个环节，每个环节中Spring都提供了一些扩展点供第三方进行功能扩展。生命周期的流程如下图所示：
![[../picture/Pasted image 20231210170025.png#pic_center|500]]
![[../picture/Pasted image 20231210170200.png]]
&emsp;&emsp;&emsp; 在 _DefaultListableBeanFactory_ 类中有个重要的字段：**`private final List<BeanPostProcessor> beanPostProcessors = new CopyOnWriteArrayList<>()`**，该字段是 _BeanPostProcessor_ 接口的集合，_BeanPostProcessor_ 接口提供了很多方法，Spring在 _Bean_ 生命周期的不同阶段调用列表中的 _BeanPostProcessor_ 中的方法来对生命周期进行扩展。***Bean* 生命周期中的所有扩展点都是依靠这个集合中的 _BeanPostProcessor_ 来实现的。**

##### 1. *Bean* 元信息环节
&emsp;&emsp;*Bean* 的元信息环节主要分为两个步骤： ***Bean* 元信息的配置** 与 ***Bean*元信息的解析**。在Spring容器启动时，会按照 *Bean* 的声明方式 ( *xml* 配置 / *@Bean*注解 / *@Component* 注解 / *Properties* 资源配置 ) 去读取声明的 *Bean* 信息，然后解析声明的信息，并封装在 ***BeanDefinition*** 对象里面。在后续环节，*Bean* 工厂就会根据 *BeanDefinition* 对象的信息，生产一个 *Bean* 实例，并对 *Bean* 进行实例化、初始化等等操作。 
&emsp;&emsp;&emsp;*BeanDefinition* 是一个接口，包含了 *Bean* 定义的各种信息，如：*Bean* 对应的 *class* 类型、*Bean* 的作用域 *scope*、*Bean* 懒加载信息 *lazy*、*Bean* 的依赖等信息。在 *Spring* 容器中，存储的就是每个 *Bean* 对应的 *BeanDefinition* 对象。
![[../picture/Pasted image 20231210175426.png#pic_center|600]]
###### (1).  ***Bean* 元信息的解析**
&emsp;&emsp;  *Bean* 元信息的解析的目的是**根据配置的 *Bean* 元信息，通过解析后生成 *BeanDefinition* 对象**。针对不同的 *Bean* 注册方式，相应的提供了的不同的 *Bean* 解析方式，Bean元信息的解析主要依赖于 ***BeanDefinitionReader* 体系**。*BeanDefinitionReader* 体系有三个实现类，分别对应3种元数据解析方式：Xml文件定义 *Bean* 的解析 *XmlBeanDefinitionReader*、*properties* 文件定义 *Bean*的解析 *PropertiesBeanDefinitionReader*、通过注解方式定义 *Bean* 的解析。
![[../picture/Pasted image 20231210175613.png#pic_center|500]]
##### 2. *Spring Bean* 注册环节
&emsp;&emsp; Spring Bean的注册环节是<font color=green>**将解析后的 *BeanDefinition* 对象注册到 IoC 容器当中**</font>，以便于后续由 Spring Bean 工厂 - *BeanFactory* 对容器中的 *BeanDefinition* 进行实例化，建立这些对象间的依赖关系，即 *BeanDefinition* -> Spring 容器 -> *BeanFactory*。Spring Bean 的注册依赖于*Bean*注册器 - *BeanDefinitionRegistry* 接口，该接口定义了关于 *BeanDefinition* 的注册，移除，查询等操作。*BeanDefinition* 注册完成之后交由 *BeanFactory* 接口的 *Bean* 工厂进行管理。因此，Bean的注册器 *BeanDefinitionRegistry* 与 Bean工厂 *BeanFactory* 有一个共同的实现类 ***DefaultListableBeanFactory***。 *DefaultListableBeanFactory* 的类关系图如下所示：
![[../picture/Pasted image 20231210175800.png#pic_center|650]]
&emsp;&emsp;&emsp;   在 *Bean* 注册到容器时，由于定义 *Bean* 的时候有父子 *Bean* 关系，此时子 *BeanDefinition* 中的信息是不完整的，比如设置属性的时候配置在父 *BeanDefinition* 中，此时子 *BeanDefinition* 中是没有这些信息的，需要将子 *Bean* 的 *BeanDefinition* 和父 *Bean* 的 *BeanDefinition* 进行合并，同时 *Bean* 定义可能存在多级父子关系，合并的时候进进行递归合并，最终得到一个包含完整信息的 *RootBeanDefinition*，*RootBeanDefinition* 包含 *Bean* 定义的所有信息，也包含了从父 *Bean* 中继继承过来的所有信息，后续 *Bean* 的所有创建工作就是依靠合并之后 *BeanDefinition* 来进行的。
###### (1). ***Bean* 注册方式**
&emsp;&emsp; Spring Bean的注册方式就是 Bean 元信息的配置方式，其注册方式分为以下几种方式，其中 *BeanFactoryPostProcessor* 为 *Bean* 工厂后置处理器，*BeanDefinitionRegistryPostProcessor* 为 *Bean* 注册后置处理器，*FactoryBean* 是在调用 `getObject()` 对 *Bean* 进行实例化的同时注册到容器中。
![[../picture/Pasted image 20231210175927.png#pic_center|480]]
▨  **基于 *XML* 配置 *Bean***

&emsp;&emsp;  基于 *XML* 文件配置 *Bean* 对象，通过` <bean id="...", class="..."/>` 来标注 *Bean* 对象及其对应的类。在Spring中，可以通过 *ClassPathXmlApplicationContext* 来获取通过XML文件配置的 *Bean* 对象容器。基于 *XML* 配置 *Bean* 是Spring最早支持的方式，现在基本上不在使用 xml 的方式来配置 *Bean*。
![[../picture/Pasted image 20231210180741.png#pic_center|680]]
▨  **使用注解 *@Component* + *@ComponentScan* 方式注册 *Bean***

&emsp;&emsp;&emsp; 使用 *XML* 文件进行配置时，*Bean* 定义信息和 *Bean* 实现类本身是分离的，同时大量的配置信息会导致配置文件臃肿。为了解决这一问题，在Spring 2.5中开始支持**基于 *@Component* 注解的配置方式，在需要加载到IoC容器的 *Bean* 实现类上标注 *@Component* 注解**，即可表示将此类标记为Spring容器中的一个 *Bean*。Spring自带四种 *Bean* 注解：*@Component* 注解及扩展 *@Repository*、*@Service*、*@Controller*。
![[../picture/Pasted image 20231210180925.png#pic_center|480]]
&emsp;&emsp;&emsp; 定义Spring Bean的第一步是使用注解 *@Component* / *@Service* / *@Repository* / *@Controller*， 但是将 *Bean* 加载到IoC容器时，Spring还需要知道这些*Bean* 实现类是定义在哪些 *package* 包中。<font color=green>默认情况下，在Spring启动过程中，Spring 只能扫描到 *@SpringBootApplication* 注解标注的启动类所在的包及其下级包中的*Bean*，定义在其他包中的 *Bean* 是无法被Spring扫描，也就无法加载到IoC容器中。因此，如果项目中所有的类都定义在启动类的包及其子包下，则不需要指定被扫描的包路径，如果存在一些类不在启动类的包及其子包下，通常在启动类中会使用 *@ComponentScan* 注解来标注哪些包需要被扫描的，被扫描到的包，包中的 *Bean* 就可以加载到IoC容器中</font>。需要注意的是，<font color=red>**一旦通过 *@ComponentScan* 指定了需要扫描的包路径，Spring将会在被指定的包及其下级包中寻找*Bean*，框架原始的默认扫描效果就无效了**</font>。
![[../picture/Pasted image 20231210181051.png#pic_center|680]]

▨ **基于 *@Configuration* 配置类 + *@Bean* 注册 *Bean* 信息**

&emsp;&emsp;  如果项目中有使用到第三方类库中的工具类时，我们是无法修改第三方打包好的类库。对此，Spring提供了 `@Configuration` 注解，<font color=green>**`@Configuration `标注在类上，相当于把该类作为 Spring 的 Xml 配置文件中的`<beans>`来配置 Spring 容器**。</font>同时被注解的类内部包含有一个或多个被 *@Bean* 注解的方法，这些方法将会被 `AnnotationConfigApplicationContext` 或 `AnnotationConfigWebApplicationContext` 类进行扫描，构建并定义 *Bean*对象，初始化Spring容器。
><font color=SlateBlue>  <u>**Q1. @Component 与 @Bean 的区别 ？**</u></font>
&emsp;&emsp;*@Component* 和 *@Bean* 的目的是一样的，都是注册 *Bean* 到IoC容器中。但两者在使用上存在以下区别：
&emsp; &emsp; ● *@Component* 作用在类上，通过类路径扫描 *@ComponentScan* 自动检测并注入到Spring容器中;
&emsp; &emsp; ● *@Bean* 不能注释在类上，只能用于在配置类中显式声明单个Bean。在应用开发的过程中，如果想要将第三方库中未装配到IoC的组件装配到IoC中，由于第三方库是已经封装好的，在这种情况下，是没有办法在它的类上添加 *@Component* 注解的，因此就不能使用自动化装配的方案，仅能在*@Configuration* 配置类中通过 *@Bean* 进行配置。

![[../picture/Pasted image 20231210181500.png#pic_center|680]]

▨ **基于 *@Import* 方式注册 *Bean* 信息**
&emsp;&emsp; 基于 *@Import* 注解提供了三种注册 *Bean* 的方法：
&emsp; &emsp;&emsp;  ① *@Import* 一个普通的类，Spring会将该类加载到 Spring 容器中;①
![[../picture/Pasted image 20231210181647.png#pic_center|680]]
&emsp; &emsp;&emsp;   ② *@Import* 一个类，该类实现了 *ImportBeanDefinitionRegistrar* 接口，在重写的 `registerBeanDefinitions()` 方法里面，能拿到 *BeanDefinitionRegistry* 的注册器，通过 *BeanDefinitionRegistry* 注册器能手动往 *beanDefinitionMap* 中注册 *beanDefinition*；
&emsp; &emsp;&emsp;   ③ *@Import* 一个类，该类实现了 *ImportSelector* 接口并重写 `selectImports()` 方法。该方法返回了 *String[ ]* 数组的对象，数组里面的类都会注入到Spring 容器当中；
###### (2). ***Bean* 注册后置处理器**
&emsp;&emsp;   后置处理器 ( *PostProcessor* ) 作为 *Spring* 框架的重要扩展点，可以通过扩展点实现自定义信息的处理。在 Bean 的注册阶段，有两个后置处理器，分别是：***Bean* 注册器后置处理器 - *BeanDefinitionRegistryPostProcessor***、***Bean* 工厂后置处理器 - *BeanFactoryPostProcessor***
![[../picture/Pasted image 20231210182316.png#pic_center|480]]
&emsp;&emsp;&emsp;   **①  注册器后置处理器 - *BeanDefinitionRegistryPostProcessor***：*BeanDefinitionRegistryPostProcessor* 继承了*BeanFactoryPostProcessor*，其注册时机是<font color=green>**在 *Bean* 定义之后，调用构造函数实例化之前执行**，主要作用是**用来注册更多的 *Bean* 到 *Spring* 容器中**</font>，可以通过其 `postProcessBeanDefinitionRegistry()` 方法的入参 `registry` 来完成对 *BeanDefinition* 的判断、注册、移除等操作，可以实现动态的注册 *BeanDefinition*。
```java
static class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor{
        @Override
        public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
            //自定义Bean的注册
            BeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(TestBean.class).getBeanDefinition();
            beanDefinition.getPropertyValues().add("name","自定义Bean注册");
            beanDefinition.getPropertyValues().add("age",100);
            registry.registerBeanDefinition("testSelfBeanDefinition",beanDefinition);
        } 
}
```
&emsp;&emsp;   **②  工厂后置处理器 - *BeanFactoryPostProcessor***：*BeanFactoryPostProcessor* 的注册时机是<font color=green>**在 *Bean* 定义之后，调用构造函数实例化之前执行**，主要作用是**用来对Bean定义进行一些改变**</font>，可以通过其 `postProcessBeanFactory()` 方法的入参 `beanFactory` 来对 *Bean* 定义的属性进行修改，定义改变之后，创建的 *Bean* 实例也会发生改变。
![[../picture/Pasted image 20231210220141.png#pic_center|680]]

##### 3. *Spring Bean* 实例化环节
&emsp;&emsp;   *Bean* 的实例化是 *Bean* 对象创建的过程，会为 *Bean* 对象在内存中分配空间。在整个实例化过程中，共分为三个阶段：**实例化前阶段、实例化阶段、实例化后阶段**。
![[../picture/Pasted image 20231210220233.png#pic_center|680]]

###### (1). 实例化前阶段
&emsp;&emsp;  *Bean* 实例化前阶段是 Spring 的一个扩展点，在目标 *Bean* 开始实例化前会触发该扩展点，并会调用实现 *InstantiationAwareBeanPostProcessor* 接口的 `postProcessBeforeInstantiation()` 方法。由于这个时候目标 *Bean* 对象还未实例化，所以在 `postProcessBeforeInstantiation()` 方法中可以通过代理对象实例来代替原本应该生成的目标对象的实例。<font color=red>**如果此方法返回非 *null* 对象，则 *Bean* 实例化过程将被短路，正常 *Bean* 实例化的后续流程不再执行**，后续只有初始化后阶段的`postProcessAfterInitialization()` 方法会调用，其它方法不再调用。</font>
![[../picture/Pasted image 20231210220322.png#pic_center|680]]

###### (2). 实例化阶段
&emsp;&emsp;  Spring Bean 实例化方法有四种：使用构造方法实例化 *Bean* ( 最常用 )、使用静态工厂实例化 *Bean*、使用实例工厂实例化 *Bean*、使用 *FactoryBean* 实例化 *Bean*。
&emsp; &emsp;&emsp;  **① 构造方法实例化**：通过无参构造的方法实例化 *Bean*，其实质是将 *Bean* 对应的类交给 Spring 自带的工厂 *BeanFactory* 来管理，由Spring自带的工厂模式帮我们创建和维护这个类。
&emsp; &emsp;&emsp;  **② 静态工厂实例化**：通过静态工厂创建并返回 *Bean*。其实质是将 *Bean* 对应的类交给我们自己的静态工厂管理。Spring只是帮我们调用了静态工厂创建实例的方法。在很多时候我们在使用第三方jar包提供给我们的类时，由于这个类没有构造方法，是通过第三方包提供的静态工厂创建的，如果我们想把第三方jar包里面的这个类交由Spring来管理的话，就可以使用Spring提供的静态工厂创建实例的配置。
&emsp;  &emsp;&emsp;   **③ 实例工厂实例化**：通过实例工厂创建并返回 *Bean*，其实质就是把创建实例的工厂类和调用工厂类的方法创建实例的方法的过程也交由Spring管理，创建实例的这个过程也是由我们自己配置的实例工厂内部实现的。如 Spring 整合 *Hibernate* 就是通过这种方式实现的。但对于没有与 Spring 整合过的工厂类，我们一般都是自己用代码来管理的。
&emsp;  &emsp;&emsp; ④ <font color=red>***FactoryBean* 实例化：**</font> *FactoryBean* 实例化是 Spring IoC 容器Bean实例化逻辑的一个扩展点，是 Spring 提供的一种整合第三方框架的常用机制，其本身也是一个**特殊的 *Bean***，其作用是<font color=red>**允许自定义一个对象通过 *FactoryBean* 的 `getObject()` 方法间接的放到 Spring 容器中成为一个 *Bean***</font>。例如：在使用Mybatis的时候，我们定义的 *Dao* 层都是接口，但是在 Spring 容器中是存在对应的 *Bean* 对象，这里就是用了 *FactoryBean* 进行注册的。因此，如果有复杂的初始化 *Bean* 逻辑，则可以选择创建自定义 *FactoryBean*，在该类中的 `getObject()` 方法中进行初始化逻辑，即可把自定义 *FactoryBean* 注入到容器中。在Spring中当一个 *Bean* 的类型是 *FactoryBean* ，此时实例化时候就会执行该对象的 `getObject()` 方法来进行实例化。<font color=green>在获取 *Bean* 对象的时候，在 *beanName* 前面加上 & 符号，则会拿到 *factoryBean* 自身的对象了，而不是`getObject()` 返回的 *Bean* 对象。</font>
![[../picture/Pasted image 20231210220704.png#pic_center|580]]

###### (3). *Bean* 元信息获取
&emsp;&emsp;  在 Bean 完成实例化后，此时还没有进行后续的 *Bean* 属性注入、初始化等流程。Spring 通过回调 *MergedBeanDefinitionPostProcessor* 的`postProcessMergedBeanDefinition()` 方法对一些元信息 (主要是 *Bean* 类上的注解) 做了收集维护处理，如*@Autowire*、*@Resource*、*@PostConstruct* 和 *@PreDestroy* 等，为后续 *Bean* 属性注入做准备。
```java
@Component
static class MyMergedBeanDefinitionPostProcessor implements MergedBeanDefinitionPostProcessor {
    @Override
    public void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName) {
        if ("beanLifeCycle".equals(beanName)) {
            log.info(">>>>>>>>>>元信息收集 ，MergedBeanDefinitionPostProcessor#postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName) nbeanDefinition = [{}]n,beanType = [{}],beanName = [{}]n", beanDefinition, beanType, beanName);
        }
    }
}
```
###### (4). 实例化后阶段
&emsp;&emsp;   *Bean* 实例化后阶段也是 Spring 的一个扩展点。在该环节时，对象已经被实例化但是该实例的属性还未被设置，都是null。*Bean* 实例化后阶段主要作用是一个**属性获取控制器**。在目标对象实例化之后会调用实现 *InstantiationAwareBeanPostProcessor* 接口的 `postProcessAfterInstantiation()` 方法。<font color=red>`postProcessAfterInstantiation()` 方法的返回值会决定要不要调用 `postProcessProperties()` 方法，如果该方法返回 *false*，那么 `postProcessProperties()` 方法就会被忽略不执行；如果返回 *true*，`postProcessProperties()` 就会被执行，通过 `postProcessProperties()` 方法可以对 *Bean* 对象的实例属性进行设置。</font>注意：`postProcessProperties()` 方法主要用来对 XML 配置 *Bean* 方式的 \<property> 标签定义的属性进行处理。
![[../picture/Pasted image 20231210220901.png#pic_center|680]]

###### (5). 实例化后 *Bean* 属性赋值阶段
&emsp;&emsp;  经过实例化后阶段，*Bean* 对象已经被实例化，且对 *Bean* 的属性进行了前置的处理，随后就需要对Bean对象的属性进行赋值，即循环处理 *PropertyValues* 中的属性值信息，通过**反射调用 set 方法将属性的值设置到 *Bean* 实例中**。*PropertyValues* 中的值是通过 Bean xml中\<property> 元素配置的，或者调用 *MutablePropertyValues* 中 add 方法设置的属性值。

##### 4. *Spring Bean* 初始化环节
&emsp;&emsp;  *Bean* 初始化是为 *Bean* 对象中的属性赋值。Bean 初始化分为以下五个阶段：***Bean Aware* 接口回调、*Bean* 初始化前操作、*Bean* 初始化、*Bean* 初始化后操作、*Bean* 初始化完成操作**。
![[../picture/Pasted image 20231210220949.png#pic_center|580]]

###### (1). *Bean Aware* 接口回调
&emsp;&emsp;  在 Spring 的思想中，**解耦并不只是业务代码间的解耦，还包括业务代码与框架间的解耦。** *Spring* 有意识的隔离了框架代码和业务代码，所以在业务代码中是无法感知和使用 *Spring* 框架的一些方法的。如果我们需要使用 *Spring* 一些方法，比如获得上下文，比如获得 *Bean* 容器，这时应该怎么办呢？Spring 提供了一种感知 Spring 框架功能的能力，那就是 Aware 接口。*Aware* 接口是一个具有标识作用的接口，其含义是“感知捕获”，主要作用是辅助 *Spring Bean* 访问 Spring 容器，并拿到 *Spring* 容器内部的一些资源。Aware 接口有很多的子接口，这些子接口表明需要使用的 Spring 资源，使用 Aware 接口，我们基本可以获得 Spring 所有的核心对象，代价是业务代码和 Spring 框架强耦合。
&emsp;&emsp;&emsp;  单纯的 *Bean* 都未实现 *Aware* 接口，这时的 *Bean* 对 *Spring* 容器是没有感知的。对于实现了 *Aware* 接口的 *Bean* 可以访问 Spring 容器，*Aware* 接口增强了 *Spring Bean* 的功能，但是也会造成 *Bean* 对 *Spring* 框架的绑定，增大了与 *Spring* 框架的耦合度。常见的 Aware 接口如下：
![[../picture/Pasted image 20231210222653.png#pic_center|580]]

###### (2). *Bean* 初始化前操作
&emsp;&emsp;  *Bean* 初始化前阶段是 Spring 的一个扩展点，在目标 *Bean* 实例化、依赖注入完毕，开始初始化前会触发该扩展点。通过调用实现 *BeanPostProcessor* 接口的 `postProcessBeforeInitialization()` 方法，完成一些自定义的初始化逻辑。<font color=green>**`postProcessBeforeInitialization()` 方法返回值不能为 *NULL*** </font>，如果返回 *NULL* 会使得从Spring IoC容器中取出 Bean 实例对象没有再次放回IoC容器中，从而导致后续初始化方法会报空指针异常或者通过 `getBean()`方法获取不到 *Bean* 实例。
```java
public interface BeanPostProcessor {
    @Nullable
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;  //初始化前操作，返回值不能为NULL
    }
}
```
###### (3). *Bean* 初始化

**▨  *Bean* 初始化方式：**

&emsp;&emsp;*Bean* 的初始化有两种初始化方式：<font color=red>**实时初始化、延迟初始化**</font>
&emsp; &emsp;&emsp; ①  **实时初始化**：在容器启动过程中被创建组装好的 *Bean*，称为实时初始化的 *Bean*，Spring中默认定义的 *Bean* 都是实时初始化的 *Bean*，这些 *Bean* 默认都是单例的，在容器启动过程中会被创建好，然后放在Spring容器中以供使用。但是如果程序中定义的 *Bean* 非常多，并且有些 *Bean* 创建的过程中比较耗时的时候，会导致系统消耗的资源比较多，会让整个启动时间比较长。实时初始化的 *Bean* 有以下优点：
&emsp; &emsp; &emsp;  ● 更早的发现 *Bean* 定义的错误：实时初始化的 *Bean* 如果定义有问题，会在容器启动过程中会抛出异常。
&emsp; &emsp; &emsp;  ● *Bean* 的查找速度更快：容器启动完成后，实时初始化的 *Bean* 已经创建完成，并缓存到Spring容器中，当我们使用时，容器直接获取 *Bean* 即可。
&emsp; &emsp;&emsp; ②  **延迟初始化**：和实时初始化刚好相反，延迟初始化的 *Bean* 在容器启动过程中不会创建，而是需要使用的时候才会去创建。当 *Bean* 被其他 *Bean* 作为依赖进行注入的时候( 如：通过构造器注入、通过setter注入、通过自动注入 ) 会导致被依赖Bean的创建，或者通过Spring 容器的 `getBean()` 方法获取Bean 时也会导致 *Bean* 的创建。延迟初始化的 Bean 无法在程序启动过程中发现 *Bean* 定义的问题，只有在创建 *Bean* 的时候 *Bean* 定义的问题才会暴露。

**▨  *Bean* 初始化步骤：**

&emsp;&emsp;*Bean* 的初始化方法为 `invokeInitMethods()`。 Bean 初始化分为两个步骤：
&emsp;&emsp;&emsp; ① 如果 *Bean* 对应的 Class 实现了 *InitializingBean* 接口，则会执行 `afterPropertiesSet()`方法；
```java
public class TestInitializingBean implements InitializingBean {
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("TestInitializingBean");
    }
}
```
&emsp;  &emsp;  ② 如果 *Bean* 的 Class 不是 *NullBean* 会执行 `initMethodName()`方法，该方法会调用定义 *Bean* 的时候指定的初始化方法，如 `@Bean(initMethod="")` 或 \<bean> 标签 *init-method* 属性指定；

###### (4). *Bean* 初始化后操作
&emsp;&emsp; 在目标 *Bean* 初始化后会触发该扩展点，通过调用实现 *BeanPostProcessor* 接口的 `postProcessAfterInitialization()` 方法，完成一些自定义的初始化逻辑。<font color=green>**`postProcessAfterInitialization()` 方法返回值不能为 *NULL*** </font>，如果返回 *NULL* 会使得从Spring IoC容器中取出 Bean 实例对象没有再次放回IoC容器中，从而导致后续初始化方法会报空指针异常或者通过 `getBean()`方法获取不到 *Bean* 实例。
```java
public interface BeanPostProcessor {
     @Nullable
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;	//初始化后操作，返回值不能为NULL
    }
} 
```
##### 5. *Spring Bean* 销毁环节
###### (1). *Bean* 销毁方式
&emsp;  &emsp; 在 *Bean* 的生命周期中，在完成了 *Bean* 的创建之后，会注册 *Bean* 销毁的逻辑。在Spring容器关闭的时候，会去销毁所有的单例 *Bean*，并不是只有注册了销毁逻辑的 *Bean* 才被销毁，注册了销毁逻辑的单例 *Bean* 在销毁之前，会调用它们注册的销毁逻辑。*Bean* 的销毁有三种方式：
&emsp;&emsp;&emsp; ① 在 *Bean* 对象中添加销毁方法，并在方法上标注 *@preDestory* 注解，该方式是标准的 *Bean* 销毁方式。
&emsp;&emsp;&emsp;  ② *Bean* 对象实现 *DisposableBean* 接口，并重写 `destroy()` 方法。
&emsp; &emsp;&emsp;  ③ 自定义销毁方法，包括 Xml 配置 ( 如：在Xml的 Bean 配置下增加 *destroy-method* 属性 -->` <bean destroy-method="bean中方法名称"/>` )、Java 注解 ( 如: 在注解 *@Bean* 中指定属性 *destoryMethod* --> ``@Bean(destroyMethod = "初始化的方法")``)、API方式指定销毁方法 ( 如：通过 *BeanDefinition* 的方法 `setDestroyMethodName()` 设置  --> `this.beanDefinition.setDestroyMethodName(methodName)`)
```java
@Bean(initMethod = "init", destroyMethod = "destroyMethod")
public BeanLifeCycle beanLifeCycle() {
     return new BeanLifeCycle();
}

public class BeanLifeCycle implements InitializingBean, MyAware, DisposableBean {
    //Bean销毁方式1：通过@Bean注解方式定义
    public void destroyMethod() {
        log.info("          BeanLifeCycle destroy-method");
    }

    //Bean销毁方式2：通过注解@PreDestroy方式
    @PreDestroy
    public void preDestory(){
        log.info("          BeanLifeCycle DisposableBean-preDestory");
    }

    //Bean销毁方式3：Bean对象实现DisposableBean接口，并重写destroy()方法
    @Override
    public void destroy() {
        log.info("          BeanLifeCycle DisposableBean-destroy");
    }
}
```
###### (2). *Bean* 销毁流程
&emsp; &emsp; 在Spring容器关闭过程时，会对Bean进行销毁。
&emsp;&emsp;&emsp; ① 首先发布 *ContextClosedEvent* 事件；
&emsp;&emsp;&emsp;  ② 其次调用 *lifecycleProcessor* 的 `onCloese()` 方法；
&emsp;&emsp;&emsp;  ③ 销毁单例 *Bean*，首先会在 `destroySingletons()` 方法中取出在缓存中存储的 *disposableBeans*，随后遍历 *disposableBeans*，把每个 *disposableBean* 从单例池中移除，调用 *disposableBean* 的 `destroy()`进行销毁。
&emsp;&emsp;&emsp;   ●  如果这个 *disposableBean* 还被其他 *Bean* 依赖了，那么也会销毁其他 *Bean*。
&emsp;&emsp;&emsp;   ●  如果这个 *disposableBean* 还包含了内部 *Beans*，将这些 *Bean* 从单例池中移除掉。
&emsp;&emsp;&emsp;  ④ 清空 *manualSingletonNames*，这是一个Set，存放的是用户手动注册的单例 *Bean* 的 *beanName*
&emsp;&emsp;&emsp;  ⑤ 清空 *allBeanNamesByType*，是一个Map，key是bean类型，value是该类型所有的 *beanName* 数组
&emsp;&emsp;&emsp;  ⑥ 清空 *singletonBeanNamesByType*，和 *allBeanNamesByType* 类似，只不过只存了单例 *Bean*
![[../picture/Pasted image 20231210223437.png#pic_center|580]]
#### 3.1.2  *Bean* 的注入

&emsp;&emsp; 配置完Bean类后，IoC容器启动时，会根据配置将Bean类实例化，并”注册“到IoC容器当中。当应用程序需要使用Bean类时，通过 ***Bean* 注入**的方式，将 *Bean* 的初始化后的对象注入到程序中，从而在程序中可以不创建对象就直接使用 *Bean*。对于 *Bean* 的注入有两种方式：**XML配置注入** ( 属性注入、构造函数注入和工厂方法注入) 和 **注解注入** ( *@Autowired、@Resource、@Required* )。

##### 1. XML配置注入
&emsp;&emsp; **● 属性注入**：属性注入通过 `setter()` 方法注入 *Bean* 的属性值或依赖。属性注入要求 *Bean* 提供一个默认的**构造函数**，并为需要注入的属性提供对应的`setter()` 方法。<font color=green>Spring先调用 *Bean* 的默认构造函数实例化 *Bean* 对象，然后通过反射的方式调用 `setter()` 方法注入属性值。</font>
&emsp;&emsp; **● 构造方法注入**：使用构造函数注入的前提是 *Bean* 必须提供带参数的构造函数
&emsp;&emsp; **● 工厂方法注入**：工厂类负责创建一个或多个目标类实例，工厂类方法一般以接口或抽象类变量的形式返回目标类实例。非静态工厂方法，必须实例化工厂类后才能调用工厂方法。静态工厂方法，无须创建工厂类实例就可以调用工厂类方法。
```java
package com.xzz.demo;
import org.springframework.beans.factory.BeanNameAware;
// Bean类
public class Service implements BeanNameAware{
    private LogDao logDao;
    private UserDao userDao;
  	public LogonService(LogDao logDao, UserDao userDao) {  //构造方法注入
        this.logDao = logDao;
        this.userDao = userDao;
    }
    public void setUserDao(UserDao userDao) {    //属性注入 - setter方法注入
        this.userDao = userDao;
    }
    public void setLogDao(LogDao logDao) {
        this.logDao = logDao;
    }   
    public LogDao getLogDao() {
        return logDao;
    }
    public UserDao getUserDao() {
        return userDao;
    }    
}
```
```xml
// bean.xml配置文件
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
         http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
         http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context-3.0.xsd"
       default-autowire="byName">
  	<!-- 注册Bean -->
    <bean id="logDao" class="com.xzz.demo.LogDao"/>
    <bean id="userDao" class="com.xzz.demo.UserDao"/>
  
    <bean class="com.xzz.demo.Service">   <!-- 属性注入 -->
       <property name="logDao" ref="logDao"></property>
       <property name="userDao" ref="userDao"></property>
    </bean>
  
  	<bean class="com.xzz.demo.Service">   <!-- 构造方法注入 -->
       <constructor-arg ref="logDao"></constructor-arg>
       <constructor-arg ref="userDao"></constructor-arg>
    </bean>
</beans>
```
##### 2. 注解注入
&emsp;&emsp; 同样的XML方式注入需要通过XML配置文件进行配置。为了减少配置文件的”冗余“，Spring提供了<font color=red> *@Autowire* </font>注解实现 *Bean* 的依赖注入。**在使用 *@Autowired* 注解时，注入的 *Bean* 必须是被 Spring 管理的，即 *Bean* 对象已经注册到 Spring IoC 容器中**。
```java
package com.xzz.demo;
import org.springframework.beans.factory.BeanNameAware;
// Bean类
@Service
public class Service implements BeanNameAware{
    @Autowired	 //通过@Autowired注入LogDao的Bean
    private LogDao logDao;
    @Autowired  //通过@Autowired注入UserDao的Bean
  	@Qualifier("userDao")  //通过@Qualifier可以对注入Bean类型进行限定
    private UserDao userDao;
  	public LogonService(LogDao logDao, UserDao userDao) {  //构造方法注入
        this.logDao = logDao;
        this.userDao = userDao;
    }
    public void setUserDao(UserDao userDao) {    //属性注入 - setter方法注入
        this.userDao = userDao;
    }
    public void setLogDao(LogDao logDao) {
        this.logDao = logDao;
    }   
    public LogDao getLogDao() {
        return logDao;
    }
    public UserDao getUserDao() {
        return userDao;
    }    
}
```
#### 3.1.3 _Bean_ 的特性

#### 3.1.4 _Bean_ 的循环依赖

### 3.2 Spring 容器
&emsp;&emsp; Spring 容器又称为IoC容器，是具有依赖注入功能的容器，其本质是**一个对象工厂，通过反射创建 *Bean* 对象**，并负责***Bean* 对象的实例化、*Bean* 对象的初始化，*Bean* 对象之间依赖关系配置、*Bean* 对象的销毁、对外提供 *Bean* 对象的查找**等操作，*Bean* 对象的整个生命周期都是由容器来控制。我们需要使用的 *Bean* 对象都由IoC容器进行管理，不需要我们再去手动通过 *new* 的方式去创建对象，由 IoC容器直接帮我们组装好，当我们需要使用的时候直接从IoC容器中直接获取 (注入) 就可以了。
#### 3.2.1 *Spring* 容器的实现方式
&emsp;&emsp;Spring 提供的 IoC 容器有两种实现方式：
&emsp;&emsp;&emsp;● ***BeanFactory*** 接口：该接口是 Spring 容器最基本的实现，提供了完整的 IoC 服务支持，它主要负责初始化各种 *Bean*，并调用它们的生命周期方法。*BeanFactory* 是 Spring 内部使用的接口，不提供给开发人员使用。
&emsp;&emsp;&emsp;● ***ApplicationContext*** 接口：*BeanFactory* 接口的子接口，也被称为应用上下文，它不仅提供了 *BeanFactory* 的所有功能，还添加了对国际化、资源访问、事件传播等方面的良好支持。*ApplicationContext* 面向 Spring 的使用者，几乎所有场合都使用 *ApplicationContext* 而不是底层的 *BeanFactory*。*ApplicationContext* 容器主要有三个实现类：
&emsp; &emsp;&emsp;    ① *ClassPathXmlApplicationContext*：该类从类路径 *ClassPath* 中寻找指定的 XML 配置文件，找到并装载完成 *ApplicationContext* 的实例化工作。
&emsp; &emsp;&emsp;   ② *FileSystemXmlApplicationContext*：该类从指定的文件系统路径中寻找指定的 XML 配置文件，找到并装载完成 *ApplicationContext* 的实例化工作。
&emsp; &emsp;&emsp;    ③ *AnnotationContigApplicationContext*：用于读取注解创建容器；

### 3.3 Spring 启动流程
&emsp;&emsp;*SpringBoot* 的启动流程主要包含两个步骤: 
&emsp;&emsp;&emsp;● **起步依赖**：通过 *pom* 依赖来解决版本管理和 *jar* 包引用的问题；
&emsp;&emsp;&emsp;● **自动装配**；引入相关的 *jar* 包后，*SpringBoot* 会自动注册一些比较关键的 *Bean*，并进行默认配置，不用我们进行特殊配置。*SpringBoot* 启动依靠的是带有 `main()` 方法的启动类，启动类的内容可以分为两个部分:
&emsp; &emsp; &emsp;  ① 启动类上*@SpringBootApplication* 这个注解
&emsp; &emsp;  &emsp; ② `main()` 方法里的 `SpringApplication.run(启动类.class，args)` 方法
#### 3.3.1 *@SpringBootApplication* 注解
&emsp;&emsp; *@SpringBootApplication* 注解主要由三个注解组成 *@ComponentScan*、*@SpringBootConfiguration*、*@EnableAutoConfiguration*。
![[../picture/Pasted image 20231210223954.png#pic_center|480]]

##### 1. *@EnableAutoConfiguration*
&emsp;&emsp; *@EnableAutoConfiguration* 此注解的作用是从 *classpath* 路径下搜索所有的 *META-INF/spring.factories* 配置文件，然后将其中 Key 为*org.springframework.boot.autoconfigure.EnableAutoConfiguration* 的Value加载到 *Spring* 容器中。*@EnableAutoConfiguration* 注解由 *@Import* + *@AutoConfigurationPackage* 两个注解组成。
&emsp; &emsp;&emsp;   ① *@AutoConfigurationPackage*：主要作用是自动配置包，并向 Spring 容器中导入组件，导入的组件由 *AutoConfigurationPackages.Registrar.class* 将主配置类所在的包以及下面所有子包里面的所有组件扫描到 *Spring* 容器中。
&emsp; &emsp;&emsp;   ② *@Import*：主要作用是给 *Spring* 容器导入组件。*@Import* 中的参数 *AutoConfigurationImportSelector* 的作用是导入哪些组件的选择器。将所需要导入的组件以全类名的方式返回数组，这些组件就会被添加到容器中。通过 `selectImports()` 方法，将配置类信息交给 *SpringFactory* 加载器进行一系列的容器创建的过程

##### 2. *@SpringBootConfiguration*
&emsp;&emsp; *@SpringBootConfiguration* (内部为 *@Configuration* )：被标注的类等于在spring的XML配置文件中( *applicationContext.xml* )，装配所有 *Bean* 事务，提供了一个 Spring 的上下文环境。

#### 3.3.2 *SpringApplication* 启动
&emsp;&emsp;  *SpringApplication* 的启动包含两个过程：1.实例化 *SpringApplication* 对象；2.调用 *SpringApplication.run()* 方法。
![[../picture/Pasted image 20231210224141.png#pic_center|680]]

### 3.4 Spring 统一资源加载策略
&emsp;&emsp; 在 Spring 中有很多 *Xml* 配置文件，同时还包括自己创建的各种 *properties* 资源文件，还有可能进行网络交互，收发各种文件、二进制流等。这些资源粗略可分为：*URL* 资源、*File* 资源、*ClassPath* 相关资源、服务器相关资源。Spring 把这些文件、二进制流统称为资源。程序对这些资源的访问，就叫做资源访问。针对不同的资源文件，其处理资源文件步骤都是相同的：**定义资源、读取资源、关闭资源**。因此 Spring 抽象出一个统一的接口<font color=red> ***Resource*** </font>接口来对这些底层资源进行统一访问，其类关系如下图所示：
![[../picture/Pasted image 20231210224216.png#pic_center|680]]

#### 3.4.1 Spring 资源的描述与定义
&emsp;&emsp; 针对不同类型的资源，Spring 基于 *Resource* 接口作为**所有资源的抽象和访问接口**，并派生出不同的资源描述类：
&emsp;&emsp;&emsp;● ***UrlResource***：代表URL资源，用于简化URL资源访问，是对 *java.net.URL* 的包装。在 Java 中，将不同来源的资源**抽象成URL**，通过注册不同的 *handler* 来处理不同来源的资源的读取逻辑。一般不同类型使用不同的前缀。
&emsp;  &emsp;  &emsp;   *HTTP*：通过标准的 HTTP 协议访问web资源，`new UrlResource(“http://地址”); `
&emsp;  &emsp;  &emsp;    *FTP*：通过 FTP 协议访问资源，`new UrlResource(“ftp://地址”); `
&emsp;  &emsp;  &emsp;     *FILE*：通过 FILE 协议访问本地文件系统资源，`new UrlResource(“file:d:/test.txt”);`
&emsp;&emsp;&emsp;●  ***ClassPathResource***：代表 *classpath* 路径的资源，*classpath* 资源存在于类路径中的文件系统中或 jar包中。*ClassPathResource* 将使用 *ClassLoader* 进行加载资源。主要优势是方便访问类加载路径下的资源，尤其是Web应用，因为它**可以自动搜索位于 WEB-INF/classes 下的资源文件**，当Spring获取资源时，如果路径字符串前缀是 `classpath:`，则系统将会**自动创建** *ClassPathResource* 对象。
&emsp;&emsp;&emsp;●  ***FileSystemResource***：代表 *java.io.File* 资源，当Spring获取资源时，如果路径字符串前缀是 `file:`，则系统将会自动创建 *FileSystemResource* 对象。
&emsp;&emsp;&emsp;●  ***ServletContextResource***：用于访问 *Web Context* 下相对路径下的资源，入参的资源位置是相对于**Web应用根路径**的位置 ( 根路径在工程文件夹下，*WEB-INF* 所在层级的文件夹 ) 。主要是用于简化 *Servlet* 容器的 *ServletContext* 接口的 `getResource()` 操作和 `getResourceAsStream()` 操作。 
&emsp;&emsp;&emsp;●  ***InputStreamResource***： 代表 *java.io.InputStream* 字节流，只有当没有合适的 *Resource* 实现时，才考虑使用 *InputStreamResource*。一般考虑使用*ByteArrayResource*。
&emsp;&emsp;&emsp;●  ***ByteArrayResource***：读取数组资源，可以把从网络传输数据或者本地资源的输入流都转换为 *byte[]* 类型，然后用 *ByteArrayResource* 转化为资源。

#### 3.4.2 Spring 资源加载策略
&emsp;&emsp;  为了更方便的获取资源，弱化对各个 *Resource* 接口实现类的感知与分辨，定义了<font color=red> ***ResourceLoader*** </font>接口，用来加载不同类型的 *Resource* 实例 。该接口中的 `Resource getResource(String location)` 方法返回的对象就是Spring 容器中 *Resource* 接口的实例。<font color=green>Spring 内所有的应用程序上下文都实现了 *ResourceLoader* 接口 ，都实现了这个方法，因此在使用 Spring 都可以会获取  *Resource* 实例去获取相关的资源数据。</font>
![[../picture/Pasted image 20231210224424.png#pic_center|680]]

### 3.5 Spring Aop
&emsp; &emsp; *AOP* 称为面向切面编程，**通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。**利用 *AOP*可以对业务逻辑的各个部分进行隔离，可以无侵入的在原本功能的切面层添加自定义代码，从而<font color=red>**增强原有代码逻辑功能，同时也使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性**</font>。切面就是把非业务逻辑相关的代码抽取出来定位到具体的连接点上的一种实现方式。
> <font color=SlateBlue>  <u>**Q1. *AOP* 与 *OOP* 的区别 ？**</u></font>
> &emsp; &emsp;&emsp;*AOP* (面向切面编程) 和 *OOP* (面向对象编程) 是不同领域的两种思想，*OOP* 主要是针对业务处理过程中的<font color=green>**实体的属性和行为的抽象与封装**</font>，以获得更加清晰高效地逻辑单元。*AOP* 是针对业务处理过程中的切面进行提取，面对的是<font color=green>**处理过程中某个步骤或阶段**</font>，以获得逻辑过程中各部分之间低耦合性的隔离效果。
> 
> <font color=SlateBlue>  <u>**Q2. *AOP* 的应用场景？**</u></font>
> &emsp;&emsp;&emsp;只要系统的业务开发过程中大部分都使用通用模块都可以通过AOP来实现。如: ① 参数校验和判空; ② 数据的埋点; ③ 日志记录; ④ 事务处理; ⑤ 权限与安全控制; ⑥ 异常处理。

![[../picture/Pasted image 20231210224551.png#pic_center|480]]
&emsp;&emsp;  *AOP* 主要有以下几个概念:
&emsp; &emsp;&emsp; **① 连接点 ( *joint point* )**：具体的切面点的抽象概念，可以是在字段、方法上。Spring中具体表现形式是切入点( *PointCut* )，仅作用在方法上，包括方法调用，对类成员的访问以及异常处理程序块的执行等。
&emsp; &emsp;&emsp; **② 增强/通知 ( *Advice* ) **：在连接点进行的具体操作，如何对原有逻辑进行增强处理的，分为前置( ***Before*** )、后置( ***After*** )、异常( ***AfterThrowing*** )、最终( ***AfterReturning*** )、环绕( ***Around*** )五种情况。
&emsp; &emsp;&emsp; **③ 目标对象 ( *Target* ) **：被 *AOP* 框架进行增强处理的对象，也被称为被增强的对象。
&emsp; &emsp;&emsp; **④ 织入 ( *Weaving* ) **：将增强处理添加到目标对象中，通过动态代理的方式创建一个被增强的对象的过程。
&emsp; &emsp;&emsp; **⑤ 切面 ( *Aspect* ) **：包含着一些 *Pointcut* 以及相应的 *Advice* 的抽象概述。
&emsp; &emsp;&emsp; **⑥ 切点 ( *Pointcut* ) **：表示一组 *joint point*，作用就是提供一组规则来匹配 *joint point*, 给满足规则的 *join point* 添加 *Advice*。
![[../picture/Pasted image 20231210224704.png#pic_center|580]]