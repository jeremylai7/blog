# 深入理解Redis 数据结构—双链表

在 Redis 数据类型中的**列表list**，对数据的添加和删除常用的命令有 lpush,rpush,lpop,rpop，其中 l 表示在左侧，r 表示在右侧，可以在左右两侧做添加和删除操作，说明这是一个双向的数据结构，而 list 数据结构正是双向链表，类似 java 中的 LinekdList 链表列表。

链表提供了高效的节点重排能力，以及顺序的节点访问方式，通过修改节点的 pre 和 next 指针来修改链表的数据。

C 语言没有内置链表的数据结构，所以 Redis 构建了自己的链表结构。

## 链表的数据结构，链表以及链表节点
链表是由链表以及链表节点组成，每个链表节点使用一个 [adlist.h/listNode](https://github.com/redis/redis/blob/2.6/src/adlist.h#L36-L40) 结构来表示：
```
typedef struct listNode {
    //前置节点
    struct listNode *prev;
    //后置节点
    struct listNode *next;
    // 节点值
    void *value;
} listNode;
```
多个 listNode 可以通过 prev 和 next 指针组成双链表的，如题所示：

![image](https://user-images.githubusercontent.com/11553237/169222510-27c1f279-b541-4596-97bb-7fd3efee56d9.png)


多个 listNode 可以组成链表，但是为了方便管理，使用 [adlist.h/list](https://github.com/redis/redis/blob/2.6/src/adlist.h#L47-L54) 管理链表，list 结构如下：
```
typedef struct list {
    // 列表头结点
    listNode *head;
    // 列表尾结构
    listNode *tail;
    // 节点值复制函数
    void *(*dup)(void *ptr);
    // 节点值释放函数
    void (*free)(void *ptr);
    // 节点值对比函数 
    int (*match)(void *ptr, void *key);
    // 列表节点数量
    unsigned long len;
} list;
```
list 结构为链表提供了表头指针 head、表尾指针 tail，以及节点数量计算 len。下图展示一个由 list 结构和三个 listNode 节点组成的链表：

![image](https://user-images.githubusercontent.com/11553237/169222558-2ad266b7-5d63-440f-9565-f326792b5da7.png)

Redis 链表实现的特征有如下的总结：
* 双向：链表节点带有 prev 和 next 指针，可以通过指针获取每一个数据
* 快速计算链表长度：通过 list 结构中的 len 属性计算 list 的长度，而时间复杂度为O(1)
* 多态： 链表节点使用 void* 指针保存节点，所以链表支持保存各种不同类型的值

## 双链表的运用
列表键，发布订阅、慢查询以及监视器等。

## 总结
* 本文通过介绍链表的数据结构，链表是由链表和链表节点组成的
* 链表节点都有一个前置和后置指针，所以 Redis 的链表是一个双向链表
* 链表可以存储头结点，尾节点，更好的管理自己的节点，len 属性快速算出链表的长度
* 链表通过 void* 以及不同的类型设定函数，所以链表可以不同的类型的值
