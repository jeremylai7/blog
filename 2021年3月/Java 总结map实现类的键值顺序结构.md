# map实现类的键值顺序结构

## map是一个接口。主要有如下实现类

1. HashMap
2. LinkedHashMap
3. TreeMap
4. ConcurrentHashMap
5.Hashtable

## 其中 
1. HashMap、Hashtable、ConcurrentHashMap 不会保存插入顺序，遍历时访问数据是随机的。
2. LinkedHashMap保存插入顺序，遍历map顺序就是插入的顺序。
3. TreeMap实现SortMap接口，根据键的值进行排序，默认是升序。
