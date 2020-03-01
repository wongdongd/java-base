
## 定义

> 基于Map接口实现的哈希表。允许null键/值，不能保证有序插入，同时不保证顺序的持久性。get和put方法为常数时间，Iteration迭代与buckets和k-v映射数量成正比。因此，初始容量不能太高(或者是负载因子不能太低)。

```
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {

    private static final long serialVersionUID = 362498820763181265L;

    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 16，默认的初始容量，必须是2的倍数

    static final int MAXIMUM_CAPACITY = 1 << 30; // 最大容量，必须为2的倍数，且小于 1<<30

    static final float DEFAULT_LOAD_FACTOR = 0.75f; // 默认负载因子

    static final int TREEIFY_THRESHOLD = 8; // 在bin(位桶)中使用tree代替list的bin数量阈值

    static final int UNTREEIFY_THRESHOLD = 6; // 在resize操作中，去树化的位桶个数阈值

    static final int MIN_TREEIFY_CAPACITY = 64; // 位桶被树化的最小表容量，为避免resize和树化的阈值冲突，最小应该设为4*TREEIFY_THRESHOLD

    transient Node<K,V>[] table;

    transient Set<Map.Entry<K,V>> entrySet;

    transient int size;

    transient int modCount;

    int threshold; // capacity*loadFactor

    final float loadFactor;
}
``` 

## HashMap中的几个关键函数

### putVal()

> 在HashMap中新增元素一般用到的是put函数，或者putIfAbsent()等，其在1.8中调用的还是putVal()

```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)  // table[]哈希表为空，resize()初始化
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)           // 新增元素对应的索引位置为null，直接补入
        tab[i] = newNode(hash, key, value, null);
    else {                                               //  新增元素对应的索引位置有值
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))   // table[i]位置元素key值与新增一致，替换 
            e = p;
        else if (p instanceof TreeNode)                               // 是TreeNode
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {                                                        // 是链表
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1)           // 链表长超过了树化阈值，进行树化
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k)))) // 链表中元素key值与新增一致，替换
                    break;
                p = e;
            }                                                           // 遍历完链表，没有key值一样的话，e为原链表末端的next
        }
        if (e != null) { // existing mapping for key       // 对e赋值value，上述过程中e已经找到合适的插入位置
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)                             // 判断是否超过了 threshold*capacity
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

由上述分析可知，在新增元素过程中，putVal()主要做了两件事：
1. 为新增的元素找到合适的插入位置，几种可能情况：
    * 插入在哈希表table[]中(原table[]为空，会进行resize()初始化)
    * 在哈希表某个索引对应的链表中
    * 在哈希表某个索引对应的TreeNode中
2. 新增完元素，判断是否超过了threshold规定的容量，进行resize()扩容

### getNode()

> jdk1.8中get(Object key)方法实际调用的是getNode(),如下所示，存在的话也是三种情况：在哈希表上，在树上，在链表上

```
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {                   // 找到key对应的索引位置
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))  // table中索引位置元素key值与要查的一致，返回
            return first;
        if ((e = first.next) != null) {                            
            if (first instanceof TreeNode)                              // table中索引位置元素key值与要查的不一致且是TreeNode,继续查
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {                                                       // table中索引位置元素key值与要查的不一致且是链表,继续查
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

### tableSizeFor()

> 该函数主要用来计算哈希表的大小，保证数组大小永远是2的整数次幂。

```
static final int tableSizeFor(int cap) {
    int n = cap - 1;                        // 找到的目标2的整数次幂最接近且大于等于cap
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```
实现的算法相当给力：假设n为2进制数：001xxx...xxx,

* 将n右移一位0001xxx...xx，在与n位或：0011xxx..xx;
* 将n右移两位000011x...x, 在与n位或：001111x...x，此时前面有四个1，
* 将n右移四位在位或，得到前面8个1，... 
* 最后n右移16位后位或，整个32位执行完，让原来1后面的数值全变成1，
* 最后n+1，得到一个比原来数值大的2的整数次幂值。

### hash()

> 在Java 1.8的实现中，是通过hashCode()的高16位异或低16位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度、功效、质量来考虑的，这么做可以在bucket的n比较小的时候，也能保证考虑到高低bit都参与到hash的计算中，同时不会有太大的开销。

## 多线程问题

HashMap是线程不安全的，主要有以下两点：

1. 在put的时候，因为该方法不是同步的，假如有两个线程A,B它们的put的key的hash值相同，不论是从头插入还是从尾插入，假如A获取了插入位置为x，但是还未插入，此时B也计算出待插入位置为x，则不论AB插入的先后顺序肯定有一个会丢失

2. 在resize的时候，jdk1.8之前是采用头插法，当两个线程同时检测到hashmap需要扩容，在进行同时扩容的时候有可能形成有环链表会造成查询死循环，主要原因就是，采用头插法，新链表与旧链表的顺序是反的，在1.8后采用尾插法就不会出现这种问题，同时1.8的链表长度如果大于8就会转变成红黑树

HashMap在用到多线程时，请使用CocurrentHashMap.

