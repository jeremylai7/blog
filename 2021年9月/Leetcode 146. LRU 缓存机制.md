# Leetcode 146. LRU 缓存机制

## 前言
缓存是一种提高数据读取性能的技术，在计算机中cpu和主内存之间读取数据存在差异，CPU和主内存之间有CPU缓存，而且在内存和硬盘有内存缓存。当主存容量远大于CPU缓存，或磁盘容量远大于主存时，哪些数据应该被应该被清理，哪些数据应该被保留，这就需要缓存淘汰策略来决定。常见的策略有三种：先进先出策略FIFO(First In,First Out)、最少使用策略LFU(Least Frequently Used)、最近最少使用策略LRU(Least Recently Used)。

## LRU描述
设计和实现一个  LRU (最近最少使用) 缓存机制 。
实现 LRUCache 类：
* LRUCache(int capacity) 以正整数作为容量 capacity 初始化 LRU 缓存
* int get(int key) 如果关键字 key 存在于缓存中，则返回关键字的值，否则返回 -1 。
* void put(int key, int value) 如果关键字已经存在，则变更其数据值；如果关键字不存在，则插入该组「关键字-值」。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。

## 解题思路 哈希表 + 双向链表
* 针对LRU的特点，选择使用双链表实现。
* 使用 gut 方法获取数据，如果有数据，把返回数据，并且把数据放在链表头部。
* 使用 put 方法存放数据，如果数据存在，直接覆盖新值；如果数据不存在，添加新值。新值都放在链表头部。此外，还需要判断缓存有没有超出容量 capacity，如果有超出，删除链表的尾结点。
* 因为是单链表，每次获取数据，或者删除数据，都需要遍历一遍链表，时间复杂度是O(n),这里使用hash来记录每个数据的位置，将数据访问的时间复杂度降到O(1)。

```
class LRUCache {

    class DLinkedNode{
		int key;
		int value;
		DLinkedNode prev;
		DLinkedNode next;

		public DLinkedNode() {}

		public DLinkedNode(int key, int value) {
			this.key = key;
			this.value = value;
		}
	}

	private int size;

	private int capacity;

	private DLinkedNode head;

	private DLinkedNode tail;

	private Map<Integer,DLinkedNode> cache = new HashMap<>();

    public LRUCache(int capacity) {
        this.size = 0;
		this.capacity = capacity;
        head = new DLinkedNode();
		tail = new DLinkedNode();
		head.next = tail;
		tail.prev = head;
    }
    
    public int get(int key) {
        DLinkedNode node = cache.get(key);
		if (node == null) {
			return -1;
		}
		//找到并移动到首位
		moveToHead(node);
		return node.value;

    }
    
    public void put(int key, int value) {
        DLinkedNode node = cache.get(key);
		if (node == null) {
			//不存在就创建一个新的节点
			DLinkedNode newNode = new DLinkedNode(key,value);
			cache.put(key,newNode);
			addToHead(newNode);
			size++;
			if (size > capacity) {
				//超出容量，移除最后节点
				DLinkedNode tail = removeTail();
				cache.remove(tail.key);
				size--;
			}
		} else {
			//key存在，覆盖value，并移到头部
			if (node.value != value) {
				node.value = value;
			}
			moveToHead(node);

		}
    }

    private DLinkedNode removeTail() {
		DLinkedNode node = tail.prev;
		removeNode(node);
		return node;
	}

	private DLinkedNode removeNode(DLinkedNode node) {
		node.next.prev = node.prev;
		node.prev.next = node.next;
		return node;
	}

	private void moveToHead(DLinkedNode node) {
		removeNode(node);
		addToHead(node);
	}

	private void addToHead(DLinkedNode node) {
		node.prev = head;
		node.next = head.next;
        head.next.prev = node;
		head.next = node;
	}
}

```

### 参考
[LRU维基百科](https://zh.wikipedia.org/wiki/%E5%BF%AB%E5%8F%96%E6%96%87%E4%BB%B6%E7%BD%AE%E6%8F%9B%E6%A9%9F%E5%88%B6)
[极客时间-王争-如何实现LRU缓存淘汰算法?](https://time.geekbang.org/column/article/41013)

