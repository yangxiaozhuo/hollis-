# 典型回答

Java 8是一个比较大的版本，目前很多人还在用，他有很多内容的升级，关于HashMap也有很多，其中最主要的就是引入了红黑树，除此之外hash、resize等方法都有些改动。

### 红黑树

Java 1.7中的HashMap使用一个数组+链表的数据结构来存储键值对，这会导致在处理hash冲突时性能下降，特别是当链表变得很长的时候时。Java 1.8中的HashMap引入了红黑树来替代链表，以解决冲突时性能问题。这意味着当链表变得过长时，HashMap可以将链表转换为树，从而提高了查找、插入和删除操作的性能。

[✅为什么在JDK8中HashMap要转成红黑树](https://www.yuque.com/hollis666/fo22bm/zx609g?view=doc_embed)

### 节点变化
在JDK 1.8中，Node用于代替了旧版本中的普通链表节点（Entry），在旧版本中，每个桶中的元素都需要一个单独的Entry对象，而在新版本中，使用Node来代替，并且为了支持红黑树还引入了TreeNode。在Node中hash字段变成了final类型，一定确定就不再可变。

1.7中的Entry：

```c
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    int hash;
}
```


1.8中的Node：
```c
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
}
```

```c
 static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
 }

```

### 尾插法

为了解决1.7中的并发场景中存在的死循环的问题，JDK1.8把头插法改成了尾插法。

[✅HashMap用在并发场景中有什么问题？](https://www.yuque.com/hollis666/fo22bm/ph44ot?view=doc_embed)

### hash方法


在JDK 1.8中，hash方法有所变化。JDK 1.7中hash方法如下：

```java
final int hash(Object k) {
    int h = 0;
    if (useAltHashing) {
        if (k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }
        h = hashSeed;
    }

    h ^= k.hashCode();

    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

而JDK 1.8中hash代码比较简单：

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

这两个算法比较大的区别就是1.8中的`位运算`和`异或`操作更少了，而1.7中之所以有这么多位运算和异或操作，主要是为了增加结果的散列性，从而减少碰撞的概率。

而JDK 1.8的源码中关于这个算法也给出了官方解释：

> There is a tradeoff between speed, utility, and quality of bit-spreading. Because many common sets of hashes are already reasonably distributed (so don't benefit from spreading), and because we use trees to handle large sets of collisions in bins, we just XOR some shifted bits in the cheapest possible way to reduce systematic lossage, as well as to incorporate impact of the highest bits that would otherwise never be used in index calculations because of table bounds.


大概意思就是说， 1.8的的哈希算法在速度、实用性和位扩散的质量之间做了权衡。修改哈希算法的目的是在保持速度的前提下，提高hashcode的质量，以减少哈希冲突和提高数据分布的均匀性。这里还提到在 JDK 1.8 中，为了解决桶上的冲突，引入了红黑树。这意味着即使哈希冲突发生，树结构可以有效地处理大规模的冲突，而不会影响性能。

> 位扩散质量： 位扩散是指在哈希算法中如何将不同位的信息组合，以生成最终的哈希码。较好的位扩散能够减少哈希冲突，即不同键映射到同一桶的可能性。因此，修改哈希算法的一个目标是提高位扩散的质量，以改善数据的分布。


### 扩容机制

在JDK 1.8中，扩容机制也有很大的变化，在JDK 1.7中，扩容是通过resize和transfer两个方法相互配合完成的。


```java
 void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    Entry[] newTable = new Entry[newCapacity];
    boolean oldAltHashing = useAltHashing;
    useAltHashing |= sun.misc.VM.isBooted() &&
            (newCapacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
    boolean rehash = oldAltHashing ^ useAltHashing;
    transfer(newTable, rehash);
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}

void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) {
        while(null != e) {
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i];
            newTable[i] = e;
            e = next;
        }
    }
}

```

而在JDK 1.8中，把transfer的逻辑直接放到resize中了：

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
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
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
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

肉眼可见的代码变长了，主要是因为1.8中引入了红黑树，需要考虑到树化的过程。

还有就是在JDK 1.8 中，元素扩容后的位置有所区别。

JDK1.7中扩容的时候，重新计算位置。JDK 1.8 则不一定，只要看看原hash值新增的那个bit位是1还是0，是0的话位置不变， 是1的话位置重新计算，为原数组下标位置加上原数组容量：

```java
 do {
    next = e.next;
    // 结果为0，放入新数组的位置和原数组的位置相同
    if ((e.hash & oldCap) == 0) {
        if (loTail == null)	
            loHead = e;
        else
            loTail.next = e;
        loTail = e;
    }
    // 结果不为0，需要重新计算存放的位置
    else {
        if (hiTail == null)	
            hiHead = e;
        else	
            hiTail.next = e;
        hiTail = e;
    }
} while ((e = next) != null);
if (loTail != null) {
    loTail.next = null;
    newTab[j] = loHead;
}
if (hiTail != null) { 
    // 新位置：所在的原数组位置下标加上原数组容量
    hiTail.next = null;
    newTab[j + oldCap] = hiHead;
}
```
