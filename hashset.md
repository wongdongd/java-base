# HashSet

This class implements the Set interface, backed by a hash table (actually a HashMap instance). It makes no guarantees as to the iteration order of the set; in particular, it does not guarantee that the order will remain constant over time. This class permits the null element.

HashSet实现了Set接口，内部实际是HashMap，内部是无序的，且同HashMap一样，会因为resize而导致元素的位置不能永远一致。

支持fail-fast机制

需要多线程时：
1. 在外部的object上使用synchronized关键字
2. 使用包装类方法：Collections.synchronizedSet()
