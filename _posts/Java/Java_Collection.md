# Collection

## List、Set和Map

- `List`：存储的元素是有序可重复的。
- `Set`：存储的元素是无序不可重复的。
- `Map`：使用键值对（kye-value）存储，Key 是无序不可重复的，value 是无序可重复的，每个键最多映射到一个值。



# List

## ArrayList

> java.util.ArrayList

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    /**
     * 默认初始容量大小
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 空集合对象共享的空数组
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * ArrayList大小，即包含的元素数量
     */
    private int size;
  	// ...
}
```



ArrayList底层是一个Object数组。



### 最佳实践

#### 删除元素

- 跳过fast-fail检测操作，使用普通for循环；
- Collection的removeIf方法，通过iterator操作；
- Stream的filter方式；
- 使用fail-safe的集合类。



### 源码解析

#### 构造方法

> java.util.ArrayList

```java
    /**
     * 指定初始容量参数的构造函数
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            // 初始容量大于0，创建initialCapacity大小的数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
         	  // 初始容量等于0，创建空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            // 初始容量小于0，抛出异常
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     * 默认构造函数，使用初始容量10构造一个空列表(无参数构造)
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 构造包含指定collection元素的列表，这些元素利用该集合的迭代器按顺序返回
     */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // 集合参数为空集合时，用默认空数组替换
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```



#### 属性

##### elementData

> java.util.ArrayList

```java
transient Object[] elementData;
```



该属性存储集合元素，空集合时该值为空数组，添加第一个元素时进行扩容

`transient`关键字修饰该属性是为了序列化时不将其中的空对象进行序列化，`ArrayList`通过`writeObject`和`readObject`重新实现了序列化的逻辑来只序列化集合中实际存在的元素。



#### 添加元素

> java.util.ArrayList

```java
    /*
     * 将指定的元素追加到此列表的末尾。
     */
		public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // 增加modCount!!
        elementData[size++] = e;
        return true;
    }

    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        // 集合数组为空时，比较默认容量和容量参数，返回较大值
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
      	// 直接返回容量参数
        return minCapacity;
    }
```



##### **判断是否扩容**

> java.util.ArrayList

```java
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        if (minCapacity - elementData.length > 0)
            // 调用扩容方法
            grow(minCapacity);
    }
```



- 没有指定默认容量时，添加第一个元素时，minCapacity为10，elementData.length为0，此时判断为true，进行扩容；

- 添加元素直到第11个元素时，minCapacity为11，elementData.length为10，此时再次进行扩容；
- 以此类推······



##### **扩容**

> java.util.ArrayList

```java
    /**
     * 要分配的最大数组大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

	  /**
     * ArrayList扩容的核心方法
     * 增加容量以确保它至少可以容纳最小容量参数指定的元素数量。
     */
    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        // 扩容后的容量是旧容量的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        // 检查新容量是否大于最小需要容量，若小于最小需要容量，就把最小需要容量当作数组新容量
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // 如果新容量大于 MAX_ARRAY_SIZE，则比较 minCapacity 和 MAX_ARRAY_SIZE
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        // 如果minCapacity大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为 MAX_ARRAY_SIZE 即 `Integer.MAX_VALUE - 8`
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```



- 当add第1个元素时，oldCapacity 为0，经比较后第一个if判断成立，newCapacity = minCapacity(为10)。但是第二个if判断不会成立，即newCapacity 不比 MAX_ARRAY_SIZE大，则不会进入 `hugeCapacity` 方法。数组容量为10，add方法中 return true,size增为1。
- 当add第11个元素进入grow方法时，newCapacity为15，比minCapacity（为11）大，第一个if判断不成立。新容量没有大于数组最大size，不会进入hugeCapacity方法。数组容量扩为15，add方法中return true,size增为11。
- 以此类推······



#### 删除元素

##### 删除指定元素

1. for循环匹配传入的值；
2. 调用fastRemove(index)删除数据，将index + 1及之后的元素向前移动一位，覆盖被删除值；
3. 清空最后一个元素。



##### 删除指定位置元素

1. 范围检测是否越界；
2. 取出旧数据；
3. 调用fastRemove(index)删除数据，将index + 1及之后的元素向前移动一位，覆盖被删除值；
4. 清空最后一个元素。



##### 缩容

1. 当容量为0时，重置为空数组；
2. 容量小于数组缓冲区容量，创建一个新的数组拷贝过去。



#### 手动扩容

> java.util.ArrayList

```java
    /**
     * 增加此list对象的容量，以确保它可以容纳由minimum capacity参数指定的元素数
     *
     * @param   minCapacity   the desired minimum capacity
     */
    public void ensureCapacity(int minCapacity) {
        // 数组不为默认空数组时，返回0，否则返回默认容量10
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            ? 0
            : DEFAULT_CAPACITY;
        if (minCapacity > minExpand) {
            // 执行判断是否扩容，根据需要对集合进行扩容操作
            ensureExplicitCapacity(minCapacity);
        }
    }
```



在添加大量元素前，调用该方法来减少重新扩容的次数，提高代码执行效率。



#### 转换为数组

> java.util.ArrayList

```java
    /**
     * 以正确的顺序返回一个包含此列表中所有元素的数组（从第一个到最后一个元素）; 
     */
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    /**
     * 返回的数组的运行时类型是指定数组的运行时类型。
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // 创建一个运行时类型的新数组
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
```



#### 序列化和反序列化

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

序列化该集合，将集合中数据写入流。



```java
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        int capacity = calculateCapacity(elementData, size);
        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```

反序列化该集合，从流中按顺序获取数据。



### 扩展

#### RandomAccess 接口

> java.util.RandomAccess

```java
public interface RandomAccess {
}
```



 `RandomAccess` 接口中什么都没有定义。所以该接口仅用于标识实现这个接口的类具有随机访问功能。

在 `binarySearch（)` 方法中，它要判断传入的 list 是否 `RamdomAccess` 的实例，如果是，调用`indexedBinarySearch()`方法，如果不是，那么调用`iteratorBinarySearch()`方法。



> java.util.Collections

```java
public static <T>
int binarySearch(List<? extends Comparable<? super T>> list, T key) {
    if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
        return Collections.indexedBinarySearch(list, key);
    else
        return Collections.iteratorBinarySearch(list, key);
}

private static <T>
int indexedBinarySearch(List<? extends Comparable<? super T>> list, T key) {
    int low = 0;
    int high = list.size()-1;

    while (low <= high) {
        int mid = (low + high) >>> 1;
        Comparable<? super T> midVal = list.get(mid);
        int cmp = midVal.compareTo(key);

        if (cmp < 0)
            low = mid + 1;
        else if (cmp > 0)
            high = mid - 1;
        else
            return mid; // key found
    }
    return -(low + 1);  // key not found
}

private static <T>
int iteratorBinarySearch(List<? extends Comparable<? super T>> list, T key)
{
    int low = 0;
    int high = list.size()-1;
    ListIterator<? extends Comparable<? super T>> i = list.listIterator();

    while (low <= high) {
        int mid = (low + high) >>> 1;
        Comparable<? super T> midVal = get(i, mid);
        int cmp = midVal.compareTo(key);

        if (cmp < 0)
            low = mid + 1;
        else if (cmp > 0)
            high = mid - 1;
        else
            return mid; // key found
    }
    return -(low + 1);  // key not found
}
```



`ArrayList` 实现了 `RandomAccess` 接口， 而 `LinkedList` 没有实现。ArrayList` 底层是数组，而 `LinkedList` 底层是链表。数组天然支持随机访问，时间复杂度为 O(1)，所以称为快速随机访问。链表需要遍历到特定位置才能访问特定位置的元素，时间复杂度为 O(n)，所以不支持快速随机访问。`ArrayList` 实现了 `RandomAccess` 接口，就表明了他具有快速随机访问功能。 `RandomAccess` 接口只是标识，并不是说 `ArrayList` 实现 `RandomAccess` 接口才具有快速随机访问功能的！



## LinkedList

> java.util.LinkedList

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;
}
```



LinkedList 底层是一个双向链表。



### 源码解析

#### 获取元素

> java.util.LinkedList

```java
    public E get(int index) {
      	// 首先判断下标是否为负数或越界
        checkElementIndex(index);
        return node(index).item;
    }

    private void checkElementIndex(int index) {
        if (!isElementIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private boolean isElementIndex(int index) {
        return index >= 0 && index < size;
    }

    Node<E> node(int index) {
      	// 判断下标属于前半部分还是后半部分，决定是从前往后遍历还是从后往前遍历
        if (index < (size >> 1)) {
          	// 从前往后遍历
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
          	// 从后往前遍历
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }

```



### ArrayList和LinkedList比较

1. **是否保证线程安全：** `ArrayList` 和 `LinkedList` 都是不同步的，也就是不保证线程安全；
2. **底层数据结构：** `Arraylist` 底层使用的是 **`Object` 数组**；`LinkedList` 底层使用的是 **双向链表** 数据结构（JDK1.6 之前为循环链表，JDK1.7 取消了循环。）
3. **插入和删除是否受元素位置的影响：** ① **`ArrayList` 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。** 比如：执行`add(E e)`方法的时候， `ArrayList` 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 O(1)。但是如果要在指定位置 i 插入和删除元素的话（`add(int index, E element)`）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。 ② **`LinkedList` 采用链表存储，所以对于`add(E e)`方法的插入，删除元素时间复杂度不受元素位置的影响，近似 O(1)，如果是要在指定位置`i`插入和删除元素的话（`(add(int index, E element)`） 时间复杂度近似为`o(n))`因为需要先移动到指定位置再插入。**
4. **是否支持快速随机访问：** `LinkedList` 不支持高效的随机元素访问，而 `ArrayList` 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于`get(int index)`方法)。
5. **内存空间占用：** ArrayList 的空 间浪费主要体现在在 list 列表的结尾会预留一定的容量空间，而 LinkedList 的空间花费则体现在它的每一个元素都需要消耗比 ArrayList 更多的空间（因为要存放直接后继和直接前驱以及数据）。



## Vector

`ArrayList` 是 `List` 的主要实现类，底层使用 `Object[ ]`存储，适用于频繁的查找工作，线程不安全 ；`Vector` 是 `List` 的古老实现类，底层使用` Object[ ]` 存储，线程安全的。



# Map

![在这里插入图片描述](../../Image/2022/08/220801-2.png)



Map的Key必须要是不可变的，如果Key对象的哈希值发生变化，Map对象很可能就定位不到映射的位置了。



## HashMap

### 概念

JDK7 HashMap结构为数组+链表（发生元素碰撞时，会将新元素添加到链表开头）

JDK8 HashMap结构为数组+链表+红黑树（发生元素碰撞时，会将新元素添加到链表末尾，当HashMap总容量大于等于64，并且某个链表的大小大于等于8，会将链表转化为红黑树（注意：红黑树是二叉树的一种））

它根据键的hashCode值存储数据，大多数情况下可以直接定位到它的值，因而具有很快的访问速度，但遍历顺序却是不确定的。



**如果需要满足线程安全**，可以用：

- Collections的synchronizedMap方法使HashMap具有线程安全的能力，
- 或者使用ConcurrentHashMap。



#### 特点

- HashMap采用了链地址法解决冲突；
- HashMap有较好的Hash算法和扩容机制；
- HashMap 最多只允许一条记录的Key为null，允许多条记录的Value为null；
- HashMap非线程安全，任一时刻可以有多个线程同时写HashMap，可能会导致数据不一致。



### 最佳实践

- 重写hashCode和equals方法

重写 key 的 hashCode 方法，降低哈希冲突，从而减少链表的产生，高效利用哈希表，达到提高性能的效果。



- 配置初始容量和负载因子

结合实际场景来设置初始容量和加载因子两个参数。当查询操作较为频繁时，可以适当地减少加载因子；如果对内存利用率要求比较高，可以适当的增加加载因子。

在预知存储数据量的情况下，提前设置初始容量（初始容量=预知数据量/加载因子），这样做的好处是可以减少 resize() 操作，提高 HashMap 的效率；



### 源码解析

#### 常量

> java.util.HashMap

```java
    /**
     * 默认初始化数组容量
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * 最大数组容量
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 默认负载因子
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * 最小红黑树化链表大小
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * 最小解除红黑树化时树的元素数量
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * 最小红黑树化数组大小.
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
```



#### 属性

> java.util.HashMap

```java
    /**
     * 哈希数组，哈希槽位数组，table桶数组，散列表，数组中的一个元素被称之为一个槽位slot
     */
    transient Node<K,V>[] table;

    /**
     * 
     */
    transient Set<Map.Entry<K,V>> entrySet;

    /**
     * 大小
     */
    transient int size;

    /**
     * 结构性修改次数
     */
    transient int modCount;

    /**
     * 边界值
     */
    int threshold;

    /**
     * 加载因子
     */
    final float loadFactor;
```



##### loadFactor

loadFactor 也是可以调整的，默认是0.75，如果 loadFactor 越大，在数组定义好 length 长度之后，所能容纳的键值对个数越多。

loadFactor 越大，对空间的利用就越充分，碰撞的机会越高，这就意味着链表的长度越长，查找效率也就越低。如果 loadFactor 太小，那么哈希表的数据将过于稀疏，对空间造成严重浪费，但是碰撞的机会越低， 查找的效率就搞，性能就越好。

默认的负载因子0.75是对空间和时间效率的一个平衡选择，一般不修改，除非在时间和空间比较特殊的情况下。分为两种情况：

- 如果内存空间很多而又对时间效率要求很高，可以降低负载因子Load factor的值；
- 相反，如果内存空间紧张而对时间效率要求不高，可以增加负载因子loadFactor的值，这个值可以大于1。



##### threshold

threshold是HashMap所能容纳的最大数据量的Node 个数，Node[] table的初始化长度length(默认值是16)。

**threshold = length * Load factor**，默认情况下 threshold = 16 * 0.75 =12。

threshold就是允许的哈希数组 最大元素数目，超过这个数目就重新resize(扩容)，扩容后的哈希数组 容量length 是之前容量length 的两倍。

threshold是通过初始容量和LoadFactor计算所得，在初始HashMap不设置参数的情况下，默认边界值为12。

如果HashMap中Node的数量超过边界值，HashMap就会调用resize()方法重新分配table数组。

这将会导致HashMap的数组复制，迁移到另一块内存中去，从而影响HashMap的效率。



##### size

size就是HashMap中实际存在的键值对数量。**size和table的长度length的区别**在于length是 哈希桶数组table的长度。



##### table

在HashMap中，哈希桶数组table的长度length大小必须为2的n次方，这是一定是一个合数，这是一种反常规的设计。



> 常规的设计是把桶数组的大小设计为素数，相对来说素数导致冲突的概率要小于合数。比如，Hashtable初始化桶大小为11，就是桶大小设计为素数的应用（Hashtable扩容后不能保证还是素数）。



HashMap采用这种非常规设计，主要是为了方便扩容。而 HashMap为了减少冲突，采用另外的方法规避：计算哈希桶索引位置时，哈希值的高位参与运算。



> java.util.HashMap.Node

```java
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```



Node类作为HashMap中的一个内部类，每个 Node 包含了一个 key-value 键值对。还定义了一个next 指针，通过链地址法解决哈希冲突。

当有哈希冲突时，HashMap 会用之前数组当中相同哈希值对应存储的 Node 对象，通过指针指向新增的相同哈希值的 Node 对象的引用。



##### modCount

表示此哈希表已被**结构性修改**的次数，**结构性修改**是指哈希表的内部结构被修改，比如桶数组被修改或者拉链被修改。

那些更改桶数组或者拉链的操作如，重新哈希。 此字段用于HashMap集合迭代器的快速失败（Fail-Fast）。



> Fail-Fast：在可能出现错误的情况下提前抛出异常终止操作。



只有当对HashMap元素个数产生影响的时候才会修改modCount，如添加不存在的Key键值对和移除键值对时。

用迭代器遍历HashMap的时调用HashMap.remove方法后，判断modCount和expectedModCount是否一致，如果不一致则会产**并发修改的异常**ConcurrentModificationException。



#### 构造方法

> java.util.HashMap

```java
    /**
     * 指定初始容量和加载因子的构造方法
     */
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }

    /**
     * 指定初始容量的构造方法
     */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * 空构造方法
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }

    /**
     * 
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
```



##### 设置table容量大小

> java.util.HashMap

```java
    /**
     *    通过5次无符号移位运算以及或运算得到：
     *    n第一次右移一位时，相当于将最高位的1右移一位，再和原来的n取或，
     *    就将最高位和次高位都变成1，也就是两个1；
     *    第二次右移两位时，将最高的两个1向右移了两位，取或后得到四个1；
     *    依次类推，右移16位再取或就能得到32个1；
     *    最后通过加一进位得到2^n。
     *
     *    比如initialCapacity = 10，那就返回16，initialCapacity = 17，那么就返回32
     *    10的二进制是1010，减1就是1001
     *    第一次右移取或： 1001 | 0100 = 1101 ；
     *    第二次右移取或： 1101 | 0011 = 1111 ；
     *    第三次右移取或： 1111 | 0000 = 1111 ；
     *    第四次第五次同理
     *    最后得到 n = 1111，返回值是 n+1 = 2 ^ 4 = 16 ;
     */
    static final int tableSizeFor(int cap) {
        // 使目标值大于或等于原值，如果cap已经是2的幂，不进行该操作则目标值将会是cap的2倍
        // 如cap为十进制8时，不进行 -1 操作的结果为16；
        // 减1后二进制为111，再进行操作则会得到原来的数值1000，即8。
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```



由于HashMap的capacity都是2的幂，因此这个方法用于找到大于等于initialCapacity的最小的2的幂（initialCapacity如果就是2的幂，则返回的还是这个数），保证 HashMap 总是使用2的幂作为哈希表的大小。



通过构造器创建HashMap时赋值threshold，不符合算式capacity * load factor，这是因为构造方法中，并没有对table这个成员变量进行初始化，table的初始化被推迟到了put方法中，在put方法中会对threshold重新计算。

在put时，会对table进行初始化，如果threshold大于0，会把threshold当作数组的长度进行table的初始化，否则创建的table的长度为16。



#### 查询元素
##### get
> java.util.HashMap
```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

当 HashMap 只存在数组，而数组中没有 Node 链表时，是 HashMap 查询数据性能最好的时候。一旦发生大量的哈希冲突，就会产生 Node 链表，这个时候每次查询元素都可能遍历 Node 链表，从而降低查询数据的性能。

特别是在链表长度过长的情况下，性能明显下降，**使用红黑树**就很好地解决了这个问题，红黑树使得查询的平均复杂度降低到了 **O(log(n))**，链表越长，使用红黑树替换后的查询效率提升就越明显。



> java.util.HashMap.getNode
```java
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 数组不为null，数组长度大于0，
    // 通过 hash & (table.length - 1)获取该key对应的数据节点的hash槽的元素不为null;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash &&
            ((k = first.key) == key || (key != null && key.equals(k))))
            // 查找的元素在数组中或者链表的第一个元素，返回该元素
            return first;
        if ((e = first.next) != null) {
            // 查找的元素在链表非首节点或红黑树中
            // 首节点是树形节点, 则进入红黑树数的取值流程, 并返回结果;
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 进入链表的取值流程, 遍历链表，元素在链表中，返回该元素;
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```



![img](../../Image/2022/08/220801-6.png)



###### 从红黑树中取值

> java.util.HashMap.TreeNode.getTreeNode
```java
final TreeNode<K,V> getTreeNode(int h, Object k) {
    return ((parent != null) ? root() : this).find(h, k, null);
}
```



> java.util.HashMap.TreeNode.root
```java
final TreeNode<K,V> root() {
    for (TreeNode<K,V> r = this, p;;) {
        if ((p = r.parent) == null)
            return r;
        r = p;
    }
}
```



> java.util.HashMap.TreeNode.find
```java
final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
    TreeNode<K,V> p = this;
    do {
        int ph, dir; K pk;
        TreeNode<K,V> pl = p.left, pr = p.right, q;
        if ((ph = p.hash) > h)
            p = pl;
        else if (ph < h)
            p = pr;
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
        else if (pl == null)
            p = pr;
        else if (pr == null)
            p = pl;
        else if ((kc != null ||
                  (kc = comparableClassFor(k)) != null) &&
                 (dir = compareComparables(kc, k, pk)) != 0)
            p = (dir < 0) ? pl : pr;
        else if ((q = pr.find(h, k, kc)) != null)
            return q;
        else
            p = pl;
    } while (p != null);
    return null;
}
```



##### containsKey

> java.util.HashMap

```java
    public boolean containsKey(Object key) {
        return getNode(hash(key), key) != null;
    }
```



查找对应Key的键值对元素存在，即表明该Key存在。



##### containsValue

> java.util.HashMap

```java
    public boolean containsValue(Object value) {
        Node<K,V>[] tab; V v;
         // 数组不为null并且长度大于0
        if ((tab = table) != null && size > 0) {
            for (int i = 0; i < tab.length; ++i) {
                // 对数组进行遍历
                for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                    // 当前节点的值等价查找的值，返回true
                    if ((v = e.value) == value ||
                        (value != null && value.equals(v)))
                        return true;
                }
            }
        }
        return false;
    }
```



##### 参考资料

- [HashMap之get方法详解](https://blog.csdn.net/weixin_39667787/article/details/86687414)



#### 添加元素

##### put
设置指定key的value值并返回旧值，如果已存在则替换旧值

> java.util.HashMap
```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```



- 首先会根据该 key 的 hashCode() 返回值，再通过 hash() 方法计算出 hash 值；
- 再**除留余数法**，取得余数，这里通过位运算来完成。(n-1) & hash 就是 hash值除以n留余数， n 代表哈希表的长度，余数 (n-1) & hash 决定该 Node 的存储位置。



把对象加入`HashMap`时，`HashMap` 会先计算对象的`hashcode`值来判断对象加入的位置，同时也会与其他加入的对象的 `hashcode` 值作比较，如果没有相符的 `hashcode`，如果发现有相同 `hashcode` 值的对象，这时会调用`equals()`方法来检查 `hashcode` 相等的对象是否真的相同。如果两者相同，在根据当前 `hashcode` 位置的值数量决定是存储为链表还是红黑树。

![在这里插入图片描述](../../Image/2022/09/220927-1.jpg)



###### 计算哈希值

> java.util.HashMap

```java
static final int hash(Object key) {
    int h;
    // 通过扰动函数处理（减少碰撞）过后得到 hash 值
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

- h >>>  16，表示无符号右移16位，高位补0，任何数跟0异或都是其本身，因此key的hash值高16位不变。
- 异或的运算法则为：0^0=0，1^0=1，0^1=1，1^1=0（同为0，异为1）



![img](../../Image/2022/08/220801-3.png)



即取 int 类型的一半，刚好可以将该二进制数对半切开，利用异或运算（如果两个数对应的位置相反，则结果为1，反之为0），这样可以避免哈希冲突。

底16位与高16位异或，其目标**尽量打乱 hashCode 真正参与运算的低16位，减少hash 碰撞**。之所以要无符号右移16位，是跟table的下标有关，位置计算方式是：（n-1)&hash 计算 Node 的存储位置。

**假如n=16，**从下图可以看出table的下标仅与hash值的低n位有关，hash值的高位都被与操作置为0了，只有hash值的低4位参与了运算。

![img](../../Image/2022/08/220801-4.png)



hashCode值为什么需要 **高16位 ^ 低16位** 主要为了优化hash算法，近可能的分散得比较均匀，尽可能的减少碰撞。

因为hashmap内部散列表，它大多数场景下，它不会特别大。

hashmap内部散列表的长度，也就是说 length - 1 对应的 二进制数，实际有效位很有限，一般都在（低）16位以内，这样的话，key的hash值高16位就等于完全浪费了，没起到作用。

所以，node的hash字段才采用了 **高16位 ^ 低16位** 这种方式来增加随机的概率，近可能的分散得比较均匀，尽可能的减少碰撞。



###### 添加键值对

> java.util.HashMap

```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // n 指的是数组的长度
        if ((tab = table) == null || (n = tab.length) == 0)
            // 通过 resize 方法得到初始化的table
            n = (tab = resize()).length;
        // 通过 (n - 1) & hash 判断当前元素存放的位置
        if ((p = tab[i = (n - 1) & hash]) == null)
            // Node不在哈希表中(链表的第一个节点位置），新增一个Node，并加入到哈希表中
            tab[i] = newNode(hash, key, value, null);
        else {
            //hash冲突了
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                // 通过equals方法比较当前位置的key相同，则覆盖
                e = p;
            else if (p instanceof TreeNode)
                // 添加红黑树节点
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                // 循环，直到链表中的某个节点为null，
                // 或者某个节点hash值和给定的hash值一致且key也相同，则停止循环。
                // binCount是一个计数器，来计算当前链表的元素个数
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        // next为空，将添加的元素置为next
                        p.next = newNode(hash, key, value, null);
                        // 插入成功后，要判断是否需要转换为红黑树，
                        // 因为插入后链表长度+1，而binCount并不包含新节点，
                        // 所以判断时要将临界阀值-1.
                        if (binCount >= TREEIFY_THRESHOLD - 1)
                            // 链表长度大于阈值（默认为8）时，
                            // 且数组长度小于64，那么就重新散列resize()
                            // 如果数组长度大于64，将链表转换为红黑树
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        // 节点hash值和给定的hash值一致且key也相同，停止循环
                        break;
                    // 如果给定的hash值不同或者key不同。将next值赋给p，
                    // 为下次循环做铺垫。即结束当前节点，对下一节点进行判断
                    p = e;
                }
            }
            // 如果e不是null，该元素存在了(也就是key相等)
            if (e != null) { 
                // 取出该元素的值
                V oldValue = e.value;
                // 如果 onlyIfAbsent 是 true，就不用改变已有的值；
                // 如果是false(默认)，或者value是null，将新的值替换老的值
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        // 对于之前不存在的key进行put的时候，对modCount有修改，为迭代服务
        ++modCount;
        // 达到了边界值，需要扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```



当链表长度太长（默认超过 8）时，链表就进行转换红黑树的操作。这里利用**红黑树快速增删改查**的特点，提高 HashMap 的性能。

当红黑树结点个数少于 6 个的时候，又会将红黑树转化为链表。因为在数据量较小的情况下，红黑树要维护平衡，比起链表来，性能上的优势并不明显。

![img](../../Image/2022/08/220801-5.png)



###### 扩容

> java.util.HashMap

```java
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        // 旧阀值
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            // 数组已经存在不需要进行初始化
            if (oldCap >= MAXIMUM_CAPACITY) {
                // 旧数组容量超过最大容量限制，不扩容直接返回旧数组
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 进行2倍扩容后的新数组容量小于最大容量和旧数组长度大于等于16
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                // 重新计算阀值为原来的2倍
                newThr = oldThr << 1;
        }
        // 初始化数组
        else if (oldThr > 0)
            // 有阀值，初始容量的值为阀值
            newCap = oldThr;
        else {     
            // 没有阀值，初始化的默认容量，重新计算阀值
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            // 有阀值，定义了新数组的容量，重新计算阀值
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        // 赋予新阀值
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        // 如果旧数组有数据，进行数据移动，如果没有数据，返回一个空数组
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    // 将旧数组的所属位置的旧元素清空
                    oldTab[j] = null;
                    if (e.next == null)
                        // 当前节点是在数组上，后面没有链表，重新计算槽位
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        // 当前节点是红黑树，红黑树重定位
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { 
                        // 当前节点是链表
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                // 不需要移位
                                if (loTail == null)
                                    // 头节点是空的，头节点放置当前遍历到的元素
                                    loHead = e;
                                else
                                    // 当前元素放到尾节点的后面
                                    loTail.next = e;
                                // 尾节点重置为当前元素
                                loTail = e;
                            }
                            else {
                                // 需要移位
                                if (hiTail == null)
                                    // 头节点是空的，头节点放置当前遍历到的元素
                                    hiHead = e;
                                else
                                    // 当前元素放到尾节点的后面
                                    hiTail.next = e;
                                // 尾节点重置为当前元素
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            // 不需要移位
                            loTail.next = null;
                            // 原位置
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            // 移动到当前hash槽位 + oldCap的位置，
                            // 即在原位置再移动2次幂的位置
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```



![img](../../Image/2022/08/220801-9.png)



- 当前节点是数组，后面没有链表，重新计算槽位，位与操作的效率比效率高

```
     定位槽位：e.hash & (newCap - 1)，用长度16, 待插入节点的hash值为21举例:
     (1)取余: 21 % 16 = 5
     (2)位与:
     21: 0001 0101
             &
     15: 0000 1111
     5:  0000 0101
```



- 遍历链表，对链表节点进行移位判断：(e.hash & oldCap) == 0

```
      比如oldCap=8,hash是3，11，19，27时，
     （1）JDK1.8中(e.hash & oldCap)的结果是0，8，0，8，这样3，19组成新的链表，index为3；而11，27组成新的链表，新分配的index为3+8；
     （2）JDK1.7中是(e.hash & newCap-1)，newCap是oldCap的两倍，也就是3，11，19，27对(16-1)与计算，也是0，8，0，8，但由于是使用了单链表的头插入方式，即同一位置上新元素总会被放在链表的头部位置；这样先放在一个索引上的元素终会被放到Entry链的尾部(如果发生了hash冲突的话），这样index为3的链表是19，3，index为3+8的链表是 27，11。
     也就是说1.7中经过resize后数据的顺序变成了倒叙，而1.8没有改变顺序。
```



**扩容时元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。**

由于扩容数组的长度是2倍关系，所以对于假设初始 tableSize=4 要扩容到8来说就是 0100 到 1000 的变化（左移一位就是2倍），在扩容中只用判断原来的 hash 值和 oldCap（旧数组容量）按位与操作是 0 或 1 就行:

- 0的话索引不变，
- 1的话索引变成原索引加扩容前数组。



之所以能通过这种“与”运算来重新分配索引，是因为 hash 值本来是随机的，而 hash 按位与上 oldCap 得到的 0 和 1 也是随机的，所以扩容的过程就能把之前哈希冲突的元素再随机分布到不同的索引中去。



###### 链表转红黑树

> java.util.HashMap

```java
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
          	// 转成红黑树前会判断当前数组的长度若小于 64，则先进行数组扩容，而不是转为红黑树
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
```



##### putAll

> java.util.HashMap

```java
    public void putAll(Map<? extends K, ? extends V> m) {
        putMapEntries(m, true);
    }

    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
            if (table == null) {
                // 数组为null则进行初始化
                float ft = ((float)s / loadFactor) + 1.0F;
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
            else if (s > threshold)
                // 容量达到了边界值，比如插入的m的定义容量是16，
                // 但当前map的边界值是12，需要对当前map进行重新计算边界值
                resize();
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                // 遍历放到当前map中
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }
```

![img](../../Image/2022/08/220801-8.png)

##### putIfAbsent

设置指定key的value值并返回旧值，如果已存在则不替换

> java.util.HashMap.putIfAbsent
```java
@Override
public V putIfAbsent(K key, V value) {
    return putVal(hash(key), key, value, true, true);
}
```



> java.util.HashMap.putVal
```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        // 如果当前key没有value，则设置value
        tab[i] = newNode(hash, key, value, null);
    else {
        // 当前key有value，则替换当前value并返回旧的value
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) {
            // 存在key的映射
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                // 判断是否将新value替换旧value
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    
    // 新增key时，计算是否要扩容并返回null
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```



##### compute

设置指定key的value值并返回新值，如果已存在则替换旧值

> java.util.HashMap.compute
```java
@Override
public V compute(K key,
                 BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    if (remappingFunction == null)
        throw new NullPointerException();
    int hash = hash(key);
    Node<K,V>[] tab; Node<K,V> first; int n, i;
    int binCount = 0;
    TreeNode<K,V> t = null;
    Node<K,V> old = null;
    if (size > threshold || (tab = table) == null ||
        (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((first = tab[i = (n - 1) & hash]) != null) {
        if (first instanceof TreeNode)
            old = (t = (TreeNode<K,V>)first).getTreeNode(hash, key);
        else {
            Node<K,V> e = first; K k;
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k)))) {
                    old = e;
                    break;
                }
                ++binCount;
            } while ((e = e.next) != null);
        }
    }
    V oldValue = (old == null) ? null : old.value;
    V v = remappingFunction.apply(key, oldValue);
    if (old != null) {
        if (v != null) {
            old.value = v;
            afterNodeAccess(old);
        }
        else
            removeNode(hash, key, null, false, true);
    }
    else if (v != null) {
        if (t != null)
            t.putTreeVal(this, tab, hash, key, v);
        else {
            tab[i] = newNode(hash, key, v, first);
            if (binCount >= TREEIFY_THRESHOLD - 1)
                treeifyBin(tab, hash);
        }
        ++modCount;
        ++size;
        afterNodeInsertion(true);
    }
    return v;
}
```



##### computeIfAbsent

设置指定key的value值并返回新值，如果已存在则不替换

> java.util.HashMap.computeIfAbsent
```java
@Override
public V computeIfAbsent(K key,
                         Function<? super K, ? extends V> mappingFunction) {
    if (mappingFunction == null)
        throw new NullPointerException();
    int hash = hash(key);
    Node<K,V>[] tab; Node<K,V> first; int n, i;
    int binCount = 0;
    TreeNode<K,V> t = null;
    Node<K,V> old = null;
    if (size > threshold || (tab = table) == null ||
        (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((first = tab[i = (n - 1) & hash]) != null) {
        if (first instanceof TreeNode)
            old = (t = (TreeNode<K,V>)first).getTreeNode(hash, key);
        else {
            Node<K,V> e = first; K k;
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k)))) {
                    old = e;
                    break;
                }
                ++binCount;
            } while ((e = e.next) != null);
        }
        V oldValue;
        if (old != null && (oldValue = old.value) != null) {
            afterNodeAccess(old);
            return oldValue;
        }
    }
    V v = mappingFunction.apply(key);
    if (v == null) {
        return null;
    } else if (old != null) {
        old.value = v;
        afterNodeAccess(old);
        return v;
    }
    else if (t != null)
        t.putTreeVal(this, tab, hash, key, v);
    else {
        tab[i] = newNode(hash, key, v, first);
        if (binCount >= TREEIFY_THRESHOLD - 1)
            treeifyBin(tab, hash);
    }
    ++modCount;
    ++size;
    afterNodeInsertion(true);
    return v;
}
```



##### computeIfPresent

> java.util.HashMap.computeIfPresent

```java
public V computeIfPresent(K key,
                          BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    if (remappingFunction == null)
        throw new NullPointerException();
    Node<K,V> e; V oldValue;
    int hash = hash(key);
    if ((e = getNode(hash, key)) != null &&
        (oldValue = e.value) != null) {
        V v = remappingFunction.apply(key, oldValue);
        if (v != null) {
            e.value = v;
            afterNodeAccess(e);
            return v;
        }
        else
            removeNode(hash, key, null, false, true);
    }
    return null;
}
```



##### 参考资料

- [JAVA8 Map新方法：compute，computeIfAbsent，putIfAbsent与put的区别](https://blog.csdn.net/wang_8101/article/details/82191146)



#### 更新元素

##### replace

> java.util.HashMap

```java
    @Override
    public V replace(K key, V value) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) != null) {
            // 根据hash计算得到槽位的节点不为null
            V oldValue = e.value;
            // 覆盖旧值并返回旧值
            e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
        return null;
    }

    @Override
    public boolean replace(K key, V oldValue, V newValue) {
        Node<K,V> e; V v;
        if ((e = getNode(hash(key), key)) != null &&
            ((v = e.value) == oldValue || (v != null && v.equals(oldValue)))) {
            // 根据hash计算得到槽位的节点不为null，并且节点的值等于旧值，然后覆盖旧值
            e.value = newValue;
            afterNodeAccess(e);
            return true;
        }
        return false;
    }
```



#### 移除元素

##### remove

> java.util.HashMap

```java
    public V remove(Object key) {
        Node<K,V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
    }

    @Override
    public boolean remove(Object key, Object value) {
        return removeNode(hash(key), key, value, true, true) != null;
    }

    final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        // 数组不为null，数组长度大于0，要删除的元素计算的槽位有元素
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
            Node<K,V> node = null, e; K k; V v;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                // 当前元素在数组中
                node = p;
            else if ((e = p.next) != null) {
                // 元素在红黑树或链表中
                if (p instanceof TreeNode)
                    // 是树节点，从树种查找节点
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                    //在链表中遍历查找hash相同，并且key相同，找到节点并结束
                    do {
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            // 找到节点了，并且值也相同
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    // 是树节点，从树中移除
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                else if (node == p)
                    // 节点在数组中，当前槽位置为null，node.next为null
                    tab[index] = node.next;
                else
                    // 节点在链表中, 将节点删除
                    p.next = node.next;
                // 进行了modCount自增操作，更新结构型修改次数，为迭代服务
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
```



![img](../../Image/2022/08/220801-7.png)



##### clear

> java.util.HashMap

```java
    public void clear() {
        Node<K,V>[] tab;
        modCount++;
        if ((tab = table) != null && size > 0) {
            // 将数组的元素格式置为0，然后遍历数组，将每个槽位的元素置为null
            size = 0;
            for (int i = 0; i < tab.length; ++i)
                tab[i] = null;
        }
    }
```



#### 遍历移除元素

> java.util.HashMap.HashIterator

```java
    abstract class HashIterator {
        Node<K,V> next;        // next entry to return
        Node<K,V> current;     // current entry
        int expectedModCount;  // for fast-fail
        int index;             // current slot

        HashIterator() {
            expectedModCount = modCount;
            Node<K,V>[] t = table;
            current = next = null;
            index = 0;
            if (t != null && size > 0) { // advance to first entry
                do {} while (index < t.length && (next = t[index++]) == null);
            }
        }

        public final boolean hasNext() {
            return next != null;
        }

        final Node<K,V> nextNode() {
            Node<K,V>[] t;
            Node<K,V> e = next;
            // 比较两者是否一致，不一致抛出异常
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            if (e == null)
                throw new NoSuchElementException();
            if ((next = (current = e).next) == null && (t = table) != null) {
                do {} while (index < t.length && (next = t[index++]) == null);
            }
            return e;
        }

        public final void remove() {
            Node<K,V> p = current;
            if (p == null)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            current = null;
            K key = p.key;
            removeNode(hash(key), key, null, false, false);
            // 迭代器的remove方法通过重新赋值expectedModCount来保证不会抛出异常
            expectedModCount = modCount;
        }
    }
    
    final class KeyIterator extends HashIterator
        implements Iterator<K> {
        public final K next() { return nextNode().key; }
    }

    final class ValueIterator extends HashIterator
        implements Iterator<V> {
        public final V next() { return nextNode().value; }
    }

    final class EntryIterator extends HashIterator
        implements Iterator<Map.Entry<K,V>> {
        public final Map.Entry<K,V> next() { return nextNode(); }
    }
```



通过上述源码可知，如果在遍历过程中调用了HashMap的remove方法，会修改modCount的值，导致和expectedModCount值不一致而抛出异常。此时如果要移除键值对，应该调用迭代器的remove方法来移除，避免抛出异常。



### 扩展

#### 与Hashtable的区别

1. **线程是否安全：** `HashMap` 是非线程安全的，`HashTable` 是线程安全的,因为 `HashTable` 内部的方法基本都经过`synchronized` 修饰。（如果你要保证线程安全的话就使用 `ConcurrentHashMap` 吧！）；
2. **效率：** 因为线程安全的问题，`HashMap` 要比 `HashTable` 效率高一点。另外，`HashTable` 基本被淘汰，不要在代码中使用它；
3. **对 Null key 和 Null value 的支持：** `HashMap` 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；HashTable 不允许有 null 键和 null 值，否则会抛出 `NullPointerException`。
4. **初始容量大小和每次扩充容量大小的不同 ：** ① 创建时如果不指定容量初始值，`Hashtable` 默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1。`HashMap` 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍。② 创建时如果给定了容量初始值，那么 Hashtable 会直接使用你给定的大小，而 `HashMap` 会将其扩充为 2 的幂次方大小（`HashMap` 中的`tableSizeFor()`方法保证，下面给出了源代码）。也就是说 `HashMap` 总是使用 2 的幂作为哈希表的大小,后面会介绍到为什么是 2 的幂次方。
5. **底层数据结构：** JDK1.8 以后的 `HashMap` 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。Hashtable 没有这样的机制。



#### HashMap 的长度为什么是2的幂次方

为了能让 HashMap 存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀。Hash 值的范围值-2147483648到2147483647，加起来大概40亿的映射空间，只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个40亿长度的数组，内存是放不下的。所以这个散列值是不能直接拿来用的。用之前还要先做对数组的长度取模运算，得到的余数才能用来要存放的位置也就是对应的数组下标。这个数组下标的计算方法是“ `(n - 1) & hash`”。（n代表数组长度）。这也就解释了 HashMap 的长度为什么是2的幂次方。

首先可能会想到采用%取余的操作来实现。但是，重点来了：**“取余(%)操作中如果除数是2的幂次则等价于与其除数减一的与(&)操作（也就是说 hash%length==hash&(length-1)的前提是 length 是2的 n 次方；）。”** 并且 **采用二进制位操作 &，相对于%能够提高运算效率，这就解释了 HashMap 的长度为什么是2的幂次方。**



#### HashMap 多线程操作导致死循环问题

主要原因在于 并发下的Rehash 会造成元素之间会形成一个循环链表。不过，jdk 1.8 后解决了这个问题，但是还是不建议在多线程下使用 HashMap,因为多线程下使用 HashMap 还是会存在其他问题比如数据丢失。并发环境下推荐使用 ConcurrentHashMap 。



详情请查看：https://coolshell.cn/articles/9606.html



**JDK8 HashMap重排序**

如果删除了HashMap中红黑树的某个元素导致元素重排序时，不需要计算待重排序的元素的HashCode码，只需要将当前元素放到HashMap总长度+当前元素在HashMap中的位置的位置即可。



## LinkedHashMap

`LinkedHashMap` 继承自 `HashMap`，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，`LinkedHashMap` 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。

LinkedHashMap是HashMap的一个子类，其优点在于： **保存了记录的插入顺序**，

在用Iterator遍历LinkedHashMap时，**先得到的记录肯定是先插入的**，也可以在构造时带参数，按照访问次序排序。



详细可以查看：[《LinkedHashMap 源码详细分析（JDK1.8）》](https://www.imooc.com/article/22931)



## Hashtable

### 概念

数组+链表组成的，数组是 `HashMap` 的主体，链表则是主要为了解决哈希冲突而存在的。

Hashtable是遗留类，很多映射的常用功能与HashMap类似，不同的是它承自Dictionary类，并且是**线程安全**的。任一时间只有一个线程能写Hashtable，并发性不如ConcurrentHashMap。后者使用了 分段保护机制，也就是 分而治之的思想。

这个是老古董，Hashtable**不建议在代码中使用**，

不需要线程安全的场合可以用HashMap替换，需要线程安全的场合可以用ConcurrentHashMap替换。



#### 特点

- Hashtable不允许Key为NULL，会报空指针异常；



## TreeMap

TreeMap实现SortedMap接口，能够把它保存的记录根据键排序，**默认是按键值的升序排序**，也可以**指定排序的比较器**，

当用Iterator遍历TreeMap时，得到的记录是排过序的。

如果使用**排序的映射**，建议使用TreeMap。

在使用TreeMap时，**key必须实现Comparable接口**, 或者在构造TreeMap传入自定义的Comparator，

否则会在运行时抛出java.lang.ClassCastException类型的异常。



## ConcurrentHashMap

### 扩展

#### ConcurrentHashMap 和 Hashtable 的区别

- **底层数据结构：** JDK1.7 的 `ConcurrentHashMap` 底层采用 **分段的数组+链表** 实现，JDK1.8 采用的数据结构跟 `HashMap1.8` 的结构一样，数组+链表/红黑二叉树。`Hashtable` 和 JDK1.8 之前的 `HashMap` 的底层数据结构类似都是采用 **数组+链表** 的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的；
- **实现线程安全的方式（重要）：** ① **在 JDK1.7 的时候，`ConcurrentHashMap`（分段锁）** 对整个桶数组进行了分割分段(`Segment`)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 **到了 JDK1.8 的时候已经摒弃了 `Segment` 的概念，而是直接用 `Node` 数组+链表+红黑树的数据结构来实现，并发控制使用 `synchronized` 和 CAS 来操作。（JDK1.6 以后 对 `synchronized` 锁做了很多优化）** 整个看起来就像是优化过且线程安全的 `HashMap`，虽然在 JDK1.8 中还能看到 `Segment` 的数据结构，但是已经简化了属性，只是为了兼容旧版本；② **`Hashtable`(同一把锁)** :使用 `synchronized` 来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。

JDK1.8 的 `ConcurrentHashMap` 不在是 **Segment 数组 + HashEntry 数组 + 链表**，而是 **Node 数组 + 链表 / 红黑树**。不过，Node 只能用于链表的情况，红黑树的情况需要使用 **`TreeNode`**。当冲突链表达到一定长度时，链表会转换成红黑树。



#### ConcurrentHashMap线程安全的具体实现方式/底层具体实现

##### JDK 1.7

首先将数据分为一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问。

**`ConcurrentHashMap` 是由 `Segment` 数组结构和 `HashEntry` 数组结构组成**。

Segment 实现了 `ReentrantLock`,所以 `Segment` 是一种可重入锁，扮演锁的角色。`HashEntry` 用于存储键值对数据。

```java
static class Segment<K,V> extends ReentrantLock implements Serializable {
}Copy to clipboardErrorCopied
```

一个 `ConcurrentHashMap` 里包含一个 `Segment` 数组。`Segment` 的结构和 `HashMap` 类似，是一种数组和链表结构，一个 `Segment` 包含一个 `HashEntry` 数组，每个 `HashEntry` 是一个链表结构的元素，每个 `Segment` 守护着一个 `HashEntry` 数组里的元素，当对 `HashEntry` 数组的数据进行修改时，必须首先获得对应的 `Segment` 的锁。



##### JDK 1.8

`ConcurrentHashMap` 取消了 `Segment` 分段锁，采用 CAS 和 `synchronized` 来保证并发安全。数据结构跟 HashMap1.8 的结构类似，数组+链表/红黑二叉树。Java 8 在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为 O(N)）转换为红黑树（寻址时间复杂度为 O(log(N))）

`synchronized` 只锁定当前链表或红黑二叉树的首节点，这样只要 hash 不冲突，就不会产生并发，效率又提升 N 倍。



# Set

## HashSet

> java.util.HashSet

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    private transient HashMap<E,Object> map;

    // 与支持 Map 中的对象关联的虚拟值
    private static final Object PRESENT = new Object();
}
```



`HashSet` 是 `Set` 接口的主要实现类 ，`HashSet` 的底层是 `HashMap`，线程不安全的，可以存储 null 值。



### 源码解析

#### 添加元素

> java.util.HashSet

```java
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```



HashSet添加元素时即调用了HashMap的put方法，此时如果map的key位置为空，则返回true；反之则返回旧值表示添加失败。



## LinkeHashSet



`LinkedHashSet` 是 `HashSet` 的子类，其内部是通过 `LinkedHashMap` 来实现，能够按照添加的顺序遍历。



## TreeSet



`TreeSet` 底层使用红黑树，能够按照添加元素的顺序进行遍历，排序的方式有自然排序和定制排序。