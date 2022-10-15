# 概念

## Java语言特点

1. 简单易学；
2. 面向对象（封装，继承，多态）；
3. 平台无关性（ Java 虚拟机实现平台无关性）；
4. 可靠性；
5. 安全性；
6. 支持多线程（ C++ 语言没有内置的多线程机制，因此必须调用操作系统的多线程功能来进行多线程程序设计，而 Java 语言却提供了多线程支持）；
7. 支持网络编程并且很方便（ Java 语言诞生本身就是为简化网络编程设计的，因此 Java 语言不仅支持网络编程而且很方便）；
8. 编译与解释并存；



## JDK、JRE和JVM



# 数据类型
## 基本数据类型

Java的每种基数据类型的大小是确定的，不会随机器硬件结构变化而变化。这种大小固定的特性是Java具备可移植性的原因之一。



### 整型

### 浮点型

### 布尔型

### 字符型

 

**字符型常量和字符串常量的区别?**

1. 形式上: 字符常量是单引号引起的一个字符; 字符串常量是双引号引起的若干个字符
2. 含义上: 字符常量相当于一个整型值( ASCII 值),可以参加表达式运算; 字符串常量代表一个地址值(该字符串在内存中存放位置)
3. 占内存大小 字符常量只占 2 个字节; 字符串常量占若干个字节 (**注意： char 在 Java 中占两个字节**)



### 总结

| 基本类型 | 大小    | 最小值    | 最大值          | 包装器类型 |
| -------- | ------- | --------- | --------------- | ---------- |
| boolean  |         |           |                 | Boolean    |
| char     | 16 bits | Unicode o | Unicode 2^16 -1 | Character  |
| byte     | 8 bits  | -128      | 127             | Byte       |
| short    | 16 bits | -2^15     | 2^15-1          | Short      |
| int      | 32 bits | -2^31     | 2^31-1          | Integer    |
| long     | 64 bits | -2^63     | 2^63-1          | Long       |
| float    | 32 bits | IEEE754   | IEEE754         | Float      |
| double   | 64 bits | IEEE754   | IEEE754         | Double     |



## 包装数据类型

### 对象创建

```java
public void create(){
    Integer i = new Integer(1);
    Integer j = 1;
}
```



- 第一种方式不会触发自动装箱的过程；而第二种方式会触发；
- 在执行效率和资源占用上的区别。第二种方式的执行效率和资源占用在一般性情况下要优于第一种情况（注意这并不是绝对的）。



### 小值缓存

通过valueOf方法创建Integer对象的时候，如果数值在[-128,127]之间，便返回指向IntegerCache.cache中已经存在的对象的引用；否则创建一个新的Integer对象。



> java.lang.Integer

```java
public final class Integer extends Number implements Comparable<Integer> {
    public static Integer valueOf(String s, int radix) throws NumberFormatException {
        return Integer.valueOf(parseInt(s, radix));
    }

    public static Integer valueOf(String s) throws NumberFormatException {
        return Integer.valueOf(parseInt(s, 10));
    }

    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
}
```



> java.lang.Integer.IntegerCache

```java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // 最大缓存值默认为127，可通过系统配置修改
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // 配置的缓存最大值不能超过int最大值 - 129
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // 配置的最大值无法解析成int值时，忽略该配置
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // 缓存值必须在 [-128, 127] 范围内
        assert Integer.IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```



Double类的valueOf方法采用与Integer类的valueOf方法不同的实现。因为在某个范围内的整型数值的个数是有限的，而浮点数却不是。

Integer、Short、Byte、Character、Long这几个类的valueOf方法的实现是类似的。Double、Float的valueOf方法的实现是类似的。



### 自动装箱与拆箱

- **装箱**：将基本类型用它们对应的引用类型包装起来；
- **拆箱**：将包装类型转换为基本数据类型；



装箱的时候自动调用的是Integer的valueOf(int)方法。而在拆箱的时候自动调用的是Integer的intValue方法。



### 等值比较

```java
public void equals(){
    Integer a = 1;
    Integer b = 2;
    Integer c = 3;
    Integer d = 3;
    Integer e = 321;
    Integer f = 321;
    Long g = 3L;
    Long h = 2L;

    System.out.println(c==d); // true
    System.out.println(e==f); // false
    System.out.println(c==(a+b)); // true
    System.out.println(c.equals(a+b)); // true
    System.out.println(g==(a+b)); // true
    System.out.println(g.equals(a+b)); // false
    System.out.println(g.equals(a+h)); // true
}
```



当 "=="运算符的两个操作数都是包装器类型的引用，则是比较指向的是否是同一个对象，而如果其中有一个操作数是表达式（即包含算术运算）则比较的是数值（即会触发自动拆箱的过程）。另外，对于包装器类型，equals方法并不会进行类型转换。



## 引用数据类型
## 数组
## 数据类型转换
# 运算符
## 算数运算符
## 赋值运算符
## 关系运算符
## 逻辑运算符
## 位运算符
## 三元运算符

## 运算符优先级别

# 进程控制
## 分支控制符
## 循环控制符
## 跳转控制符
# 面向对象

## 面向对象和面向过程

- **面向过程** ：**面向过程性能比面向对象高。** 因为类调用时需要实例化，开销比较大，比较消耗资源，所以当性能是最重要的考量因素的时候，比如单片机、嵌入式开发、Linux/Unix 等一般采用面向过程开发。但是，**面向过程没有面向对象易维护、易复用、易扩展。**
- **面向对象** ：**面向对象易维护、易复用、易扩展。** 因为面向对象有封装、继承、多态性的特性，所以可以设计出低耦合的系统，使系统更加灵活、更加易于维护。但是，**面向对象性能比面向过程低**。



> 这个并不是根本原因，面向过程也需要分配内存，计算内存偏移量，Java 性能差的主要原因并不是因为它是面向对象语言，而是 Java 是半编译语言，最终的执行代码并不是可以直接被 CPU 执行的二进制机械码。
>
> 而面向过程语言大多都是直接编译成机械码在电脑上执行，并且其它一些面向过程的脚本语言性能也并不一定比 Java 好。



## 类

### 构造方法

构造方法主要作用是完成对类对象的初始化工作。一个类即使没有声明构造方法也会有默认的空构造方法。



#### 特性

1. 名字与类名相同。
2. 没有返回值，但不能用 void 声明构造函数。
3. 生成类的对象时自动执行，无需调用。



#### 空构造方法

空构造方法指的是没有任何参数的构造方法。

Java 程序在执行子类的构造方法之前，如果没有用 super()来调用父类特定的构造方法，则会调用父类中空构造方法来进行初始化。因此，如果父类中只定义了有参数的构造方法，而在子类的构造方法中又没有用 super()来调用父类中特定的构造方法，则编译时将发生错误，因为 Java 程序在父类中找不到没有参数的构造方法可供执行。解决办法是在父类里加上一个不做事且没有参数的构造方法。



#### 对象创建

通过 new 运算符创建对象，new 创建对象实例（对象实例在堆内存中），对象引用指向对象实例（对象引用存放在栈内存中）。一个对象引用可以指向 0 个或 1 个对象（一根绳子可以不系气球，也可以系一个气球）;一个对象可以有 n 个引用指向它（可以用 n 条绳子系住一个气球）。



### 属性变量

**成员变量与局部变量的区别有哪些？**

1. 从语法形式上看：成员变量是属于类的，而局部变量是在方法中定义的变量或是方法的参数；成员变量可以被 public,private,static 等修饰符所修饰，而局部变量不能被访问控制修饰符及 static 所修饰；但是，成员变量和局部变量都能被 final 所修饰。
2. 从变量在内存中的存储方式来看：如果成员变量是使用static修饰的，那么这个成员变量是属于类的，如果没有使用static修饰，这个成员变量是属于实例的。对象存于堆内存，如果局部变量类型为基本数据类型，那么存储在栈内存，如果为引用数据类型，那存放的是指向堆内存对象的引用或者是指向常量池中的地址。
3. 从变量在内存中的生存时间上看：成员变量是对象的一部分，它随着对象的创建而存在，而局部变量随着方法的调用而自动消失。
4. 成员变量如果没有被赋初值，则会自动以类型的默认值而赋值。被 final 修饰的成员变量也必须显式地赋值，而局部变量则不会自动赋值。



### 方法

#### 方法结构

权限修饰符 + 返回值 + 方法名 + 参数



**返回值**

方法的返回值是指获取到的某个方法体中的代码执行后产生的结果，使得它可以用于其他的操作！





#### 方法重载

重载就是同样的一个方法能够根据输入数据的不同，做出不同的处理。

发生在同一个类中，方法名必须相同，参数类型不同、个数不同、顺序不同，方法返回值和访问修饰符可以不同。Java可以重载任何方法。



#### 方法重写

重写就是当子类继承自父类的相同方法，输入数据一样，但要做出有别于父类的响应时，你就要覆盖父类方法。

重写发生在运行期，是子类对父类的允许访问的方法的实现过程进行重新编写。

1. 返回值类型、方法名、参数列表必须相同，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类。
2. 如果父类方法访问修饰符为 `private/final/static` 则子类就不能重写该方法，但是被 static 修饰的方法能够被再次声明。
3. 构造方法无法被重写

综上：重写就是子类对父类方法的重新改造，外部样子不能改变，内部逻辑可以改变。



**方法的重写要遵循“两同两小一大”**

- “两同”即方法名相同、形参列表相同；
- “两小”指的是子类方法返回值类型应比父类方法返回值类型更小或相等，子类方法声明抛出的异常类应比父类方法声明抛出的异常类更小或相等；
- “一大”指的是子类方法的访问权限应比父类方法的访问权限更大或相等。

关于**重写的返回值类**型 这里需要额外多说明一下，如果方法的返回类型是void和基本数据类型，则返回值重写时不可修改。但是如果方法的返回值是引用类型，重写时是可以返回该引用类型的子类的。



#### 重载和重写总结

| 区别点     | 重载方法 | 重写方法                                                     |
| ---------- | -------- | ------------------------------------------------------------ |
| 发生范围   | 同一个类 | 子类                                                         |
| 参数列表   | 必须修改 | 一定不能修改                                                 |
| 返回类型   | 可修改   | 子类方法返回值类型应比父类方法返回值类型更小或相等           |
| 异常       | 可修改   | 子类方法声明抛出的异常类应比父类方法声明抛出的异常类更小或相等； |
| 访问修饰符 | 可修改   | 一定不能做更严格的限制（可以降低限制）                       |
| 发生阶段   | 编译期   | 运行期                                                       |



## 三大特性

### 封装

封装把一个对象的属性私有化，同时提供一些可以被外界访问的属性的方法，如果属性不想被外界访问，可不提供方法给外界访问。但是如果一个类没有提供给外界访问的方法，那么这个类也没有什么意义了。



### 继承

继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承我们能够非常方便地复用以前的代码。



**关于继承如下 3 点请记住：**

1. 子类拥有父类对象所有的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法子类是无法访问，**只是拥有**。
2. 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。
3. 子类可以用自己的方式实现父类的方法。（以后介绍）。



### 多态

所谓多态就是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，即一个引用变量到底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。

在 Java 中有两种形式可以实现多态：继承（多个子类对同一方法的重写）和接口（实现接口并覆盖接口中同一方法）。



## 抽象和接口

1. 接口的方法默认是 `public`，所有方法在接口中不能有实现(Java 8 开始接口方法可以有默认实现），而抽象类可以有非抽象的方法。
2. 接口中除了 `static`、`final` 变量，不能有其他变量，而抽象类中则不一定。
3. 一个类可以实现多个接口，但只能实现一个抽象类。接口自己本身可以通过 `extends` 关键字扩展多个接口。
4. 接口方法默认修饰符是 `public`，抽象方法可以有 `public`、`protected` 和 `default` 这些修饰符（抽象方法就是为了被重写所以不能使用 `private` 关键字修饰！）。
5. 从设计层面来说，抽象是对类的抽象，是一种模板设计，而接口是对行为的抽象，是一种行为的规范。



> 1. 在 JDK8 中，接口也可以定义静态方法，可以直接用接口名调用。实现类和实现是不可以调用的。如果同时实现两个接口，接口中定义了一样的默认方法，则必须重写，不然会报错。
> 2.  JDK 8 的时候接口可以有默认方法和静态方法功能。
> 3.  JDK 9 的接口被允许定义私有方法和私有静态方法 。





## 枚举

# 关键字

## static

### 作用范围

static可以用来修饰类、属性、方法和代码块。



由于静态方法可以不通过对象进行调用，因此在静态方法里，不能调用其他非静态变量，也不可以访问非静态变量成员。



### 静态方法

**静态方法和实例方法有何不同**

1. 在外部调用静态方法时，可以使用"类名.方法名"的方式，也可以使用"对象名.方法名"的方式。而实例方法只有后面这种方式。也就是说，调用静态方法可以无需创建对象。
2. 静态方法在访问本类的成员时，只允许访问静态成员（即静态成员变量和静态方法），而不允许访问实例成员变量和实例方法；实例方法则无此限制。

## final

### 作用范围

final可以用来修饰类、属性和方法。



### 作用

1. 对于一个 final 变量，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。
2. 当用 final 修饰一个类时，表明这个类不能被继承。final 类中的所有成员方法都会被隐式地指定为 final 方法。
3. 使用 final 方法的原因有两个。第一个原因是把方法锁定，以防任何继承类修改它的含义；第二个原因是效率。在早期的 Java 实现版本中，会将 final 方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升（现在的 Java 版本已经不需要使用 final 方法进行这些优化了）。类中所有的 private 方法都隐式地指定为 final。



## transient

### 作用范围

transient可以用来修饰属性。



### 作用

阻止实例中那些用此关键字修饰的的变量序列化；当对象被反序列化时，被 transient 修饰的变量值不会被持久化和恢复。transient 只能修饰变量，不能修饰类和方法。



# 序列化

- 序列化：把对象转换为字节序列的过程称为对象的序列化.
- 反序列化：把字节序列恢复为对象的过程称为对象的反序列化.



## 序列化特性

- transient修饰的属性都不会参与序列化，该属性反序列化后的结果为null；
- static修饰的属性不会参与序列化，序列化是针对对象而言的，而static属性优先于对象存在，随着类的加载而加载，所以不会被序列化；
- 基本数据类型能够序列化和反序列化。

```java
@Data
public class SerializableField implements Serializable {
    private Long money;
    private int age;
    private transient String name;
    private static String TITLE = "title";
}
```



```java
@Test
@SneakyThrows
public void serializeAndDeserialize() {
    SerializableField serializableField = new SerializableField();
    serializableField.setMoney(100L);
    serializableField.setAge(28);
    serializableField.setName("name");

    ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(new File("D:\\serialize.txt")));
    oos.writeObject(serializableField);
    oos.close();

    ObjectInputStream ois = new ObjectInputStream(new FileInputStream(new File("D:\\serialize.txt")));
    SerializableField deserializeField = (SerializableField) ois.readObject();
    System.err.println(deserializeField);
}
```



测试结果如下：

```bash
SerializableField(money=100, age=28, name=null)
```



## Serializable接口

**只要对内存中的对象进行持久化或网络传输, 这个时候都需要序列化和反序列化。**Java中序列化和反序列化都需要实现Serializable接口。

```java
public interface Serializable {
}
```



和浏览器交互是是将对象转换为JSON格式，实质上是String类型的数据，String实现了Serializable接口；将对象持久化到数据库中时，实质上是持久化了对象中的属性，而这些属性都是实现了Serializable接口的。



如果类没有实现Serializable接口并执行序列化或反序列化操作，则会抛出异常：

```java
@Data
public class NoSerializableBean {
    private String title;
}
```



```java
@Test
@SneakyThrows
public void noSerialize() {
    NoSerializableBean noSerializableBean = new NoSerializableBean();
    noSerializableBean.setTitle("noSerialize");

    ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(new File("D:\\noSerialize.txt")));
    oos.writeObject(noSerializableBean);
    oos.close();
}
```



测试结果如下：

```bash
java.io.NotSerializableException: com.demo.jdk.base.serialize.NoSerializableBean
	at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1184)
	at java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:348)
```



## serialVersionUID

对需要序列化的类实现Serializable接口后，还需要显示指定serialVersionUID值。否则在反序列化属性被修改的相同类时会反序列化失败。

```java
@Data
public class SerialVersionUID implements Serializable {
    private static final long serialVersionUID = 1L;
    
    private String title;
}
```



如果不显示指定serialVersionUID，JVM在序列化时根据属性自动生成serialVersionUID，然后与属性一起序列化，再进行持久化或网络传输。在反序列化时，JVM会再根据属性自动生成一个新版serialVersionUID，然后将这个新版serialVersionUID与序列化时生成的旧版serialVersionUID进行比较，如果相同则反序列化成功，否则报错。

若显示指定serialVersionUID，JVM在序列化和反序列化时仍都会生成serialVersionUID，但值为显示指定的值，这样在反序列化时新旧版本的serialVersionUID就一致了。



```java
@Data
public class NoSerialVersionUID implements Serializable {
    // 不显示指定serialVersionID
    private String title;
}
```



```java
@Test
@SneakyThrows
public void serialize() {
    NoSerialVersionUID noSerialVersionUID = new NoSerialVersionUID();
    noSerialVersionUID.setTitle("serialize");

    ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(new File("D:\\serialize.txt")));
    oos.writeObject(noSerialVersionUID);
    oos.close();
}

@Test
@SneakyThrows
public void deserialize() {
    ObjectInputStream ois = new ObjectInputStream(new FileInputStream(new File("D:\\serialize.txt")));
    NoSerialVersionUID deserializeBean = (NoSerialVersionUID) ois.readObject();
    System.err.println(deserializeBean);
}
```

在不修改属性的情况下，不显示指定serialVersionUID能够正常序列化和反序列化。



```java
@Test
@SneakyThrows
public void serializeWhenFieldChanged() {
    NoSerialVersionUID noSerialVersionUID = new NoSerialVersionUID();
    noSerialVersionUID.setTitle("serialize");

    ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(new File("D:\\serialize.txt")));
    oos.writeObject(noSerialVersionUID);
    oos.close();
}
```



先将NoSerialVersionUID序列化，然后修改类，添加name属性后进行反序列化。

```java
@Data
public class NoSerialVersionUID implements Serializable {
    private String title;
    private String name;
}
```



```java
@Test
@SneakyThrows
public void deSerializeWhenFieldChanged() {
    ObjectInputStream ois = new ObjectInputStream(new FileInputStream(new File("D:\\serialize.txt")));
    NoSerialVersionUID deserializeBean = (NoSerialVersionUID) ois.readObject();
    System.err.println(deserializeBean);
}
```



此时因为类发生了变化，且没有显示指定serialVersionUID属性，反序列化时抛出异常：

```bash
java.io.InvalidClassException: com.demo.jdk.base.serialize.NoSerialVersionUID; local class incompatible: stream classdesc serialVersionUID = -3783502947229734359, local class serialVersionUID = -8289838064664040038

	at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:699)
	at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1885)
```



如果在类型显示指定serialVersionUID属性，重复上述操作则不会抛出异常，而是正常序列化和反序列化。





# 扩展

## 值传递

**按值调用(call by value)表示方法接收的是调用者提供的值，而按引用调用（call by reference)表示方法接收的是调用者提供的变量地址。一个方法可以修改传递引用所对应的变量值，而不能修改传递值调用所对应的变量值。** 

**Java 程序设计语言总是采用按值调用。也就是说，方法得到的是所有参数值的一个拷贝，也就是说，方法不能修改传递给它的任何参数变量的内容。**



### 基本数据类型

```java
public static void main(String[] args) {
    int num1 = 10;
    int num2 = 20;

    swap(num1, num2);

    System.out.println("num1 = " + num1); // num1 = 10
    System.out.println("num2 = " + num2); // num2 = 20
}

public static void swap(int a, int b) {
    int temp = a;
    a = b;
    b = temp;

    System.out.println("a = " + a); // a = 20
    System.out.println("b = " + b); // b = 10
}
```



在 swap 方法中，a、b 的值进行交换，并不会影响到 num1、num2。因为，a、b 中的值，只是从 num1、num2 的复制过来的。也就是说，a、b 相当于 num1、num2 的副本，副本的内容无论怎么修改，都不会影响到原件本身。

**一个方法不能修改一个基本数据类型的参数**



### 对象引用参数

```java
 public static void main(String[] args) {
        int[] arr = { 1, 2, 3, 4, 5 };
        System.out.println(arr[0]); // 1
        change(arr);
        System.out.println(arr[0]); // 0
    }

    public static void change(int[] array) {
        // 将数组的第一个元素变为0
        array[0] = 0;
    }
```



**方法得到的是对象引用的拷贝，对象引用及其他的拷贝同时引用同一个对象，因此数据内容的修改是作用于同一个对象。**



```java
public class Test {

    public static void main(String[] args) {
        // TODO Auto-generated method stub
        Student s1 = new Student("小张");
        Student s2 = new Student("小李");
        Test.swap(s1, s2);
        System.out.println("s1:" + s1.getName()); // s1:小张
        System.out.println("s2:" + s2.getName()); // s2:小李
    }

    public static void swap(Student x, Student y) {
        Student temp = x;
        x = y;
        y = temp;
        System.out.println("x:" + x.getName()); // x:小李
        System.out.println("y:" + y.getName()); // y:小张
    }
}
```



**方法并没有改变存储在变量 s1 和 s2 中的对象引用。swap 方法的参数 x 和 y 被初始化为两个对象引用的拷贝，这个方法交换的是这两个拷贝。**



### 总结

Java 程序设计语言对对象采用的不是引用调用，对象引用是按值传递的。下面再总结一下 Java 中方法参数的使用情况：

- 一个方法不能修改一个基本数据类型的参数（即数值型或布尔型）。
- 一个方法可以改变一个对象参数的状态。
- 一个方法不能让对象参数引用一个新的对象。



## 深拷贝 vs 浅拷贝

1. **浅拷贝**：对基本数据类型进行值传递，对引用数据类型进行引用传递般的拷贝，此为浅拷贝。
2. **深拷贝**：对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容，此为深拷贝。



### 参考资料