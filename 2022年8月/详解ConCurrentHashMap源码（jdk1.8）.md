# 详解ConCurrentHashMap源码（jdk1.8）

`ConCurrentHashMap`是一个支持高并发集合，常用的集合之一，在`jdk1.8`中`ConCurrentHashMap`的结构和操作和`HashMap`都很类似：
* 数据结构基于`数组+链表/红黑树`。
* `get`通过计算hash值后取模数组长度确认索引来查询元素。
* `put`方法也是先找索引位置，然后不存在就直接添加，存在相同`key`就替换。
* 扩容都是创建新的`table`数组，原来的数据转移到新的`table`数组中。

**唯一不同的是**，`HashMap`不支持并发操作，`ConCurrentHashMap`是支持并发操作的。所以`ConCurrentHashMap`的设计也比`HashMap`也复杂的多，通过阅读`ConCurrentHashMap`的源码，也更加了解一些并发的操作,比如：
* `volatile` 线程可见性
* `CAS` 乐观锁
* `synchronized` 同步锁/悲观锁

详见`HashMap`相关文章：

[详解HashMap源码解析（上）](https://www.cnblogs.com/jeremylai7/p/16441845.html)

[详解HashMap源码解析（下）](https://www.cnblogs.com/jeremylai7/p/16445127.html)

# 数据结构

`ConCurrentHashMap`是由`数组+链表/红黑树`组成的：

![image](https://user-images.githubusercontent.com/11553237/184280956-ff0513c8-c948-4cb5-8a1c-ae80638e7a95.png)

其中左侧部分是一个`哈希表`,通过`hash算法`确定元素在数组的下标:
```
transient volatile Node<K,V>[] table;
```

链表是为了解决`hash冲突`,当发生冲突的时候。采用`链表法`,将元素添加到链表的尾部。其中`Node`节点存储数据：

```
 static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;
    volatile Node<K,V> next;

    Node(int hash, K key, V val, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.val = val;
        this.next = next;
    }
}
```

`Node`节点包含：
* `hash` hash值
* `key` 值
* `value` 值
* `next` next指针

# 主要属性字段

```
// 最大容量
int MAXIMUM_CAPACITY = 1 << 30;

// 初始化容量
int DEFAULT_CAPACITY = 16

// 控制数组初始化或者扩容，为负数时，表示数组正在初始化或者扩容。-1表示正在初始化。其他情况-n表示n线程正在扩容。
private transient volatile int sizeCtl;

// 装载因子
float LOAD_FACTOR = 0.75f

// 链表长度为 8 转成红黑树
int TREEIFY_THRESHOLD = 8

// 红黑树长度小于6退化成链表
int UNTREEIFY_THRESHOLD = 6;

```


# 获取数据get

```
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    // 计算hash值
    int h = spread(key.hashCode());
    // 判断 tab 不为空并且 tab对应的下标不为空 
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        // eh < 0 表示遇到扩容
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        // 遍历链表，直到遍历key相等的值    
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

* 获取数据流程：
    * 调用`spread`获取`hash`值,通过`(n - 1) & h`取余获取数组下标的数据。
    * 首节点符合就返回数据。
    * `eh<0`表示遇到了扩容，会调用正在扩容节点`ForwardingNode`的`find`方法，查找该节点，匹配就返回。
    * 遍历链表，匹配到数据就返回。
    * 以上都不符合，返回`null`。



## get如何实现线程安全

get方法里面没有使用到锁，那是如何实现线程安全。主要使用到了`volatile`。

* `volatile` 

一个线程对共享变量的修改，另外一个线程能够立刻看到，我们称为`可见性`。

`cpu`运行速度比内存速度快很多，为了均衡和内存之间的速度差异，增加了`cpu缓存`,如果在cpu缓存中存在`cpu`需要数据，说明命中了`cpu`缓存，就不经过访问内存。如果不存在，则要先把内存的数据载入到`cpu`缓存中，在返回给`cpu`处理器。

在`多核cpu`的服务器中，每个`cpu`都有自己的缓存，`cpu`之间的缓存是不共享的。 当多个线程在不同的`cpu`上执行时，比如下图中，`线程A`操作的是`cpu-1`上的缓存，`线程B`操作的是`cpu-2`上的缓存，这个时候，`线程A`对`变量V`的操作对于`线程B`是`不可见的`。

![image](https://user-images.githubusercontent.com/11553237/184280993-1f4c0bca-0e2d-407e-ac7d-57cc8ca3b211.png)

但是一个变量被`volatile`声明，它的意思是：

> 告诉编译器，对这个变量的读写，不能使用cpu缓存，必须从内存中读取或者写入。

上面的变量V被`volatile`声明，线程A在cup-1中修改了数据，会直接写到内存中，不会写入到cpu缓存中。而线程B无法从cpu缓存读取变量，需要从主内存拉取数据。

* 总结:
  * 使用`volatile`关键字的变量会将修改的变量强制写入内存中。
  * 其他线程读取变量时，会直接从内存中读取变量。

## `volatile`在`get`应用

* `table`哈希表
```
transient volatile Node<K,V>[] table;
```

使用`volatile`声明数组，表示`引用地址`是`volatile`而不是`数组元素`是`volatile`。

> 既然不是`数组元素`被修饰成`volatile`，那实现线程安全在看`Node`节点。

* `Node`节点
```
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;
    volatile Node<K,V> next;
}
```

其中`val`和`next`都用了`volatile`修饰，在多线程环境下，线程A修改节点`val`或者新增节点对别人线程是`可见的`。
所以`get`方法使用无锁操作是可以`保证线程安全`。

>既然`volatile`修饰数组对`get`操作没有效果，那加在`volatile`上有什么目的呢？

**是为了数组在扩容的时候对其他线程具有可见性。**

* jdk 1.8 的get操作不使用锁，主要有两个方面：
    * Node节点的`val`和`next`都用`volatile`修饰，保证线程修改或者新增节点对别人线程是可见的。
    * `volatile`修饰`table`数组，保证数组在扩容时其它线程是具有可见性的。

# 添加数据put

`put(K key, V value)`直接调用`putVal(key, value, false)`方法。

```
public V put(K key, V value) {
    return putVal(key, value, false);
}
```

`putVal()`方法：

```
final V putVal(K key, V value, boolean onlyIfAbsent) {
        // key或者value为空，报空指针错误
        if (key == null || value == null) throw new NullPointerException();
        // 计算hash值
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                // tab为空或者长度为0，初始化table
                tab = initTable();
            // 使用volatile查找索引下的数据
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                // 索引位置没有数据，使用cas添加数据
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            // MOVED表示数组正在进行数组扩容，当前进行也参加到数组复制
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                // 数组不在扩容和也有值，说明数据下标处有值
                // 链表中有数据，使用synchronized同步锁
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        // 为链表
                        if (fh >= 0) {
                            binCount = 1;
                            // 遍历链表
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                // hash 以及key相同，替换value值
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                // 遍历到链表尾，添加链表节点
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        // 红黑树，TreeBin哈希值固定为-2
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
                    // 链表转红黑树
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

* 添加数据流程：
  * 判断`key`或者`value`为`null`都会报空指针错误。
  * 计算`hash`值,然后开启**没有终止条件**的循环。
  * 如果`table`数组为`null`,初始化数组。
  * 数组`table`不为空，通过`volatile`找到数组对应下标是否为空，为空就使用`CAS`添加头结点。
  * 节点的`hash`=`-1`表示数组正在扩容，一起进行扩容操作。
  * 以上不符合，说明索引处有值，使用`synchronized`锁住当前位置的节点，防止被其他线程修改。
       * 如果是链表，遍历链表，匹配到相同的`key`替换`value`值。如果链表找不到，就添加到链表尾部。
       * 如果是红黑树，就添加到红黑树中。
  * 节点的链表个数大于`8`,链表就转成红黑树。
   
>`ConcurrentHashMap`键值对为什么都不能为`null`,而`HashMap`就可以？

通过`get`获取数据时，如果获取的数据是`null`，**就无法判断，是put时的value为null，还是找个key就没做过映射**。HashMap是非并发的，可以通过`contains(key)`判断，而支持并发的`ConcurrentHashMap`在调用`contains`方法和`get`方法的时候，`map`可能已经不同了。[参考](http://cs.oswego.edu/pipermail/concurrency-interest/2006-May/002485.html)

如果数组`table`为空调用`initTable`初始化数组：

```
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    // table 为 null
    while ((tab = table) == null || tab.length == 0) {
       
        if ((sc = sizeCtl) < 0)
         // sizeCtl<0表示其它线程正在初始化数组数组，当前线程需要让出CPU
            Thread.yield(); // lost initialization race; just spin
        // 调用CAS初始化table表    
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```
`initTable`判断`sizeCtl`值，如果`sizeCtl`为`-1`表示有其他线程正在初始化数组，当前线程调用`Thread.yield`让出`CPU`。而正在初始化数组的线程通过`Unsafe.compareAndSwapInt`方法将`sizeCtl`改成`-1`。

`initTable`最外层一直使用`while`循环，而非`if`条件判断，就是确保数组可以初始化成功。

数组初始化成功之后，再执行添加的操作，调用`tableAt`通过`volatile`的方式找到`(n-1)&hash`处的`bin`节点。
* 如果为空，使用`CAS`添加节点。
* 不为空，需要使用`synchronized`锁，索引对应的`bin`节点，进行添加或者更新操作。
  
> Insertion (via put or its variants) of the first node in an
  empty bin is performed by just CASing it to the bin.  This is
  by far the most common case for put operations under most
  key/hash distributions.  Other update operations (insert,
  delete, and replace) require locks.  We do not want to waste
  the space required to associate a distinct lock object with
  each bin, so instead use the first node of a bin list itself as
  a lock. Locking support for these locks relies on builtin
  "synchronized" monitors. 
 
如果f的hash值为-1，说明当前f是ForwaringNode节点，意味着有其它线程正在扩容，则一起进行扩容操作。

完成添加或者更新操作之后，才执行`break`终止最外层**没有终止条件的for循环**，确保数据可以添加成功。

最后执行`addCount`方法。

```
private final void addCount(long x, int check) {
    CounterCell[] as; long b, s;
    // 利用CAS更新baseCoount
    if ((as = counterCells) != null ||
        !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
        CounterCell a; long v; int m;
        boolean uncontended = true;
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
            !(uncontended =
              U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
            fullAddCount(x, uncontended);
            return;
        }
        if (check <= 1)
            return;
        s = sumCount();
    }
    // check >= 0,需要检查是否需要进行扩容操作
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);
            if (sc < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
            s = sumCount();
        }
    }
}
```

# 扩容transfer

## 什么时候会扩容

*插入一个新的节点：
  * 新增节点，所在的链表元素个数达到阈值8，则会调用`treeifyBin`把链表转成红黑树，在转成之前，会判断数组长度小于`MIN_TREEIFY_CAPACITY`,默认是`64`,触发扩容。
  * 调用`put`方法，在结尾`addCount`方法记录元素个数，并检查是否进行扩容，数组元素达到阈值时，触发扩容。


不使用加锁的，支持多线程扩容。利用并发处理减少扩容带来性能的影响。

```
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    if (nextTab == null) {            // initiating
        try {
            @SuppressWarnings("unchecked")
            // 创建nextTab，容量为原来容量的两倍
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            // 扩容是抛出异常，将阈值设置成最大，表示不再扩容。
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }
    int nextn = nextTab.length;
    // 创建 ForwardingNode 节点，作为标记位，表明当前位置已经做过桶处理
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    // advance = true 表明该节点已经处理过了
    boolean advance = true;
    boolean finishing = false; // to ensure sweep before committing nextTab
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        // 控制 --i，遍历原hash表中的节点
        while (advance) {
            int nextIndex, nextBound;
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            // 用CAS计算得到的transferIndex
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        // 将原数组中的节点赋值到新的数组中，nextTab赋值给table，清空nextTable。
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            // 所有节点完成复制工作，
            if (finishing) {
                nextTable = null;
                table = nextTab;
                // 设置新的阈值为原来的1.5倍
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            // 利用CAS方法更新扩容的阈值，sizeCtl减一，说明新加入一个线程参与到扩容中
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
                finishing = advance = true;
                i = n; // recheck before commit
            }
        }
        // 遍历的节点为null，则放入到ForwardingNode指针节点
        else if ((f = tabAt(tab, i)) == null)
            advance = casTabAt(tab, i, null, fwd);
        // f.hash==-1表示遍历到ForwardingNode节点，说明该节点已经处理过了    
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed
        else {
            // 节点加锁
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    // fh>=0，表示为链表节点
                    if (fh >= 0) {
                        // 构建两个链表，一个是原链表，另一个是原链表的反序链表
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
                        // 在nextTable i 位置处插入链表
                        setTabAt(nextTab, i, ln);
                        // 在nextTable i+n 位置处插入链表
                        setTabAt(nextTab, i + n, hn);
                        // 在table i的位置处插上ForwardingNode，表示该节点已经处理过
                        setTabAt(tab, i, fwd);
                        // 可以执行 --i的操作，再次遍历节点
                        advance = true;
                    }
                    // TreeBin红黑树，按照红黑树处理，处理逻辑和链表类似
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
                        // 扩容后树节点的个数<=6,红黑树转成链表
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

扩容过程有的复杂，主要涉及到多线程的并发扩容，`ForwardingNode`的作用就是支持扩容操作，将已经处理过的节点和空节点置为`ForwardingNode`，并发处理时多个线程处理`ForwardingNode`表示已经处理过了，就往后遍历。

# 总结

* `ConcurrentHashMap`是基于`数组+链表/红黑树`的数据结构，添加、删除、更新都是先通过计算`key`的`hash`值确定数据的索引值，这和`HashMap`是类似的，只不过`ConcurrentHashMap`针对并发做了更多的处理。
* `get`方法获取数据，先计算`hash`值再再和数组长度取余操作获取索引位置。
    * 通过`volatile`关键字找到`table`保证多线程环境下，数组扩容具有`可见性`,而`Node`节点中`val`和`next`指针都使用`volatile`修饰保证数据修改后别的线程是`可见的`。这就保证了`ConcurrentHashMap`的`线程安全性`。
    * 如果遇到数组扩容，就参与到扩容中。
    * 首节点值匹配到数据就直接返回数据，否则就遍历链表或者红黑树，直到匹配到数据。
* `put`方法添加或者更新数据。
    * 如果`key`或`value`为空，就报错。这是因为在调用`get`方法获取数据为`null`,无法判断是获取的数据为null，还是对应的`key`就不存在映射,`HashMap`可以通过`contains(key)`判断，而`ConcurrentHashMap`在多线程环境下调用`contains`和`get`方法的时候，`map`可能就不同了。
    * 如果`table`数组为空，先初始化数组，先通过`sizeCtl`控制并发，如果小于0表示有别的线程正在初始化数组，就让出`CPU`,否则使用`CAS`将`sizeCtl`设置成`-1`。
    * 初始化数组之后，如果节点为空，使用`CAS`添加节点。
    * 不为空，就锁住该节点，进行添加或者更新操作。
    
* `transfer`扩容
   * 在新增一个节点时，链表个数达到阈值`8`，会将链表转成红黑树，在转成之前，会先判断数组长度小于`64`,会触发扩容。还有集合个数达到阈值时也会触发扩容。
   * 扩容数组的长度是原来数组的两倍。
   * 为了支持多线程扩容创建`ForwardingNode`节点作为标记位，如果遍历到该节点，说明已经做过处理。
   * 遍历赋值原来的数据给新的数组。
  

# 参考

[为什么ConcurrentHashMap的读操作不需要加锁](https://www.cnblogs.com/keeya/p/9632958.html)

[Java并发——ConcurrentHashMap(JDK 1.8）](https://juejin.cn/post/6844903646476369933)
