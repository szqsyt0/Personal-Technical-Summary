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
4. JDK1.8引入了红黑树，优化了HashMap性能
