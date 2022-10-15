# Spring

> 本文档所有代码均基于5.1.14版本

Spring 是一种轻量级开发框架，旨在提高开发人员的开发效率以及系统的可维护性。恰当使用 DI 或 IOC 时，可以开发松耦合应用，松耦合应用的单元测试可以很容易的进行。

为了降低Java开发的复杂性，Spring采取了以下4种关键策略

- 基于POJO的轻量级和最小侵入性编程；
- 通过依赖注入和面向接口实现松耦合；
- 基于切面和惯例进行声明式编程；
- 通过切面和模板减少样板式代码。



**Spring设计目标**：Spring为开发者提供一个一站式轻量级应用开发平台；

**Spring设计理念**：在JavaEE开发中，支持POJO和JavaBean开发方式，使应用面向接口开发，充分支持OO（面向对象）设计方法；Spring通过IoC容器实现对象耦合关系的管理，并实现依赖反转，将对象之间的依赖关系交给IoC容器，实现解耦；

**Spring框架的核心**：IoC容器和AOP模块。通过IoC容器管理POJO对象以及他们之间的耦合关系；通过AOP以动态非侵入的方式增强服务。IoC让相互协作的组件保持松散的耦合，而AOP编程允许你把遍布于应用各层的功能分离出来形成可重用的功能组件。



## Spring组成

Spring框架至今已集成了20多个模块，这些模块分布在以下模块中：

- 核心容器（Core Container）
- 数据访问/集成（Data Access/Integration）层
- Web层
- AOP（Aspect Oriented Programming）
- 设备支持（Instrumentation）
- 消息传输（Messaging）
- 测试（Test）



![1](../../Image/2021/04/210407.png)



| 组成部分                      | 概述                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| Core                          | 提供了框架的基本组成部分，包括控制反转（Inversion of Control，IOC）和依赖注入（Dependency Injection，DI）功能。 |
| Beans                         | 提供了BeanFactory，是工厂模式的一个经典实现，Spring将管理对象称为Bean。 |
| Context                       | 建立在Core和Beans模块的基础之上，提供一个框架式的对象访问方式，是访问定义和配置的任何对象的媒介。ApplicationContext接口是Context模块的焦点。 |
| Context-Support               | 持整合第三方库到Spring应用程序上下文，特别是用于高速缓存（EhCache、JCache）和任务调度（CommonJ、Quartz）的支持。 |
| Expression                    | 提供了强大的表达式语言去支持运行时查询和操作对象图。这是对JSP2.1规范中规定的统一表达式语言（Unified EL）的扩展。该语言支持设置和获取属性值、属性分配、方法调用、访问数组、集合和索引器的内容、逻辑和算术运算、变量命名以及从Spring的IOC容器中以名称检索对象。它还支持列表投影、选择以及常用的列表聚合。 |
| AOP                           | 提供了一个符合AOP要求的面向切面的编程实现，允许定义方法拦截器和切入点，将代码按照功能进行分离，以便干净地解耦。 |
| Aspects                       | 提供了与AspectJ的集成功能，AspectJ是一个功能强大且成熟的AOP框架。 |
| Instrument                    | 提供了类植入（Instrumentation）支持和类加载器的实现，可以在特定的应用服务器中使用。 |
| Messaging                     | 该模块提供了对消息传递体系结构和协议的支持。                 |
| JDBC                          | 提供了一个JDBC的抽象层，消除了烦琐的JDBC编码和数据库厂商特有的错误代码解析。 |
| orm                           | 为流行的对象关系映射（Object-Relational Mapping）API提供集成层，包括JPA和Hibernate。使用Spring-orm模块可以将这些O/R映射框架与Spring提供的所有其他功能结合使用，例如声明式事务管理功能。 |
| oxm                           | 提供了一个支持对象/XML映射的抽象层实现，例如JAXB、Castor、JiBX和XStream。 |
| jms（Java Messaging Service） | 指Java消息传递服务，包含用于生产和使用消息的功能。自Spring4.1以后，提供了与Spring-messaging模块的集成。 |
| tx                            | 支持用于实现特殊接口和所有POJO（普通Java对象）类的编程和声明式事务管理。 |
| web                           | 提供了基本的Web开发集成功能，例如多文件上传功能、使用Servlet监听器初始化一个IOC容器以及Web应用上下文。 |
| webmvc                        | 也称为Web-Servlet模块，包含用于web应用程序的Spring MVC和REST Web Services实现。Spring MVC框架提供了领域模型代码和Web表单之间的清晰分离，并与Spring Framework的所有其他功能集成。 |
| websocket                     | Spring4.0以后新增的模块，它提供了WebSocket和SocketJS的实现。 |
| Portlet                       | （已废弃）类似于Servlet模块的功能，提供了Portlet环境下的MVC实现。 |
| Spring-test                   | 支持使用JUnit或TestNG对Spring组件进行单元测试和集成测试。    |



## 设计模式

- **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。



### 工厂模式

Spring使用工厂模式可以通过 `BeanFactory` 或 `ApplicationContext` 创建 bean 对象。

**两者对比：**

- `BeanFactory` ：延迟注入(使用到某个 bean 的时候才会注入),相比于`BeanFactory`来说会占用更少的内存，程序启动速度更快。
- `ApplicationContext` ：容器启动的时候，不管你用没用到，一次性创建所有 bean 。`BeanFactory` 仅提供了最基本的依赖注入支持，`ApplicationContext` 扩展了 `BeanFactory` ,除了有`BeanFactory`的功能还有额外更多功能，所以一般开发人员使用`ApplicationContext`会更多。

ApplicationContext的三个实现类：

1. `ClassPathXmlApplication`：把上下文文件当成类路径资源。
2. `FileSystemXmlApplication`：从文件系统中的 XML 文件载入上下文定义信息。
3. `XmlWebApplicationContext`：从Web系统中的XML文件载入上下文定义信息。



### 代理模式

AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

**Spring AOP 就是基于动态代理的**，如果要代理的对象，实现了某个接口，那么Spring AOP会使用**JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用**Cglib** ，这时候Spring AOP会使用 **Cglib** 生成一个被代理对象的子类来作为代理，如下图所示：



### 单例模式

在我们的系统中，有一些对象其实我们只需要一个，比如说：线程池、缓存、对话框、注册表、日志对象、充当打印机、显卡等设备驱动程序的对象。事实上，这一类对象只能有一个实例，如果制造出多个实例就可能会导致一些问题的产生，比如：程序的行为异常、资源使用过量、或者不一致性的结果。



**使用单例模式的好处:**

- 对于频繁使用的对象，可以省略创建对象所花费的时间，这对于那些重量级对象而言，是非常可观的一笔系统开销；
- 由于 new 操作的次数减少，因而对系统内存的使用频率也会降低，这将减轻 GC 压力，缩短 GC 停顿时间。



**Spring 实现单例的方式：**

- xml:<bean id="userService" class="top.snailclimb.UserService" scope="singleton"/>``
- 注解：`@Scope(value = "singleton")`



Spring 通过 `ConcurrentHashMap` 实现单例注册表的特殊方式实现单例模式。Spring 实现单例的核心代码如下：

```java
// 通过 ConcurrentHashMap（线程安全） 实现单例注册表
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(64);

public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(beanName, "'beanName' must not be null");
        synchronized (this.singletonObjects) {
            // 检查缓存中是否存在实例  
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
                //...省略了很多代码
                try {
                    singletonObject = singletonFactory.getObject();
                }
                //...省略了很多代码
                // 如果实例对象在不存在，我们注册到单例注册表中。
                addSingleton(beanName, singletonObject);
            }
            return (singletonObject != NULL_OBJECT ? singletonObject : null);
        }
    }
    //将对象添加到单例注册表
    protected void addSingleton(String beanName, Object singletonObject) {
            synchronized (this.singletonObjects) {
                this.singletonObjects.put(beanName, (singletonObject != null ? singletonObject : NULL_OBJECT));

            }
        }
}
```



### 模板方法

模板方法模式是一种行为设计模式，它定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。 模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤的实现方式。

Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。一般情况下，我们都是使用继承的方式来实现模板模式，但是 Spring 并没有使用这种方式，而是使用Callback 模式与模板方法模式配合，既达到了代码复用的效果，同时增加了灵活性。



### 观察者模式

观察者模式是一种对象行为型模式。它表示的是一种对象与对象之间具有依赖关系，当一个对象发生改变的时候，这个对象所依赖的对象也会做出反应。

Spring 事件驱动模型就是观察者模式很经典的一个应用。



#### Spring驱动模型

##### 三种角色

###### 事件角色

`ApplicationEvent` (`org.springframework.context`包下)充当事件的角色,这是一个抽象类，它继承了`java.util.EventObject`并实现了 `java.io.Serializable`接口。

Spring 中默认存在以下事件，他们都是对 `ApplicationContextEvent` 的实现(继承自`ApplicationContextEvent`)：

- `ContextStartedEvent`：`ApplicationContext` 启动后触发的事件;
- `ContextStoppedEvent`：`ApplicationContext` 停止后触发的事件;
- `ContextRefreshedEvent`：`ApplicationContext` 初始化或刷新完成后触发的事件;
- `ContextClosedEvent`：`ApplicationContext` 关闭后触发的事件。



###### 事件监听者角色

`ApplicationListener` 充当了事件监听者角色，它是一个接口，里面只定义了一个 `onApplicationEvent（）`方法来处理`ApplicationEvent`。`ApplicationListener`接口类源码如下，可以看出接口定义看出接口中的事件只要实现了 `ApplicationEvent`就可以了。所以，在 Spring中我们只要实现 `ApplicationListener` 接口实现 `onApplicationEvent()` 方法即可完成监听事件。



###### 事件发布者角色

`ApplicationEventPublisher` 充当了事件的发布者，它也是一个接口。

`ApplicationEventPublisher` 接口的`publishEvent（）`这个方法在`AbstractApplicationContext`类中被实现，阅读这个方法的实现，你会发现实际上事件真正是通过`ApplicationEventMulticaster`来广播出去的。



##### 事件流程总结

1. 定义一个事件: 实现一个继承自 `ApplicationEvent`，并且写相应的构造函数；
2. 定义一个事件监听者：实现 `ApplicationListener` 接口，重写 `onApplicationEvent()` 方法；
3. 使用事件发布者发布消息: 可以通过 `ApplicationEventPublisher` 的 `publishEvent()` 方法发布消息。



```java
// 定义一个事件,继承自ApplicationEvent并且写相应的构造函数
public class DemoEvent extends ApplicationEvent{
    private static final long serialVersionUID = 1L;

    private String message;

    public DemoEvent(Object source,String message){
        super(source);
        this.message = message;
    }

    public String getMessage() {
         return message;
          }


// 定义一个事件监听者,实现ApplicationListener接口，重写 onApplicationEvent() 方法；
@Component
public class DemoListener implements ApplicationListener<DemoEvent>{

    //使用onApplicationEvent接收消息
    @Override
    public void onApplicationEvent(DemoEvent event) {
        String msg = event.getMessage();
        System.out.println("接收到的信息是："+msg);
    }

}
// 发布事件，可以通过ApplicationEventPublisher  的 publishEvent() 方法发布消息。
@Component
public class DemoPublisher {

    @Autowired
    ApplicationContext applicationContext;

    public void publish(String message){
        //发布事件
        applicationContext.publishEvent(new DemoEvent(this, message));
    }
}
```



当调用 `DemoPublisher` 的 `publish()` 方法的时候，比如 `demoPublisher.publish("你好")` ，控制台就会打印出:`接收到的信息是：你好` 。



### 适配器模式

适配器模式(Adapter Pattern) 将一个接口转换成客户希望的另一个接口，适配器模式使接口不兼容的那些类可以一起工作，其别名为包装器(Wrapper)。



#### Spring AOP

Spring AOP 的实现是基于代理模式，但是 Spring AOP 的增强或通知(Advice)使用到了适配器模式，与之相关的接口是`AdvisorAdapter` 。Advice 常用的类型有：`BeforeAdvice`（目标方法调用前,前置通知）、`AfterAdvice`（目标方法调用后,后置通知）、`AfterReturningAdvice`(目标方法执行结束后，return之前)等等。每个类型Advice（通知）都有对应的拦截器:`MethodBeforeAdviceInterceptor`、`AfterReturningAdviceAdapter`、`AfterReturningAdviceInterceptor`。Spring预定义的通知要通过对应的适配器，适配成 `MethodInterceptor`接口(方法拦截器)类型的对象（如：`MethodBeforeAdviceInterceptor` 负责适配 `MethodBeforeAdvice`）。



#### Spring MVC

在Spring MVC中，`DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由`HandlerAdapter` 适配器处理。`HandlerAdapter` 作为期望接口，具体的适配器实现类用于对目标类进行适配，`Controller` 作为需要适配的类。

Spring MVC 中的 `Controller` 种类众多，不同类型的 `Controller` 通过不同的方法来对请求进行处理。如果不利用适配器模式的话，`DispatcherServlet` 直接获取对应类型的 `Controller`，需要的自行来判断。



### 装饰者模式

装饰者模式可以动态地给对象添加一些额外的属性或行为。相比于使用继承，装饰者模式更加灵活。简单点儿说就是当我们需要修改原有的功能，但我们又不愿直接去修改原有的代码时，设计一个Decorator套在原有代码外面。

在 JDK 中就有很多地方用到了装饰者模式，比如 `InputStream`家族，`InputStream` 类下有 `FileInputStream` (读取文件)、`BufferedInputStream` (增加缓存,使读取文件速度大大提升)等子类都在不修改`InputStream` 代码的情况下扩展了它的功能。

Spring 中用到的包装器模式在类名上含有 `Wrapper`或者 `Decorator`。这些类基本上都是动态地给一个对象添加一些额外的职责。



## 总结

### 优点

- 方便解耦，简化开发。Spring就是一个大工厂，可以将所有对象的创建和依赖关系的维护，交给Spring管理。
- AOP编程的支持。Spring提供面向切面编程，可以方便的实现对程序进行权限拦截、运行监控等功能。
- 声明式事务的支持。只需要通过配置就可以完成对事务的管理，而无需手动编程。
- 方便程序的测试。Spring对Junit4支持，可以通过注解方便的测试Spring程序。
- 方便集成各种优秀框架。Spring不排斥各种优秀的开源框架，其内部提供了对各种优秀框架的直接支持（如：Struts、Hibernate、MyBatis等）。
- 降低JavaEE API的使用难度。Spring对JavaEE开发中非常难用的一些API（JDBC、JavaMail、远程调用等），都提供了封装，使这些API应用难度大大降低。



### 缺点

- Spring明明一个很轻量级的框架，却给人感觉大而全
- Spring依赖反射，反射影响性能
- 使用门槛升高，入门Spring需要较长时间
- Spring的配置是重量级的，需要大量的XML配置。



# Spring Core

这是基本的Spring模块，提供spring 框架的基础功能，BeanFactory 是 任何以spring为基础的应用的核心。Spring 框架建立在此模块之上，它使Spring成为一个容器。

Bean 工厂是工厂模式的一个实现，提供了控制反转功能，用来把应用的配置和依赖从真正的应用代码中分离。最常用的就是org.springframework.beans.factory.xml.XmlBeanFactory ，它根据XML文件中的定义加载beans。该容器从XML 文件读取配置元数据并用它去创建一个完全配置的系统或应用。



## IOC

### 概念

IoC（Inverse of Control:控制反转）是一种**设计思想**，就是 **将原本在程序中手动创建对象的控制权，交由Spring框架来管理。** IoC 在其他语言中也有应用，并非 Spring 特有。 **IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个Map（key，value）,Map 中存放的是各种对象。**

 **IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。**



控制反转即IoC (Inversion of Control)，它把传统上由程序代码直接操控的对象的调用权交给容器，通过容器来实现对象组件的装配和管理。所谓的“控制反转”概念就是对组件对象控制权的转移，从程序代码本身转移到了外部容器。

Spring IOC 负责创建对象，管理对象（通过依赖注入（DI），装配对象，配置对象，并且管理这些对象的整个生命周期。



#### IOC作用

- 管理对象的创建和依赖关系的维护。对象的创建并不是一件简单的事，在对象关系比较复杂时，如果依赖关系需要程序猿来维护的话，那是相当头疼的
- 解耦，由容器去维护具体的对象
- 托管了类的产生过程，比如我们需要在类的产生过程中做一些处理，最直接的例子就是代理，如果有容器程序可以把这部分处理交给容器，应用程序则无需去关心类是如何完成代理的



#### IOC优点

- IOC 或 依赖注入把应用的代码量降到最低。
- 它使应用容易测试，单元测试不再需要单例和JNDI查找机制。
- 最小的代价和最小的侵入性使松散耦合得以实现。
- IOC容器支持加载服务时的饿汉式初始化和懒加载。



#### 缺点

1. 生成一个对象的步骤变复杂了。
2. 对象生成因为是使用反射编程，在效率上有些损耗。



#### IOC功能

Spring 的 IoC 设计支持以下功能：

- 依赖注入
- 依赖检查
- 自动装配
- 支持集合
- 指定初始化方法和销毁方法
- 支持回调某些方法（但是需要实现 Spring 接口，略有侵入）

其中，最重要的就是依赖注入，从 XML 的配置上说，即 ref 标签。对应 Spring RuntimeBeanReference 对象。对于 IoC 来说，最重要的就是容器。容器管理着 Bean 的生命周期，控制着 Bean 的依赖注入。



### 实现原理

IOC原理就是通过 Java 的反射技术来实现的，通过反射我们可以获取类的所有信息(成员变量、类名等等等)，再通过配置文件(xml)或者注解来描述类与类之间的关系。



### 容器
`Spring` 容器对象有 `ApplicationContext` 和 `BeanFactory`，`ApplicationContext` 包含 `BeanFactory` 的所有功能，建议优先使用 `ApplicationContext`。

Spring 作者 Rod Johnson 设计了两个接口用以表示容器。

- BeanFactory
- ApplicationContext

BeanFactory 简单粗暴，可以理解为就是个 HashMap，Key 是 BeanName，Value 是 Bean 实例。通常只提供注册（put），获取（get）这两个功能。我们可以称之为 **“低级容器”**。

ApplicationContext 可以称之为 **“高级容器”**。因为他比 BeanFactory 多了更多的功能。他继承了多个接口。因此具备了更多的功能。例如资源的获取，支持多种消息（例如 JSP tag 的支持），对 BeanFactory 多了工具级别的支持等待。所以你看他的名字，已经不是 BeanFactory 之类的工厂了，而是 “应用上下文”， 代表着整个大容器的所有功能。该接口定义了一个 refresh 方法，此方法是所有阅读 Spring 源码的人的最熟悉的方法，用于刷新整个容器，即重新加载/刷新所有的 bean。

当然，除了这两个大接口，还有其他的辅助接口，这里就不介绍他们了。

BeanFactory和ApplicationContext的关系

为了更直观的展示 “低级容器” 和 “高级容器” 的关系，这里通过常用的 ClassPathXmlApplicationContext 类来展示整个容器的层级 UML 关系。

![img](../../Image/2022/08/220803-18.png)



最上面的是 BeanFactory，下面的 3 个绿色的，都是功能扩展接口，这里就不展开讲。

看下面的隶属 ApplicationContext 粉红色的 “高级容器”，依赖着 “低级容器”，这里说的是依赖，不是继承哦。他依赖着 “低级容器” 的 getBean 功能。而高级容器有更多的功能：支持不同的信息源头，可以访问文件资源，支持应用事件（Observer 模式）。

通常用户看到的就是 “高级容器”。 但 BeanFactory 也非常够用啦！

左边灰色区域的是 “低级容器”， 只负载加载 Bean，获取 Bean。容器其他的高级功能是没有的。例如上图画的 refresh 刷新 Bean 工厂所有配置，生命周期事件回调等。

小结

说了这么多，不知道你有没有理解Spring IoC？ 这里小结一下：IoC 在 Spring 里，只需要低级容器就可以实现，2 个步骤：

1. 加载配置文件，解析成 BeanDefinition 放在 Map 里。
2. 调用 getBean 的时候，从 BeanDefinition 所属的 Map 里，拿出 Class 对象进行实例化，同时，如果有依赖关系，将递归调用 getBean 方法 —— 完成依赖注入。

上面就是 Spring 低级容器（BeanFactory）的 IoC。

至于高级容器 ApplicationContext，他包含了低级容器的功能，当他执行 refresh 模板方法的时候，将刷新整个容器的 Bean。同时其作为高级容器，包含了太多的功能。一句话，他不仅仅是 IoC。他支持不同信息源头，支持 BeanFactory 工具类，支持层级容器，支持访问文件资源，支持事件发布通知，支持接口回调等等。



#### BeanFactory

BeanFactory是IOC容器的核心接口，它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。BeanFactory只是个接口，并不是IOC容器的具体实现，但是Spring容器给出了很多种实现，如 DefaultListableBeanFactory、XmlBeanFactory、ApplicationContext等，其中XmlBeanFactory就是常用的一个，该实现将以XML方式描述组成应用的对象及对象间的依赖关系。

原始的BeanFactory无法支持spring的许多插件，如AOP功能、Web应用等。



##### 实现 `BeanFactoryAware` 接口获取BeanFactory
```java
@Service
public class ObtainContextByBeanFactoryAware implements BeanFactoryAware {
    private BeanFactory beanFactory;

    @Override
    public void setBeanFactory(@NonNull BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    public String getContent() {
        ContextBean contextBean = (ContextBean)beanFactory.getBean("contextBean");
        return contextBean.getContent();
    }
}
```



##### 源码解析

```java
public interface BeanFactory {  
    String FACTORY_BEAN_PREFIX = "&";

    /**
     * 返回给定名称注册的bean实例。根据bean的配置情况，如果是singleton模式将返回一个共享实例，
     * 否则将返回一个新建的实例，如果没有找到指定bean,该方法可能会抛出异常
     */
    Object getBean(String name) throws BeansException;

    /**
     * 返回以给定名称注册的bean实例，并转换为给定class类型
     */
    <T> T getBean(String name, Class<T> requiredType) throws BeansException;
    Object getBean(String name, Object... args) throws BeansException;
    <T> T getBean(Class<T> requiredType) throws BeansException;
    <T> T getBean(Class<T> requiredType, Object... args) throws BeansException;
    <T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);
    <T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);

    /**
     * 判断工厂中是否包含给定名称的bean定义，若有则返回true
     */
    boolean containsBean(String name);

    /**
     * 判断给定名称的bean定义是否为单例模式
     */
    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;  boolean isPrototype(String name) throws NoSuchBeanDefinitionException;  boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;  boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;    

    /**
     * 返回给定名称的bean的Class,
     * 如果没有找到指定的bean实例，则抛出NoSuchBeanDefinitionException异常
     */
    @Nullable
    Class<?> getType(String name) throws NoSuchBeanDefinitionException;

    /**
     * 返回给定bean名称的所有别名 
     */
    @Nullable
    Class<?> getType(String name, boolean allowFactoryBeanInit) throws NoSuchBeanDefinitionException;

    /**
     * 返回给定bean名称的所有别名 
     */
    String[] getAliases(String name);
}
```



#### ApplicationContext 

ApplicationContext以一种更向面向框架的方式工作以及对上下文进行分层和实现继承，ApplicationContext包还提供了以下的功能： 

- MessageSource, 提供国际化的消息访问 
- 资源访问，如URL和文件 
- 事件传播 
- 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层; 



##### 接口实现类

**FileSystemXmlApplicationContext** ：此容器从一个XML文件中加载beans的定义，XML Bean 配置文件的全路径名必须提供给它的构造函数。

**ClassPathXmlApplicationContext**：此容器也从一个XML文件中加载beans的定义，这里，你需要正确设置classpath因为这个容器将在classpath里找bean配置。

**WebXmlApplicationContext**：此容器加载一个XML文件，此文件定义了一个WEB应用的所有bean。



##### 获取ApplicationContext

###### 实现 `ApplicationContextAware` 接口
```java
@Service
public class ObtainContextByApplicationContextAware implements ApplicationContextAware {
    private ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(@Nonnull ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    public String getContent() {
        ContextBean contextBean = (ContextBean)applicationContext.getBean("contextBean");
        return contextBean.getContent();
    }
}
```



###### 实现 `ApplicationListener` 接口
```java
@Service
public class ObtainContextByApplicationListener implements ApplicationListener<ContextRefreshedEvent> {
    private ApplicationContext applicationContext;

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        this.applicationContext = event.getApplicationContext();
    }

    public String getContent() {
        ContextBean contextBean = (ContextBean)applicationContext.getBean("contextBean");
        return contextBean.getContent();
    }
}
```



##### 和BeanFactory有什么区别

BeanFactory和ApplicationContext是Spring的两大核心接口，都可以当做Spring的容器。其中ApplicationContext是BeanFactory的子接口。



| 区别     | BeanFactory                                         | ApplicationContext                                  |
| -------- | --------------------------------------------------- | --------------------------------------------------- |
| 依赖关系 | 最底层的接口                                        | BeanFactory派生接口                                 |
| 加载方式 | 延迟加载注入Bean，调用时才抛出异常                  | 一次性创建了所有的Bean，启动时就可以发现配置错误    |
| 创建方式 | 编程方式创建                                        | 编程方式创建和声明方式创建，如ContextLoader         |
| 注册方式 | 手动注册BeanPostProcessor、BeanFactoryPostProcessor | 自动注册BeanPostProcessor、BeanFactoryPostProcessor |



#### AnnotationConfigApplicationContext

主要是注解开发获取ioc中的bean实例。



## DI

### 概念

控制反转IoC是一个很大的概念，可以用不同的方式来实现。其主要实现方式有两种：依赖注入和依赖查找

依赖注入：相对于IoC而言，依赖注入(DI)更加准确地描述了IoC的设计理念。所谓依赖注入（Dependency Injection），即组件之间的依赖关系由容器在应用系统运行期来决定，也就是由容器动态地将某种依赖关系的目标对象实例注入到应用系统中的各个关联的组件之中。组件不做定位查询，只提供普通的Java方法让容器去决定依赖关系。



#### 基本原则

依赖注入的基本原则是：应用组件不应该负责查找资源或者其他依赖的协作对象。配置对象的工作应该由IoC容器负责，“查找资源”的逻辑应该从应用组件的代码中抽取出来，交给IoC容器负责。容器全权负责组件的装配，它会把符合依赖关系的对象通过属性（JavaBean中的setter）或者是构造器传递给需要的对象。



#### 优点

依赖注入之所以更流行是因为它是一种更可取的方式：让容器全权负责依赖查询，受管组件只需要暴露JavaBean的setter方法或者带参数的构造器或者接口，使容器可以在初始化时组装对象的依赖关系。其与依赖查找方式相比，主要优势为：

- 查找定位操作与应用代码完全无关。
- 不依赖于容器的API，可以很容易地在任何容器以外使用应用对象。
- 不需要特殊的接口，绝大多数对象可以做到完全不必依赖容器。



### 对象注入

#### 接口注入

接口注入（Interface Injection）由于在灵活性和易用性比较差，现在从Spring4开始已被废弃。



#### 属性注入

属性注入非常简洁，没有任何多余代码，非常有效的提高了java的简洁性。即使再多几个依赖一样能解决掉这个问题。

```java
@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserMapper userMapper;


    //...
}
```



#### Setter方法注入

在使用set（Setter Injection）方式时，这是一种选择注入，可有可无，即使没有注入这个依赖，那么也不会影响整个类的运行。

```java
@Service
public class UserServiceImpl implements UserService {
    private UserMapper userMapper;
    @Autowired
    public void setUserMapper(UserMapper userMapper) {
        this.userMapper = userMapper;
    }
}
```



#### 构造方法注入

在使用构造器方式（Constructor Injection）时已经显式注明必须强制注入。通过强制指明依赖注入来保证这个类的运行。

变量方式注入应该尽量避免，使用set方式注入或者构造器注入，这两种方式的选择就要看这个类是强制依赖的话就用构造器方式，选择依赖的话就用set方法注入。

```java
@Service
public class UserServiceImpl implements UserService {
    private final UserMapper userMapper;
    
    @Autowired
    public UserServiceImpl(UserMapper userMapper) {
        this.userMapper = userMapper;
    }
}
```



#### 总结

| **构造函数注入**           | **setter** **注入**        |
| -------------------------- | -------------------------- |
| 没有部分注入               | 有部分注入                 |
| 不会覆盖 setter 属性       | 会覆盖 setter 属性         |
| 任意修改都会创建一个新实例 | 任意修改不会创建一个新实例 |
| 适用于设置很多属性         | 适用于设置少量属性         |



两种依赖方式都可以使用，构造器注入和Setter方法注入。最好的解决方案是用构造器参数实现强制依赖，setter方法实现可选依赖。



### 对象初始化

#### xml中的 `init-method` 配置
这种方式现在已经很少使用，推荐下面两种初始化方式。



#### 实现 `InitializingBean` 接口
```java
@Slf4j
@Service
public class InitByInterface implements InitializingBean {
    @Override
    public void afterPropertiesSet() {
        log.info("通过实现InitializingBean接口来进行初始化");
    }
}
```



#### `@PostConstruct` 注解
```java
@Slf4j
@Service
public class InitByAnnotation {
    @PostConstruct
    public void init() {
        log.info("通过@PostConstruct注解进行初始化操作");
    }
}
```

##### 源码解析
> org.springframework.beans.factory.config.BeanPostProcessor
```java
public interface BeanPostProcessor {
	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
}
```

> org.springframework.context.annotation.CommonAnnotationBeanPostProcessor
```java
public class CommonAnnotationBeanPostProcessor extends InitDestroyAnnotationBeanPostProcessor
		implements InstantiationAwareBeanPostProcessor, BeanFactoryAware, Serializable {
    public CommonAnnotationBeanPostProcessor() {
        setOrder(Ordered.LOWEST_PRECEDENCE - 3);
        // 调用父类方法设置被@PostConstruct注解
        setInitAnnotationType(PostConstruct.class);
        setDestroyAnnotationType(PreDestroy.class);
        ignoreResourceType("javax.xml.ws.WebServiceContext");
    }
}
```

> org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor
```java
public class InitDestroyAnnotationBeanPostProcessor
		implements DestructionAwareBeanPostProcessor, MergedBeanDefinitionPostProcessor, PriorityOrdered, Serializable {
    // 通过CommonAnnotationBeanPostProcessor设置initAnnotationType为@PostConstruct
    public void setInitAnnotationType(Class<? extends Annotation> initAnnotationType) {
        this.initAnnotationType = initAnnotationType;
    }
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        // 获取初始化和销毁方法
        LifecycleMetadata metadata = findLifecycleMetadata(bean.getClass());
        try {
            // 通过反射执行初始化方法
            metadata.invokeInitMethods(bean, beanName); 
        }
        // ...
        // 执行完初始化方法后返回对象并执行下一个BeanPostProcessor的初始化方法
        return bean;
    } 

    private LifecycleMetadata buildLifecycleMetadata(final Class<?> clazz) {
        // ...
        List<LifecycleElement> initMethods = new ArrayList<>();
        // ...
        Class<?> targetClass = clazz;
        do {
            final List<LifecycleElement> currInitMethods = new ArrayList<>();
            // ...
            ReflectionUtils.doWithLocalMethods(targetClass, method -> {
                if (this.initAnnotationType != null && method.isAnnotationPresent(this.initAnnotationType)) {
                    LifecycleElement element = new LifecycleElement(method);
                    // 获取被@PostConstruct标注的方法并将其添加到
                    currInitMethods.add(element);
                    // ...
                }
                // ...
            });
            initMethods.addAll(0, currInitMethods);
            // ...
            targetClass = targetClass.getSuperclass();
        }
        while (targetClass != null && targetClass != Object.class);
        return (initMethods.isEmpty() && destroyMethods.isEmpty() ? this.emptyLifecycleMetadata :
                new LifecycleMetadata(clazz, initMethods, destroyMethods));
    }
}
```

#### 对象初始化顺序
初始化方式的顺序如下：

Constructor构造方法 -> `@Autowired` -> `@PostConstruct` -> `InitializingBean` -> `init-method`
```java
@Slf4j
@Service
public class InitOrder implements InitializingBean {
    private ContextBean contextBean;
    public InitOrder() {
        log.info("1、先执行构造器初始化");
        if(contextBean == null) {
            log.info("此时contextBean为null");
        }
    }

    @Autowired
    public void setContextBean(ContextBean contextBean) {
        log.info("2、再执行@Autowired注解初始化");
        this.contextBean = contextBean;
        if(contextBean != null) {
            log.info("此时contextBean不为null");
        }
    }

    @PostConstruct
    public void init() {
        log.info("3、再执行@PostConstruct注解初始化");
    }

    @Override
    public void afterPropertiesSet() {
        log.info("4、最后执行afterPropertiesSet方法和init-method进行初始化");
    }
}
```

##### 源码解析
决定他们调用顺序的关键代码在 `AbstractAutowireCapableBeanFactory` 类的 `initializeBean` 方法中。

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory
```java
 abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
		implements AutowireCapableBeanFactory {
    protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
        // ...
        Object wrappedBean = bean;
        if (mbd == null || !mbd.isSynthetic()) {
            // 调用BeanPostProcessor，注解@PostConstruct就是通过InitDestroyAnnotationBeanPostProcessor实现的
            wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
        }
    
        try {
            // 调用初始化方法，包括afterPropertiesSet和init-method
            invokeInitMethods(beanName, wrappedBean, mbd);
        }
        // ...
    }

    @Override
    public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
            throws BeansException {
        Object result = existingBean;
        for (BeanPostProcessor processor : getBeanPostProcessors()) {
            // 调用BeanPostProcessor的postProcessBeforeInitialization进行初始化
            Object current = processor.postProcessBeforeInitialization(result, beanName);
            if (current == null) {
                return result;
            }
            result = current;
        }
        return result;
    }

    protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
        // 先调用InitializingBean接口的afterPropertiesSet方法初始化对象
        boolean isInitializingBean = (bean instanceof InitializingBean);
        if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
            // ...
            if (System.getSecurityManager() != null) {
                try {
                    AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
                        ((InitializingBean) bean).afterPropertiesSet();
                        return null;
                    }, getAccessControlContext());
                }
                // ...
            }
            else {
                ((InitializingBean) bean).afterPropertiesSet();
            }
        }
    
        // 再调用init-method方法初始化对象
        if (mbd != null && !Objects.equals(bean.getClass(), NullBean.class)) {
            String initMethodName = mbd.getInitMethodName();
            if (StringUtils.hasLength(initMethodName) &&
                    !(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
                    !mbd.isExternallyManagedInitMethod(initMethodName)) {
                invokeCustomInitMethod(beanName, bean, mbd);
            }
        }
    }
}
```



#### @Scope
#### @Scope注解衍生注解
`@RequestScope`, `@SessionScope` 和 `@ApplicationScope` 注解都是 `@Scope` 注解的衍生注解，其功能相当于设置 `@Scope` 注解的对应 `value` 属性。



> org.springframework.web.context.annotation.RequestScope
```java
@Scope(WebApplicationContext.SCOPE_REQUEST)
public @interface RequestScope {
	// ...
}
```



> org.springframework.web.context.annotation.SessionScope
```java
@Scope(WebApplicationContext.SCOPE_SESSION)
public @interface SessionScope {
	// ...
}
```



> org.springframework.web.context.annotation.ApplicationScope 
```java
@Scope(WebApplicationContext.SCOPE_APPLICATION)
public @interface ApplicationScope {
	// ...
}
```



# Spring Context

## 事件

Spring 提供了以下5种标准的事件：

1. 上下文更新事件（ContextRefreshedEvent）：在调用ConfigurableApplicationContext 接口中的refresh()方法时被触发。
2. 上下文开始事件（ContextStartedEvent）：当容器调用ConfigurableApplicationContext的Start()方法开始/重新开始容器时触发该事件。
3. 上下文停止事件（ContextStoppedEvent）：当容器调用ConfigurableApplicationContext的Stop()方法停止容器时触发该事件。
4. 上下文关闭事件（ContextClosedEvent）：当ApplicationContext被关闭时触发该事件。容器被关闭时，其管理的所有单例Bean都被销毁。
5. 请求处理事件（RequestHandledEvent）：在Web应用中，当一个http请求（request）结束触发该事件。如果一个bean实现了ApplicationListener接口，当一个ApplicationEvent 被发布以后，bean会自动被通知。



## 注解

### @Configuration

用来标记类可以当做一个bean的定义，被Spring IOC容器使用。



#### 源码解析

> org.springframework.context.annotation.Configuration

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {
	/**
	 * 
	 */
	@AliasFor(annotation = Component.class)
	String value() default "";
}
```



### @Bean

表示此方法将要返回一个对象，作为一个bean注册进Spring应用上下文。

该注解作用于方法，通常是在标有该注解的方法中定义产生这个 Bean，`@Bean`告诉了Spring这是某个类的示例，当我需要用它的时候还给我。

`@Bean` 注解比 `Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。

@Bean注解主要的作用是告知Spring，被此注解所标注的类将需要纳入到Bean管理工厂中。

​	

#### 源码解析

> org.springframework.context.annotation.Bean

```java
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Bean {
	/**
	 * 
	 */
	@AliasFor("name")
	String[] value() default {};

	/**
	 *
	 */
	@AliasFor("value")
	String[] name() default {};

	/**
	 * 
	 */
	@Deprecated
	Autowire autowire() default Autowire.NO;

	/**
	 * 
	 */
	boolean autowireCandidate() default true;

	/**
	 * 
	 */
	String initMethod() default "";

	/**
	 * 
	 */
	String destroyMethod() default AbstractBeanDefinition.INFER_METHOD;

}
```



### @Component 

这将 java 类标记为 bean。它是任何 Spring 管理组件的通用构造型。spring 的组件扫描机制现在可以将其拾取并将其拉入应用程序环境中。

该注解作用于类，通过类路径扫描来自动侦测以及自动装配到Spring容器中（可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。

@Component注解用于标注一个普通的组件类，它没有明确的业务范围，只是通知Spring被此注解的类需要被纳入到Spring Bean容器中并进行管理。



#### 源码解析

> org.springframework.stereotype.Component

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component {
	/**
	 * 
	 */
	String value() default "";
}
```



### @ComponentScan

@ComponentScan注解用于配置Spring需要扫描的被组件注解注释的类所在的包。可以通过配置其basePackages属性或者value属性来配置需要扫描的包路径。属性是basePackages的别名。



### @Controller

> @Controller注解属于Spring Context模块下的注解

`@Controller`是`@Component`注解的一个延伸，Spring 会自动扫描并配置被该注解标注的类。此注解用于标注Spring MVC的控制器。

在Spring MVC 中，控制器Controller 负责处理由DispatcherServlet 分发的请求，它把用户请求的数据经过业务处理层处理之后封装成一个Model ，然后再把该Model 返回给对应的View 进行展示。在Spring MVC 中提供了一个非常简便的定义Controller 的方法，你无需继承特定的类或实现特定的接口，只需使用@Controller 标记一个类是Controller ，然后使用@RequestMapping 和@RequestParam 等一些注解用以定义URL 请求和Controller 方法之间的映射，这样的Controller 就能被外界访问到。此外Controller 不会直接依赖于HttpServletRequest 和HttpServletResponse 等HttpServlet 对象，它们可以通过Controller 的方法参数灵活的获取到。

@Controller 用于标记在一个类上，使用它标记的类就是一个Spring MVC Controller 对象。分发处理器将会扫描使用了该注解的类的方法，并检测该方法是否使用了@RequestMapping 注解。@Controller 只是定义了一个控制器类，而使用@RequestMapping 注解的方法才是真正处理请求的处理器。单单使用@Controller 标记在一个类上还不能真正意义上的说它就是Spring MVC 的一个控制器类，因为这个时候Spring 还不认识它。那么要如何做Spring 才能认识它呢？这个时候就需要我们把这个控制器类交给Spring 来管理。有两种方式：

- 在Spring MVC 的配置文件中定义MyController 的bean 对象。
- 在Spring MVC 的配置文件中告诉Spring 该到哪里去找标记为@Controller 的Controller 控制器。



这将一个类标记为 Spring Web MVC 控制器。标有它的 Bean 会自动导入到 IoC 容器中。



#### 源码解析

>org.springframework.stereotype.Controller

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {
	/**
	 * 
	 */
	@AliasFor(annotation = Component.class)
	String value() default "";
}
```



### @Repository

这个注解是具有类似用途和功能的 @Component 注解的特化。它为 DAO 提供了额外的好处。它将 DAO 导入 IoC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException。

`@Repository`注解也是`@Component`注解的延伸，与`@Component`注解一样，被此注解标注的类会被Spring自动管理起来，`@Repository`注解用于标注DAO层的数据持久化类。



#### 源码解析

> org.springframework.stereotype.Repository

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Repository {
	/**
	 * 
	 */
	@AliasFor(annotation = Component.class)
	String value() default "";
}
```



### @Service

此注解是组件注解的特化。它不会对 @Component 注解提供任何其他行为。您可以在服务层类中使用 @Service 而不是 @Component，因为它以更好的方式指定了意图。

`@Service`注解是`@Component`的一个延伸（特例），它用于标注业务逻辑类。与`@Component`注解一样，被此注解标注的类，会自动被Spring所管理。



#### 源码解析

> org.springframework.stereotype.Service

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {
	/**
	 * 
	 */
	@AliasFor(annotation = Component.class)
	String value() default "";
}
```



### @DependsOn

@DependsOn注解可以配置Spring IoC容器在初始化一个Bean之前，先初始化其他的Bean对象。



### @Scope

@Scope注解可以用来定义@Component标注的类的作用范围以及@Bean所标记的类的作用范围。@Scope所限定的作用范围有：singleton、prototype、request、session、globalSession或者其他的自定义范围。这里以prototype为例子进行讲解。

当一个Spring Bean被声明为prototype（原型模式）时，在每次需要使用到该类的时候，Spring IoC容器都会初始化一个新的改类的实例。在定义一个Bean时，可以设置Bean的scope属性为prototype：scope=“prototype”,也可以使用@Scope注解设置。



### @Primary

当系统中需要配置多个具有相同类型的bean时，@Primary可以定义这些Bean的优先级。



### @Conditional

@Conditional 注解可以控制更为复杂的配置条件。在Spring内置的条件控制注解不满足应用需求的时候，可以使用此注解定义自定义的控制条件，以达到自定义的要求。



### @Async

Spring内部线程池，其实是`SimpleAsyncTaskExecutor`，它**不会复用线程的**，它的设计初衷就是执行大量的短时间的任务。也就是说来了一个请求，就会新建一个线程！因此要避免使用该注解来异步执行任务。



#### 使用示例

- 使用 `@EnableAsync` 启用异步注解

```java
@EnableAsync
public class BaseServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(BaseServiceApplication.class, args);
    }
}
```



- 自定义线程池

```java
@Slf4j
@Configuration
public class ThreadPoolExecutorConfig {
    @Bean(name = "defaultThreadPoolExecutor", destroyMethod = "shutdown")
    public ThreadPoolExecutor systemCheckPoolExecutorService() {
        return new ThreadPoolExecutor(3, 10, 60, TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(10000),
            new ThreadFactoryBuilder().setNameFormat("default-executor-%d").build(),
            (r, executor) -> log.error("system pool is full! "));
    }
}
```



- 在异步处理的方法上添加注解 @Async ，当对方法调用时，通过自定义的线程池 defaultThreadPoolExecutor 异步化执行方法。

```java
@Async("defaultThreadPoolExecutor")
@Override
public void executeAsyncTaskWithoutReturn (int i) {
    System.out.println("线程" + Thread.currentThread().getName() + " 执行异步任务：" + i);
}
```



# Spring Beans

Spring beans 是那些形成Spring应用的主干的java对象。它们被Spring IOC容器初始化，装配，和管理。这些beans通过容器中配置的元数据创建。比如，以XML文件中 的形式定义。

一个Spring Bean 的定义包含容器必知的所有配置元数据，包括如何创建一个bean，它的生命周期详情及它的依赖。



## 概念

### Bean相关

#### 内部Bean

在Spring框架中，当一个bean仅被用作另一个bean的属性时，它能被声明为一个内部bean。内部bean可以用setter注入“属性”和构造方法注入“构造参数”的方式来实现，内部bean通常是匿名的，它们的Scope一般是prototype。



#### Bean装配

装配，或bean 装配是指在Spring 容器中把bean组装到一起，前提是容器需要知道bean的依赖关系，如何通过依赖注入来把它们装配到一起。



## 对象自动装配

概念

在Spring框架中，在配置文件中设定bean的依赖关系是一个很好的机制，Spring 容器能够自动装配相互合作的bean，这意味着容器不需要和配置，能通过Bean工厂自动处理bean之间的协作。这意味着 Spring可以通过向Bean Factory中注入的方式自动搞定bean之间的依赖关系。自动装配可以设置在每个bean上，也可以设定在特定的bean上。

在spring中，对象无需自己查找或创建与其关联的其他对象，由容器负责把需要相互协作的对象引用赋予各个对象，使用autowire来配置自动装载模式。



自动装配类型

在Spring框架xml配置中共有5种自动装配：

- no：默认的方式是不进行自动装配的，通过手工设置ref属性来进行装配bean。
- byName：通过bean的名称进行自动装配，如果一个bean的 property 与另一bean 的name 相同，就进行自动装配。
- byType：通过参数的数据类型进行自动装配。
- constructor：利用构造函数进行装配，并且构造函数的参数通过byType进行装配。
- autodetect：自动探测，如果有构造方法，通过 construct的方式自动装配，否则使用 byType的方式自动装配。





### @Autowired注解

使用@Autowired注解来自动装配指定的bean。在使用@Autowired注解之前需要在Spring配置文件进行配置，<context:annotation-config />。

在启动spring IoC时，容器自动装载了一个AutowiredAnnotationBeanPostProcessor后置处理器，当容器扫描到@Autowied、@Resource或@Inject时，就会在IoC容器自动查找需要的bean，并装配给该对象的属性。在使用@Autowired时，首先在容器中查询对应类型的bean：

- 如果查询结果刚好为一个，就将该bean装配给@Autowired指定的数据；
- 如果查询的结果不止一个，那么@Autowired会根据名称来查找；
- 如果上述查找的结果为空，那么会抛出异常。解决方法时，使用required=false。



## 对象配置方式

### 注解方式

一般使用 `@Autowired` 注解自动装配 bean，要想把类标识成可用于 `@Autowired` 注解自动装配的 bean 的类，采用以下注解可实现：

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个Bean不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao层。
- `@Controller` : 对应 Spring MVC 控制层，主要用户接受用户请求并调用 Service 层返回数据给前端页面。



### XML配置文件方式

Spring配置文件是个XML 文件，这个文件包含了类信息，描述了如何配置它们，以及如何相互调用。



### 基于Java代码方式



## 对象作用域

当定义一个 在Spring里，我们还能给这个bean声明一个作用域。它可以通过bean 定义中的scope属性来定义。如，当Spring要在需要的时候每次生产一个新的bean实例，bean的scope属性被指定为prototype。另一方面，一个bean每次使用的时候必须返回同一个实例，这个bean的scope 属性 必须设为 singleton。

| 作用域            | 描述                 | 适用场景 |
| ----------------- | -------------------- | -------- |
| singleton（默认） | 单例                 | 所有场景 |
| prototype         | 多例                 | 所有场景 |
| request           | 在一次http请求内有效 | Web场景  |
| session           | 在一个用户会话内有效 | Web场景  |
| globalSession     | 在全局会话内有效     | Web场景  |



### singleton

bean在每个Spring ioc 容器中只有一个实例。

Springboot的注入默认范围是单例，在容器中通过对象引入配置注入和通过容器的getBean()方法返回的实例都是同一个bean。容器在启动的时候，自动实例化所有的singleton的bean并缓存于容器当中。

1、对bean提前的实例化操作，会及早发现一些潜在的配置的问题。

2、Bean以缓存的方式运行，当运行到需要使用该bean的时候，就不需要再去实例化了。加快了运行效率。



#### 线程安全问题

单例 bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。

常见解决办法：

1. 在Bean对象中尽量避免定义可变的成员变量。
2. 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式），注意使用完成后要remove掉。
3. 使用并发安全的类
4. 改变为非单例模式，通过注解 @`Scope("prototype")` 或 `@Scope("request")`



不是，Spring框架中的单例bean不是线程安全的。

spring 中的 bean 默认是单例模式，spring 框架并没有对单例 bean 进行多线程的封装处理。

实际上大部分时候 spring bean 无状态的（比如 dao 类），所有某种程度上来说 bean 也是安全的，但如果 bean 有状态的话（比如 view model 对象），那就要开发者自己去保证线程安全了，最简单的就是改变 bean 的作用域，把“singleton”变更为“prototype”，这样请求 bean 相当于 new Bean()了，所以就可以保证线程安全了。

- 有状态就是有数据存储功能。
- 无状态就是不会保存数据。



在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域，因为Spring对一些Bean中非线程安全状态采用ThreadLocal进行处理，解决线程安全问题。



#### 实现原理

> org.springframework.beans.factory.support.AbstractBeanFactory

```java
// 一级缓存
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
                          @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
	// ...
    if (mbd.isSingleton()) {
        sharedInstance = getSingleton(beanName, () -> {
            try {
                return createBean(beanName, mbd, args);
            } catch (BeansException ex) {
                destroySingleton(beanName);
                throw ex;
            }
        });
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
    }
	// ...
	return (T) bean;
}

public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
    synchronized (this.singletonObjects) {
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {
            // ...
            try {
                singletonObject = singletonFactory.getObject();
                newSingleton = true;
            }
            catch (IllegalStateException ex) {
                singletonObject = this.singletonObjects.get(beanName);
                if (singletonObject == null) {
                    throw ex;
                }
            }
            // ...
            finally {
                // ...
                afterSingletonCreation(beanName);
            }
            if (newSingleton) {
                addSingleton(beanName, singletonObject);
            }
        }
        return singletonObject;
    }
}

protected void addSingleton(String beanName, Object singletonObject) {
    synchronized (this.singletonObjects) {
        this.singletonObjects.put(beanName, singletonObject);
        this.singletonFactories.remove(beanName);
        this.earlySingletonObjects.remove(beanName);
        this.registeredSingletons.add(beanName);
    }
}
```



获取bean时判断如果scope是singleton，则调用getSingleton方法获取实例。getSingleton方法的主要逻辑如下：

1. 根据beanName先从singletonObjects集合中获取bean实例。
2. 如果bean实例不为空，则直接返回该实例。
3. 如果bean实例为空，则通过getObject方法创建bean实例，然后通过addSingleton方法，将该bean实例添加到singletonObjects集合中。
4. 下次再通过beanName从singletonObjects集合中，就能获取到bean实例了。



### prototype

一个bean的定义可以有多个实例。

是指每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，相当于执行new Bean()的操作。在默认情况下，容器在启动时不实例化prototype的Bean，容器将prototype的实例交给调用者之后便不在管理他的生命周期了。

对于有状态的Bean应该使用prototype，对于无状态的Bean则使用singleton

使用 prototype 作用域需要慎重的思考，因为频繁创建和销毁 bean 会带来很大的性能开销。



### request

每次http请求都会创建一个bean，该作用域仅在基于web的Spring ApplicationContext情形下有效。

对应一个http请求和生命周期，当http请求调用作用域为request的bean的时候,Spring便会创建一个新的bean，在请求处理完成之后便及时销毁这个bean。



### session

在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。

Session中所有http请求共享同一个请求的bean实例。Session结束后就销毁bean。



### globalSession

在一个全局的HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。

与session大体相同，但仅在portlet应用中使用。Portlet规范定义了全局session的概念。请求的bean被组成所有portlet的自portlet所共享。如果不是在portlet这种应用下，globalSession则等价于session作用域。



## 对象生命周期

### 实现流程

Spring Bean的生命周期有4个阶段，如下：

1. 实例化 Instantiation
2. 属性赋值 Populate
3. 初始化 Initialization
4. 销毁 Destruction



![图片](../../Image/2022/09/220908-1.jpg)



#### 入口

##### 源码解析

> org.springframework.context.support.AbstractApplicationContext

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        try {
            // ...
            // 实例化剩下的普通的Bean对象
            finishBeanFactoryInitialization(beanFactory);
            // ...
        }
        // ...
    }
}

protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // ...
    // 实例化所有剩下的非懒加载的单例对象
    beanFactory.preInstantiateSingletons();
}
```



> org.springframework.beans.factory.support.DefaultListableBeanFactory

```java
@Override
public void preInstantiateSingletons() throws BeansException {
    // ...
    for (String beanName : beanNames) {
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            if (isFactoryBean(beanName)) {
                // ...
            }
            else {
                getBean(beanName);
            }
        }
    }
    // ...
}
```



> org.springframework.beans.factory.support.AbstractBeanFactory

```java
@Override
public Object getBean(String name) throws BeansException {
    return doGetBean(name, null, null, false);
}

protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
                          @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
	// ...
    if (sharedInstance != null && args == null) {
        // ...
    }
    else {
        // ...
        try {
            // ...
            if (dependsOn != null) {
                // ...
            }
            // 创建对象实例
            if (mbd.isSingleton()) {
                sharedInstance = getSingleton(beanName, () -> {
                    try {
                        // 创建对象
                        return createBean(beanName, mbd, args);
                    }
                    // ...
                });
                bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
            }
            else if (mbd.isPrototype()) {
                // ...
            }
            else {
                // ...
            }
        }
        catch (BeansException ex) {
            cleanupAfterBeanCreationFailure(beanName);
            throw ex;
        }
    }
	// ...
    return (T) bean;
}
```



> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
@Override
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
    throws BeanCreationException {
	// ...
    try {
        // Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
        Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
        if (bean != null) {
            return bean;
        }
    }
 	// ...
    try {
        Object beanInstance = doCreateBean(beanName, mbdToUse, args);
        return beanInstance;
    }
    // ...
}

protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
    throws BeanCreationException {
    // 实例化对象.
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) {
        instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    if (instanceWrapper == null) {
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }

    // 初始化对象
    Object exposedObject = bean;
    try {
        // 属性赋值
        populateBean(beanName, mbd, instanceWrapper);
        // 初始化
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }
    catch (Throwable ex) {
        // ...
    }
    // ...
    return exposedObject;
}
```



#### 实例化

实例化，创建一个Bean对象。

Bean实例化的时机也分为两种，BeanFactory管理的Bean是在使用到Bean的时候才会实例化Bean，ApplicantContext管理的Bean在容器初始化的时候就回完成Bean实例化。



##### 源码解析

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
	// ...
    // 无特殊处理的空构造器的对象
    return instantiateBean(beanName, mbd);
}

protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {
    try {
        Object beanInstance;
        final BeanFactory parent = this;
        if (System.getSecurityManager() != null) {
            // ...
        }
        else {
            beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, parent);
        }
        BeanWrapper bw = new BeanWrapperImpl(beanInstance);
        initBeanWrapper(bw);
        return bw;
    }
	// ...
}
```



> org.springframework.beans.factory.support.SimpleInstantiationStrategy

```java
@Override
public Object instantiate(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner) {
    if (!bd.hasMethodOverrides()) {
        Constructor<?> constructorToUse;
        synchronized (bd.constructorArgumentLock) {
            // ...
        }
        // 构造器创建对象
        return BeanUtils.instantiateClass(constructorToUse);
    }
    else {
        return instantiateWithMethodInjection(bd, beanName, owner);
    }
}
```



#### 属性赋值

填充属性，为属性赋值。



##### 源码解析

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    // ...
    if (pvs != null) {
        applyPropertyValues(beanName, mbd, bw, pvs);
    }
}

protected void applyPropertyValues(String beanName, BeanDefinition mbd, BeanWrapper bw, PropertyValues pvs) {
    // 设置属性
    try {
        bw.setPropertyValues(new MutablePropertyValues(deepCopy));
    }
	// ...
}
```



> org.springframework.beans.AbstractPropertyAccessor

```java
@Override
public void setPropertyValues(PropertyValues pvs) throws BeansException {
    setPropertyValues(pvs, false, false);
}

@Override
public void setPropertyValues(PropertyValues pvs, boolean ignoreUnknown, boolean ignoreInvalid)
    throws BeansException {
    // ...
    for (PropertyValue pv : propertyValues) {
        try {
          	// 填充属性
            setPropertyValue(pv);
        }
        // ...
    }
	// ...
}
```



> org.springframework.beans.AbstractNestablePropertyAccessor

```java
@Override
public void setPropertyValue(PropertyValue pv) throws BeansException {
    PropertyTokenHolder tokens = (PropertyTokenHolder) pv.resolvedTokens;
    if (tokens == null) {
        // ...
    }
    else {
        setPropertyValue(tokens, pv);
    }
}

protected void setPropertyValue(PropertyTokenHolder tokens, PropertyValue pv) throws BeansException {
    if (tokens.keys != null) {
        processKeyedProperty(tokens, pv);
    }
    else {
        processLocalProperty(tokens, pv);
    }
}

private void processLocalProperty(PropertyTokenHolder tokens, PropertyValue pv) {
    PropertyHandler ph = getLocalPropertyHandler(tokens.actualName);
	// ...

    Object oldValue = null;
    try {
        Object originalValue = pv.getValue();
        Object valueToApply = originalValue;
        // ...
        ph.setValue(valueToApply);
    }
	// ...
}
```



> org.springframework.beans.BeanWrapperImpl.BeanPropertyHandler

```java
@Override
public void setValue(final @Nullable Object value) throws Exception {
    final Method writeMethod = (this.pd instanceof GenericTypeAwarePropertyDescriptor ?
                                ((GenericTypeAwarePropertyDescriptor) this.pd).getWriteMethodForActualAccess() :
                                this.pd.getWriteMethod());
    if (System.getSecurityManager() != null) {
       	// ...
    }
    else {
        ReflectionUtils.makeAccessible(writeMethod);
        writeMethod.invoke(getWrappedInstance(), value);
    }
}
```



#### 初始化

- 如果实现了`xxxAware`接口，通过不同类型的Aware接口拿到Spring容器的资源
- 如果实现了BeanPostProcessor接口，则会回调该接口的`postProcessBeforeInitialzation`和`postProcessAfterInitialization`方法
- 如果配置了`init-method`方法，则会执行`init-method`配置的方法



##### 初始化方式

因为这几个方法对应的都是同一个生命周期，只是实现方式不同，我们一般只采用其中一种方式。这几种定义方式的调用顺序如下：

**Constructor > @PostConstruct > InitializingBean > init-method**



###### @PostConstruct注解

使用@PostConstruct注解，该注解作用于void方法上



###### 配置init-method

在配置文件中配置init-method方法



###### 实现InitializingBean接口

将类实现InitializingBean接口



##### 源码解析

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
    if (System.getSecurityManager() != null) {
        // ...
    }
    else {
        // 调用Aware接口相关方法
        invokeAwareMethods(beanName, bean);
    }

    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
       	// 调用BeanPostProcesser接口
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }

    try {
        // 初始化方法
        invokeInitMethods(beanName, wrappedBean, mbd);
    }
    // ...
    if (mbd == null || !mbd.isSynthetic()) {
        // 调用BeanPostProcesser接口
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }

    return wrappedBean;
}

protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)
    throws Throwable {

    boolean isInitializingBean = (bean instanceof InitializingBean);
    if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
        // ...
        if (System.getSecurityManager() != null) {
            // ...
        }
        else {
            // 调用实现InitializingBean接口的方法
            ((InitializingBean) bean).afterPropertiesSet();
        }
    }

    if (mbd != null && bean.getClass() != NullBean.class) {
        String initMethodName = mbd.getInitMethodName();
        if (StringUtils.hasLength(initMethodName) &&
            !(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
            !mbd.isExternallyManagedInitMethod(initMethodName)) {
            // 调用自定义初始化方法
            invokeCustomInitMethod(beanName, bean, mbd);
        }
    }
}
```



#### 销毁

- 容器关闭后，如果Bean实现了`DisposableBean`接口，则会回调该接口的`destroy`方法
- 如果配置了`destroy-method`方法，则会执行`destroy-method`配置的方法



##### 销毁方式

###### @PreDestroy注解



###### 配置destroy-method



###### 实现了DisposableBean接口



##### 源码解析

销毁，是在容器关闭时调用的，详见 ConfigurableApplicationContext#close()

> org.springframework.context.ConfigurableApplicationContext

```java
	/**
	 * 
	 */
	@Override
	void close();
```



ConfigurableApplicationContext接口定义了close方法，由AbstractApplicationContext实现。



> org.springframework.context.support.AbstractApplicationContext

```java
@Override
public void close() {
    synchronized (this.startupShutdownMonitor) {
        doClose();
        // ...
    }
}

protected void doClose() {
    // 检查实际关闭尝试是否必要
    if (this.active.get() && this.closed.compareAndSet(false, true)) {
        // ...
        // 销毁所有缓存的单例对象
        destroyBeans();
        // ...
    }
}

protected void destroyBeans() {
    getBeanFactory().destroySingletons();
}
```



> org.springframework.beans.factory.support.DefaultListableBeanFactory

```java
@Override
public void destroySingletons() {
    super.destroySingletons();
    updateManualSingletonNames(Set::clear, set -> !set.isEmpty());
    clearByTypeCache();
}
```



> org.springframework.beans.factory.support.DefaultSingletonBeanRegistry

```java
public void destroySingletons() {
	// ...
    String[] disposableBeanNames;
    synchronized (this.disposableBeans) {
        disposableBeanNames = StringUtils.toStringArray(this.disposableBeans.keySet());
    }
    for (int i = disposableBeanNames.length - 1; i >= 0; i--) {
        destroySingleton(disposableBeanNames[i]);
    }
	// ...
}
```



> org.springframework.beans.factory.support.DefaultListableBeanFactory

```java
@Override
public void destroySingleton(String beanName) {
    super.destroySingleton(beanName);
    removeManualSingletonName(beanName);
    clearByTypeCache();
}
```



> org.springframework.beans.factory.support.DefaultSingletonBeanRegistry

```java
public void destroySingleton(String beanName) {
    // Remove a registered singleton of the given name, if any.
    removeSingleton(beanName);

    // Destroy the corresponding DisposableBean instance.
    DisposableBean disposableBean;
    synchronized (this.disposableBeans) {
        disposableBean = (DisposableBean) this.disposableBeans.remove(beanName);
    }
    destroyBean(beanName, disposableBean);
}

protected void destroyBean(String beanName, @Nullable DisposableBean bean) {
    // Trigger destruction of dependent beans first...
    Set<String> dependencies;
    synchronized (this.dependentBeanMap) {
        // Within full synchronization in order to guarantee a disconnected Set
        dependencies = this.dependentBeanMap.remove(beanName);
    }
    if (dependencies != null) {
        // ...
        for (String dependentBeanName : dependencies) {
            destroySingleton(dependentBeanName);
        }
    }

    // Actually destroy the bean now...
    if (bean != null) {
        try {
            // 调用DispoableBean接口实现的方法销毁对象
            bean.destroy();
        }
		// ...
    }

    // Trigger destruction of contained beans...
    Set<String> containedBeans;
    synchronized (this.containedBeanMap) {
        // Within full synchronization in order to guarantee a disconnected Set
        containedBeans = this.containedBeanMap.remove(beanName);
    }
    if (containedBeans != null) {
        for (String containedBeanName : containedBeans) {
            destroySingleton(containedBeanName);
        }
    }

    // Remove destroyed bean from other beans' dependencies.
    synchronized (this.dependentBeanMap) {
        for (Iterator<Map.Entry<String, Set<String>>> it = 
             this.dependentBeanMap.entrySet().iterator(); it.hasNext();) {
            Map.Entry<String, Set<String>> entry = it.next();
            Set<String> dependenciesToClean = entry.getValue();
            dependenciesToClean.remove(beanName);
            if (dependenciesToClean.isEmpty()) {
                it.remove();
            }
        }
    }

    // Remove destroyed bean's prepared dependency information.
    this.dependenciesForBeanMap.remove(beanName);
}
```



### 扩展点

InstantiationAwareBeanPostProcessor作用于**实例化**阶段的前后，BeanPostProcessor作用于**初始化**阶段的前后。

![img](../../Image/2022/08/220804-1.png)

#### InstantiationAwareBeanPostProcessor

```java
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {
	/**
	 * 
	 */
	@Nullable
	default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
		return null;
	}

	/**
	 * 
	 */
	default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
		return true;
	}
}
```



##### 调用时机

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
    throws BeanCreationException {
    try {
        // 让 BeanPostProcessors 有机会返回一个代理而不是目标 bean 实例。
        Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
        if (bean != null) {
            return bean;
        }
    }
    catch (Throwable ex) {
        // ...
    }

    try {
        // 创建对象
        Object beanInstance = doCreateBean(beanName, mbdToUse, args);
        // ...
        return beanInstance;
    }
    catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
        // ...
    }
}
```



> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
    Object bean = null;
    if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
        // Make sure bean class is actually resolved at this point.
        if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
            Class<?> targetType = determineTargetType(beanName, mbd);
            if (targetType != null) {
                bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
                if (bean != null) {
                    bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                }
            }
        }
        mbd.beforeInstantiationResolved = (bean != null);
    }
    return bean;
}
```



postProcessBeforeInstantiation在doCreateBean之前调用，也就是在bean实例化之前调用的，英文源码注释解释道该方法的返回值会替换原本的Bean作为代理，这也是Aop等功能实现的关键点。



###### postProcessBeforeInstantiation

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
@Nullable
protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        if (bp instanceof InstantiationAwareBeanPostProcessor) {
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
            Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName);
            if (result != null) {
                return result;
            }
        }
    }
    return null;
}
```



###### postProcessAfterInstantiation

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    // ...
    // 让 InstantiationAwareBeanPostProcessors 在设置属性之前修改 bean 的状态。
    // 例如，这可以用于支持字段注入的样式。
    // 方法作为属性赋值的前置检查条件，在属性赋值之前执行，能够影响是否进行属性赋值！
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof InstantiationAwareBeanPostProcessor) {
                InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
                if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                    return;
                }
            }
        }
    }
    // ...
}
```



#### Aware接口

Aware接口的作用是让Bean拿到Spring容器中的资源**所有的Aware方法都是在初始化阶段之前调用的！**

Aware之前的名字就是可以拿到什么资源，例如BeanNameAware可以拿到BeanName，以此类推。调用时机需要注意：



`Aware` 是一个空接口，里面不包括任何方法。该接口表示已感知，可以通过该接口的实现获取指定对象。

> org.springframework.beans.factory.Aware

```java
public interface Aware {
}
```



Aware接口具体可以分为两组，如下所示：



- 对象相关：

1. BeanNameAware
2. BeanClassLoaderAware
3. BeanFactoryAware



- ApplicationContext相关：

1. EnvironmentAware
2. EmbeddedValueResolverAware 实现该接口能够获取Spring EL解析器，用户的自定义注解需要支持spel表达式的时候可以使用，非常方便。
3. ApplicationContextAware(ResourceLoaderAware\ApplicationEventPublisherAware\MessageSourceAware) 这几个接口可能让人有点懵，实际上这几个接口可以一起记，其返回值实质上都是当前的ApplicationContext对象，因为ApplicationContext是一个复合接口。



该接口的部分实现如下：

- ApplicationEventPublisherAware 
- ServletContextAware 
- MessageSourceAware 
- ResourceLoaderAware 
- SchedulerContextAware 
- NotificationPublisherAware 
- EnvironmentAware 
- BeanFactoryAware 
- EmbeddedValueResolverAware 
- ImportAware 
- ServletConfigAware 
- BootstrapContextAware 
- LoadTimeWeaverAware
- BeanNameAware
- BeanClassLoaderAware
- ApplicationContextAware



并不是所有的Aware接口都使用同样的方式调用。Bean××Aware都是在代码中直接调用的，而ApplicationContext相关的Aware都是通过BeanPostProcessor#postProcessBeforeInitialization()实现的。



##### 调用时机

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
		if (System.getSecurityManager() != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
                // 调用对象相关的Aware
				invokeAwareMethods(beanName, bean);
				return null;
			}, getAccessControlContext());
		}
		else {
            // 调用对象相关的Aware
			invokeAwareMethods(beanName, bean);
		}
		// ...
	}
```



###### 对象相关

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	private void invokeAwareMethods(final String beanName, final Object bean) {
		if (bean instanceof Aware) {
			if (bean instanceof BeanNameAware) {
				((BeanNameAware) bean).setBeanName(beanName);
			}
			if (bean instanceof BeanClassLoaderAware) {
				ClassLoader bcl = getBeanClassLoader();
				if (bcl != null) {
					((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
				}
			}
			if (bean instanceof BeanFactoryAware) {
				((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
			}
		}
	}
```



###### ApplicationContext相关

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	@Override
	public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
			Object current = processor.postProcessBeforeInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
	}

```



> org.springframework.context.support.ApplicationContextAwareProcessor

```java
	public Object postProcessBeforeInitialization(final Object bean, String beanName) throws BeansException {
		AccessControlContext acc = null;

		if (System.getSecurityManager() != null &&
				(bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware ||
						bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware ||
						bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware)) {
			acc = this.applicationContext.getBeanFactory().getAccessControlContext();
		}

		if (acc != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
				invokeAwareInterfaces(bean);
				return null;
			}, acc);
		}
		else {
			invokeAwareInterfaces(bean);
		}

		return bean;
	}

	private void invokeAwareInterfaces(Object bean) {
		if (bean instanceof Aware) {
			if (bean instanceof EnvironmentAware) {
				((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
			}
			if (bean instanceof EmbeddedValueResolverAware) {
				((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
			}
			if (bean instanceof ResourceLoaderAware) {
				((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
			}
			if (bean instanceof ApplicationEventPublisherAware) {
				((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
			}
			if (bean instanceof MessageSourceAware) {
				((MessageSourceAware) bean).setMessageSource(this.applicationContext);
			}
			if (bean instanceof ApplicationContextAware) {
				((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
			}
		}
	}
```

ApplicationContext相关的Aware是通过BeanPostProcessor（ApplicationContextAwareProcessor）实现的。



#### BeanPostProcessor

从refresh方法来看,BeanPostProcessor 实例化比正常的bean早，从initializeBean方法看,每个bean初始化前后都调用所有BeanPostProcessor的postProcessBeforeInitialization和postProcessAfterInitialization方法。

> org.springframework.beans.factory.config.BeanPostProcessor

```java
public interface BeanPostProcessor {
	/**
	 * 
	 */
	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	/**
	 * 
	 */
	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
}
```



##### 注册时机

BeanPostProcessor会在业务Bean之前初始化完成。



> org.springframework.context.support.AbstractApplicationContext

```java
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// ...
			try {
				// ...
				// 注册拦截bean创建的BeanPostProcessor
				registerBeanPostProcessors(beanFactory);

                // ...
                
				// 实例化并初始化bean
				finishBeanFactoryInitialization(beanFactory);
				// ...
			}
			// ...
		}
	}
```



Spring是先执行registerBeanPostProcessors()进行BeanPostProcessors的注册，然后再执行finishBeanFactoryInitialization创建我们的单例非懒加载的Bean。



##### 执行顺序

BeanPostProcessor有很多个，而且每个BeanPostProcessor都影响多个Bean，其执行顺序至关重要，必须能够控制其执行顺序才行。关于执行顺序这里需要引入两个排序相关的接口：PriorityOrdered、Ordered

- PriorityOrdered是一等公民，首先被执行，PriorityOrdered公民之间通过接口返回值排序
- Ordered是二等公民，然后执行，Ordered公民之间通过接口返回值排序
- 都没有实现是三等公民，最后执行



> org.springframework.context.support.AbstractApplicationContext

```java
	protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
		PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
	}
```



> org.springframework.context.support.PostProcessorRegistrationDelegate

```java
	public static void registerBeanPostProcessors(
			ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {

		String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);

		// Register BeanPostProcessorChecker that logs an info message when
		// a bean is created during BeanPostProcessor instantiation, i.e. when
		// a bean is not eligible for getting processed by all BeanPostProcessors.
		int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
		beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));

		// Separate between BeanPostProcessors that implement PriorityOrdered,
		// Ordered, and the rest.
		List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
		List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();
		List<String> orderedPostProcessorNames = new ArrayList<>();
		List<String> nonOrderedPostProcessorNames = new ArrayList<>();
		for (String ppName : postProcessorNames) {
			if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
				BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
				priorityOrderedPostProcessors.add(pp);
				if (pp instanceof MergedBeanDefinitionPostProcessor) {
					internalPostProcessors.add(pp);
				}
			}
			else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
				orderedPostProcessorNames.add(ppName);
			}
			else {
				nonOrderedPostProcessorNames.add(ppName);
			}
		}

		// 首先注册实现了PriorityOrdered的
		sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
		registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);

		// 然后注册实现了Ordered的BeanPostProcessor并排序
		List<BeanPostProcessor> orderedPostProcessors = new ArrayList<>();
		for (String ppName : orderedPostProcessorNames) {
			BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
			orderedPostProcessors.add(pp);
			if (pp instanceof MergedBeanDefinitionPostProcessor) {
				internalPostProcessors.add(pp);
			}
		}
		sortPostProcessors(orderedPostProcessors, beanFactory);
		registerBeanPostProcessors(beanFactory, orderedPostProcessors);

		// 最后注册其它BeanPostProcessor
		List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<>();
		for (String ppName : nonOrderedPostProcessorNames) {
			BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
			nonOrderedPostProcessors.add(pp);
			if (pp instanceof MergedBeanDefinitionPostProcessor) {
				internalPostProcessors.add(pp);
			}
		}
		registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);

		// Finally, re-register all internal BeanPostProcessors.
		sortPostProcessors(internalPostProcessors, beanFactory);
		registerBeanPostProcessors(beanFactory, internalPostProcessors);

		// Re-register post-processor for detecting inner beans as ApplicationListeners,
		// moving it to the end of the processor chain (for picking up proxies etc).
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
	}
```



###### PriorityOrdered

> org.springframework.core.PriorityOrdered

```java
public interface PriorityOrdered extends Ordered {
}
```



###### Ordered

> org.springframework.core.Ordered

```java
public interface Ordered {
	/**
	 * 
	 */
	int HIGHEST_PRECEDENCE = Integer.MIN_VALUE;

	/**
	 * 
	 */
	int LOWEST_PRECEDENCE = Integer.MAX_VALUE;

	/**
	 * 
	 */
	int getOrder();
}
```

根据排序接口返回值排序，默认升序排序，返回值越低优先级越高。

PriorityOrdered、Ordered接口作为Spring整个框架通用的排序接口，在Spring中应用广泛，也是非常重要的接口。



##### 调用时机

###### postProcessBeforeInitialization

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
    // ...
    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }
    // ...
    return wrappedBean;
}
```



> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
    throws BeansException {

    Object result = existingBean;
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        Object current = processor.postProcessBeforeInitialization(result, beanName);
        if (current == null) {
            return result;
        }
        result = current;
    }
    return result;
}
```



###### postProcessAfterInitialization

- 场景一

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
	// ...
                if (bean != null) {
                    bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
	// ...
}
```



> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
			Object current = processor.postProcessAfterInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
	}
```



- 场景二

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
		// ...
		if (mbd == null || !mbd.isSynthetic()) {
			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
		}

		return wrappedBean;
	}
```



#### InitializingBean

InitializingBean顾名思义，是初始化Bean相关的接口。



> org.springframework.beans.factory.InitializingBean

```java
public interface InitializingBean {
	/**
	 * 
	 */
	void afterPropertiesSet() throws Exception;

}
```



##### 调用时机

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
		try {
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
		// ...
		return wrappedBean;
	}
```



> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)
			throws Throwable {

		boolean isInitializingBean = (bean instanceof InitializingBean);
		if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
			// ...
			if (System.getSecurityManager() != null) {
				// ...
			}
			else {
				((InitializingBean) bean).afterPropertiesSet();
			}
		}
		// ...
	}
```



看方法名，是在读完Properties文件，之后执行的方法。afterPropertiesSet()方法是在初始化过程中被调用的。

InitializingBean 对应生命周期的初始化阶段，在上面源码的invokeInitMethods(beanName, wrappedBean, mbd);方法中调用。

有一点需要注意，因为Aware方法都是执行在初始化方法之前，所以可以在初始化方法中放心大胆的使用Aware接口获取的资源，这也是我们自定义扩展Spring的常用方式。



#### DisposableBean

DisposableBean 类似于InitializingBean，对应生命周期的销毁阶段，**以ConfigurableApplicationContext#close()方法作为入口**，实现是通过循环获取所有实现了DisposableBean接口的Bean然后调用其destroy()方法 。



> org.springframework.beans.factory.DisposableBean

```java
public interface DisposableBean {
	/**
	 * 
	 */
	void destroy() throws Exception;
}
```



也就是说，在对象销毁的时候，会去调用DisposableBean的destroy方法。在进入到销毁过程时先去调用一下DisposableBean的destroy方法，然后后执行 destroy-method声明的方法（用来销毁Bean中的各项数据）。



> org.springframework.beans.factory.support.DefaultSingletonBeanRegistry

```java
protected void destroyBean(String beanName, @Nullable DisposableBean bean) {
    // ...
    // 实际销毁对象
    if (bean != null) {
        try {
            bean.destroy();
        }
        // ...
    }
    // ...
}
```



### 总结

![img](../../Image/2022/08/220804-2.png)



Spring Bean的生命周期分为`四个阶段`和`多个扩展点`。扩展点又可以分为`影响多个Bean`和`影响单个Bean`。整理如下：



#### 四个阶段

**实例化 > 属性赋值 > 初始化 > 销毁**



#### 多个扩展点

- 影响多个Bean
	- BeanPostProcessor
	- InstantiationAwareBeanPostProcessor
- 影响单个Bean
	- Aware
		- 对象相关Aware
			- BeanNameAware
			- BeanClassLoaderAware
			- BeanFactoryAware
		- ApplicationContext相关Aware
			- EnvironmentAware
			- EmbeddedValueResolverAware
			- ApplicationContextAware(ResourceLoaderAware\ApplicationEventPublisherAware\MessageSourceAware)
	- 生命周期
		- InitializingBean
		- DisposableBean



## FactoryBean

一般情况下，Spring通过反射机制利用<bean>的class属性指定实现类实例化Bean，在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在<bean>中提供大量的配置信息。配置方式的灵活性是受限的，这时采用编码的方式可能会得到一个简单的方案。Spring为此提供了一个org.springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑。FactoryBean接口对于Spring框架来说占用重要的地位，Spring自身就提供了70多个FactoryBean的实现。

```java
@Service
public class DefaultFactoryBean implements FactoryBean<SchoolBean> {
    private String name = "default factory bean";

    @Override
    public SchoolBean getObject() {
        return new SchoolBean(1L, "第一中学");
    }

    @Override
    public Class<?> getObjectType() {
        return SchoolBean.class;
    }
}
```



### 源码解析

```java
public interface FactoryBean<T> {
    String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";
    
    /**
     * 返回由FactoryBean创建的Bean实例，
     * 如果isSingleton()返回true，则该实例会放到Spring容器中单实例缓存池中；
     */
    @Nullable
    T getObject() throws Exception;
    
    /**
     * 返回FactoryBean创建的Bean类型。
     */
    @Nullable
    Class<?> getObjectType();
    
    /**
     * 返回由FactoryBean创建的Bean实例的作用域是singleton还是prototype
     */
    default boolean isSingleton() {
        return true;
    }
}
```



FactoryBean 接口提供三个方法，用来创建对象，FactoryBean 具体返回的对象是由getObject 方法决定的。



### 扩展

#### BeanFactory 和 FactoryBean 的区别

BeanFactory 是 Bean 的工厂， ApplicationContext 的父类，IOC 容器的核心，负责生产和管理 Bean 对象。

FactoryBean 是 Bean，可以通过实现 FactoryBean 接口定制实例化 Bean 的逻辑，通过代理一个Bean对象，对方法前后做一些操作。



`BeanFactory` 是接口，提供了IOC容器最基本的形式，给具体的IOC容器的实现提供了规范；

`FactoryBean` 也是接口，为IOC容器中Bean的实现提供了更加灵活的方式，`FactoryBean` 在IOC容器的基础上给Bean的实现加上了一个简单工厂模式和装饰模式，可以在getObject()方法中灵活配置。

```java
@Service
public class DefaultBeanFactory implements BeanFactoryAware {
    private BeanFactory beanFactory;

    @Override
    public void setBeanFactory(@NonNull BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    public void execute() {
        String beanName = "defaultFactoryBean";
        Object bean = beanFactory.getBean(beanName);
        if(bean instanceof SchoolBean) {
            System.err.println("bean类型是SchoolBean");
        }
        Object factoryBean = beanFactory.getBean(BeanFactory.FACTORY_BEAN_PREFIX + beanName);
        if(factoryBean instanceof FactoryBean) {
            System.err.println("bean类型是FactoryBean");
        }
    }
}
```



## 注解

### @Autowired

@Autowired默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在（可以设置它required属性为false）。@Autowired 注解提供了更细粒度的控制，包括在何处以及如何完成自动装配。它的用法和@Required一样，修饰setter方法、构造器、属性或者具有任意名称和/或多个参数的PN方法。

@Autowired注解用于标记Spring将要解析和注入的依赖项。此注解可以作用在构造函数、字段和setter方法上。



#### 源码解析

> org.springframework.beans.factory.annotation.Autowired

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {
   /**
    * 
    */
   boolean required() default true;
}
```



#### 扩展

##### @Autowired和@Resource之间的区别

@Autowired可用于：构造函数、成员变量、Setter方法

@Autowired和@Resource之间的区别

- @Autowired默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在（可以设置它required属性为false）。
- @Resource默认是按照名称来装配注入的，只有当找不到与名称匹配的bean才会按照类型来装配注入



### @Qualifier 

当您创建多个相同类型的 bean 并希望仅使用属性装配其中一个 bean 时，您可以使用@Qualifier 注解和 @Autowired 通过指定应该装配哪个确切的 bean 来消除歧义。

当系统中存在同一类型的多个Bean时，@Autowired在进行依赖注入的时候就不知道该选择哪一个实现类进行注入。此时，我们可以使用@Qualifier注解来微调，帮助@Autowired选择正确的依赖项。



#### 源码解析

> org.springframework.beans.factory.annotation.Qualifier

```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Qualifier {
   String value() default "";
}
```



### @Required

> 在Spring 5.1版本废弃



这个注解表明bean的属性必须在配置的时候设置，通过一个bean定义的显式的属性值或通过自动装配，若@Required注解的bean属性未被设置，容器将抛出BeanInitializationException。



#### 源码解析

> org.springframework.beans.factory.annotation.Required

```java
@Deprecated
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Required {

}
```





## 循环依赖

循环依赖：说白是一个或多个对象实例之间存在直接或间接的依赖关系，这种依赖关系构成了构成一个环形调用。



### 三级缓存

解决循环依赖的问题就是三级缓存，通过三级缓存提前拿到未初始化的对象。

第一级缓存：用来保存实例化、初始化都完成的对象

第二级缓存：用来保存实例化完成，但是未初始化完成的对象

第三级缓存：用来保存一个对象工厂，提供一个匿名内部类，用于创建二级缓存中的对象

![图片](../../Image/2022/08/220804-9.png)

因为三级缓存中放的是生成具体对象的匿名内部类，他可以生成代理对象，也可以是普通的实例对象。使用三级缓存主要是为了保证不管什么时候使用的都是一个对象。

假设只有二级缓存的情况，往二级缓存中放的显示一个普通的Bean对象，`BeanPostProcessor`去生成代理对象之后，覆盖掉二级缓存中的普通Bean对象，那么多线程环境下可能取到的对象就不一致了。



> org.springframework.beans.factory.support.DefaultSingletonBeanRegistry

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
}
```



### 循环依赖场景

循环依赖主要场景如下：

- 单例的setter注入（能解决）；
- 多例的setter注入（不能解决）；
- 构造器注入（不能解决）；
- 单例的代理对象setter注入（有可能解决）；
- DependsOn循环依赖（不能解决）；



#### 单例setter注入

##### 两个Bean互相依赖

```java
@Service
publicclass TestService1 {
    @Autowired
    private TestService2 testService2;
}

@Service
publicclass TestService2 {
    @Autowired
    private TestService1 testService1;
}
```

spring内部有三级缓存：

- singletonObjects 一级缓存，用于保存实例化、注入、初始化完成的bean实例
- earlySingletonObjects 二级缓存，用于保存实例化完成的bean实例
- singletonFactories 三级缓存，用于保存bean创建工厂，以便于后面扩展有机会创建代理对象。



![img](../../Image/2022/08/220804-3.png)



A对象的创建过程：

1. 创建对象A，实例化的时候把A对象工厂放入三级缓存
2. A注入属性时，发现依赖B，转而去实例化B
3. 同样创建对象B，注入属性时发现依赖A，一次从一级到三级缓存查询A，从三级缓存通过对象工厂拿到A，把A放入二级缓存，同时删除三级缓存中的A，此时，B已经实例化并且初始化完成，把B放入一级缓存。
4. 接着继续创建A，顺利从一级缓存拿到实例化且初始化完成的B对象，A对象创建也完成，删除二级缓存中的A，同时把A放入一级缓存
5. 最后，一级缓存中保存着实例化、初始化都完成的A、B对象



##### 多个Bean互相依赖

上述场景中**第二级缓存**作用不大，二级缓存主要用于多个Bean互相依赖的情况。如下所示：



```java
@Service
publicclass TestService1 {
    @Autowired
    private TestService2 testService2;
    @Autowired
    private TestService3 testService3;
}

@Service
publicclass TestService2 {
    @Autowired
    private TestService1 testService1;
}

@Service
publicclass TestService3 {
    @Autowired
    private TestService1 testService1;
}
```



TestService1依赖于TestService2和TestService3，而TestService2依赖于TestService1，同时TestService3也依赖于TestService1。按照上图的流程可以把TestService1注入到TestService2，并且TestService1的实例是从第三级缓存中获取的。

假设不用第二级缓存，TestService1注入到TestService3的流程如图：

![img](../../Image/2022/08/220804-4.png)



TestService1注入到TestService3又需要从第三级缓存中获取实例，而第三级缓存里保存的并非真正的实例对象，而是`ObjectFactory`对象。**两次从三级缓存中获取都是`ObjectFactory`对象，而通过它创建的实例对象每次可能都不一样的。**



> 第三级缓存中添加`ObjectFactory`对象，而不是直接保存实例对象，主要是为了对添加到三级缓存中的实例对象进行增强。



为了解决这个问题，spring引入的第二级缓存。前一个图其实TestService1对象的实例已经被添加到第二级缓存中了，而在TestService1注入到TestService3时，只用从第二级缓存中获取该对象即可。

![img](../../Image/2022/08/220804-5.png)



###### 源码解析

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {
		// ...
		// 即使被 BeanFactoryAware 等生命周期接口触发，也缓存单例以解析循环引用。
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			// ...
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}
    }

	/*
	 * 获取对指定 bean 的早期访问的引用，通常用于解析循环引用。
	 */
	protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
		Object exposedObject = bean;
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
					SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
					exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
				}
			}
		}
		return exposedObject;
	}
```



这种场景下Spring定义了一个匿名内部类，通过`getEarlyBeanReference`方法获取代理对象，其实底层是通过`AbstractAutoProxyCreator`类的`getEarlyBeanReference`生成代理对象。



#### 多例的setter注入

```java
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Service
publicclass TestService1 {
    @Autowired
    private TestService2 testService2;
}

@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Service
publicclass TestService2 {
    @Autowired
    private TestService1 testService1;
}
```



这种情况下虽然是原型场景下的循环依赖，但是程序能够正常启动。因为Spring中非抽象、单例 并且非懒加载的类才能被提前初始bean。而多例即`SCOPE_PROTOTYPE`类型的类，非单例，不会被提前初始化bean，所以程序能够正常启动。



```java
@Service
publicclass TestService3 {
    @Autowired
    private TestService1 testService1;
}
```



如果再定义一个单例的类，在它里面注入TestService1，此时就会要初始化TestService1导致抛出异常。执行结果：

```
Requested bean is currently in creation: Is there an unresolvable circular reference?
```



这种循环依赖问题是无法解决的，因为它没有用缓存，每次都会生成一个新对象。



##### 原型场景

原型(Prototype)的场景是不支持循环依赖的， 单例的场景才能存在循环依赖。原型(Prototype)的场景通常会走到AbstractBeanFactory类中下面的判断，抛出异常。



> org.springframework.beans.factory.support.AbstractBeanFactory

```java
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
		// ...
		if (sharedInstance != null && args == null) {
			// ...
		}

		else {
			// 如果已经在创建这个 bean 实例，则失败：可能在循环引用中。
			if (isPrototypeCurrentlyInCreation(beanName)) {
				throw new BeanCurrentlyInCreationException(beanName);
			}
			// ...
			}
		}
```



因为创建新的A时，发现要注入原型字段B，又创建新的B发现要注入原型字段A。这就套娃了, Spring就先抛出了BeanCurrentlyInCreationException。



##### 源码解析

在`AbstractApplicationContext`类的`refresh`方法中会调用`finishBeanFactoryInitialization`方法，该方法的作用是为了spring容器启动的时候提前初始化一些bean。该方法的内部又调用了`preInstantiateSingletons`方法。



> org.springframework.context.support.AbstractApplicationContext

```java
	protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
		// ...
		// 实例化所有剩余的（非惰性初始化）单例
		beanFactory.preInstantiateSingletons();
	}
```



> org.springframework.beans.factory.support.DefaultListableBeanFactory

```java
	public void preInstantiateSingletons() throws BeansException {
		// ...
		for (String beanName : beanNames) {
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
                // 非抽象、单例 并且非懒加载的类才能被提前初始bean。
				// ... 
			}
		}
		// ...
	}
```



#### 构造器注入

```java
@Service
publicclass TestService1 {
    public TestService1(TestService2 testService2) {
    }
}
@Service
publicclass TestService2 {
    public TestService2(TestService1 testService1) {
    }
}
```



运行结果：

```
Requested bean is currently in creation: Is there an unresolvable circular reference?
```



出现了循环依赖，构造器注入没能添加到三级缓存，也没有使用缓存，所以也无法解决循环依赖问题。如下所示：

![img](../../Image/2022/08/220804-6.png)



由于把实例化和初始化的流程分开了才能解决循环依赖，所以如果都是用构造器的话，就没法分离这个操作，所以都是构造器的话就无法解决循环依赖的问题了。



#### 单例的代理对象setter注入

这种注入方式其实也比较常用，比如平时使用：`@Async`注解的场景，会通过`AOP`自动生成代理对象。

```
@Service
publicclass TestService1 {
    @Autowired
    private TestService2 testService2;
    @Async
    public void test1() {
    }
}
@Service
publicclass TestService2 {
    @Autowired
    private TestService1 testService1;
    public void test2() {
    }
}
```



程序启动会报错，出现了循环依赖：

```
org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'testService1': Bean with name 'testService1' has been injected into other beans [testService2] in its raw version as part of a circular reference, but has eventually been wrapped. This means that said other beans do not use the final version of the bean. This is often the result of over-eager type matching - consider using 'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.
```



![img](../../Image/2022/08/220804-7.png)



bean初始化完成之后，后面还有一步去检查：第二级缓存 和 原始对象 是否相等。



##### 源码解析

> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory

```java
	protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {
		// ...
		if (earlySingletonExposure) {
			Object earlySingletonReference = getSingleton(beanName, false);
            // 二级缓存和原始对象不相等，因此抛出了异常
			if (earlySingletonReference != null) {
				if (exposedObject == bean) {
					exposedObject = earlySingletonReference;
				}
				else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
					String[] dependentBeans = getDependentBeans(beanName);
					Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
					for (String dependentBean : dependentBeans) {
						if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
							actualDependentBeans.add(dependentBean);
						}
					}
					if (!actualDependentBeans.isEmpty()) {
						throw new BeanCurrentlyInCreationException(beanName,
								"Bean with name '" + beanName + "' has been injected into other beans [" +
								StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
								"] in its raw version as part of a circular reference, but has eventually been " +
								"wrapped. This means that said other beans do not use the final version of the " +
								"bean. This is often the result of over-eager type matching - consider using " +
								"'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
					}
				}
			}
		}
		// ...
	}
```



##### 扩展

###### bean加载顺序

上述循环依赖的情况下，如果这时候把TestService1改个名字，改成：TestService6，其他的都不变。

```java
@Service
publicclass TestService6 {
    @Autowired
    private TestService2 testService2;
    @Async
    public void test1() {
    }
}
```



此时启动程序就不会出现循环依赖的异常了，因为spring是按照文件完整路径递归查找的，按路径+文件名排序，排在前面的先加载。所以TestService1比TestService2先加载，而改了文件名称之后，TestService2比TestService6先加载。

![img](../../Image/2022/08/220804-8.png)



这种情况testService6中其实第二级缓存是空的，不需要跟原始对象判断，所以不会抛出循环依赖。



#### DependsOn循环依赖

还有一种有些特殊的场景，比如我们需要在实例化Bean A之前，先实例化Bean B，这个时候就可以使用`@DependsOn`注解。

```java
@DependsOn(value = "testService2")
@Service
publicclass TestService1 {
    @Autowired
    private TestService2 testService2;
}
@DependsOn(value = "testService1")
@Service
publicclass TestService2 {
    @Autowired
    private TestService1 testService1;
}
```



程序启动之后，执行结果：

```
Circular depends-on relationship between 'testService2' and 'testService1'
```



这个例子中本来如果TestService1和TestService2都没有加`@DependsOn`注解是没问题的，反而加了这个注解会出现循环依赖问题。

在AbstractBeanFactory类的doGetBean方法中会检查dependsOn的实例有没有循环依赖，如果有循环依赖则抛异常。



##### 源码解析

> org.springframework.beans.factory.support.AbstractBeanFactory

```java
	protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
				// ...

				// 保证当前 bean 所依赖的 bean 的初始化。
				String[] dependsOn = mbd.getDependsOn();
				if (dependsOn != null) {
					for (String dep : dependsOn) {
						if (isDependent(beanName, dep)) {
							throw new BeanCreationException(mbd.getResourceDescription(), beanName,
									"Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
						}
						registerDependentBean(dep, beanName);
						try {
							getBean(dep);
						}
						catch (NoSuchBeanDefinitionException ex) {
							throw new BeanCreationException(mbd.getResourceDescription(), beanName,
									"'" + beanName + "' depends on missing bean '" + dep + "'", ex);
						}
					}
				}
				// ...
	}
```



### 循环依赖解决

#### 生成代理对象产生的循环依赖

这类循环依赖问题解决方法很多，主要有：

1. 使用`@Lazy`注解，延迟加载
2. 使用`@DependsOn`注解，指定加载先后关系
3. 修改文件名称，改变循环依赖类的加载顺序



#### 使用@DependsOn产生的循环依赖 

这类循环依赖问题要找到`@DependsOn`注解循环依赖的地方，迫使它不循环依赖就可以解决问题。



#### 多例循环依赖

这类循环依赖问题可以通过把bean改成单例的解决。



#### 构造器循环依赖

这类循环依赖问题可以通过使用`@Lazy`注解解决



# Spring AOP

## 概念

OOP(Object-Oriented Programming)面向对象编程，允许开发者定义纵向的关系，但并适用于定义横向的关系，导致了大量代码的重复，而不利于各个模块的重用。

AOP(Aspect-Oriented Programming)，一般称为面向切面编程，作为面向对象的一种补充，用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑，抽取并封装为一个可重用的模块，这个模块被命名为“切面”（Aspect），减少系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性。可用于权限认证、日志、事务处理等。



AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**，并**有利于未来的可拓展性和可维护性**。

**Spring AOP就是基于动态代理的**，如果要代理的对象，实现了某个接口，那么Spring AOP会使用**JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用**Cglib** ，这时候Spring AOP会使用 **Cglib** 生成一个被代理对象的子类来作为代理，如下图所示：

![SpringAOPProcess](../../Image/2022/07/220719-1.png)



通过在代理类中包裹切面，Spring在运行期把切面织入到Spring管理的bean中。代理封装了目标类，并拦截被通知方法的调用，再把调用转发给真正的目标bean。当代理拦截到方法调用时，在调用目标bean方法之前，会执行切面逻辑。

直到应用需要被代理的bean时，Spring才创建代理对象。如果使用的是ApplicationContext的话，在ApplicationContext从BeanFactory中加载所有bean的时候，Spring才会创建被代理的对象。因为Spring运行时才创建代理对象，所以我们不需要特殊的编译器来织入SpringAOP的切面。



### 基础名词概述

#### 切面

（1）切面（Aspect）：切面是通知和切点的结合。通知和切点共同定义了切面的全部内容。 在Spring AOP中，切面可以使用通用类（基于模式的风格） 或者在普通类中以 @AspectJ 注解来实现。

它既包含了横切逻辑的定义, 也包括了连接点的定义. Spring AOP 就是负责实施切面的框架, 它将切面所定义的横切逻辑编织到切面所指定的连接点中.
AOP 的工作重心在于如何将增强编织目标对象的连接点上, 这里包含两个工作:

- 如何通过 pointcut 和 advice 定位到特定的 joinpoint 上
- 如何在 advice 中编写切面代码.

可以简单地认为, 使用 @Aspect 注解的类就是切面。



#### 连接点

（2）连接点（Join point）：指方法，在Spring AOP中，一个连接点 总是 代表一个方法的执行。 应用可能有数以千计的时机应用通知。这些时机被称为连接点。连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。

**因为Spring基于动态代理，所以Spring只支持方法连接点。**Spring缺少对字段连接点的支持，而且它不支持构造器连接点。方法之外的连接点拦截功能，我们可以利用Aspect来补充。



#### 通知

（3）通知（Advice）：在AOP术语中，切面的工作被称为通知，实际上是程序执行时要通过SpringAOP框架触发的代码段。



##### 通知类型

Spring切面可以应用5种类型的通知：

![img](../../Image/2022/08/220804-10.png)



1. 前置通知（Before）：在目标方法被调用之前调用通知功能；
2. 后置通知（After）：在目标方法完成之后调用通知，此时不会关心方法的输出是什么；
3. 返回通知（After-returning ）：在目标方法成功执行之后调用通知；
4. 异常通知（After-throwing）：在目标方法抛出异常后调用通知；
5. 环绕通知（Around）：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。



#### 切入点

（4）切入点（Pointcut）：切点的定义会匹配通知所要织入的一个或多个连接点。我们通常使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。



#### 引入

（5）引入（Introduction）：引入允许我们向现有类添加新方法或属性。



#### 目标对象

（6）目标对象（Target Object）： 被一个或者多个切面（aspect）所通知（advise）的对象。它通常是一个代理对象。也有人把它叫做 被通知（adviced） 对象。 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个 被代理（proxied） 对象。



#### 织入

（7）织入（Weaving）：织入是把切面应用到目标对象并创建新的代理对象的过程。在目标对象的生命周期里有多少个点可以进行织入：

- 编译期：切面在目标类编译时被织入。AspectJ的织入编译器是以这种方式织入切面的。
- 类加载期：切面在目标类加载到JVM时被织入。需要特殊的类加载器，它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ5的加载时织入就支持以这种方式织入切面。
- 运行期：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。SpringAOP就是以这种方式织入切面。



### 扩展

#### Spring AOP 和 AspectJ AOP 有什么区别？

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多。



AOP实现的关键在于 代理模式，AOP代理主要分为静态代理和动态代理。静态代理的代表为AspectJ；动态代理则以Spring AOP为代表。

静态代理与动态代理区别在于生成AOP代理对象的时机不同，相对来说AspectJ的静态代理方式具有更好的性能，但是AspectJ需要特定的编译器进行处理，而Spring AOP则无需特定的编译器处理。



（1）AspectJ是静态代理的增强，所谓静态代理，就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强，他会在编译阶段将AspectJ(切面)织入到Java字节码中，运行的时候就是增强之后的AOP对象。

（2）Spring AOP使用的动态代理，所谓的动态代理就是说AOP框架不会去修改字节码，而是每次运行时在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。



#### JDK动态代理和CGLIB动态代理的区别

Spring AOP中的动态代理主要有两种方式，JDK动态代理和CGLIB动态代理：

JDK动态代理只提供接口的代理，不支持类的代理。核心InvocationHandler接口和Proxy类，InvocationHandler 通过invoke()方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起；接着，Proxy利用 InvocationHandler动态创建一个符合某一接口的的实例, 生成目标类的代理对象。

如果代理类没有实现接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。



## 通知

### 通知类型

#### 前置通知

#### 后置通知

#### 返回通知

#### 异常通知

#### 环绕通知



### 通知执行顺序

#### 正常情况

around before > before > target method 执行 > around after > after > afterReturning



#### 异常情况

around before > before > target method 执行 > around after > after >  afterThrowing



# Spring Web MVC

## 概念

### MVC框架

mvc是一种设计模式（设计模式就是日常开发中编写代码的一种好的方法和经验的总结）。模型（model）-视图（view）-控制器（controller），三层架构的设计模式。用于实现前端页面的展现与后端业务数据处理的分离。

mvc设计模式的好处：

1.分层设计，实现了业务系统各个组件之间的解耦，有利于业务系统的可扩展性，可维护性。

2.有利于系统的并行开发，提升开发效率。



### Spring MVC

Spring MVC是一个基于Java的实现了MVC设计模式的请求驱动类型的轻量级Web框架，通过把模型-视图-控制器分离，将web层进行职责解耦，把复杂的web应用分成逻辑清晰的几部分，简化开发，减少出错，方便组内开发人员之间的配合。

Spring MVC 提供了一种分离式的方法来开发 Web 应用。通过运用像 DispatcherServelet，MoudlAndView 和 ViewResolver 等一些简单的概念，开发 Web 应用将会变的非常简单。



### Spring MVC优点

（1）可以支持各种视图技术,而不仅仅局限于JSP；

（2）与Spring框架集成（如IoC容器、AOP等）；

（3）清晰的角色分配：前端控制器(dispatcherServlet) , 请求到处理器映射（handlerMapping), 处理器适配器（HandlerAdapter), 视图解析器（ViewResolver）。

（4） 支持各种请求资源的映射策略。



### 核心组件

#### DispatcherServlet

（1）前端控制器 DispatcherServlet（不需要程序员开发）

作用：接收请求、响应结果，相当于转发器，有了DispatcherServlet 就减少了其它组件之间的耦合度。

Spring的MVC框架是围绕DispatcherServlet来设计的，它用来处理所有的HTTP请求和响应。



#### HandlerMapping

（2）处理器映射器HandlerMapping（不需要程序员开发）

作用：根据请求的URL来查找Handler



#### HandlerAdapter

（3）处理器适配器HandlerAdapter

注意：在编写Handler的时候要按照HandlerAdapter要求的规则去编写，这样适配器HandlerAdapter才可以正确的去执行Handler。



#### Handler

（4）处理器Handler（需要程序员开发）

控制器提供一个访问应用程序的行为，此行为通常通过服务接口实现。控制器解析用户输入并将其转换为一个由视图呈现给用户的模型。Spring用一个非常抽象的方式实现了一个控制层，允许用户创建多种用途的控制器。

Spring MVC的控制器是单例模式,所以在多线程访问的时候有线程安全问题,不要用同步,会影响性能的,解决方案是在控制器里面不能写字段。



#### ViewResolver

（5）视图解析器 ViewResolver（不需要程序员开发）

作用：进行视图的解析，根据视图逻辑名解析成真正的视图（view）



#### View

（6）视图View（需要程序员开发jsp）

View是一个接口， 它的实现类支持不同的视图类型（jsp，freemarker，pdf等等）



## 原理

### 核心执行流程

![SpringMVC运行原理](../../Image/2022/07/220719-3.png)



1. 客户端（浏览器）发送请求，直接请求到 `DispatcherServlet`。
2. `DispatcherServlet` 根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`。
3. 解析到对应的 `Handler`（也就是我们平常说的 `Controller` 控制器）后，开始由 `HandlerAdapter` 适配器处理。
4. `HandlerAdapter` 会根据 `Handler `来调用真正的处理器开处理请求，并处理相应的业务逻辑。
5. 处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，`View` 是个逻辑上的 `View`。
6. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
7. `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
8. 把 `View` 返回给请求者（浏览器）



（1）用户发送请求至前端控制器DispatcherServlet；

（2） DispatcherServlet收到请求后，调用HandlerMapping处理器映射器，请求获取Handle；

（3）处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet；

（4）DispatcherServlet 调用 HandlerAdapter处理器适配器；

（5）HandlerAdapter 经过适配调用 具体处理器(Handler，也叫后端控制器)；

（6）Handler执行完成返回ModelAndView；

（7）HandlerAdapter将Handler执行结果ModelAndView返回给DispatcherServlet；

（8）DispatcherServlet将ModelAndView传给ViewResolver视图解析器进行解析；

（9）ViewResolver解析后返回具体View；

（10）DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）

（11）DispatcherServlet响应用户。



```java
o.s.w.servlet.FrameworkServlet#doGet
o.s.w.servlet.FrameworkServlet#processRequest
o.s.w.servlet.DispatcherServlet#doService
o.s.w.servlet.DispatcherServlet#doDispatch
o.s.w.servlet.DispatcherServlet#getHandler
o.s.w.servlet.handler.AbstractHandlerMethodMapping#getHandlerInternal
o.s.w.servlet.handler.AbstractHandlerMethodMapping#lookupHandlerMethod
o.s.w.servlet.HandlerAdapter#handle
o.s.w.servlet.mvc.method.annotation.RequestMappingHandlerAdapter#invokeHandlerMethod
o.s.w.servlet.mvc.method.annotation.ServletInvocableHandlerMethod#invokeAndHandle
o.s.w.method.support.InvocableHandlerMethod#invokeForRequest
o.s.w.method.support.InvocableHandlerMethod#getMethodArgumentValues
o.s.w.method.support.InvocableHandlerMethod#doInvoke
o.s.w.method.support.HandlerMethodReturnValueHandlerComposite#handleReturnValue
o.s.w.servlet.mvc.method.annotation.RequestMappingHandlerAdapter#getModelAndView
```



## 注解

### @RestController

@RestController是在Spring 4.0开始引入的，这是一个特定的控制器注解。此注解相当于@Controller和@ResponseBody的快捷方式。当使用此注解时，不需要再在方法上使用@ResponseBody注解。



### @RequestMapping

> @RequestMapping注解属于Spring Web模块下的注解



RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。Spring MVC和Spring WebFlux都通过`RquestMappingHandlerMapping`和`RequestMappingHndlerAdapter`两个类来提供对@RequestMapping注解的支持。

@RequestMapping 注解用于将特定 HTTP 请求方法映射到将处理相应请求的控制器中的特定类/方法。此注释可应用于两个级别：

- 类级别：映射请求的 URL
- 方法级别：映射 URL 以及 HTTP 请求方法



#### 源码解析

> org.springframework.web.bind.annotation.RequestMapping

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {

	/**
	 * 
	 */
	String name() default "";

	/**
	 * 指定请求的实际地址，指定的地址可以是URI Template 模式（后面将会说明）；
	 */
	@AliasFor("path")
	String[] value() default {};

	/**
	 * 
	 * @since 4.2
	 */
	@AliasFor("value")
	String[] path() default {};

	/**
	 * 指定请求的method类型， GET、POST、PUT、DELETE等；
	 */
	RequestMethod[] method() default {};

	/**
	 * 指定request中必须包含某些参数值是，才让该方法处理。
	 */
	String[] params() default {};

	/**
	 * 指定request中必须包含某些指定的header值，才能让该方法处理请求。
	 */
	String[] headers() default {};

	/**
	 * 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;
	 */
	String[] consumes() default {};

	/**
	 * 指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回；
	 */
	String[] produces() default {};

}
```



### @GetMapping

@GetMapping注解用于处理HTTP GET请求，并将请求映射到具体的处理方法中。具体来说，@GetMapping是一个组合注解，它相当于是@RequestMapping(method=RequestMethod.GET)的快捷方式。



### @PostMapping

@PostMapping注解用于处理HTTP POST请求，并将请求映射到具体的处理方法中。@PostMapping与@GetMapping一样，也是一个组合注解，它相当于是@RequestMapping(method=HttpMethod.POST)的快捷方式。



### @PutMapping

@PutMapping注解用于处理HTTP PUT请求，并将请求映射到具体的处理方法中，@PutMapping是一个组合注解，相当于是@RequestMapping(method=HttpMethod.PUT)的快捷方式。



### @DeleteMapping

@DeleteMapping注解用于处理HTTP DELETE请求，并将请求映射到删除方法中。@DeleteMapping是一个组合注解，它相当于是@RequestMapping(method=HttpMethod.DELETE)的快捷方式。



### @PatchMapping

@PatchMapping注解用于处理HTTP PATCH请求，并将请求映射到对应的处理方法中。@PatchMapping相当于是@RequestMapping(method=HttpMethod.PATCH)的快捷方式。



### @RequestBody

> @RequestBody注解属于Spring Web模块下的注解



注解实现接收http请求的json数据，将json转换为java对象。

@RequestBody在处理请求方法的参数列表中使用，它可以将请求主体中的参数绑定到一个对象中，请求主体参数是通过`HttpMessageConverter`传递的，根据请求主体中的参数名与对象的属性名进行匹配并绑定值。此外，还可以通过@Valid注解对请求主体中的参数进行校验。



### @ResponseBody

> @ResponseBody注解属于Spring Web模块下的注解



注解实现将conreoller方法返回对象转化为json对象响应给客户。

该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区。

使用时机：返回的数据不是html标签的页面，而是其他某种格式的数据时（如json、xml等）使用；

@ResponseBody会自动将控制器中方法的返回值写入到HTTP响应中。特别的，@ResponseBody注解只能用在被@Controller注解标记的类中。如果在被@RestController标记的类中，则方法不需要使用@ResponseBody注解进行标注。@RestController相当于是@Controller和@ResponseBody的组合注解。



### @PathVariable

`@PathVariable`注解是将方法中的参数绑定到请求URI中的模板变量上。可以通过`@RequestMapping`注解来指定URI的模板变量，然后使用`@PathVariable`注解将方法中的参数绑定到模板变量上。

特别地，`@PathVariable`注解允许我们使用value或name属性来给参数取一个别名。如果参数是一个非必须的，可选的项，则可以在`@PathVariable`中设置`require = false`。



模板变量名需要使用`{ }`进行包裹，如果方法的参数名与URI模板变量名一致，则在`@PathVariable`中就可以省略别名的定义。

请求路径上有个id的变量值，可以通过@PathVariable来获取 

```java
@RequestMapping(value = “/page/{id}”, method = RequestMethod.GET)
```



### @RequestParam

@RequestParam用来获得静态的URL请求入参 spring注解时action里用到。

@RequestParam注解用于将方法的参数与Web请求的传递的参数进行绑定。使用@RequestParam可以轻松的访问HTTP请求参数的值。

该注解的其他属性配置与@PathVariable的配置相同，特别的，如果传递的参数为空，还可以通过defaultValue设置一个默认值。



### @ModelAttribute

通过此注解，可以通过模型索引名称来访问已经存在于控制器中的model。与`@PathVariable`和`@RequestParam`注解一样，如果参数名与模型具有相同的名字，则不必指定索引名称。

如果使用`@ModelAttribute`对方法进行标注，Spring会将方法的返回值绑定到具体的Model上。在Spring调用具体的处理方法之前，被`@ModelAttribute`注解标注的所有方法都将被执行。



### @ControllerAdvice

`@ControllerAdvice`是@Component注解的一个延伸注解，Spring会自动扫描并检测被@ControllerAdvice所标注的类。`@ControllerAdvice`需要和`@ExceptionHandler`、`@InitBinder`以及`@ModelAttribute`注解搭配使用，主要是用来处理控制器所抛出的异常信息。

首先，我们需要定义一个被`@ControllerAdvice`所标注的类，在该类中，定义一个用于处理具体异常的方法，并使用@ExceptionHandler注解进行标记。

此外，在有必要的时候，可以使用`@InitBinder`在类中进行全局的配置，还可以使用@ModelAttribute配置与视图相关的参数。使用`@ControllerAdvice`注解，就可以快速的创建统一的，自定义的异常处理类。



### @ExceptionHandler

@ExceptionHander注解用于标注处理特定类型异常类所抛出异常的方法。当控制器中的方法抛出异常时，Spring会自动捕获异常，并将捕获的异常信息传递给被@ExceptionHandler标注的方法。



### @ResponseStatus

@ResponseStatus注解可以标注请求处理方法。使用此注解，可以指定响应所需要的HTTP STATUS。特别地，我们可以使用HttpStauts类对该注解的value属性进行赋值。



### **@CrossOrigin**

`@CrossOrigin`注解将为请求处理类或请求处理方法提供跨域调用支持。如果我们将此注解标注类，那么类中的所有方法都将获得支持跨域的能力。使用此注解的好处是可以微调跨域行为。



#### @InitBinder

@InitBinder注解用于标注初始化WebDataBinider 的方法，该方法用于对Http请求传递的表单数据进行处理，如时间格式化、字符串处理等。



## 拦截器

### 实现HandlerInterceptor接口

> org.springframework.web.servlet.HandlerInterceptor

```java
public interface HandlerInterceptor {
	/**
	 * 
	 */
	default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {

		return true;
	}

	/**
	 * 
	 */
	default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable ModelAndView modelAndView) throws Exception {
	}

	/**
	 * 
	 */
	default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable Exception ex) throws Exception {
	}
}
```



### 继承适配器类

> org.springframework.web.servlet.handler.HandlerInterceptorAdapter

```java
public abstract class HandlerInterceptorAdapter implements AsyncHandlerInterceptor {
	/**
	 * This implementation always returns {@code true}.
	 */
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		return true;
	}

	/**
	 * This implementation is empty.
	 */
	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable ModelAndView modelAndView) throws Exception {
	}

	/**
	 * This implementation is empty.
	 */
	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable Exception ex) throws Exception {
	}

	/**
	 * This implementation is empty.
	 */
	@Override
	public void afterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response,
			Object handler) throws Exception {
	}
}
```



## 扩展

### 重定向和转发

（1）转发：在返回值前面加"forward:"，譬如"forward:user.do?name=method4"

（2）重定向：在返回值前面加"redirect:"，譬如"redirect:[http://www.baidu.com](http://www.baidu.com/)"





### 方法里面得到Request,或者Session

直接在方法的形参中声明request,Spring MVC就自动把request对象传入。



# Spring Transactions

## 概念

**事务能否生效数据库引擎是否支持事务是关键。比如常用的 MySQL 数据库默认使用支持事务的 `innodb`引擎。但是，如果把数据库引擎变为 `myisam`，那么程序也就不再支持事务了！**

Spring支持两种类型的事务管理：

**编程式事务管理**：这意味你通过编程的方式管理事务，给你带来极大的灵活性，但是难维护。

**声明式事务管理**：这意味着你可以将业务代码和事务管理分离，你只需用注解和XML配置来管理事务。



### 事务管理优点

- 为不同的事务API 如 JTA，JDBC，Hibernate，JPA 和JDO，提供一个不变的编程模式。
- 为编程式事务管理提供了一套简单的API而不是一些复杂的事务API
- 支持声明式事务管理。
- 和Spring各种数据访问抽象层很好得集成。



## 编程式事务

编程式事务管理使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。对于编程式事务管理，spring推荐使用TransactionTemplate。



### 实际使用

#### TransactionTemplate

```java
@Autowired
private TransactionTemplate transactionTemplate;
public void testTransaction() {

        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {

                try {

                    // ....  业务代码
                } catch (Exception e){
                    //回滚
                    transactionStatus.setRollbackOnly();
                }

            }
        });
}
```



#### TransactionManager

```java
@Autowired
private PlatformTransactionManager transactionManager;

public void testTransaction() {

  TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
          try {
               // ....  业务代码
              transactionManager.commit(status);
          } catch (Exception e) {
              transactionManager.rollback(status);
          }
}
```



## 声明式事务

声明式事务管理建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。



### 基于XML

基于XML的声明式事务。



### 基于注解

Spring在TransactionDefinition接口中规定了7种类型的事务传播行为，它们规定了事务方法和事务方法发生嵌套调用时事务如何进行传播：

#### @Transactional注解

##### 作用范围

当`@Transactional`注解作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。如果类或者方法加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚。

`@Transactional` 注解也可以添加到类级别上，当把`@Transactional` 注解放在类级别时，表示所有该类的公共方法都配置相同的事务属性信息。当类级别配置了`@Transactional`，方法级别也配置了`@Transactional`，应用程序会以方法级别的事务属性信息来管理事务，换言之，方法级别的事务属性信息会覆盖类级别的相关配置信息。



1. **方法** ：推荐将注解使用于方法上，不过需要注意的是：**该注解只能应用到 public 方法上，否则不生效。**
2. **类** ：如果这个注解使用在类上的话，表明该注解对该类中所有的 public 方法都生效。
3. **接口** ：不推荐在接口上使用。



##### rollbackFor

在`@Transactional`注解中如果不配置`rollbackFor`属性,那么事物只会在遇到`RuntimeException`的时候才会回滚,加上`rollbackFor=Exception.class`,可以让事物在遇到非运行时异常时也回滚。

默认情况下，如果在事务中抛出了未检查异常（继承自 `RuntimeException` 的异常）或者 `Error`，则 Spring 将回滚事务；除此之外，Spring 不会回滚事务。（**此处需要留意`@Transactional`默认不会对检测型异常回滚**）

如果在事务中抛出其他类型的异常，并期望 Spring 能够回滚事务，可以指定 rollbackFor。例：

```java
@Transactional(propagation= Propagation.REQUIRED,rollbackFor= MyException.class)
```

若在目标方法中抛出的异常是 `rollbackFor` 指定的异常的子类，事务同样会回滚。



##### readOnly

对于只有读取数据查询的事务，可以指定事务类型为 readonly，即只读事务。只读事务不涉及数据的修改，数据库会提供一些优化手段，适合用在有多条数据库查询操作的方法中。



> MySQL 默认对每一个新建立的连接都启用了`autocommit`模式。在该模式下，每一个发送到 MySQL 服务器的`sql`语句都会在一个单独的事务中进行处理，执行结束后会自动提交事务，并开启一个新的事务。



如果不加`Transactional`，每条`sql`会开启一个单独的事务，中间被其它事务改了数据，都会实时读取到最新值，可能会出现读数据不一致的状态。



##### 源码解析

> org.springframework.transaction.annotation.Transactional

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {

	/**
	 * 当在配置文件中有多个 TransactionManager , 可以用该属性指定选择哪个事务管理器。
	 */
	@AliasFor("transactionManager")
	String value() default "";

	@AliasFor("value")
	String transactionManager() default "";

	/**
	 * 事务的传播行为，默认值为 REQUIRED。
	 */
	Propagation propagation() default Propagation.REQUIRED;

	/**
	 * 事务的隔离度，默认值采用 DEFAULT。
	 */
	Isolation isolation() default Isolation.DEFAULT;

	/**
	 * 事务的超时时间，单位是秒，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。
	 */
	int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;

	/**
	 * 指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，
	 * 比如读取数据，可以设置 read-only 为 true。
	 */
	boolean readOnly() default false;

	/**
	 * 	用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔。
	 */
	Class<? extends Throwable>[] rollbackFor() default {};

	String[] rollbackForClassName() default {};

	/**
	 * 抛出 no-rollback-for 指定的异常类型，不回滚事务。
	 */
	Class<? extends Throwable>[] noRollbackFor() default {};

	String[] noRollbackForClassName() default {};
```



## 事务隔离级别

TransactionDefinition 接口中定义了五个表示隔离级别的常量。



| 类型                           | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| **ISOLATION_DEFAULT**          | 使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别。 |
| **ISOLATION_READ_UNCOMMITTED** | 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读** |
| **ISOLATION_READ_COMMITTED**   | 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生** |
| **ISOLATION_REPEATABLE_READ**  | 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。** |
| **ISOLATION_SERIALIZABLE**     | 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。 |



## 事务传播行为

TransactionDefinition 接口中定义了七个表示事务传播行为的常量。



| 类型                          | 描述                                           |
| ----------------------------- | ---------------------------------------------- |
| **PROPAGATION_REQUIRED**      | 当前存在事务则加入，否则新建事务执行（默认）   |
| **PROPAGATION_SUPPORTS**      | 当前存在事务则加入，否则非事务执行             |
| **PROPAGATION_MANDATORY**     | 当前存在事务则加入，否则抛出异常               |
| **PROPAGATION_NESTED**        | 当前存在事务则加入为嵌套事务，否则新建事务执行 |
| **PROPAGATION_REQUIRES_NEW**  | 新建事务执行，当前存在事务则挂起当前事务       |
| **PROPAGATION_NOT_SUPPORTED** | 非事务执行行，当前存在事务则挂起当前事务       |
| **PROPAGATION_NEVER**         | 非事务执行行，当前存在事务则抛出异常。         |



### REQUIRED

这个级别通常能满足处理大多数的业务场景。

**外围方法未开启事务的情况下`Propagation.REQUIRED`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。**

**外围方法开启事务的情况下`Propagation.REQUIRED`修饰的内部方法会加入到外围方法的事务中，所有`Propagation.REQUIRED`修饰的内部方法和外围方法均属于同一事务，只要一个方法回滚，整个事务均回滚。**



### SUPPORTS

**并非所有的包在**`transactionTemplate.execute`**中的代码都会有事务支持**。这个通常是用来处理那些并非原子性的非核心业务逻辑操作。应用场景较少。（事务可能不会发生回滚）



### MANDATORY

配置该方式的传播级别是**有效的控制上下文调用代码遗漏添加事务控制的保证手段**。比如一段代码不能单独被调用执行，但是一旦被调用，就必须有事务包含的情况，就可以使用这个传播级别。



### NESTED

**上下文中存在事务，则嵌套事务执行，如果不存在事务，则新建事务**。

嵌套是子事务套在父事务中执行，**子事务是父事务的一部分，在进入子事务之前，父事务建立一个回滚点，叫save point，然后执行子事务**，这个子事务的执行也算是父事务的一部分，然后子事务执行结束，父事务继续执行。重点就在于那个save point。看几个问题就明了了：

**如果子事务回滚，会发生什么？**
父事务会回滚到进入子事务前建立的save point，然后尝试其他的事务或者其他的业务逻辑，父事务之前的操作不会受到影响，更不会自动回滚。

**如果父事务回滚，会发生什么？**
父事务回滚，子事务也会跟着回滚！为什么呢，因为父事务结束之前，子事务是不会提交的，我们说子事务是父事务的一部分，正是这个道理。那么：

**事务的提交，是什么情况？**
是父事务先提交，然后子事务提交，还是子事务先提交，父事务再提交？答案是第二种情况，还是那句话，子事务是父事务的一部分，由父事务统一提交。



**外围方法未开启事务的情况下`Propagation.NESTED`和`Propagation.REQUIRED`作用相同，修饰的内部方法都会新开启自己的事务，且开启的事务相互独立，互不干扰。**

**外围方法开启事务的情况下`Propagation.NESTED`修饰的内部方法属于外部事务的子事务，外围主事务回滚，子事务一定回滚，而内部子事务可以单独回滚而不影响外围主事务和其他子事务**。



### REQUIRES_NEW

该传播级别的特点是，**每次都会新建一个事务，并且同时将上下文中的事务挂起，执行当前新建事务完成以后，上下文事务恢复再执行**。

这是一个很有用的传播级别，举一个应用场景：现在有一个发送100个红包的操作，在发送之前，要做一些系统的初始化、验证、数据记录操作，然后发送100封红包，然后再记录发送日志，发送日志要求100%的准确，如果日志不准确，那么整个父事务逻辑需要回滚。

通过这个PROPAGATION_REQUIRES_NEW 级别的事务传播控制就可以完成，发送红包的子事务不会直接影响到父事务的提交和回滚。

**外围方法未开启事务的情况下`Propagation.REQUIRES_NEW`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。**

**外围方法开启事务的情况下`Propagation.REQUIRES_NEW`修饰的内部方法依然会单独开启独立事务，且与外部方法事务也独立，内部方法之间、内部方法和外部方法事务均相互独立，互不干扰。**



### NOT_SUPPORTED

**上下文中存在事务，则挂起事务，执行当前逻辑，结束后恢复上下文的事务**。（事务将不会发生回滚）

将事务极可能的缩小。一个事务越大，它存在的风险也就越多。所以在处理事务的过程中，要保证尽可能的缩小范围。比如一段代码，是每次逻辑操作都必须调用的，比如循环1000次的某个非核心业务逻辑操作。这样的代码如果包在事务中，势必造成事务太大，导致出现一些难以考虑周全的异常情况。所以这个事务这个级别的传播级别就派上用场了。用当前级别的事务模板抱起来就可以了。



### NEVER

**上下文中不能存在事务，一旦有事务，就抛出**`RuntimeException`，强制停止执行！



### 总结

#### **NESTED 和 REQUIRED**

**NESTED 和 REQUIRED 修饰的内部方法都属于外围方法事务，如果外围方法抛出异常，这两种方法的事务都会被回滚。但是 REQUIRED 是加入外围方法事务，所以和外围事务同属于一个事务，一旦 REQUIRED 事务抛出异常被回滚，外围方法事务也将被回滚。而 NESTED 是外围方法的子事务，有单独的保存点，所以 NESTED 方法抛出异常被回滚，不会影响到外围方法的事务。**



#### **NESTED 和 REQUIRES_NEW**

**NESTED 和 REQUIRES_NEW 都可以做到内部方法事务回滚而不影响外围方法事务。但是因为 NESTED 是嵌套事务，所以外围方法回滚之后，作为外围方法事务的子事务也会被回滚。而 REQUIRES_NEW 是通过开启新的事务实现的，内部事务和外围事务是两个事务，外围事务回滚不会影响内部事。**





## 事务失效场景

- 类内部自调用
- 修饰非public方法

- 修饰接口方法但是非基于接口的JDK代理
- 错误配置事务注解
- 数据库引擎不支持事务
- 没有被Spring管理
- 数据源没有配置事务管理器

```java
@Bean
public PlatformTransactionManager transactionManager(DataSource dataSource) {
    return new DataSourceTransactionManager(dataSource);
}
```



- 异常被catch了或非rollback指定异常及其子异常



### 类内部调用

事务实现原理实际上是先调用了代理对象中被增强的方法，然后在代理对象中，又会调用我们实际的目标对象中的方法。而类内部调用实际上是调用的原目标对象，而没有走代理对象，因此事务失效。



**解决方案**

- 将事务方法放到另一个类中进行调用。

```java
@Service
public class A {
  @Resource
  private B b;
  
  public void a(){
    // 省略其他业务代码
    b.b();
  }
}

@Service
public class B {
  @Transactional
  public void b(){
  
  }
}
```



- 获取本对象的代理对象，再进行调用。

```java
@Service
public class OrderService {
  public void insert() {
    OrderService proxy = (OrderService) AopContext.currentProxy();
     proxy.insertOrder();
  }

  @Transactional
  public void insertOrder() {
      //SQL操作
  }
}
```



- 利用非 this 调用。

```java
@Service
public class OrderService {
  public void insert() {
    SpringUtil.getBean(OrderService.class).insertOrder();
    // SpringUtil 获取 bean 对象
  }

  @Transactional
  public void insertOrder() {
      //SQL操作
  }
}
```



### 非public方法

因为 @Transactional 使用的是 Spring AOP 实现的，而 Spring AOP 是通过动态代理实现的，而 @Transactional 在生成代理时会判断，如果方法为非 public 修饰的方法，则不生成代理对象，这样也就没办法自动回滚事务了。



```java
protected TransactionAttribute computeTransactionAttribute(Method method, Class<?> targetClass) {
   // 非 public 方法，设置为 null
   if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
      return null;
   }
   // 后面代码省略....
 }
```



### 异常捕获

@Transactional 注解在其实现时，需要感知到异常才会自动回滚，在代码中加入了 try/catch 之后，@Transactional 就无法感知到异常了，那么也就不能自动回滚事务了。

此问题的解决方案有两种：一种是在 catch 中将异常重新抛出去，另一种是使用代码手动将事务回滚。

```java
@Override
@Transactional(rollbackFor = Exception.class)
public void exceptionCatch() {
    try {
        int i = 1 / 0;
    } catch (Exception e) {
        // 异常被捕获导致事务失效
    }
}
```



#### 将异常重新抛出

```java
@Override
@Transactional(rollbackFor = Exception.class)
public void throwException() {
    try {
        int i = 1 / 0;
    } catch (Exception e) {
        // 将异常抛出
        throw new ServiceException("error");
    }
}
```



#### 手动回滚事务

```java
@Override
@Transactional(rollbackFor = Exception.class)
public void handRollback() {
    try {
        int i = 1 / 0;
    } catch (Exception e) {
        // 手动设置事务回滚
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
    }
}
```



### 检查异常

检查异常不回滚事务的原因是因为，@Transactional 默认只回滚运行时异常 RuntimeException 和 Error，而对于检查异常默认是不回滚的。

此问题的解决方案是给 @Transactional 注解上，添加 rollbackFor 参数并设置 Exception.class 值。

```java
@Override
@Transactional(rollbackFor = Exception.class)
public void handRollback() {
    // ...
}
```



### 数据库不支持

如果数据库本身不支持事务，比如 MySQL 中设置了使用 MyISAM 引擎，因为它本身是不支持事务的，这种情况下，即使在程序中添加了 @Transactional 注解，那么依然不会有事务的行为，也就不会执行事务的自动回滚了。



## 实现原理

**`@Transactional` 的工作机制是基于 AOP 实现的，AOP 又是使用动态代理实现的。如果目标对象实现了接口，默认情况下会采用 JDK 的动态代理，如果目标对象没有实现了接口,会使用 CGLIB 动态代理。**

如果一个类或者一个类中的 public 方法上被标注`@Transactional` 注解的话，Spring 容器就会在启动的时候为其创建一个代理类，在调用被`@Transactional` 注解的 public 方法的时候，实际调用的是，`TransactionInterceptor` 类中的 `invoke()`方法。这个方法的作用就是在目标方法之前开启事务，方法执行过程中如果遇到异常的时候回滚事务，方法调用完成之后提交事务。



Spring 框架中，事务管理相关最重要的 3 个接口如下：

- **`PlatformTransactionManager`**： （平台）事务管理器，Spring 事务策略的核心。
- **`TransactionDefinition`**： 事务定义信息(事务隔离级别、传播行为、超时、只读、回滚规则)。
- **`TransactionStatus`**： 事务运行状态。

 **`PlatformTransactionManager`** 接口可以被看作是事务上层的管理者，而 **`TransactionDefinition`** 和 **`TransactionStatus`** 这两个接口可以看作是事务的描述。

**`PlatformTransactionManager`** 会根据 **`TransactionDefinition`** 的定义比如事务超时时间、隔离级别、传播行为等来进行事务管理 ，而 **`TransactionStatus`** 接口则提供了一些方法来获取事务相应的状态比如是否新事务、是否可以回滚等等。




在应用系统调用声明@Transactional 的目标方法时，Spring Framework 默认使用 AOP 代理，在代码运行时生成一个代理对象，根据@Transactional 的属性配置信息，这个代理对象决定该声明@Transactional 的目标方法是否由拦截器 TransactionInterceptor 来使用拦截，在 TransactionInterceptor 拦截时，会在在目标方法开始执行之前创建并加入事务，并执行目标方法的逻辑, 最后根据执行情况是否出现异常，利用抽象事务管理器AbstractPlatformTransactionManager 操作数据源 DataSource 提交或回滚事务。



![Spring事务的配置、参数详情及其原理介绍(Transactional)_weixin_38166560的博客-CSDN博客](../../Image/2022/07/220723-1.png)



Spring AOP 代理有 CglibAopProxy 和 JdkDynamicAopProxy 两种，图 1 是以 CglibAopProxy 为例，对于 CglibAopProxy，需要调用其内部类的 DynamicAdvisedInterceptor 的 intercept 方法。对于JdkDynamicAopProxy，需要调用其 invoke 方法。

正如上文提到的，事务管理的框架是由抽象事务管理器 AbstractPlatformTransactionManager 来提供的，而具体的底层事务处理实现，由 PlatformTransactionManager 的具体实现类来实现，如事务管理器 DataSourceTransactionManager。不同的事务管理器管理不同的数据资源 DataSource，比如 DataSourceTransactionManager 管理 JDBC 的 Connection。



### 事务管理器

**Spring 并不直接管理事务，而是提供了多种事务管理器** 。Spring 事务管理器的接口是： **`PlatformTransactionManager`** 。

定义 PlatformTransactionManager 接口是因为要将事务管理行为抽象出来，然后不同的平台去实现它，这样可以保证提供给外部的行为不变，方便扩展。



通过这个接口，Spring 为各个平台如 JDBC(`DataSourceTransactionManager`)、Hibernate(`HibernateTransactionManager`)、JPA(`JpaTransactionManager`)等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情了。



PlatformTransactionManager，AbstractPlatformTransactionManager 及具体实现类关系如图所示。



![透彻的掌握 Spring 中 @Transactional的使用_事务管理_02](../../Image/2022/07/220723-2.png)



### 事务属性

事务管理器接口 **`PlatformTransactionManager`** 通过 **`getTransaction(TransactionDefinition definition)`** 方法来得到一个事务，这个方法里面的参数是 **`TransactionDefinition`** 类 ，这个类就定义了一些基本的事务属性。

**什么是事务属性呢？** 事务属性可以理解成事务的一些基本配置，描述了事务策略如何应用到方法上。

事务属性包含了 5 个方面：

- 隔离级别
- 传播行为
- 回滚规则
- 是否只读
- 事务超时

`TransactionDefinition` 接口中定义了 5 个方法以及一些表示事务属性的常量比如隔离级别、传播行为等等。



### 事务状态

`TransactionStatus`接口用来记录事务的状态 该接口定义了一组方法,用来获取或判断事务的相应状态信息。

`PlatformTransactionManager.getTransaction(…)`方法返回一个 `TransactionStatus` 对象。



### 源码解析

> org.springframework.transaction.PlatformTransactionManager

```java
public interface PlatformTransactionManager {
   /**
    * 
    */
   TransactionStatus getTransaction(@Nullable TransactionDefinition definition)
         throws TransactionException;

   /**
    * 
    */
   void commit(TransactionStatus status) throws TransactionException;

   /**
    * 
    */
   void rollback(TransactionStatus status) throws TransactionException;
}
```



> org.springframework.transaction.TransactionDefinition

```java
public interface TransactionDefinition {

   /**
    * 
    */
   int PROPAGATION_REQUIRED = 0;

   /**
    * 
    */
   int PROPAGATION_SUPPORTS = 1;

   /**
    * 
    */
   int PROPAGATION_MANDATORY = 2;

   /**
    * 
    */
   int PROPAGATION_REQUIRES_NEW = 3;

   /**
    * 
    */
   int PROPAGATION_NOT_SUPPORTED = 4;

   /**
    * 
    */
   int PROPAGATION_NEVER = 5;

   /**
    * 
    */
   int PROPAGATION_NESTED = 6;


   /**
    * 
    */
   int ISOLATION_DEFAULT = -1;

   /**
    * 
    */
   int ISOLATION_READ_UNCOMMITTED = Connection.TRANSACTION_READ_UNCOMMITTED;

   /**
    * 
    */
   int ISOLATION_READ_COMMITTED = Connection.TRANSACTION_READ_COMMITTED;

   /**
    * 
    */
   int ISOLATION_REPEATABLE_READ = Connection.TRANSACTION_REPEATABLE_READ;

   /**
    * 
    */
   int ISOLATION_SERIALIZABLE = Connection.TRANSACTION_SERIALIZABLE;


   /**
    * 
    */
   int TIMEOUT_DEFAULT = -1;


   /**
    * 返回事务的传播行为，默认值为 REQUIRED。
    */
   int getPropagationBehavior();

   /**
    * 返回事务的隔离级别，默认值是 DEFAULT
    */
   int getIsolationLevel();

   /**
    * 返回事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。
    */
   int getTimeout();

   /**
    * 返回是否为只读事务，默认值为 false
    */
   boolean isReadOnly();

   /**
    * 
    */
   @Nullable
   String getName();

}
```



> org.springframework.transaction.TransactionStatus

```java
public interface TransactionStatus extends SavepointManager, Flushable {
   /**
    * 是否是新的事务
    */
   boolean isNewTransaction();

   /**
    * 是否有恢复点
    */
   boolean hasSavepoint();

   /**
    * 设置为只回滚
    */
   void setRollbackOnly();

   /**
    * 是否为只回滚
    */
   boolean isRollbackOnly();

   /**
    * 
    */
   @Override
   void flush();

   /**
    * 是否已完成
    */
   boolean isCompleted();
```



> org.springframework.transaction.interceptor.AbstractFallbackTransactionAttributeSource

```java
	@Nullable
	protected TransactionAttribute computeTransactionAttribute(Method method, @Nullable Class<?> targetClass) {
		// 方法不是public修饰就不会获取@Transactional的属性配置信息
		if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
			return null;
		}
		// ...
	}
```



## 实际使用

1. 通过@EnableTransactionManagement 注解可以启用事务管理功能。
2. 将@Transactional 注解添加到合适的方法上，并设置合适的属性信息。



## 扩展

### 避免 Spring 的 AOP 的自调用问题

在 Spring 的 AOP 代理下，只有目标方法由外部调用，目标方法才由 Spring 生成的代理对象来管理，这会造成自调用问题。若同一类中的方法调用自己类中有`@Transactional`注解的方法，被调用的方法的`@Transactional`被忽略，不会发生回滚。



### @Transactional不能使用在private方法

当客户代码所持有的引用是一个Test类的代理的时候，当调用a()方法时候,首先会调用原始a()方法上的@Before的代码逻辑，然后调用原始的a()方法，原始a()方法内的b()调用的是原始对象的this.b()，即**一旦调用最终抵达了目标对象 (此处为Test类的引用)，任何对自身的调用例如this.b()将对this引用进行调用而非代理。**



为解决这两个问题，可以使用`AspectJ`取代`Spring AOP` ，通过字节码在编译时生成代理类。



## 总结

虽然 `@Transactional` 注解可以作用于接口、接口方法、类以及类方法上，但是 Spring 建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。

另外， `@Transactional` 注解应该只被应用到 `public` 方法上，这是由 `Spring AOP` 的本质决定的。如果你在 `protected`、`private` 或者默认可见性的方法上使用 `@Transactional` 注解，这将被忽略，也不会抛出任何异常。

默认情况下，只有来自外部的方法调用才会被AOP代理捕获，也就是说类内部方法调用本类内部的其他方法并不会引起事务行为，即使被调用方法使用`@Transactional`注解进行修饰。



# Spring JavaConfig

Spring JavaConfig 是 Spring 社区的产品，它提供了配置 Spring IoC 容器的纯Java 方法。因此它有助于避免使用 XML 配置。使用 JavaConfig 的优点在于：

（1）面向对象的配置。由于配置被定义为 JavaConfig 中的类，因此用户可以充分利用 Java 中的面向对象功能。一个配置类可以继承另一个，重写它的@Bean 方法等。

（2）减少或消除 XML 配置。基于依赖注入原则的外化配置的好处已被证明。但是，许多开发人员不希望在 XML 和 Java 之间来回切换。JavaConfig 为开发人员提供了一种纯 Java 方法来配置与 XML 配置概念相似的 Spring 容器。从技术角度来讲，只使用 JavaConfig 配置类来配置容器是可行的，但实际上很多人认为将JavaConfig 与 XML 混合匹配是理想的。

（3）类型安全和重构友好。JavaConfig 提供了一种类型安全的方法来配置 Spring容器。由于 Java 5.0 对泛型的支持，现在可以按类型而不是按名称检索 bean，不需要任何强制转换或基于字符串的查找。
