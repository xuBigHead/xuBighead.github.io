# 设计模式概览

![1](../../Image/2021/03/210324.png)

**创建型设计模式**

用于描述“怎样创建对象”，主要特点是“将对象的创建与使用分离”。GoF中提供了单例、原型、工厂方法、抽象工厂、建造者5种创建型模式。



| 模式名称     | **业务场景**                                         | **实现要点**                                                 |
| ------------ | ---------------------------------------------------- | :----------------------------------------------------------- |
| 工厂方法模式 | 多种类型商品不同接口，统一发奖服务搭建场景           | 定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。 |
| 抽象工厂模式 | 替换Redis双集群升级，代理类抽象场景                  | 提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。 |
| 单例模式     | 7种单例模式案例，Effective Java 作者推荐枚举单例模式 | 一个类只有一个实例，并提供一个访问它的全局访问点，以便外部获取该实例。 |
| 原型模式     | 上机考试多套试，每人题目和答案乱序排列场景           | 通过对原型对象复制而获取多个和原型对象类似的实例。           |
| 建造者模式   | 各项装修物料组合套餐选配场景                         | 将一个复杂对象分解成多个相对简单的部分，然后根据不同的需要分别创建它们，最后构建成该复杂对象。 |



**结构型设计模式**

用于描述如何将类或对象按某种布局组成更大的结构，GoF中提供了代理、适配器、桥接、装饰、外观、享元、组合7种结构型模式。



| 模式名称   | **业务场景**                                                 | **实现要点**                                                 |
| :--------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 适配器模式 | 从多个MQ消息体中，抽取指定字段值场景                         | 将一个类的接口转换成另外一个接口，使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。 |
| 桥接模式   | 多支付渠道(微信、支付宝)与多支付模式(刷脸、指纹)场景         | 将抽象与实现分离，使它们可以独立变化，用组合关系代替继承关系，降低了抽象和实现的耦合度。 |
| 组合模式   | 营销差异化人群发券，决策树引擎搭建场景                       | 将对象组合成树形结构以表示”部分-整体”的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。 |
| 装饰模式   | SSO单点登录功能扩展，增加拦截用户访问方法范围场景            | 动态地给对象添加额外功能。                                   |
| 外观模式   | 基于SpringBoot开发门面模式中间件，统一控制接口白名单场景     | 为多个复杂的子系统提供一个一致的接口，使这些子系统更加容易被访问。 |
| 享元模式   | 基于Redis秒杀，提供活动与库存信息查询场景                    | 运用共享技术有效地支持大量细粒度的对象的复用。               |
| 代理模式   | 模拟mybatis-spring中定义DAO接口，使用代理类方式操作数据库原理实现场景 | 提供一种代理来控制对这个对象的访问，从而限制、增强或修改该对象的一些特性。 |



**行为型设计模式**

用于描述类或对象之间怎样相互协作共同完成单个对象无法单独完成的任务，以及怎样分配职责。GoF中提供了模板方法、策略、命令、职责链、状态、观察者、中介者、迭代器、访问者、备忘录、解释器11种行为型模式。



| 模式名称     | **业务场景**                                                 | **实现要点**                                                 |
| :----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 责任链模式   | 模拟618电商大促期间，项目上线流程多级负责人审批场景          | 把请求从链中的一个对象传到下一个对象，直到请求被响应为止。通过这种方式可以去除对象之间的耦合。 |
| 命令模式     | 模拟高档餐厅八大菜系，小二点单厨师烹饪场景                   | 将一个请求封装成一个对象，使发出请求的责任和执行请求的责任分割开。 |
| 迭代器模式   | 模拟公司组织架构树结构关系，深度迭代遍历人员信息输出场景     | 提供一种方法顺序访问一个聚合对象中各个元素,  而又无须暴露该对象的内部表示。 |
| 中介者模式   | 按照Mybatis原理手写ORM框架，给JDBC方式操作数据库增加中介者场景 | 定义一个中介对象来简化原有对象之间的交互关系，降低系统中对象间的耦合度，使原有对象之间不必相互了解。 |
| 备忘录模式   | 模拟互联网系统上线过程中，配置文件回滚场景                   | 在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。 |
| 观察者模式   | 模拟类似小客车指标摇号过程，监听消息通知用户中签场景         | 定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。 |
| 状态模式     | 模拟系统营销活动，状态流程审核发布上线场景                   | 允许一个对象在其内部状态发生改变时改变其行为能力。           |
| 策略模式     | 模拟多种营销类型优惠券，折扣金额计算策略场景                 | 定义并封装一系列的算法， 使它们可相互替换。算法的改变不会影响使用算法的客户。 |
| 模板方法模式 | 模拟爬虫各类电商商品，生成营销推广海报场景                   | 定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。 |
| 访问者模式   | 模拟家长与校长，对学生和老师的不同视角信息的访问场景         | 在不改变集合元素的前提下，为一个集合中的每个元素提供多种访问方式，即每个元素有多个访问者对象访问。 |
| 解释器       |                                                              | 提供如何定义语言的文法，以及对语言句子的解释方法，即解释器。 |

# 设计模式01-工厂方法模式（Factory）

## 概述

## 简单实现

## 基于JDK的实现

## 基于IOC容器的实现

## 源码解析

## 优点

## 使用场景

## 参考资料

# 设计模式02-抽象工厂模式（Abstract Factory）
## 概述

## 简单实现

## 基于JDK的实现

## 基于IOC容器的实现

## 源码解析

## 优点

## 使用场景

## 参考资料

# 设计模式03-单例模式（Singleton）
## 概述

单例模式（Singleton）要私有化构造器，在类中创建私有实例并提供公有方法给外部类获取该实例对象，确保只会存在唯一的一个实例。



特点：

- 私有化构造器；
- 对外提供获取实例方法。



单例模式分为：

- **懒汉式**（线程不安全，调用效率高，但是不能延时加载）；
- **饿汉式**（线程安全，调用效率不高，可以延时加载）；
- **双重检测锁式**（由于JVM底层内部模型原因，偶尔会出问题，不建议使用）；
- **静态内部类式**（线程安全，调用效率高，可以延时加载）；
- **枚举单例**（线程安全，调用效率高，不能延时加载）；



## 简单实现

### 基于JDK的实现

#### 懒汉式

线程不安全的懒汉式：

```java
public class Singleton {
    private static Singleton instance = null;
    private Singleton() {}
    public static Singleton getInstance() {
        if(instance == null) { 
            instance = new Singleton();
        }
        return instance;
    }
}
```

这种方式是最基本的实现方式，这种实现最大的问题就是不支持多线程。因为没有加锁 synchronized，所以严格意义上它并不算单例模式。



线程安全的懒汉式：

```java
public class Singleton {
    private static Singleton instance = null;
    private Singleton() {}
    public static synchronized Singleton getInstance() {
        if(instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```



这种方式具备很好的 lazy loading，能够在多线程中很好的工作，但是，效率很低，99% 情况下不需要同步。

优点：第一次调用才初始化，避免内存浪费。

缺点：必须加锁 synchronized 才能保证单例，但加锁会影响效率。



#### 饿汉式

```java
public class Singleton {
    private static final Singleton instance = new Singleton();
    private Singleton() { }
    public static Singleton getInstance() { return instance; }
}
```



优点：没有加锁，执行效率会提高。

缺点：类加载时就初始化，浪费内存，容易产生垃圾对象。



饿汉式基于 classloader 机制避免了多线程的同步问题，在类装载时就实例化。

虽然导致类装载的原因有很多种，在单例模式中大多数都是调用 getInstance 方法， 但是也不能确定有其他的方式（或者其他的静态方法）导致类装载，这时候初始化 instance 显然没有达到 lazy loading 的效果。



#### 双检锁式

```java
public class Singleton {
    private volatile static Singleton instance = null;
    private Singleton() {}
    /**
     * 静态工程方法，创建实例，此时毫无线程安全保护，因此要添加synchronized
     * @return      单例对象
     */
    public static Singleton getInstance() {
        if(instance == null) {
            synchronized(Singleton.class) {
                if(instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```



##### synchronized的作用

主要用来解决的是多线程同步问题，其可以保证在被其修饰的代码任意时刻只有一个线程执行。



##### 两次校空

两次较空的目的在于防止同时有两个线程执行到synchronized，第一个线程执行完创建实例后释放锁，第二个线程获取到锁后没有再次校验为NULL而继续执行创建实例，导致重复创建实例。



##### **volatile的作用**

volatile的主要作用：

1. 保持内存可见性；使所有线程都能看到共享内存的最新状态。
2. 防止指令重排的问题；



这种方式采用双检锁机制（DCL，即 double-checked locking），安全且在多线程情况下能保持高性能。

uniqueInstance = new Singleton(); 这段代码其实是分为三步执行：

1. 为实例分配内存空间
2. 初始化实例
3. 将实例指向分配的内存地址



由于 JVM 指令重排，执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 getUniqueInstance 后发现实例不为空，因此返回实例，但此时实例还未被初始化。

使用 `volatile` 可以禁止 JVM 的指令重排，保证在多线程环境下也能正常运行。



#### 静态内部类式

单例模式使用内部类来实现，JVM 内部的机制能够保证当一个类被加载的时候，这个类的加载过程是线程互斥的。当第一次调用getInstance的时候，JVM 能够保证instance只被创建一次，并且会把赋值给instance的内存初始化完毕。 同时该方法也只会在第一次调用的时候使用互斥机制，这样就解决了低性能问题。

```java
// 通过静态内部类的方式来实现单例模式，线程安全，调用效率高，可以延时加载。
public class Singleton {
    private Singleton () {
        // 避免通过反射方式实例化对象
        if(SingletonFactory.INSTANCE != null) {
            throw new RuntimeException("不支持重复实例化");
        }
    }

    // 使用一个内部类来维护单例
    private static class SingletonFactory{
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return SingletonFactory.INSTANCE;
    }
    
    // 防止反序列化实例化重复对象
    private Object readResolve() throws ObjectStreamException {
    	return Inner.INSTANCE;
	}
}
```



这种方式能达到双检锁方式一样的功效，但实现更简单。对静态域使用延迟初始化，应使用这种方式而不是双检锁方式。这种方式只适用于静态域的情况，双检锁方式可在实例域需要延迟初始化时使用。

这种方式同样利用了 classloader 机制来保证初始化 instance 时只有一个线程，跟饿汉式不同的是饿汉式只要 Singleton 类被装载了，那么 instance 就会被实例化，没有懒加载的效果，而这种方式是 Singleton 类被装载了，instance 不一定被初始化。因为 SingletonHolder 类没有被主动使用，只有通过显式调用 getInstance 方法时，才会显式装载 SingletonHolder 类，从而实例化 instance。

如果实例化 instance 很消耗资源，想让它延迟加载，另外一方面，又不希望在 Singleton 类加载时就实例化，因为不能确保 Singleton 类还可能在其他的地方被主动使用从而被加载，那么这个时候实例化 instance 显然是不合适的。这个时候，这种方式相比饿汉式就显得很合理。

 

**防止反射创建对象**

为了防止通过Class类型对象调用newInstance方法来创建实例，可以在私有空构造器中添加内部类静态属性是否为空校验。



**防止反序列化创建对象**

为了防止放序列化创建对象，可以添加readResolve方法，程序在反序列化获取对象时，会去寻找readResolve()方法。

- 如果该方法不存在，则直接返回新对象。
- 如果该方法存在，则按该方法的内容返回对象。
- 如果之前没有实例化单例对象，则会返回null。



#### 创建获取分离方式

 因为只需要在创建类的时候进行同步，所以只要将创建和getInstance()分开，单独为创建加synchronized关键字也可以。整个程序只需创建一次实例，所以性能也不会有什么影响。



```java
public class Singleton {
    private static Singleton instance = null;
    private Singleton() {
    }
    public static synchronized void syncInit() {
        if(instance == null) {
            instance = new Singleton();
        }
    }
    public static Singleton getInstance() {
        if(instance == null) {
            syncInit();
        }
        return instance;
    }
}
```



类的静态方法也能实现单例模式，两者的不同处：

- 静态类不能实现接口，因为接口中不允许有static修饰的方法

- 单例可以被延迟（比较庞大的类延迟加载有助于提升性能）初始化，静态类一般在第一次加载时初始化。

- 单例类比较灵活，可以实现一些其它功能，静态类不行。



#### 枚举式

```java
public enum Singleton {
    INSTANCE;
}
```

这种方式是实现单例模式的最佳方法。它更简洁，不仅能避免多线程同步问题，而且还自动支持序列化机制，防止反序列化重新创建新的对象。



### 基于IOC容器的实现

## 源码解析

### Runtime

> java.lang.Runtime

```java
public class Runtime {
    private static Runtime currentRuntime = new Runtime();
    public static Runtime getRuntime() {
        return currentRuntime;
    }
    private Runtime() {}
}
```



### DefaultNamespaceHandlerResolver

>org.springframework.beans.factory.xml.DefaultNamespaceHandlerResolver

```java
public class DefaultNamespaceHandlerResolver implements NamespaceHandlerResolver {
    @Nullable
	private volatile Map<String, Object> handlerMappings;
    	private Map<String, Object> getHandlerMappings() {
		Map<String, Object> handlerMappings = this.handlerMappings;
		if (handlerMappings == null) {
			synchronized (this) {
				handlerMappings = this.handlerMappings;
				if (handlerMappings == null) {
					// ...
				}
			}
		}
		return handlerMappings;
	}
}
```



### LogFactory

> org.apache.ibatis.logging.LogFactory

```java
public final class LogFactory {
    private static Constructor<? extends Log> logConstructor;
    static {
        tryImplementation(LogFactory::useSlf4jLogging);
        // ...
    }

    private LogFactory() {}

    public static Log getLog(Class<?> clazz) {
        return getLog(clazz.getName());
    }

    public static Log getLog(String logger) {
        try {
            return logConstructor.newInstance(logger);
        } catch (Throwable t) {
           	// ...
        }
    }

    public static synchronized void useSlf4jLogging() {
        setImplementation(org.apache.ibatis.logging.slf4j.Slf4jImpl.class);
    }
	// ...
    private static void tryImplementation(Runnable runnable) {
        if (logConstructor == null) {
            try {
                runnable.run();
            } catch (Throwable t) {
                // ignore
            }
        }
    }

    private static void setImplementation(Class<? extends Log> implClass) {
        try {
            Constructor<? extends Log> candidate = implClass.getConstructor(String.class);
            logConstructor = candidate;
        } catch (Throwable t) {
		// ...
        }
    }
}
```



## 优点

单例模式优点在于在内存里只有一个实例，减少了内存的开销，尤其是在频繁的创建和销毁实例场景下，避免对资源的多重占用；

缺点在于没有接口，不能继承，与单一职责原则冲突，一个类应该只关心内部逻辑，而不关心外面怎么样来实例化。



## 使用场景

## 参考资料


# 设计模式04-原型模式（Prototype）
## 概述

## 简单实现

## 基于JDK的实现

## 基于IOC容器的实现

## 源码解析

## 优点

## 使用场景

## 参考资料


# 设计模式05-建造者模式（Builder）
## 概述

## 简单实现

## 基于JDK的实现

## 基于IOC容器的实现

## 源码解析

## 优点

## 使用场景

## 参考资料
# 设计模式06-适配器模式（Adapter）
## 概述 
对适配器模式的功能很好理解，就是把一个类的接口变换成客户端所能接受的另一种接口，从而使两个接口不匹配而无法在一起工作的两个类能够在一起工作。

适配器的重点在于将一个接口的功能转换为另一个接口的功能。

### 优点
- 更好的复用性

系统需要使用现有的类，而此类的接口不符合系统的需要。那么通过适配器模式就可以让这些功能得到更好的复用。

- 更好的扩展性

在实现适配器功能的时候，可以调用自己开发的功能，从而自然地扩展系统的功能。

### 缺点
过多的使用适配器，会让系统非常零乱，不易整体进行把握。比如，明明看到调用的是A接口，其实内部被适配成了B接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构。

## 简单实现
### JDK实现
1、已有 `AdapterSource` 接口及其实现类 `AdapterSourceImpl`
> com.demo.design.pattern.structural.adapter.AdapterSource
```java
public interface AdapterSource {
    /**
     * adapter method  
     */
    void execute();
}
```

> com.demo.design.pattern.structural.adapter.AdapterSourceImpl
```java
@Slf4j
public class AdapterSourceImpl implements AdapterSource {
    @Override
    public void execute() {
        log.info("default adapter function");
    }
}
```

2、现在要在使用 `AdapterTarget` 接口处调用 `AdapterSource` 接口方法而不修改 `AdapterSource` 相关类。
> com.demo.design.pattern.structural.adapter.AdapterTarget
```java
public interface AdapterTarget {
    /**
     * call adapter method in this method
     */
    void executeTarget();
}
```

3、可以通过两种适配方式来在 `AdapterTarget` 接口的实现类中调用 `AdapterSource` 实现类的功能。
第一种是通过持有 `AdapterSource` 实现类对象的方式来调用 `AdapterSource` 方法，相比于下述的继承方式，更推荐通过组装的方式完成 `AdapterSource` 接口的适配。
> com.demo.design.pattern.structural.adapter.HoldObjectAdapter
```java
public class HoldObjectAdapter implements AdapterTarget {
    private final AdapterSource source;

    HoldObjectAdapter(AdapterSource source) {
        super();
        this.source = source;
    }

    @Override
    public void executeTarget() {
        source.execute();
    }
}
```

第二种是通过继承 `AdapterSource` 实现类的方式来调用 `AdapterSource` 方法。
> com.demo.design.pattern.structural.adapter.ExtendClassAdapter
```java
public class ExtendClassAdapter extends AdapterSourceImpl implements AdapterTarget {
    @Override
    public void executeTarget() {
        execute();
    }
}
```

示例
```java
public class StructuralPatternTest {
    @Test
    public void adapterPattern(){
        AdapterSource adapterSource = new AdapterSourceImpl();
        AdapterTarget holdObjAdapter = new HoldObjectAdapter(adapterSource);
        holdObjAdapter.executeTarget();
        
        AdapterTarget extendAdapter = new ExtendClassAdapter();
        extendAdapter.executeTarget();           
    }
}
```


## 源码解析
### JDK
> java.io.InputStreamReader
```java
public class InputStreamReader extends Reader {
    private final StreamDecoder sd;
    public InputStreamReader(InputStream in) {
        // ...
    }
}
```

`InputStreamReader` 通过持有不同接口或父类 `InputStream` 类型的实现来将 `InputStream` 适配适配到 `Reader`

### Mybatis Log
> org.apache.ibatis.logging.Log
```java
public interface Log {
    // ...
}
```

`Mybatis` 为了适配多种日志实现类，就使用了适配器模式，将多种日志的实现适配为同一个接口 `Log` 的实现。

> org.apache.ibatis.logging.log4j2.Log4j2LoggerImpl
```java
// ...
import org.apache.logging.log4j.Logger;

public class Log4j2LoggerImpl implements Log {
    private static final Marker MARKER = MarkerManager.getMarker(LogFactory.MARKER);
    // 这里就适配了 apache 的 log4j2 日志实现类
    private final Logger log;
    public Log4j2LoggerImpl(Logger logger) {
        log = logger;
    }
}
```

> org.apache.ibatis.logging.slf4j.Slf4jLoggerImpl
```java
class Slf4jLoggerImpl implements Log {
    private final Logger log;
    // 这里就适配了 slf4j 的日志实现类
    public Slf4jLoggerImpl(Logger logger) {
        log = logger;
    }
```

## 参考资料


# 设计模式07-装饰模式（Decorator）
## 概述
通过实现与被装饰类实现的相同接口或父类，并将被装饰类作为属性注入到装饰器对象中来完成对装饰器模式的应用。

装饰器模式重点在于调用方对整个过程无感知，仍然调用原先实现的接口或父类方法即可。

### 和适配器模式的比较
装饰器与适配器都有一个别名叫做 包装模式(Wrapper)，它们看似都是起到包装一个类或对象的作用，但是使用它们的目的很不一一样。

适配器模式的意义是要将一个接口转变成另一个接口，它的目的是通过改变接口来达到重复使用的目的。

而装饰器模式不是要改变被装饰对象的接口，而是恰恰要保持原有的接口，但是增强原有对象的功能，或者改变原有对象的处理方式而提升性能。

所以这两个模式设计的目的是不同的。

## 简单实践
### JDK实现
1、现在有接口 `DecoratorSource` 及其实现类 `DecoratorSourceImpl`，如果想在不修改已有类且继续通过 `Source` 类型的声明来调用的情况下对execute功能做修改，即可使用装饰器模式。

> com.demo.design.pattern.structural.decorator.DecoratorSource
```java
public interface DecoratorSource {
    /**
     * enhanced target method
     */
    void execute();
}
```

> com.demo.design.pattern.structural.decorator.DecoratorSourceImpl
```java
@Data
@Slf4j
public class DecoratorSourceImpl implements DecoratorSource {
    @Override
    public void execute() {
        log.info("default decorator source");
    }
}
```

2、创建 `SourceDecorator` 的装饰器类，该类要实现 `DecoratorSource` 接口并将要被装饰的 `DecoratorSource` 类型的实现类作为属性。
> com.demo.design.pattern.structural.decorator.SourceDecorator
```java
@Slf4j
public class SourceDecorator implements DecoratorSource {
    private final DecoratorSource source;

    public SourceDecorator(DecoratorSource source) {
        super();
        this.source = source;
    }

    @Override
    public void execute() {
        log.info("before decorator!");
        source.execute();
        log.info("after decorator!");
    }
}
```

3、调用装饰器类 `SourceDecorator` 方式示例如下：
> com.demo.design.pattern.StructuralPatternTest
```java
public class StructuralPatternTest {
    @Test
    public void decoratorPattern(){
        DecoratorSourceImpl decoratorSource = new DecoratorSourceImpl();
        SourceDecorator sourceDecorator = new SourceDecorator(decoratorSource);
        sourceDecorator.execute();
    }
}
```

## 源码解析
### JDK InputStream流
> java.io.FilterInputStream
```java
public class FilterInputStream extends InputStream {
    protected volatile InputStream in;

    protected FilterInputStream(InputStream in) {
        this.in = in;
    }
```

`FilterInputStream` 通过持有实现相同父类的对象，来对该对象功能作增强。

## 参考资料


# 设计模式08-桥接模式（Bridge）

## 概述

## 简单实现

## 基于JDK的实现

## 基于IOC容器的实现

## 源码解析

## 优点

## 使用场景

## 参考资料


# 设计模式09-组合模式（Composite）
## 概述

## 简单实现

## 基于JDK的实现

## 基于IOC容器的实现

## 源码解析

## 优点

## 使用场景

## 参考资料


# 设计模式10-外观模式（Facade）
## 概述

## 简单实现

## 基于JDK的实现

## 基于IOC容器的实现

## 源码解析

## 优点

## 使用场景

## 参考资料


# 设计模式11-享元模式（Flyweight）
## 概述
> Use sharing to support large numbers of fine-grained objects efficiently.

享元模式（Flyweight Pattern）：使用共享对象可有效地支持大量的细粒度的对象，从而节省内存。

## 简单实现
### JDK实现
> com.demo.design.pattern.structural.flyweight.Circle
```java
@Data
public class Circle {
    private int radius;
    private Font font;

    public void draw() {
        System.out.println("Circle: [Font : " + font + ", radius : " + radius + "]");
    }
}
```

调用示例

```java
@Test
public void flyWeightPattern(){
    // 事先定义好需要的Circle对象的共享属性
    Map<Integer, Font> fontMap = new HashMap<>(10);
    fontMap.put(1, new Font("Arial Bold", Font.PLAIN, 1));
    fontMap.put(2, new Font("Arial Bold", Font.BOLD, 2));
    
    List<Circle> circles = new ArrayList<>(100000);
    Circle circle;
    for (int i = 0; i < 100000; ++i) {
        circle = new Circle();
        // 需要大量Circle对象时，共享属性属性可以引用同一个对象
        circle.setFont(fontMap.get(i & 1));
        
        // 非共享属性则每个Circle独有
        circle.setRadius(new Random().nextInt(100));
        circles.add(circle);
    }
    circles.forEach(Circle::draw);
}
```
## 源码解析
### JDK
> java.lang.Integer
```java
public final class Integer extends Number implements Comparable<Integer> {
    /**
     * Returns an {@code Integer} instance representing the specified
     * {@code int} value.  If a new {@code Integer} instance is not
     * required, this method should generally be used in preference to
     * the constructor {@link #Integer(int)}, as this method is likely
     * to yield significantly better space and time performance by
     * caching frequently requested values.
     *
     * This method will always cache values in the range -128 to 127,
     * inclusive, and may cache other values outside of this range.
     *
     * @param  i an {@code int} value.
     * @return an {@code Integer} instance representing {@code i}.
     * @since  1.5
     */
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
}
```

Integer缓存对象缓存了 -128 到 127 之间的整型值，这是最常用的一部分整型值，JDK也提供了方法来自定义缓存的最大值。
           
> java.lang.Integer.IntegerCache
```java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```
## 优点
减少应用程序创建的对象， 降低程序内存的占用， 增强程序的性能。

## 缺点
提高了系统复杂性， 需要分离出外部状态和内部状态， 而且外部状态具有固化特性， 不应该随内部状态改变而改变， 否则导致系统的逻辑混乱。

## 使用场景
- 系统中存在大量的相似对象。

- 细粒度的对象都具备较接近的外部状态， 而且内部状态与环境无关， 也就是说对象没有特定身份。

- 需要缓冲池的场景。

## 参考资料
- [Java设计模式之（十一）——享元模式](https://www.cnblogs.com/ysocean/p/15621446.html)


# 设计模式12-代理模式（Proxy）
## 概述

## 简单实现

## 基于JDK的实现

## 基于IOC容器的实现

## 源码解析

## 优点

## 使用场景

## 参考资料


# 设计模式13-责任链模式（Chain of Responsibility）
## 概述

## 简单实现

## 基于JDK的实现

## 基于IOC容器的实现

## 源码解析

## 优点

## 使用场景

## 参考资料


# 设计模式14-命令模式
## 概述

命令模式（Command Pattern）是一种数据驱动的设计模式，它属于行为型模式。请求以命令的形式包裹在对象中，并传给调用对象。调用对象寻找可以处理该命令的合适的对象，并把该命令传给相应的对象，该对象执行命令。

命令模式很好理解，举个例子，司令员下令让士兵去干件事情，从整个事情的角度来考虑，司令员的作用是，发出口令，口令经过传递，传到了士兵耳朵里，士兵去执行。这个过程好在，三者相互解耦，任何一方都不用去依赖其他人，只需要做好自己的事儿就行，司令员要的是结果，不会去关注到底士兵是怎么实现的。



命令模式会将一个请求封装为一个接口对象，由各种功能去实现其方法，在详细命令类中，通过聚合的方式将具体方法进行抽取调用。

在软件系统中，行为请求者与行为实现者通常是一种紧耦合的关系，但某些场合，比如需要对行为进行记录、撤销或重做、事务等处理时，这种无法抵御变化的紧耦合的设计就不太合适。



**命令模式的目的就是达到命令的发出者和执行者之间解耦，实现请求和执行分开。**



### 优点

1、降低了系统耦合度。 

2、新的命令可以很容易添加到系统中去。



### 缺点

使用命令模式可能会导致某些系统有过多的具体命令类。



### 使用场景

认为是命令的地方都可以使用命令模式，比如： 1、GUI 中每一个按钮都是一条命令。 2、模拟 CMD。

系统需要支持命令的撤销(Undo)操作和恢复(Redo)操作，也可以考虑使用命令模式，见命令模式的扩展。



## 简单实现

### 基于JDK

```java
public interface Command {
    void print();
    void setReceiver(Receiver receiver);
}

public class PrintCommand implements Command {
    private Receiver receiver;
    PrintCommand(Receiver receiver) {
        this.receiver = receiver;
    }

    @Override
    public void setReceiver(Receiver receiver) {
        this.receiver = receiver;
    }

    @Override
    public void print() {
        receiver.action();
    }
}
```



```java
public interface Receiver {
    void action();
}

public class ErrorPrintReceiver implements Receiver {
    @Override
    public void action() {
        System.err.println("errorPrintReceiver: command received!");
    }
}

public class NormalPrintReceiver implements Receiver {
    @Override
    public void action() {
        System.out.println("normalPrintReceiver: command received!");
    }
}
```



```java
public class Invoker{
    private Command command;
    Invoker(Command command) {
        this.command = command;
    }
    void action() {
        command.print();
    }
}
```



```java
public class CommandPattern {
    public static void main(String[] args) {
        // 执行正常打印命令
        Receiver normalPrintReceiver = new NormalPrintReceiver();
        Command cmd = new PrintCommand(normalPrintReceiver);
        Invoker invoker = new Invoker(cmd);
        invoker.action();
        
        // 执行错误打印命令
        Receiver errorPrintReceiver = new ErrorPrintReceiver();
        cmd.setReceiver(errorPrintReceiver);
        invoker.action();
    }
}
```



### 基于IOC



## 源码解析




# 设计模式15-迭代器模式（Iterator）
## 概述

## 简单实现

## 基于JDK的实现

## 基于IOC容器的实现

## 源码解析

## 优点

## 使用场景

## 参考资料


# 设计模式16-中介者模式（Mediator）
## 概述

## 简单实现

## 基于JDK的实现

## 基于IOC容器的实现

## 源码解析

## 优点

## 使用场景

## 参考资料


# 设计模式17-访问者模式（Visitor）
## 概述

## 简单实现

## 基于JDK的实现

## 基于IOC容器的实现

## 源码解析

## 优点

## 使用场景

## 参考资料


# 设计模式18-备忘录模式（Memento）
## 概述

## 简单实现

## 基于JDK的实现

## 基于IOC容器的实现

## 源码解析

## 优点

## 使用场景

## 参考资料


# 设计模式19-模板方法模式（Template）
## 概述
> Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm’s structure.

定义一个操作中的算法的框架， 而将一些步骤延迟到子类中。 使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

## 简单实现

## 基于JDK的实现

## 基于IOC容器的实现

## 源码解析

## 优缺点
### 优点
- 封装不变部分，扩展可变部分

把认为是不变部分的算法封装到父类实现，而可变部分的则可以通过继承来继续扩展。

- 提取公共部分代码，便于维护

- 行为由父类控制，子类实现

基本方法是由子类实现的，因此子类可以通过扩展的方式增加相应的功能，符合开闭原则。

### 缺点
子类执行的结果影响了父类的结果，在复杂项目中，会带来阅读上的难度。

可能引起子类泛滥和为了继承而继承的问题

## 使用场景

## 参考资料
- [Java设计模式之（十三）——模板方法模式 ](https://www.cnblogs.com/ysocean/p/15631116.html)


# 设计模式20-观察者模式（Observer）
## 概述
> Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

在对象之间定义一个一对多的依赖，当一个对象状态改变的时候，所有依赖的对象都会得到通知并自动更新。也叫发布订阅模式，能够很好的解耦一个对象改变，自动改变另一个对象这种情况。

## 简单实现
### JDK实现
- 定义被观察者
> com.demo.design.pattern.behavioral.observer.Subject
```java
public interface Subject {
    /**
     * 添加观察者（如果有必要，可以添加remove方法移除观察者接口并在抽象类中实现）
     *
     * @param observer 观察者
     */
    void add(Observer observer);

    /**
     * 被观察者执行操作发生变化
     */
    void operation();
}
```

> com.demo.design.pattern.behavioral.observer.AbstractSubject
```java
public abstract class AbstractSubject implements Subject {
    /**
     * 观察者对象，可以有多个不同的观察者执行不同的行为
     */
    private final List<Observer> observers = new ArrayList<>();

    @Override
    public void add(Observer observer) {
        observers.add(observer);
    }

    void notifyObservers() {
        // 唤醒所有的观察者并同步更新
        for (Observer observer : observers) {
            observer.update();
        }
    }
}
```

> com.demo.design.pattern.behavioral.observer.SubjectImpl
```java
public class SubjectImpl extends AbstractSubject {
    @Override
    public void operation() {
        System.out.println("update self!");
        this.notifyObservers();
    }
}
```

- 定义被观察者
> com.demo.design.pattern.behavioral.observer.Observer
```java
interface Observer {
    /**
     * 更新操作
     */
    void update();
}
```

> com.demo.design.pattern.behavioral.observer.ObserverImpl
```java
public class ObserverImpl implements Observer {
    @Override
    public void update() {
        System.out.println("observer has received and update");
    }
}
```

- 使用示例
```java
@Test
public void observerPattern(){
    Subject sub = new SubjectImpl();
    sub.add(new ObserverImpl());
    sub.operation();
}
```

## 源码解析
### JDK
> java.util.Observer
```java
/**
 * 通过实现 Observer 接口来接受被观察对象发生改变而发出的通知
 */
public interface Observer {
    /**
     * 当被观察对象发生改变时会调用此方法。当改变发生时，调用被观察对象的唤醒方法来通知所有观察者。
     *
     * @param   o     被观察对象.
     * @param   arg   被观察对象唤醒方法参数
     */
    void update(Observable o, Object arg);
}
```

> java.util.Observable
```java
/**
 * This class represents an observable object, or "data"
 * in the model-view paradigm. It can be subclassed to represent an
 * object that the application wants to have observed.
 * <p>
 * An observable object can have one or more observers. An observer
 * may be any object that implements interface <tt>Observer</tt>. After an
 * observable instance changes, an application calling the
 * <code>Observable</code>'s <code>notifyObservers</code> method
 * causes all of its observers to be notified of the change by a call
 * to their <code>update</code> method.
 * <p>
 * The order in which notifications will be delivered is unspecified.
 * The default implementation provided in the Observable class will
 * notify Observers in the order in which they registered interest, but
 * subclasses may change this order, use no guaranteed order, deliver
 * notifications on separate threads, or may guarantee that their
 * subclass follows this order, as they choose.
 * <p>
 * Note that this notification mechanism has nothing to do with threads
 * and is completely separate from the <tt>wait</tt> and <tt>notify</tt>
 * mechanism of class <tt>Object</tt>.
 * <p>
 * When an observable object is newly created, its set of observers is
 * empty. Two observers are considered the same if and only if the
 * <tt>equals</tt> method returns true for them.
 *
 * @author  Chris Warth
 * @see     java.util.Observable#notifyObservers()
 * @see     java.util.Observable#notifyObservers(java.lang.Object)
 * @see     java.util.Observer
 * @see     java.util.Observer#update(java.util.Observable, java.lang.Object)
 * @since   JDK1.0
 */
public class Observable {
    private boolean changed = false;
    private Vector<Observer> obs;

    /** Construct an Observable with zero Observers. */

    public Observable() {
        obs = new Vector<>();
    }

    /**
     * Adds an observer to the set of observers for this object, provided
     * that it is not the same as some observer already in the set.
     * The order in which notifications will be delivered to multiple
     * observers is not specified. See the class comment.
     *
     * @param   o   an observer to be added.
     * @throws NullPointerException   if the parameter o is null.
     */
    public synchronized void addObserver(Observer o) {
        if (o == null)
            throw new NullPointerException();
        if (!obs.contains(o)) {
            obs.addElement(o);
        }
    }

    /**
     * Deletes an observer from the set of observers of this object.
     * Passing <CODE>null</CODE> to this method will have no effect.
     * @param   o   the observer to be deleted.
     */
    public synchronized void deleteObserver(Observer o) {
        obs.removeElement(o);
    }

    /**
     * If this object has changed, as indicated by the
     * <code>hasChanged</code> method, then notify all of its observers
     * and then call the <code>clearChanged</code> method to
     * indicate that this object has no longer changed.
     * <p>
     * Each observer has its <code>update</code> method called with two
     * arguments: this observable object and <code>null</code>. In other
     * words, this method is equivalent to:
     * <blockquote><tt>
     * notifyObservers(null)</tt></blockquote>
     *
     * @see     java.util.Observable#clearChanged()
     * @see     java.util.Observable#hasChanged()
     * @see     java.util.Observer#update(java.util.Observable, java.lang.Object)
     */
    public void notifyObservers() {
        notifyObservers(null);
    }

    /**
     * If this object has changed, as indicated by the
     * <code>hasChanged</code> method, then notify all of its observers
     * and then call the <code>clearChanged</code> method to indicate
     * that this object has no longer changed.
     * <p>
     * Each observer has its <code>update</code> method called with two
     * arguments: this observable object and the <code>arg</code> argument.
     *
     * @param   arg   any object.
     * @see     java.util.Observable#clearChanged()
     * @see     java.util.Observable#hasChanged()
     * @see     java.util.Observer#update(java.util.Observable, java.lang.Object)
     */
    public void notifyObservers(Object arg) {
        /*
         * a temporary array buffer, used as a snapshot of the state of
         * current Observers.
         */
        Object[] arrLocal;

        synchronized (this) {
            /* We don't want the Observer doing callbacks into
             * arbitrary code while holding its own Monitor.
             * The code where we extract each Observable from
             * the Vector and store the state of the Observer
             * needs synchronization, but notifying observers
             * does not (should not).  The worst result of any
             * potential race-condition here is that:
             * 1) a newly-added Observer will miss a
             *   notification in progress
             * 2) a recently unregistered Observer will be
             *   wrongly notified when it doesn't care
             */
            if (!changed)
                return;
            arrLocal = obs.toArray();
            clearChanged();
        }

        for (int i = arrLocal.length-1; i>=0; i--)
            ((Observer)arrLocal[i]).update(this, arg);
    }

    /**
     * Clears the observer list so that this object no longer has any observers.
     */
    public synchronized void deleteObservers() {
        obs.removeAllElements();
    }

    /**
     * Marks this <tt>Observable</tt> object as having been changed; the
     * <tt>hasChanged</tt> method will now return <tt>true</tt>.
     */
    protected synchronized void setChanged() {
        changed = true;
    }

    /**
     * Indicates that this object has no longer changed, or that it has
     * already notified all of its observers of its most recent change,
     * so that the <tt>hasChanged</tt> method will now return <tt>false</tt>.
     * This method is called automatically by the
     * <code>notifyObservers</code> methods.
     *
     * @see     java.util.Observable#notifyObservers()
     * @see     java.util.Observable#notifyObservers(java.lang.Object)
     */
    protected synchronized void clearChanged() {
        changed = false;
    }

    /**
     * Tests if this object has changed.
     *
     * @return  <code>true</code> if and only if the <code>setChanged</code>
     *          method has been called more recently than the
     *          <code>clearChanged</code> method on this object;
     *          <code>false</code> otherwise.
     * @see     java.util.Observable#clearChanged()
     * @see     java.util.Observable#setChanged()
     */
    public synchronized boolean hasChanged() {
        return changed;
    }

    /**
     * Returns the number of observers of this <tt>Observable</tt> object.
     *
     * @return  the number of observers of this object.
     */
    public synchronized int countObservers() {
        return obs.size();
    }
}
```

### EventBus
事件总线，提供了实现观察者模式的骨架代码。Google Guava EventBus 就是一个比较著名的 EventBus 框架，它不仅仅支持异步非阻塞模式，同时也支持同步阻塞模式。

> com.google.common.eventbus.EventBus
```java
@Beta
public class EventBus {
    private static final Logger logger = Logger.getLogger(EventBus.class.getName());
    private final String identifier;
    private final Executor executor;
    private final SubscriberExceptionHandler exceptionHandler; 
    private final SubscriberRegistry subscribers = new SubscriberRegistry(this);
    private final Dispatcher dispatcher;
    // ...
}
```

利用 `EventBus` 框架实现的观察者模式，不需要定义 `Observer` 接口，任意类型的对象都可以注册到 `EventBus` 中，通过 `@Subscribe` 注解来标明类中哪个函数可以接收被观察者发送的消息。

## 优点
- 观察者和被观察者之间抽象耦合

不管是增加观察者还是被观察者都非常容易扩展，在系统扩展方面会得心应手。

- 建立一套触发机制

被观察者变化引起观察者自动变化。但是需要注意的是，一个被观察者，多个观察者，Java的消息通知默认是顺序执行的，如果一个观察者卡住，会导致整个流程卡住，这就是同步阻塞。

所以实际开发中没有先后顺序的考虑使用异步，异步非阻塞除了能够实现代码解耦，还能充分利用硬件资源，提高代码的执行效率。

另外还有不同进程间的观察者模式，通常基于消息队列来实现，用于实现不同进程间的观察者和被观察者之间的交互。

## 使用场景

- 关联行为场景。如用户注册或登陆成功后发送短信提醒，就可以通过观察注册或登陆是否成功来发送短信。

- 事件多级触发场景。

- 跨系统的消息交换场景， 如消息队列的处理机制。

## 参考资料
- [Java设计模式之（十二）——观察者模式](https://www.cnblogs.com/ysocean/p/15627036.html)


# 设计模式21-状态模式（State）
## 概述
如果一个实体具备状态，且在不同状态下会在同一业务场景执行不同的业务逻辑时，就可以考虑使用状态模式。

### 优点
- 容易新加状态，封装了状态转移规则，每个状态可以被复用和共享。
- 避免大量的if else结构。

### 缺点
- 状态类膨胀。
- 新加入状态时，可能需要修改现有的状态实现。

## 简单实现
### JDK实现
```java
public interface State {
    /**
     * open mobile
     */
    void open();
    
    /**
     * close mobile
     */
    void close();
}
```

```java
@Slf4j
public class WaitingState implements State{
    private final MobileModel model;

    public WaitingState(MobileModel model) {
        this.model = model;
    }

    @Override
    public void open() {
        log.info("开启手机中。。。");
        this.model.setState(new OpenState(this.model));
    }

    @Override
    public void close() {
        log.info("关闭手机中。。。");
        this.model.setState(new CloseState(this.model));
    }
}
```

```java
@Slf4j
public class OpenState implements State {
    private final MobileModel model;

    public OpenState(MobileModel model) {
        this.model = model;
    }

    @Override
    public void open() {
        log.info("手机已开启");
    }

    @Override
    public void close() {
       log.info("关闭手机中。。。");
        model.setState(new CloseState(this.model));
    }
}
```

```java
@Slf4j
public class CloseState implements State {
    private final MobileModel model;

    public CloseState(MobileModel model) {
        this.model = model;
    }

    @Override
    public void open() {
        log.info("开启手机中。。。");
        this.model.setState(new OpenState(this.model));
    }

    @Override
    public void close() {
        log.info("手机已关闭");
    }
}
```

状态模式中具体实现功能的代码被封装到了 `State` 的实现类中，上下文通过设置不同的 `State` 实现类。
```java
public class MobileModel {
    private State state;

    public MobileModel() { }

    public MobileModel(State state) {
        this.state = state;
    }

    public void setState(State state) {
        this.state = state;
    }

    public void open() {
        state.open();
    }

    public void close() {
        state.close();
    }
}
```

在同一场景中根据不同的状态来调用不同的业务逻辑。
```java
public class DesignPattenTest {
    @Test
    public void stateDesignPattern(){
        MobileModel model = new MobileModel();
        model.setState(new WaitingState(model));
        model.open();
        model.open();
        model.close();
        model.close();
        model.open();
    }
}
```

## 参考资料


# 设计模式22-策略模式
## 概述
策略模式（Strategy）的重点在于其实现可以去感知随意替换，根据不同的场景调用不同的实现。



### 优点
 - 算法可以自由切换。 
 - 免使用多重条件判断。 
 - 扩展性良好。



### 缺点
- 策略类膨胀。 
- 所有策略类都需要对外暴露。



### 和状态模式的比较
- 策略模式的各个策略之间没有关系，状态模式中要实现状态的流转；
- 策略模式的策略调用由客户端决定，状态模式客户端定义状态的流转；
- 策略模式强调策略的选择，状态模式强调状态的变化。



## 简单实现


### JDK实现
策略模式需要设计一个接口，为一系列策略模式实现类提供统一的方法对外提供调用。
```java
interface Strategy {
    /**
     * print message
     *
     * @param message message 
     */
    void print(String message);
}
```



通过不同的实现提供不同的业务逻辑。

```java
public class NormalStrategy implements Strategy {
    @Override
    public void print(String message) {
        System.out.println(message);
    }
}
```

```java
public class ErrorStrategy implements Strategy {
    @Override
    public void print(String message) {
        System.err.println(message);
    }
}
```



在不同的场景中，通过调用不同的实现类来完成不同的业务逻辑。

```java
public class DesignPattenTest {
    @Test
    public void strategyDesignPattern() {
        String message = "this is a message";
        Strategy printStrategy = new NormalStrategy();
        printStrategy.print(message);
        printStrategy = new ErrorStrategy();
        printStrategy.print(message);
    }
}
```



### 基于Spring

通过SpringBoot实现时可以结合工厂模式，将不同的策略实现类注入到容器中。

1. 定义策略类工厂，并提供注册和获取策略实现类方法。

```java
public class StrategyFactory {
    private final static Map<String, IStrategy> STRATEGY_MAP = new ConcurrentHashMap<>();
    
    public static IStrategy getStrategy(String strategyMark) {
        return STRATEGY_MAP.get(strategyMark);
    }
    
    public static void register(String strategyMark, IStrategy strategy) {
        STRATEGY_MAP.put(strategyMark, strategy);
    }
}
```

2. 定义策略接口，如果需要可以定义策略抽象类实现该接口，增加中间层。

```java
public interface IStrategy {
    /**
     * execute strategy method
     */
    void execute();
}
```

```java
public abstract class BaseStrategy implements IStrategy {

}
```

3. 定义具体的策略实现类继承策略抽象类或策略接口
```java
@Slf4j
@Service
@AllArgsConstructor
public class FirstStrategy extends BaseStrategy implements InitializingBean {
    @Override
    public void execute() {
        log.info("execute first strategy method");
    }

    @Override
    public void afterPropertiesSet() {
        log.info("register first strategy implement class when init context");
        StrategyFactory.register("first", this);
    }
}
```

```java
@Slf4j
@Service
@AllArgsConstructor
public class SecondStrategy extends BaseStrategy implements InitializingBean {
    @Override
    public void execute() {
        log.info("execute second strategy method");
    }

    @Override
    public void afterPropertiesSet() {
        log.info("register second strategy implement class when init context");
        StrategyFactory.register("first", this);
    }
}
```



## 源码解析

### 线程池拒绝策略

ThreadPoolExecutor有4中默认的拒绝策略，通过在构造线程池时指定拒绝策略来实现策略模式。

> java.util.concurrent.ThreadPoolExecutor

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
	// ...
    this.handler = handler;
}
```



> java.util.concurrent.ThreadPoolExecutor

```java
public void execute(Runnable command) {
    // ...
    else if (!addWorker(command, false))
        // 添加执行任务失败时调用拒绝策略
        reject(command);
}

final void reject(Runnable command) {
    // 由构造线程池时指定的拒绝策略来实现拒绝逻辑
    handler.rejectedExecution(command, this);
}
```

