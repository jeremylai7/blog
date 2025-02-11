# LinkedList 添加元素源码解析

## jdk版本：1.8
LinkedList添加元素有两个方法：add(E e)和add(int index,E e)。
## add(E e)
```
/**
*  Appends the specified element to the end of this list.
*  在列表最后添加指定元素
*/
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
```
add(E e)是直接在队尾添加元素。再看一下linkLast(E e)方法，源码如下。
```
    void linkLast(E e) {
        //找到链表的最后一个节点，赋值给l，
        final Node<E> l = last;
        //创建节点，这个节点的上一个节点就是上面last节点
        final Node<E> newNode = new Node<>(l, e, null);
       //将新的节点赋值给最后节点
        last = newNode;
        if (l == null)
            //如果没有最后节点，表示添加链表为空，赋值给首节点。
            first = newNode;
        else
            //有最后一个节点，将最后节点的next指针指向新建的节点
            l.next = newNode;
        size++;
        modCount++;
    }
```
1. LinkedList会记录链表的最后一个节点last，
2. 首先创建新的节点，新节点的pre就是队列的最后一个节点last，新节点的next为null，
3. 如果last为空表示这个链表为空，新节点就是首节点first。
4. 如果last不为空表示链表不为空，将last节点的next指针指向新节点。
## add(int index,E e)
add(int index,E e)是根据元素插入到指定位置上，index表示链表的位置
```
/**
     * Inserts the specified element at the specified position in this list.
     * 插入指定的元素到列表指定位置上
     */
    public void add(int index, E element) {
        //检查位置是否越界
        checkPositionIndex(index);
        //如果插入的下标等于链表的大小，直接就是添加到队尾，
        if (index == size)
            linkLast(element);
        else
            linkBefore(element, node(index));
    }
```
1. 首先检查index是否超过链表大小，即index 是否会大于链表的size。
2. 如果index等于链表的大小，添加元素就是在链表队尾添加元素，和add(E e) 操作一致。
3. 如果不等会size大小就调用linkBefore方法，首先在该方法的第二个参数使用了node方法。node方法源码如下：
```
 Node<E> node(int index) {
       //size >>1 表示size右移，表示size的一半
      //如果index小于size一半，从首节点往后遍历
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
       //如果index大于size一半，从最后一个节点往前遍历
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```
node方法通过链表的位置找到链表的元素。这里用到了一个size的右移运算，size>>1表示size/2。首先判断index是在链表的前一半还是后一半，因为linkedList是双链表，可以往前和往后进行遍历。如果在前半部分就从首节点往后遍历，如果在后半部分就从最后一个节点往前遍历，**这样最多遍历size的一半，避免遍历整个链表。**
找到index对应的元素后执行linkedBefore方法
```
    /**
     * Inserts element e before non-null Node succ.
    *   往succ节点前面插入元素
     */
    void linkBefore(E e, Node<E> succ) {
        //succ的前一个节点
        final Node<E> pred = succ.prev;
       //创建新节点，新节点的pre就是succ的pre节点，新节点的next就是succ节点
        final Node<E> newNode = new Node<>(pred, e, succ);
       //succ.pre指向新节点
        succ.prev = newNode;
        //如果succ前节点为空，表示succ就是首节点，新节点即为首节点
        if (pred == null)
            first = newNode;
        else
        //succ的上一节点的next指向新节点
            pred.next = newNode;
        size++;
        modCount++;
    }

```
1. 新建节点，节点pre就是succ的pre，节点的next就是succ。
2. 将succ.pre指向新节点。
3. succ的pre为null，succ即为首节点，将first赋值给首节点
4. succ的pre不为空，则把succ的上一节点的next指向新节点

## 总结
1.  LinkedList，只有两种添加方式，一种是在列表最后添加(linkLast)，一种是在列表某个元素前面添加 (linkBefore)。
2. LinkLast，首先创建一个新节点，节点pre指向最后一个节点，最后一个节点的next指向新节点。
3. LinkBefore，首先根据index下标获取到元素的位置，新建新节点，新节点的pre就是元素的pre，新节点的next就是该元素。元素的pre的next指向新元素。
## 为何Linked要用双链表而不是单链表
LinkedList为何是双链表，链表主要缺点是查询速度很慢，添加或者删除都要找到要添加和删除的节点，而使用双链表，每次遍历循环前，都会判断一下索引是在链表的前半部分还是后半部分。如果是前半部分的话，从首部遍历到中间。如果是后半部分，从尾部遍历到中间。

