# Object类

## equals方法

### 源码解析

> java.lang.Object

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```



通常会重写equals()方法：若两个对象的内容相等，则equals()方法返回true；否则，返回fasle。



### equals方法特性要求

1. 对称性：如果x.equals(y)返回是"true"，那么y.equals(x)也应该返回是"true"。
2. 反射性：x.equals(x)必须返回是"true"。
3. 类推性：如果x.equals(y)返回是"true"，而且y.equals(z)返回是"true"，那么z.equals(x)也应该返回是"true"。
4. 一致性：如果x.equals(y)返回是"true"，只要x和y内容一直不变，不管你重复x.equals(y)多少次，返回都是"true"。
5. 非空性，x.equals(null)，永远返回是"false"；x.equals(和x不同类型的对象)永远返回是"false"。



### == 与 equals

**==** 的作用是判断两个对象的地址是不是相等。即判断两个对象是不是同一个对象(基本数据类型==比较的是值，引用数据类型==比较的是内存地址)。

**equals()** : 它的作用也是判断两个对象是否相等。但它一般有两种使用情况：

- 情况 1：类没有覆盖 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过“==”比较这两个对象。
- 情况 2：类覆盖了 equals() 方法。一般，覆盖 equals() 方法来比较两个对象的内容是否相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)。



```java
public void equals(){
    String a = new String("ab"); // a 为一个引用
    String b = new String("ab"); // b为另一个引用,对象的内容一样
    String aa = "ab"; // 放在常量池中
    String bb = "ab"; // 从常量池中查找
    if (aa == bb) // true
        System.out.println("aa==bb");
    if (a == b) // false，非同一对象
        System.out.println("a==b");
    if (a.equals(b)) // true
        System.out.println("a equals b");
    if (42 == 42.0) { // true
        System.out.println("true");
    }
}
```



## hashCode方法

> java.lang.Object

```java
public native int hashCode();
```

hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个 int 整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。hashCode()定义在 JDK 的 Object 类中，这就意味着 Java 中的任何类都包含有 hashCode() 函数。另外需要注意的是： Object 的 hashcode 方法是本地方法，也就是用 c 语言或 c++ 实现的，该方法通常用来将对象的 内存地址 转换为整数之后返回。

哈希表存储的是键值对(key-value)，它的特点是：能根据“键”快速的检索出对应的“值”。这其中就利用到了哈希码！

**hashCode() 在哈希表中才有用，在其它情况下没用。**在哈希表中hashCode() 的作用是获取对象的哈希码，进而确定该对象在哈希表中的位置。

哈希表根据元素的哈希码计算出元素在哈希表中的位置，然后将元素插入该位置即可。对于相同的元素只保存了一个。

Java集合中本质是哈希表的类，如HashMap，Hashtable，HashSet。这些类通过hashCode()方法保证数据的唯一性。



**为什么要有 hashCode？**

当把对象加入 HashSet 时，HashSet 会先计算对象的 hashcode 值来判断对象加入的位置，同时也会与其他已经加入的对象的 hashcode 值作比较，如果没有相符的 hashcode，HashSet 会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用 equals() 方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。这样大大减少了 equals 的次数，相应就大大提高了执行速度。



**为什么重写 equals 时必须重写 hashCode 方法**

如果两个对象相等，则 hashcode 一定也是相同的。两个对象相等,对两个对象分别调用 equals 方法都返回 true。

但是，两个对象有相同的 hashcode 值，它们也不一定是相等的 。

**因此，equals 方法被覆盖过，则 `hashCode` 方法也必须被覆盖。**

hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。



**为什么两个对象有相同的 hashcode 值，它们也不一定是相等的**

因为 hashCode() 所使用的杂凑算法也许刚好会让多个对象传回相同的杂凑值。越糟糕的杂凑算法越容易碰撞，但这也与数据值域分布的特性有关（所谓碰撞也就是指的是不同的对象得到相同的 hashCode。

如果 HashSet 在对比的时候，同样的 hashcode 有多个对象，它会使用 equals() 来判断是否真的相同。也就是说 hashcode 只是用来缩小查找成本。



# String

String 类中使用 final 关键字修饰字符数组来保存字符串，所以 String 对象是不可变的。



> java.lang.String

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```



> 在 Java 9 之后，String的实现改用 byte 数组存储字符串。



## 创建String对象

当创建 String 类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个 String 对象。



## equals方法

String 中的 equals 方法是被重写过的，因为 object 的 equals 方法是比较的对象的内存地址，而 String 的 equals 方法比较的是对象的值。



> java.lang.String

```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```



# StringBuilder和StringBuffer

而 StringBuilder 与 StringBuffer 都继承自 AbstractStringBuilder 类，在 AbstractStringBuilder 中也是使用字符数组保存字符串char[]value 但是没有用 final 关键字修饰，所以这两种对象都是可变的。

StringBuilder 与 StringBuffer 的构造方法都是调用父类构造方法也就是 AbstractStringBuilder 实现的。



> java.lang.StringBuilder

```java
public final class StringBuilder
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence
{
}
```



> java.lang.StringBuffer

```java
public final class StringBuffer
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence
{
}
```



>java.lang.AbstractStringBuilder

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    /**
     * The value is used for character storage.
     */
    char[] value;

    /**
     * The count is the number of characters used.
     */
    int count;
}
```



> 在 Java 9 之后，StringBuilder` 与 `StringBuffer的实现改用 byte 数组存储字符串。



## String、StringBuilder和StringBuffer

## **线程安全性**

String 中的对象是不可变的，也就可以理解为常量，线程安全。AbstractStringBuilder 是 StringBuilder 与 StringBuffer 的公共父类，定义了一些字符串的基本操作，如 expandCapacity、append、insert、indexOf 等公共方法。StringBuffer 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。StringBuilder 并没有对方法进行加同步锁，所以是非线程安全的。



## 性能

每次对 String 类型进行改变的时候，都会生成一个新的 String 对象，然后将指针指向新的 String 对象。StringBuffer 每次都会对 StringBuffer 对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用 StringBuilder 相比使用 StringBuffer 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。



## 总结

1. 操作少量的数据: 适用 String
2. 单线程操作字符串缓冲区下操作大量数据: 适用 StringBuilder
3. 多线程操作字符串缓冲区下操作大量数据: 适用 StringBuffer



# StringTable

## String的基本特性

- String: 字符串，使用一对""引起来表示
- String声明为final的，**不可被继承**
- String实现了Serializable接口：表示字符串是支持序列化的。
- 实现了Comparable接口：表示String可以比较大小。
- String在jdk8及以前内部定义了final char[] value用于存储字符串数据。jdk9时改为byte[]
- String：**代表不可变的字符序列**。简称:不可变性。
    - 当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值。
    - 当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。
    - 当调用String的replace()方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。
- 通过字面量的方式(区别于new)给一个字符串赋值，此时的字符串值声明在字符串常量池中。
- **字符串常量池中是不会存储相同内容的字符串的**
    - String的String Pool是一个固定大小的Hashtable，默认值大小长度是1009。如果放进string Pool的String非常多， 就会造成Hash冲突严重，从而导致链表会很长，而链表长了后直接会造成的影响就是当调用String.intern时性能会大幅下降。
    - 使用-XX: StringTableSi ze可设置StringTable的长度
    - 在jdk6中StringTable是固定的，就是1009的长度，所以如果常量池中的字符串过多就会导致效率下降很快。StringTableSize设置没有要求
    - 在jdk7中，StringTable的长度默认值是60013，1009是可设置的最小值。

## String的内存分配

- 在Java语言中有8种基本数据类型和一种比较特殊的类型string。这些类型为了使它们在运行过程中速度更快、更节省内存，都提供了一种常量池的概念。
- 常量池就类似一个Java系统级别提供的缓存。8种基本数据类型的常量池都是系统协调的，String类型的常量池比较特殊。它的主要使用方法有两种。
- 直接使用双引号声明出来的String对象会直接存储在常量池中。
    - 比如: String info = "atguigu.com" ;
- 如果不是用双引号声明的String对象，可以使用String提供的
    - intern()方法
- Java 6及以前，字符串常量池存放在永久代。
- Java 7中Oracle的工程师对字符串池的逻辑做了很大的改变，即*将**字符串常量池的位置调整到Java堆内***。
    - 所有的字符串都保存在堆(Heap)中，和其他普通对象一样，这样可以让你在进行调优应用时仅需要调整堆大小就可以了。
    - 字符串常量池概念原本使用得比较多，但是这个改动使得我们有足够的理由让我们重新考虑在Java 7中使用String.intern()。
- Java8元空间，字符串常量在堆

**StringTable为什么要调整？**

1. permSize默认比较小
2. 永久代垃圾回收频率低

## 字符串的拼接操作

1. 常量与常量的拼接结果在常量池，原理是编译期优化
2. 常量池中不会存在相同内容的常量。
3. 只要其中有一个是变量，结果就在堆中。变量拼接的原理是StringBuilder
4. 如果拼接的结果调用intern()方法，则主动将常量池中还没有的字符串对象放入池中，并返回此对象地址。
    - intern()：判断字符串常量池中是否存在这个值，如果存在，则返回常量池中的地址，如果不存在，则再常量池中加载一个新的，并返回对象的地址
5. 字符串拼接操作不一定使用的是StringBuilder，如果拼接符号左右两边都是字符串常量或常量引用，则仍然使用编译器优化，即非StringBuilder的方式。
6. **注意**：虽然编译器会将+号的拼接操作转换为StringBuilder，但是多次拼接会多次转换为StringBuilder，并不是一次，因此会很浪费时间和空间。

```java
public void test4(){
	final String s1 = "a";
	final String s2 = "b";
	String s3 = "ab";
	String s4 = S1 + s2;
	System. out.println(s3 == s4);//true
}
```



> 针对于final修饰类、方法、基本数据类型、引用数据类型的量的结构时，能使用上final的时候建议使用上。



## intern()的使用

如果不是用双引号声明的String对象，可以使用String提供的intern方法: intern方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中。



```java
String myInfo = new String("I love atguigu").intern();
```

也就是说，如果在任意字符串上调用String. intern方法，那么其返回结果所指向的那个类实例，必须和直接以常量形式出现的字符串实例完全相同。因此，下列表达式的值必定是true:

```java
("a" + "b" + "c").intern() == "abc" 
```

通俗点讲，Interned String就是确保字符串在内存里只有一份拷贝，这样可以节约内存空间，加快字符串操作任务的执行速度。注意，这个值会被存放在字符串内部池(String Intern Pool) 。

**new String("ab")会创建几个对象？**

1. new关键字在堆空间创建的
2. 字符串常量池中的对象。字节码指令：ldc

**new String("a") + new String("b")呢？**

1. new Stringbuilder
2. new String("a")
3. 常量池中的 "a"
4. new String("b")
5. 常量池中的 "b"
6. toString(): new String("ab") 但是不在字符串常量池中生成 "ab"



**分析以下代码：**

```java
public class StringIntern {
    public static void main(String[] args) {
        String s = new String("1");
        s.intern(); // 调用前常量池中已有1
        String s2 = "1";
        System.out.println(s == s2); // jdk6、7、8 false
        
        String s3 = new String("1") + new String("1"); // s3变量的记录地址为 new String("11")
        // 调用前常量池中无11
        // 在常量池中生成11
        // jdk6 创建了一个新的对象11，也就有新的地址了
        // jdk7 此时常量中并没有创建11 而是创建了一个指向堆空间new String("11")的地址 
        // 为了节省空间的做法
        s3.intern();
        String s4 = "11"; // 使用的是常量池中11的地址 
        System.out.println(s3 == s4); // jdk6 fasle, jdk7/8 true
    }
}
```



**总结String的intern()的使用**

jdk1.6中， 将这个字符串对象尝试放入串池。

- 如果串池中有，则并不会放入。返回已有的串池中的对象的地址
- 如果没有，会把此**对象复制一份**，放入串池，并返回串池中的对象地址

Jdk1.7起，将这个字符串对象尝试放入串池。

- 如果串池中有，则并不会放入。返回已有的串池中的对象的地址
- 如果没有，则会把**对象的引用地址复制一份**， 放入串池，并返回串池中的引用地址

**intern()的空间效率测试**

```java
for (int i = 0; i < 10000; i++) {
	// 会不断创建new String对象
    arr[i] = new String(String.valueof(i % 10));
    // 虽然也创建了new对象，但是会被GC清理掉，更优
    arr[i] = new String(String.valueof(i % 10)).intern();
}
```

对于程序中大量存在的字符串，尤其其中存在的很多重复字符串，使用intern()可以节省内存空间



## G1的String去重操作

**背景**:对许多Java应用(有大的也有小的)做的测试得出以下结果:

- 堆存活数据集合里面String对象占了25%
- 堆存活数据集合里面重复的string对象有13.5%
- String对象的平均长度是45

许多大规模的Java应用的瓶颈在于内存，测试表明，在这些类型的应用里面，Java堆中存活的数据集合差不多25%是String对象。更进一步，这里面差不多一半string对象是重复的，重复的意思是说: string1.equals(string2)=true。堆上存在重复的String对象必然是一种内存的浪费。这个项目将在G1垃圾收集器中实现自动持续对重复的strinq对象进行去重，这样就能避免浪费内存。

**实现**

- 当垃圾收集器工作的时候，会访问堆上存活的对象。*对每一个访问的对象都会检查是否是候选的要去重的String对象*。
- 如果是，把这个对象的一个引用插入到队列中等待后续的处理。一个去重的线程在后台运行，处理这个队列。处理队列的一个元素意味着从队列删除这个元素，然后尝试去重它引用的String对象。
- 使用一个hashtable来 记录所有的被String对象使用的不重复的char数组。当去重的时候，会查这个hashtable,来看堆上是否已经存在一个一模一样的char数组。
- 如果存在，String对象会被调整引用那个数组，释放对原来的数组的引用，最终会被垃圾收集器回收掉。
- 如果查找失败，char数组会被插入到hashtable,这样以后的时候就可以共享这个数组了。

# Arrays

## copyOf

> java.util.Arrays

```java
    public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```



`Arrays.copyOf()`内部实际调用了 `System.arraycopy()` 方法

`System.arraycopy()` 需要目标数组，将原数组拷贝到自定义的数组里或者原数组，而且可以选择拷贝的起点和长度以及放入新数组中的位置 `Arrays.copyOf()` 是系统自动在内部新建一个数组，并返回该数组。

# Optional
> @since Java8

从Java8开始提供的Optional容器类，用于尽量避免空指针异常。

> java.util.Optional
```java
public final class Optional<T> {
    /**
     * 默认的空实例
     */
    private static final Optional<?> EMPTY = new Optional<>();

    /**
     * Optional容器封装的对象
     */
    private final T value;
}
```

## Optional创建
> java.util.Optional.Optional()
```java
    private Optional() {
        this.value = null;
    }
```

> java.util.Optional.Optional(T)
```java
    // 私有有参构造器，参数不能为null
    private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }
```
由于Optional私有化了空构造器，因此只能调用其提供的方法来实例化Optional类。
    
```java
    // 获取空数据的Optional容器类实例
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }

    // of方法生成Optional实例，参数不能为null
    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }
    
    // ofNullable方法生成Optional实例，参数允许为null
    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }
```

## Optional方法
## get
获取封装对象，如果为null则抛出异常

> java.util.Optional.get

```java
    public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }
```

## isPresent
判断封装对象是否为null

> java.util.Optional.isPresent

```java
    public boolean isPresent() {
        return value != null;
    }
```

```java
    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }
```

```java
    public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }
```

```java
    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }
```

```java
    public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Objects.requireNonNull(mapper.apply(value));
        }
    }
```

```java
    public T orElse(T other) {
        return value != null ? value : other;
    }
```

```java
    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }
```

```java
    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }
```