
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

