# HashSet 如何保证元素不重复——hash码

HashSet 不重复主要add 方法实现，使用 add 方法找到是否存在元素，存在就不添加，不存在就添加。HashSet 主要是基于HashMap 实现的，HashMap 的key就是 HashSet 的元素，HashSet 基于hash 函数实现元素不重复。
首先看 add 方法：
```
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```
HashMap 的put 方法，map 的 put 方法调用 putVal 方法。
```
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```
hash 方法就是计算hash值,和右移16位^做运算使得hash分布更均衡
```
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
看一下 putVal 方法：
```
    /**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //如果table 为null，table数组做扩容
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //在tab数组的位置上找不到元素，直接添加元素
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        //在tab数组上存在元素
        else {
            Node<K,V> e; K k;
            //hash值一致，key也是一致的话，表示原来的位置的 key 和现在插入的 key 是一致的，直接替换
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // value 类型是 TreeNode，引入红黑树，红黑树不存在，直接添加。存在则替换
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
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
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
       //不存在原来的 key,返回 null
        return null;
    }
```
注解上看 return 注解 
```
return previous value, or null if none
```
**返回以前的值，如果不存在就返回null**。
通过源码分析：
* 在 tab 数组找是否存在元素
    * 不存在元素直接直接创建节点
    *  如果存在，保存原来的值
* 判断原来的值是否为null
   * 如果不为空返回原来的值
   * 为空，返回null
 
### 总结
HashSet 主要是利用的 hash 算法的唯一性，每个元素的hash值是唯一的。
* hash 码不同，说明元素不存在。存数据
* hash 码相同，并且 equles 判断相等，说明元素已存在，不存数据
* hash 码相同，并且 equles 判断不相等，说明元素不存在，存数据

> 这里的 equles 在源码是直接调用 Object 的 equles 方法，是比较内存地址。但是在实际调用时，都是使用 String，或者 Integer 等封装类型，这些类型会重写 equles 方法。这是最开始有些不懂的地方。
