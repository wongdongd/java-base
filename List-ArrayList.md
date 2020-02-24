## List

Java中的List是有序的集合(也称为序列)，是一个接口。此接口的用户可以精确控制每个元素在列表中的插入位置，可以通过元素的索引(list中的位置)获取和查找该元素。

和sets不同，list允许元素重复，甚至可以允许多个null存在。官方提示，不提倡自己实现一个禁止重复的List。List继承了Collection的iterator、add、remove、equals和hashCode方法。

## ArrayList

实现List接口的容量可变**数组**。与Vector大致一样，但不是同步的。其中： size、isEmpty、get、set、iterator和listIterator时间复杂度都是O(1)。add操作时间复杂度与个数有关，新增n个则为O(n)。其他操作为线性复杂度，且比LinkedList要快。

### 2.1 定义：
```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    private static final int DEFAULT_CAPACITY = 10;  // 默认数组初始大小

    private static final Object[] EMPTY_ELEMENTDATA = {}; // 空数组

    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {}; // 与EMPTY_ELEMENTDATA区分开，是用来衡量添加了第一个元素后，数组扩容了多大

    transient Object[] elementData; // 存放数据的数组，当新增第一个元素时，容量就到DEFAULT_CAPACITY


    private int size; // ArrayList大小

```

* 继承了AbstractList，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能
* 实现了RandmoAccess接口，即提供了随机访问功能。快速随机访问，可以通过元素的序号快速获取元素对象。
* 实现了Cloneable接口，即覆盖了函数clone()，能被克隆
* 实现java.io.Serializable接口，这意味着ArrayList支持序列化，能通过序列化去传输

### 2.2 初始化

ArrayList提供了三种初始化方法,分别为：

* ```public ArrayList(){}```: 将其中的elementData初始化为一个容量为DEFAULT_CAPACITY(10)的数组，这其实是在add操作中实现的。
* ```public ArrayList(int initialCapacity) {}```：构造指定容量的数组列表
* ```public ArrayList(Collection<? extends E> c) {}```：借助Arrays.copyOf()构成指定类型的数组列表

### 2.3 增

```
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```
其中：当前数组被装满的时候，ArrayList的实现就会按照原来的容量+容量/2来扩容，再使用Arrays.copyOf()将原来的数组复制到新的数组中，具体在ensureCapacityInternal()实现，扩容的关键方法是grow():
```
private static int calculateCapacity(Object[] elementData, int minCapacity) { 
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) { // 只有在第一次添加新元素时才会使用
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0) // 当前的数组已经满了，需要扩容
        grow(minCapacity);
}
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)  // 扩的太大了，超过了上界
        newCapacity = hugeCapacity(minCapacity);  // 将新的容量按照(minCapacity > MAX_ARRAY_SIZE) ?Integer.MAX_VALUE:MAX_ARRAY_SIZE来选
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

### 2.4 读

由于ArrayList是数组实现的，所以支持按照下标的随机访问，故读取很简单直接按下标索引即可：
```
public E get(int index) {
    rangeCheck(index); // 越界判断

    return elementData(index);
}
```

### 2.5 删除

ArrayList提供了两种删除元素的方式，其实可以归结为数组中按下标删，然后再将下标后面的前移
* 按下标删除
```
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                            numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```
* 按元素值删除：会按序删除第一个相等的
```
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);  // 同上，只是不做越界判断，不返回值
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
```

## Fail-Fast机制

快速失败”也就是fail-fast，它是Java集合的一种错误检测机制。当多个线程对集合进行结构上的改变的操作时，有可能会产生fail-fast机制。记住是有可能，而不是一定。例如：假设存在两个线程（线程1、线程2），线程1通过Iterator在遍历集合A中的元素，在某个时候线程2修改了集合A的结构（是结构上面的修改，而不是简单的修改集合元素的内容），那么这个时候程序就会抛出 ConcurrentModificationException 异常，从而产生fail-fast机制。

ArrayList中的Fail-Fast机制是由modCount来保证的，其定义在AbstractList中，为全局量，在执行add，remove等都会改变modCount的值，导致与expectedModCount不一样，从而抛出ConcurrentModificationException异常。

解决办法：
* 直接使用Collections.synchronizedList：不推荐，因为增删造成的同步锁可能会阻塞遍历操作
* 使用CopyOnWriteArrayList：会在add、remove数据的时候，都先copy一份，在copy的数据上进行操作，但是会产生大量的空间损耗。