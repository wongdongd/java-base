# LinkedList

双端链表实现了List和Deque接口，允许null值。对链表的索引将会从更接近索引位置的头或尾来开始遍历。

不支持同步。如果需要多线程修改list的结构(新增或者删除其中的元素，仅仅改变其中某个值不算是结构性的改变)，需要依赖外部同步(弄个object的synchronized)，或者是直接使用Collections.synchronizedList()方法。

iterators迭代支持fail-fast机制。

## 定义

```
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient int size = 0;

    transient Node<E> first; // 头结点

    transient Node<E> last;  // 尾节点

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
}
```

## 新增元素

> 在jdk1.8中，LinkedList提供了几种新增元素的方法，如下：

```
public void addFirst(E e) {
    linkFirst(e);
}

public void addLast(E e) {
    linkLast(e);
}

public boolean add(E e) {
    linkLast(e);
    return true;
}

public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}

public boolean offer(E e) {
    return add(e);
}

public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}

public boolean offerLast(E e) {
    addLast(e);
    return true;
}

public void push(E e) {
    addFirst(e);
}
```
由上可以看出，在LinkedList上新增元素主要包括3种：
1. 在头部插入新元素: addFirst()、offerFirst()、push()
2. 在尾部插入新元素: addLast()、add()、offer()、offerLast()
3. 在指定位置上插入: add(int index, E element)

### 在头部插入元素

```
private void linkFirst(E e) {
    final Node<E> f = first;
    final Node<E> newNode = new Node<>(null, e, f);
    first = newNode;
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
    size++;
    modCount++;
}
```

### 在尾部插入元素(大多数)

```
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```
### 在指定位置插入

```
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}
 
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
```
可以看到，链表的好处就是在某个位置插入元素时，可以不用将后边所有的元素移动位置了。

## 查询元素

同新增元素类似，查询可分为以下三种：
1. 查头元素：getFirst()、peek()、poll()、peekFirst()、pollFirst()、pop()
2. 查尾元素：getLast()、peekLast()、pollLast()
3. 按位置查找：get(int index)

其中，peek()和poll()的主要区别是：peek()只返回相应元素，poll()返回元素后会删除该元素。查找头元素和尾元素相当简单，在按位置查找时，会根据索引的位置选择按头还是尾部遍历：

```
Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {    // 索引位置小于一半，从头开始
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {                      // 否则从尾部开始
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```
