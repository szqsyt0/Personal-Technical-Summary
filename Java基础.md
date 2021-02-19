## 面向对象基本特性和基本原则

1. 面向对象基本特性
   - 封装：把客观事物封装成抽象的类，隐藏内部的实现细节，只对外开放必要的接口
   - 继承：通过继承可以获得另一个的属性和方法
   - 多态：一个类实例的相同方法在不同情形下有不同的表现形式

2. 面向对象基本原则
   - 单一职责原则**SRP(Single Responsibility Principle)**：类的功能具备单一性
   - 开放封闭原则**OCP(Open－Close Principle)**：对扩展开放，对修改封闭
   - 里氏替换原则**LSP(the Liskov Substitution Principle LSP)**：子类可以替换父类，并出现在任何父类可以出现的地方
   - 依赖原则DIP(the Dependency Inversion Principle DIP)：高层次的模块不应该依赖低层次的模块，都应该依赖于抽象；抽象不应该依赖于具体实现，具体实现应该依赖于抽象
   - 接口分离原则**ISP(the Interface Segregation Principle ISP)**：接口端不应该依赖它不需要的接口，一个类对另一个类的依赖应该建立在最小的接口上。如果一个提供接口的类中对于它的子类来说不是最小的接口，那么它的子类在实现该类时必须实现一些自己不需要的功能，整个系统会逐渐变得臃肿难以维护。



## 关键字原理及用法

### transient

1. transient关键字只能修饰变量，不能修饰方法和类
2. 被transient修饰的变量不会被序列化，一个静态变量不论是否被transient修饰，都不能被序列化，该变量内容在序列化后无法获得访问

### volatile

volatile关键字可以用于禁止指令重排序并保障变量的可见性，被volatile关键字修饰的变量在修改后会被立即刷回主存，并强制使其他线程工作内存中的变量副本失效，从主存重新读取。

volatile是基于内存屏障实现的，内存屏障有两个作用：

- 禁止指令重排序
- 强制刷回主存并使其他线程工作内存中的副本失效

### final

final是一个非访问修饰符，仅适用于类、变量或方法

1. final修饰变量：被修饰的变量必须被初始化（赋值），且后续不能修改其值，实质上是常量
2. final修饰方法：被修饰的方法无法被子类重写
3. final修饰类：被修饰的类不能被继承，并且final类中的成员方法都会被隐式地指定为final方法，但成员变量不会

### static

1. static修饰变量：静态变量或类变量，静态变量在内存中只有一个拷贝，JVM只为静态变量分配一次内存，在加载类的过程中完成静态变量的内存分配，可用类名直接访问。
2. static修饰方法：静态方法可以直接通过类名调用，任何实例都可以调用。静态方法不能访问所属类的实例变量和实例方法。
3. static代码块：是在类中独立于类成员的语句块，可以有多个，位置随便放，不在任何方法体内，JVM加载类时会执行这些静态的代码块。
4. static修饰内部类：静态内部类和非静态内部类最大的区别是：静态内部类没有指向创建它的外围类的引用。这也意味着静态内部类创建不需要依赖于外围类的对象；静态内部类不能使用任何外部类的非static成员变量和方法

## 集合类

### ArrayList

ArrayList继承自AbstractList，实现了List接口，底层使用数组实现，支持自动扩容

#### 自动扩容

在添加元素时，判断数组是否需要扩容，每次扩容默认在原有基础上扩容50%，如果扩容后依然小于所需的minCapacity，将扩容到minCapacity，代码如下

```java
    // 判断数组是否需要扩容，如需要，调用grow方法扩容
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }   
    
    // 扩容数组。
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        // 右移一位，即原有基础上扩容50%
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

### LinkedList

LinkedList基于双向链表实现，因此在插入和删除方面优于ArrayList，随机访问劣于ArrayList。除了List接口之外，LinkedList还实现了Deque、Cloneable、Serializable三个接口，表明LinkedList支持队列、克隆和序列化操作。和ArrayList一样，允许插入null元素。

LinkedList有以下三个成员变量：

```java
 transient int size = 0;
 transient Node<E> first;
 transient Node<E> last;
```

Node是双链表结构，代码如下：

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

### HashMap

HashMap是使用频率最高的用于映射的数据类型。在JDK1.8中对HashMap底层实现进入了优化，引入了红黑树的数据结构和扩容的优化等。

Java为数据结构中的映射定义了一个接口java.util.Map，此接口主要有四个常用的实现类，分别是HashMap、Hashtable、LinkedHashMap和TreeMap，类继承关系如下图所示：

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/f7fe16a2.png)

#### 内部实现

HashMap是基于数组+链表+红黑树（JDK1.8新增）实现的，如下图所示：

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/e4a19398.png)

HashMap内部是一个Node数组，Node是HashMap的内部类，实现了Map.Entry接口，本质是一个键值对。

```java
    /**
     * The table, initialized on first use, and resized as
     * necessary. When allocated, length is always a power of two.
     * (We also tolerate length zero in some operations to allow
     * bootstrapping mechanics that are currently not needed.)
     */
    transient Node<K,V>[] table;
    
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;    //用来定位数组索引位置
        final K key;
        V value;
        Node<K,V> next;   //链表的下一个node

        Node(int hash, K key, V value, Node<K,V> next) { ... }
        public final K getKey(){ ... }
        public final V getValue() { ... }
        public final String toString() { ... }
        public final int hashCode() { ... }
        public final V setValue(V newValue) { ... }
        public final boolean equals(Object o) { ... }
	}
```

HashMap采用链地址法解决Hash冲突，在每个数组元素上都有一个链表结构，当数据被Hash后，得到数组下标，把数据放在对应下标元素的链表上。

阅读源码可知，HashMap初始化时会对以下字段赋值：

```java
     int threshold;             // 所能容纳的key-value对极限 
     final float loadFactor;    // 负载因子
     int modCount;  // 记录HashMap内部结构发生变化的次数，用于迭代的快速失败。
     int size;  // 实际存在的键值对数量
```

Node[] table的默认长度是16，loadFactor默认0.75，threshold=length*loadFactor，当Map中的键值对数量超过threshold时就会进行扩容，扩容后的HashMap是原来的两倍。默认的loadFactor0.75是对空间和时间效率的一个平衡选择，一般情况下不建议修改。

当出现链表长度过长时，HashMap会将链表转换为红黑树（JDK1.8），提升增删改查的效率。

#### Hash寻址

HashMap通过hash算法确定元素在数组中的下标，代码如下：

```java
方法一：
static final int hash(Object key) {   //jdk1.8 & jdk1.7
     int h;
     // h = key.hashCode() 为第一步 取hashCode值
     // h ^ (h >>> 16)  为第二步 高位参与运算
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
方法二：
static int indexFor(int h, int length) {  //jdk1.7的源码，jdk1.8没有这个方法，但是实现原理一样的
     return h & (length-1);  //第三步 取模运算
}
```

寻址算法本质上分为三步：对key进行hash、高位运算、取模运算

对于任意给定的对象，只要其hashCode方法返回相同的结果，那么调用方法一计算得到的hash码都是一致的。为了元素尽量在hash数组中分布尽量均匀，可以把hash值对数组长度取模运算，但是模运算的消耗比较大，因此在HashMap中使用方法二来计算元素在hash数组中的下标。

方法二通过h & (length - 1)得到元素的索引，而HashMap底层数组的长度总是2的n次方，这是HashMap在速度上的优化。当length总是2的n次方时，h&(length-1)等价于对length取模，而且性能更高。

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/45205ec2.png)

#### put方法

1. 判断键值对数组table是否为空或者length=0，如果是调用resize进行扩容
2. 根据键值key计算hash值得到索引i，如果table[i]为空则新建节点插入，否则执行第三步
3. 判断table[i]的首个元素是否和key相同（hashcode&equals），如果是直接覆盖value
4. 判断table[i]是否为TreeNode，如果是，则直接在红黑树中插入键值对
5. 遍历table[i]，将node插入到链表的尾部，判断链表长度是否大于8，如果大于8则转换为红黑树；如果在遍历时找到相同的key，则进行覆盖
6. 插入成功后，判断实际存在的键值对是否超过threshold，如果超过则进行扩容

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/d669d29c.png)

```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

	final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 步骤①：tab为空则创建
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 步骤②：计算index，并对null做处理 
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            // 步骤③：节点key存在，直接覆盖value
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 步骤④：判断该链为红黑树
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            // 步骤⑤：该链为链表
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //链表长度大于8转换为红黑树进行处理
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // key已经存在直接覆盖value
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 步骤⑥：超过最大容量 就扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

#### 扩容机制

1. 确定newCap大小
2. 拷贝元素到新数组
3. 元素的重新hash

JDK8扩容机制相比之前版本进行了优化，HashMap大小始终是2次幂的扩展，所以元素的位置要么是原位置，要么是原位置+oldCap。如下图所示，n为table的长度，图(a)表示扩容前的key1和key2两种key确定索引位置的示例，图(b)表示扩容后key1和key2确定索引位置的示例，其中hash1是对应的key1的哈希与高位运算结果。

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/4d8022db.png)

元素在重新hash之后，因为n变成2倍，那么n-1的mask范围在高位多1bit（红色），因此新的index就会发生如下图所示的变化：

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/d773f86e.png)

因此，在扩充HashMap的时候，不需要想JDK7一样重新计算hash，只需要看看原来的hash值新增的bit是1还是0即可。

```java
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            // 容量超过MAXIMUM_CAPACITY,后不再扩容，将threshold扩大到MAX_VALUE
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 容量在原有基础上扩大两倍，threshold也扩大两倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        // 链表优化重hash的代码块
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            // 原索引
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            // 原索引 + oldCap
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        // 原索引放到bucket里
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        // 原索引 + oldCap放到bucket里
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

#### 小结

1. 扩容操作非常消耗性能，所以在使用HashMap的时候，估算map的大小，初始化的时候给一个大致的估值，避免频繁扩容
2. 负载因子可以修改，但除非特殊情况，否则不建议修改
3. HashMap是线程不安全的，不能在并发环境使用HashMap
4. JDK1.8引入了红黑树，优化了HashMap性能1. 

### LinkedHashMap

LinkedHashMap是HashMap的子类，保存了记录的插入顺序，遍历时按照插入顺序返回结果，也可以在构造时带参数，按照访问次序排序。

```java
    // 双向链表记录节点插入顺序
	static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
    
    /**
     * The head (eldest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> head;

    /**
     * The tail (youngest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> tail;
```

### TreeMap

TreeMap继承自AbstractMap，并实现了NavigableMap、Cloneable和Serializable接口，AbstractMap实现了大部分Map接口支持的方法，NavigableMap接口继承自SortedMap接口，SortedMap存储的是有序的键值对，其内部维护了一个Comparator比较器，因此TreeMap具备以下特性：

- 有序
- 支持克隆
- 支持序列化

#### 存储结构

TreeMap内部使用红黑树存储元素。

```java
private transient Entry<K,V> root; //根节点

static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;//存储键
    V value;//存储值
    Entry<K,V> left;//左子树节点引用
    Entry<K,V> right;//右子树节点引用
    Entry<K,V> parent;//父节点引用
    boolean color = BLACK;//颜色
    /**
     * Make a new cell with given key, value, and parent, and with
     * {@code null} child links, and BLACK color.
     */
    Entry(K key, V value, Entry<K,V> parent) {
        this.key = key;
        this.value = value;
        this.parent = parent;
    }
}
```

#### put方法

将一个节点添加到红黑树中，通常需要以下几个步骤：

1. 将红黑树当成普通二叉查找树，将节点插入
2. 将新插入的节点设置为红色
3. 通过旋转和着色，使其恢复平衡，重新变成一颗符合规则的红黑树

```java
public V put(K key, V value) {
    Entry<K,V> t = root;
    if (t == null) {//如果root为null 说明是添加第一个元素 直接实例化一个Entry 赋值给root
        compare(key, key); // type (and possibly null) check
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    Entry<K,V> parent;//如果root不为null，说明已存在元素 
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator; 
    if (cpr != null) { //如果比较器不为null 则使用比较器
        //找到元素的插入位置
        do {
            parent = t; //parent赋值
            cmp = cpr.compare(key, t.key);
            //当前key小于节点key 向左子树查找
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)//当前key大于节点key 向右子树查找
                t = t.right;
            else //相等的情况下 直接更新节点值
                return t.setValue(value);
        } while (t != null);
    }
    else { //如果比较器为null 则使用默认比较器
        if (key == null)//如果key为null  则抛出异常
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
       
        //找到元素的插入位置
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    Entry<K,V> e = new Entry<>(key, value, parent);//定义一个新的节点
    //根据比较结果决定插入到左子树还是右子树
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;
    fixAfterInsertion(e);//保持红黑树性质  插入后进行修正
    size++;//元素树自增
    modCount++; 
    return null;
}
```

#### 总结

1. TreeMap基于红黑树实现，该映射基于key的自然顺序进行排序，或者根据创建映射时提供的Comparator进行排序
2. 不允许插入为Null的key
3. 可以插入值为Null的Value
4. 非线程安全
5. 根据Key排序，Key必须实现Comparator接口，也可自定义比较器实现排序

### CouncurrentHashMap

在JDK8中，ConcurrentHashMap代码结构和HashMap基本相同，都是数组+链表+红黑树，其主要区别在ConcurrentHashMap支持并发：

1. 使用Unsafe方法操作数组内部元素，保证可见性
2. 在更新和移动节点时，直接锁住对应的哈希桶，锁粒度更小
3. 针对扩容操作慢进行优化
4. 优化哈希表计数器，采用LongAdder、Striped64类似思想

#### 哈希计算

```java
static final int MOVED     = -1;          // hash for forwarding nodes
static final int TREEBIN   = -2;          // hash for roots of trees
static final int RESERVED  = -3;          // hash for transient reservations
static final int HASH_BITS = 0x7fffffff;  // usable bits of normal node hash

// 让高位16位，参与哈希桶定位运算的同时，保证 hash 为正
static final int spread(int h) {
  return (h ^ (h >>> 16)) & HASH_BITS;
}
```

除此之外还有：

- tableSizeFor：将容量转为大于n，且最小的2的幂
- 除留余数法：hash % length = hash & (length - 1)
- 扩容后哈希桶定位：(e.hash & oldCap)， 0-位置不变，1-原来的位置+oldCap

#### 哈希桶可见性

一个数组即使声明为volatile，也只能保证这个数组引用本身的可见性，其内部元素的可见性是无法保证的，如果每次都加锁，则效率必然大大降低，在CHM中使用Unsafe方法来保证可见性：

```java
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
  return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}

static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i, Node<K,V> c, Node<K,V> v) {
  return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}

static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
  U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
}
```

#### 关键方法源码分析

##### initTable

ConcurrentHashMap在put方法被调用的时候对数组进行初始化，由于存在多个线程同时调用put的可能性，需要保证初始化的线程安全。通过initTable代码可知，多个线程的竞争通过对sizeCtl进行CAS操作实现的，如果某个线程成功把sizeCtl设置为-1，该线程就拥有了初始化的权利，进入初始化的代码模式，等到初始化完成再把sizeCtl设置回去，其他线程则一直执行while循环，自旋等待，指导数组不为null，即当初始化结束后，退出整个循环。

```java
private final Node<K,V>[] initTable() {
  Node<K,V>[] tab; int sc;
  while ((tab = table) == null || tab.length == 0) {
    if ((sc = sizeCtl) < 0) Thread.yield();  // 有其他线程在初始化
    else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {  // 设置状态 -1
      try {
        if ((tab = table) == null || tab.length == 0) {
          int n = (sc > 0) ? sc : DEFAULT_CAPACITY;  // 注意此时的 sizeCtl 表示初始容量，完毕后表示扩容阈值
          @SuppressWarnings("unchecked")
          Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
          table = tab = nt;
          sc = n - (n >>> 2);  // 同 0.75n
        }
      } finally {
        sizeCtl = sc;  // 注意这里没有 CAS 更新，这就是状态变量的高明了，因为前面设置了 -1，此时这里没有竞争
      }
      break;
    }
  }
  return tab;
}
```

##### get

```java
public V get(Object key) {
  Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
  int h = spread(key.hashCode()); // 计算 hash
  if ((tab = table) != null && (n = tab.length) > 0 &&  // 确保 table 已经初始化
    
    // 确保对应的哈希桶不为空，注意这里是 Volatile 语义获取；因为扩容的时候，是完全拷贝，所以只要不为空，则链表必然完整
    (e = tabAt(tab, (n - 1) & h)) != null) {
    if ((eh = e.hash) == h) {
      if ((ek = e.key) == key || (ek != null && key.equals(ek)))
        return e.val;
    }
    
    // hash < 0，可能为红黑色节点或者当前正在扩容
    else if (eh < 0) return (p = e.find(h, key)) != null ? p.val : null;
    
    while ((e = e.next) != null) { // 遍历链表
      if (e.hash == h &&
        ((ek = e.key) == key || (ek != null && key.equals(ek))))
        return e.val;
    }
  }
  return null;
}
```

##### put

put流程如下图所示：

![chmput](https://img2018.cnblogs.com/blog/1119937/201904/1119937-20190429200101369-1250566623.png)

```java
    public V put(K key, V value) {
        return putVal(key, value, false);
    }

    /** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```

##### 扩容

 CHM 中巧妙的利用任务划分，使得多个线程可能同时参与扩容；另外扩容条件也有两个：

- 有链表长度超过 8，但是容量小于 64 的时候，发生扩容；
- 节点数超过阈值的时候，发生扩容；

其扩容的过程可描述为：

- 首先扩容过程的中，节点首先移动到过度表 **nextTable** ，所有节点移动完毕时替换散列表 **table**；
- 移动时先将散列表定长等分，然后逆序依次领取任务扩容，设置 **sizeCtl** 标记正在扩容；
- 移动完成一个哈希桶或者遇到空桶时，将其标记为 **ForwardingNode** 节点，并指向 **nextTable** ；
- 后有其他线程在操作哈希表时，遇到 **ForwardingNode** 节点，则先帮助扩容（继续领取分段任务），扩容完成后再继续之前的操作；

先从treeifyBin(tab, i)方法分析。

```java
    /**
     * Replaces all linked nodes in bin at given index unless table is
     * too small, in which case resizes instead.
     */
    private final void treeifyBin(Node<K,V>[] tab, int index) {
        Node<K,V> b; int n, sc;
        if (tab != null) {
            if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
                tryPresize(n << 1);
            else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
                synchronized (b) {
                    if (tabAt(tab, index) == b) {
                        TreeNode<K,V> hd = null, tl = null;
                        for (Node<K,V> e = b; e != null; e = e.next) {
                            TreeNode<K,V> p =
                                new TreeNode<K,V>(e.hash, e.key, e.val,
                                                  null, null);
                            if ((p.prev = tl) == null)
                                hd = p;
                            else
                                tl.next = p;
                            tl = p;
                        }
                        setTabAt(tab, index, new TreeBin<K,V>(hd));
                    }
                }
            }
        }
    }
```

从以上代码可以看出，当数组长度没有超过MIN_TREEIFY_CAPACITY时，不会转换为红黑树，只扩容数组。

在tryPresize(int size)内部调用了一个核心函数transfer(Node<K, V>[] tab, Node<K, V>[] nextTab)方法。代码如下

```java
    /**
     * Moves and/or copies the nodes in each bin to new table. See
     * above for explanation.
     */
    private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride;
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE) 
            stride = MIN_TRANSFER_STRIDE; // subdivide range // 根据CPU数量计算任务步长
        if (nextTab == null) {            // initiating //初始化nextTab
            try {
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1]; // 扩容2倍
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            transferIndex = n; // 初始的transferIndex为旧HashMap的数组长度
        }
        int nextn = nextTab.length;
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab); // 标记空桶或已经完成移动的桶
        boolean advance = true;
        boolean finishing = false; // to ensure sweep before committing nextTab
        for (int i = 0, bound = 0;;) { // i为遍历的下标，bound为遍历的边界，如果成功拿到一个任务，则i = nextIndex - 1,bound = 
            Node<K,V> f; int fh;       // nextIndex - stride，如果拿不到任务，i = 0, bound = 0
            while (advance) {
                int nextIndex, nextBound;
                if (--i >= bound || finishing)
                    advance = false;
                else if ((nextIndex = transferIndex) <= 0) { // transferIndex <= 0，表示迁移已经完成
                    i = -1;
                    advance = false;
                }
                else if (U.compareAndSwapInt //对transferIndex进行CAS操作，即为当前线程分配一个stride，若CAS操作成功，线程成功拿到一个
                         (this, TRANSFERINDEX, nextIndex, // stride任务。CAS操作不成功，线程没抢到任务，会继续执行while循环，自旋
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                if (finishing) {
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            else {
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        if (fh >= 0) {
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }
    }
```

#### 总结

- JDK8中的CHM没有使用Segment分段锁，而且直接锁定单个哈希桶
- 对数组中的哈希桶使用CAS操作，保证可见性
- 扩容使用任务拆分、多线程同时扩容的方式，加速扩容
- 对size使用计数器思想
- CHM中对状态变量的应用，使得很多操作都可以无锁化进行- 

### HashSet

HashSet是一个value为同一Object对象的HashMap

```java
    // HashSet初始化方法
    public HashSet() {
        map = new HashMap<>();
    }
	// 添加元素
    public boolean add(E e) {
        return map.put(e, PRESENT)==null; // PRESENT = new Object()
    }
	// 删除元素
    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }
```

### TreeSet

和HashSet一样，TreeSet是TreeMap的马甲，内部维护了一个TreeMap的变量。

```java
    private transient NavigableMap<E,Object> m;
    public TreeSet() {
        this(new TreeMap<E,Object>());
    }
```

### LinkedHashSet

LinkedHashSet是LinkedHashMap的小马甲

```java
    public LinkedHashSet() {
        // 调用父类HashSet的初始化
        super(16, .75f, true);
    }
    
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        // 创建一个LinkedHashMap
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
```

### 集合Stream API

Stream将要处理的元素看作一种流，流在管道中传输，并且可以在管道的节点上进行处理，比如筛选、排序、聚合等。元素流在管道中经过中间操作的处理，最后由最终操作得到前面处理的结果。

#### 生成流

- **stream()** − 为集合创建串行流。
- **parallelStream()** − 为集合创建并行流。

#### 流操作

- forEach:迭代流中的每个数据

```java
Random random = new Random();
random.ints().limit(10).forEach(System.out::println); // 使用forEach输出了10个随机数
```

- map：映射元素到对应的结果

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
// 获取对应的平方数
List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());
```

- filter: 通过设置的条件筛选元素

```java
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
// 获取空字符串的数量
long count = strings.stream().filter(string -> string.isEmpty()).count();
```

- limit: 获取指定数量的流

```java
Random random = new Random();
random.ints().limit(10).forEach(System.out::println);
```

- sorted: 对流进行排序

```java
Random random = new Random();
random.ints().limit(10).sorted().forEach(System.out::println); //对输出的10个随机数进行排序
```

#### Collectors

Collectors类实现了很多归约操作，例如将流转换成集合和聚合元素。Collectors可用于返回列表或字符串

```java
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
 
System.out.println("筛选列表: " + filtered);
String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", "));
System.out.println("合并字符串: " + mergedString);
```

#### 统计

Java提供了产生统计结果的收集器，主要用于int、double、long等基本类型上，可以用来产生如下的统计结果

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
IntSummaryStatistics stats = numbers.stream().mapToInt((x) -> x).summaryStatistics();
System.out.println("列表中最大的数 : " + stats.getMax());
System.out.println("列表中最小的数 : " + stats.getMin());
System.out.println("所有数之和 : " + stats.getSum());
System.out.println("平均数 : " + stats.getAverage());
```

## Java IO

### IO模型 - UNIX IO模型

#### 阻塞式IO

应用程序被阻塞，指导数据复制到应用程序缓冲区内才返回

![img](https://www.pdai.tech/_images/io/java-io-model-0.png)

#### 非阻塞式IO

应用进程执行系统调用之后，内核返回一个错误码，应用进程可以继续执行，但是需要不断的执行系统调用来获知IO是否完成，这种方式称为轮询（polling），由于CPU要处理很多的系统调用，因此这种模型比较低效。

![img](https://www.pdai.tech/_images/io/java-io-model-1.png)

#### IO复用

使用select和poll等待数据，并且可以等待多个socket中任何一个变为可读，这一过程会被阻塞，当某一个socket可读时返回，之后再使用recvfrom把数据从内核复制到进程内。IO复用可以使单个进行具有处理多个IO事件的能力。

如果一个Web服务器没有IO复用，那么每一个socket连接都需要创建一个线程去处理，如果同时有几万个连接，那么就需要创建相同数量的线程。并且相比于多进程和多线程技术，IO复用不需要进程线程创建和切换的开销。

![img](https://www.pdai.tech/_images/io/java-io-model-2.png)

#### 信号驱动式IO(SIGIO)

应用进程使用sigaction系统调用，内核立即返回，应用程序可以继续执行，也就是说等待数据阶段应用进程是非阻塞的，内核在数据到达时向应用进程发送SIGIO信号，应用程序收到之后在信号处理程序中调用recvfrom将数据从内核复制到应用程序中。相比于非阻塞式IO的轮询方式，信号驱动IO的CPU利用率更高。

![img](https://www.pdai.tech/_images/io/java-io-model-3.png)

#### 异步IO(AIO)

进行aio_read系统调用会立即返回，应用程序继续执行，不会阻塞，内核会在所有操作完成后向应用程序发送信号。异步IO和信号驱动IO的区别在于异步IO的信号是通知应用程序IO完成，而信号驱动IO的信息时通知应用程序可以开始IO。
