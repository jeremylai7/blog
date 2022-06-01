# 【leetcode 206】 反转链表（简单）

## 链表
概念: 
* 区别于数组，链表中的元素不是存储在内存中连续的一片区域，链表中的数据存储在每一个称之为「结点」复合区域里，在每一个结点除了存储数据以外，还保存了到下一个结点的指针（Pointer）。

![image](https://user-images.githubusercontent.com/11553237/131843264-90ac32ca-643d-4701-90cc-1ace9f2050f0.png)

* 由于不必按顺序存储，链表在插入数据的时候可以达到 O(1)O(1) 的复杂度，但是查找一个结点或者访问特定编号的结点则需要 O(n) 的时间。

## 应用
* HashMap Node 节点，Node节点有自身的值和 next 指向：
```
//HashMap Node 部分源码
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
}
```

* LinkedList Node 结点使用双链表

```
//LinkedList 部分源码
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
```

## 题目描述

![image](https://user-images.githubusercontent.com/11553237/171378975-6e823dcd-5a6a-429a-82a8-52dcfc207f2b.png)


## 解题思路

单链表的反转就是把链表的**指向换一个方向**，由从左往右变成从右变左。

主要解题思路是拆分每一个指针。

![image](https://user-images.githubusercontent.com/11553237/171379043-500b334e-390f-4a56-beb4-ac2f6f9270b5.png)


上图从 1 指向 2 变成 2 指向1，也就是需要设置 `2 节点的next = 1`，在做指向的改变之前要先将 2 节点的 next 存起来，然后改变 2 节点的next：

![image](https://user-images.githubusercontent.com/11553237/171379087-74216a3c-eb7e-4c7e-872a-3583166cf293.png)


* 创建一个空链表 node，用来存储反转的链表。
* 存储链表的 next。
* 链表的 next 指向 node。
* 当前链表赋值给 node。
* 循环遍历下一个节点

Java 解题代码：
```
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode cur = head;
        // 创建一个空链表
        ListNode node= null;
        while(cur != null) {
            // 存储 next
            ListNode next = cur.next;
            // 当前链表指向指向新的链表
            cur.next = node;
            // 当前节点赋值给 node
            node = cur;
            // 遍历下一个节点
            cur = next; 
        }
        return pre;

    }
}

```
   
