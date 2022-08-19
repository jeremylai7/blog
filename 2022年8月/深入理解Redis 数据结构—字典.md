# 深入理解Redis 数据结构—字典

>字典，又称为**符号表、关联数组或映射**，是一种用于保存**键值对**的抽象数据结构。在字典中，一个键可以和一个值进行关联，这些关联的键和值称为键值对。键值对中键是`唯一的`,我们可以根据`键key`通过映射查找或者更新对应的`值value`。


很多高级开发语言有对应集合支持字典这种数据结构，比如`Java`中的`Map`集合。`C语言`并未内置字典这种数据结构，`Redis`构建了自己的字典实现。

# 应用

字典在`Redis`中应用非常广泛，`Redis`数据库就是使用字典作为数据底层的实现。对数据的**增、删、改、查**操作也是建立在字典之上操作。

当执行命令：
```
set msg "hello"
```
在数据库中创建一个键为 `msg`，值为 `hello` 的键值对，这个`键值对`就用字典来实现的。创建其他数据类型的键值对，比如`list`、`hash`、`set`、`sort set`也是用字典来实现的。

处理用来表示数据中的键值对，字典还是`hash`数据类型底层实现之一，比如一个`hash`数据类型`website`，包含`100`个键值对，这些键值对中的键是`网址名称`，值是`网页地址`:

```
redis> HGETALL website
1)"Redis"
2)"Redis.io"
3)"nacos"
4)"nacos.io"
.....
```
`website`键的底层就是一个字典，包好了`100`个`键值对`，例如：
* 键值对中的键为`"Redis"`，值为`"Redis.io"`。
* 键值对中的键为`"nacos"`，值为`"nacos.io"`。


# 字典的实现

`Redis`字典使用哈希表作为底层实现，一个哈希表里面有多个哈希表节点，每个哈希表节点保存字典中的键值对。

## 哈希表

`Redis`字典使用的哈希表由 [dict.h/dictht](https://github.com/redis/redis/blob/2.6/src/dict.h#L68-L73) 结构来表示：
```
/* This is our hash table structure. Every dictionary has two of this as we
 * implement incremental rehashing, for the old to the new table. */
typedef struct dictht {
     // table 数组 
    dictEntry **table;
    // 哈希表的大小
    unsigned long size;
    // 等于size-1，用于计算索引值
    unsigned long sizemask;
    // 已有的键值对数量
    unsigned long used;
} dictht;
```
注释：这是哈希表结构，每个字典有两个实现增量重散列，从旧的哈希表到新的哈希表。

* `table`属性的是一个数组，数组中的每个元素都指向哈希节点`dictEntry`，每个`dictEntry`结构都保存一个键值对。
* `size`记录了哈希表的大小，也就是`table`数组的大小。
* `used`属性记录了哈希表目前已有的键值对数量。`sizemask`的值等于`size-1`，这个属性和哈希表一起决定键应该放在 `table`数组的那个位置上。

下图展示一个大小为`4`空哈希表（没有包含任务的键值对）：

![image](https://user-images.githubusercontent.com/11553237/185521797-56034888-2efb-4574-8825-b6000b4a89c0.png)


## 哈希表节点
哈希表节点使用[dictEntry](https://github.com/redis/redis/blob/2.6/src/dict.h#L47-L55)结构来表示，每个`dictEntry`结构都保存着一个`键值对`：
```
typedef struct dictEntry {
    // 键
    void *key;
    // 值   
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;
    // 指向下一个哈希表节点，形成链表
    struct dictEntry *next;
} dictEntry;
```
其中：
* `key`保存键值
* `v`保持值，`v`可以是一个指针，可以是`uint64_t`整数，也可以是一个`int64_t`整数。
* `next`指向另一个哈希表节点的指针，这个指针将多个`哈希值相同`的键值对连接在一起，以此解决`hash冲突`的问题。

下图展示两个`键的hash值`相同的哈希表节点`k0`和`k1`，两者通过`next`指针连接在一起。

![image](https://user-images.githubusercontent.com/11553237/185521838-e4ebf837-ec8a-43bd-a75d-038acc8e8c5a.png)

## 字典

`Redis` 中的字典由[dict.h/dict](https://github.com/redis/redis/blob/2.6/src/dict.h#L75-L81)结构表示：
```
typedef struct dict {
    // 类型特定的函数 
    dictType *type;
   // 私有函数
    void *privdata;
    // 哈希表
    dictht ht[2];
    // rehash 索引 
    int rehashidx; /* rehashing not in progress if rehashidx == -1 */
    int iterators; /* number of iterators currently running */
} dict;
```

* `type`属性和`privadata`属性是针对不同类型的键值对，为创建多态的字典而设置。
* `type`是指向`dictType`结构的指针，每个`dictType`包含几组针对特定类型的键值对操作的函数，`Redis`会为用途不同的字典设置不同的函数。下图展示`dictType`字典类型：
```
typedef struct dictType {
    // 计算哈希值
    unsigned int (*hashFunction)(const void *key);
    // 复制键
    void *(*keyDup)(void *privdata, const void *key);
    // 复制值
    void *(*valDup)(void *privdata, const void *obj);
    // 对比键
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    // 销毁键
    void (*keyDestructor)(void *privdata, void *key);
   // 销毁值 
   void (*valDestructor)(void *privdata, void *obj);
} dictType;

```
* `privdata`属性保存针对不同的类型操作的函数传的可选参数。

* `ht[2]`是包含两个数大小的数组，类型为`dictht`哈希表。字典只使用`ht[0]`哈希表，ht`[1]`只会对`ht[0]`哈希表进行`rehash`时使用。
* `rehashidx`记录了`rehash`的进度，如果目前没有进行的`rehash`，那么它的值为`-1`。

下图为一个普通状态下（没有进行rehash）的字典：

![image](https://user-images.githubusercontent.com/11553237/185521873-0be34851-df62-41b5-8061-69e61887c349.png)

# 哈希算法

当要将一个新的键值对添加到字典中，程序需要先根据键值对中的键计算出哈希值和索引值，然后根据索引值，将包含新键值的哈希表放在哈希表数组的指定索引上。

`Redis`计算哈希值和索引值的步骤如下：

1. 使用字典设置的哈希函数，计算键的哈希值。
> **hash = dict—>type->hashFunction(key)**
2. 使用哈希表的`sizemask`属性和哈希值，取余计算出哈希值。
>**index = hash & dict ->ht[x].sizemask** 
   
**了解过`HashMap`底层原理的同学应该知道，上面计算索引值和`HashMap`找到索引下标的原理是类似的。**  

>什么是取余`&`运算？

取余就是计算两数相除的余数， 比如一个数组长度为4，索引范围是`0~3`,需要放置`0,1,7`，放置如下图所示：


![image](https://user-images.githubusercontent.com/11553237/185522049-c09a5006-cf47-42c8-94d5-25c34f14c1fc.png)


举个例子，要将一个键值对`k0`和`v0`添加到下方的空字典表中：

![image](https://user-images.githubusercontent.com/11553237/185521912-76ad2bed-d97e-4e6f-b95a-f09bf6498647.png)

首先计算键的哈希值：
```
hash = dict—>type->hashFunction(key)
```
计算键`k0`的哈希值。
假设计算出来的哈希值为`8`，然后计算索引值：
```
index = dict -> hash & ht[0].sizemask = 8 & 3 = 0;
```

计算出键`k0`的索引值`0`，这表示键值对`k0`和`v0`的节点放置到哈希表数组下标`0`的位置上，如下图所示：

![image](https://user-images.githubusercontent.com/11553237/185521948-f101e638-a0b9-4583-af72-ec2b510b0c6b.png)

# 键冲突

当两个或者两个以上的计算出数组索引值一致时，就发生了`键冲突`。

Redis的哈希表采用`链表法`来解决键冲突，每个哈希表的节点都有一个`next`指针，多个哈希表节点用`next`指针组成一个单链表，被分配到同一个数组索引上的多个节点使用单向链表连接起来，这就很好的解决了`键冲突`的问题。

举个例子，程序要将一个键值对`k2`和`v2`添加到下图的哈希表中，并且计算`k2`的索引值为`2`,那么键`k1`和`k2`将发生冲突：

![image](https://user-images.githubusercontent.com/11553237/185522136-e4b5071c-ccf2-4bf1-a0bb-de1444cf71a5.png)

**解决冲突的办法**就是使用`next`指针将`k2`和`k1`所在的节点连接起来，如下图所示：

![image](https://user-images.githubusercontent.com/11553237/185521977-c4ee8cc2-4a0b-4332-9ea3-4b1437111cd0.png)

# 总结

* 字典是一种映射的`键值对`数据结构，其中键是唯一的，通过唯一的键可以快速找到对应的值。
* 字典包含广泛用在`Redis`数据库中。
   * 其中所有数据类型的键值对都使用字典作为底层实现。
   * `Hash`类型的键值对也是基于字典实现。
* 字典的结构
   * 包含一个字典，包含根据特定类型处理的函数`dictType`、两个哈希表`ht[2]`,字典只用到了`ht[0]`,遇到了扩容才会使用`ht[1]`。
   * 一个字典包含`两个哈希表`，每个哈希表`dictht`包含一个`table`数组，`size`记录数组的大小，`sizemask`等于`size-1`,`sizemask`和哈希值决定数据存在在`table`的位置。`used`记录已有的键值对。
   * 哈希表节点dictEntry`结构保存一个键值对，其中`key`保存键，`V`保存值，`V`可以是一个指针、可以是`uint64_t`整数、也可以是`int64_t`的整数。`next`是为了解决`键hash冲突`，两个键的计算出的索引在数组的同一个地方，就使用`next`指针连接在一起。
* 新增一个键值对，首先通过调用`dict—>type->hashFunction(key)`计算`键的哈希值`,再和`dictht`的`sizemask`做取余操作，计算得到要存放`table`数组的索引位置。如果发生`键冲突`时，使用`链表法`将多个哈希节点通过`next`指针组成一个单链表。
* `Redis`字典的实现和`Java`中的`HashMap`数据结构有以下类似的点：
   * **确定索引位置：** 键首先使用哈希算法算出哈希值，再和数组的`长度-1`做取余操作，确定存放数组的下标。
   * **解决hash冲突:** 两个键值计算的索引一致，采用`链表法`，将多个节点通过`next`指针连在一起。

# 参考

[Redis设计与实现](https://book.douban.com/subject/25900156/)
   
