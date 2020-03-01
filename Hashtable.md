# Hashtable

java通过Hashtable函数实现了哈希表，作为k-v映射。只允许**非空**的k-v值。

同HashMap一样，也有两个重要参数：initial capacity 和 load factor，只不过默认值分别为**11和0.75**。

同样在产生"哈希冲突"时，采用链表来解决

具有fail-fast机制：当使用iterator迭代时，除了使用iterator自带的remove方法，其他增加删除等修改操作都将抛出*ConcurrentModificationException* 异常。同样是modCount来统计操作的次数。

因为Hashtable继承自Dictory类，可以使用Enumerations来遍历，但是这种遍历不支持fail-fast机制。


## 一个傻逼问题

> Hashtable是线程安全的，为什么还会抛出ConcurrentModificationException异常呢？

首先，ConcurrentModificationException异常与多线程无关。当单线程对Hashtable进行iterator遍历时，中间进行增删操作同样会抛出该异常，导致fail-fast。


## 线程安全

Hashtable在每个方法，例如put、size、isEmpty、containsKey、get等都添加了synchronized关键字来保证同步，所以Hashtable是线程安全但低效的。
官方建议：在不需要多线程时使用HashMap，多线程时使用ConcurrentHashMap。