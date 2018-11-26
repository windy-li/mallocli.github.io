## 链表

链表 (linked list) 是一种这样的数据结构，其中的各对象按线性顺序排列。数组的线性顺序是由数组下标决定的，然而与数组不同的是，链表的顺序是由各个对象里的指针决定的。

双向链表 (doubly linked list) 的每个元素都是一个对象，每个对象有一个关键字 key 和两个指针：next 和 prev。对象中还可以包含其它的辅助数据（或称卫星数据）。设 x 为链表的一个元素，x.next 指向它在链表中的后继元素，x.prev 指向它的前驱元素。如果 x.prev == null,则元素 x 没有前驱，因此是链表的第一个元素，即链表的头。如果 x.next == null，则元素 x 没有后继，因此是链表的最后一个元素，即链表的尾。

链表可以有多种形式，它可以是单链接的或双链接的，可以是已排序的或未排序的，可以是循环的或非循环的。如果一个链表是单链接 (singly linked) 的，则省略每个元素的 prev 指针。如果链表是已排序 (sorted) 的，则链表的线性顺序与链表元素中关键字的线性顺序一致，据此，最小的元素就是表头元素，而最大的元素就是表尾元素。如果链表是未排序的，则各元素可以可以以任何顺序出现。在循环链表 (circular list) 中，表头元素的 prev 指针指向表尾元素，而表尾元素的 next 指针指向表头元素，我们可以把循环链表想象成一个各元素组成的圆环。在余下的部分，我们所处理的链表都是未排序的且是双链接的。

### 链表的搜索

可以采用简单的线性搜索方法，用于查找链表中第一个关键字为k的元素，并返回指向该元素的指针，如果链表中没有关键字为k的元素，则返回 null。要搜索一个有 n 个对象的链表，在最坏情况下运行时间为 Θ(n)，因为可能需要搜索整个链表。

```
public class LinkedList {
    Node head;
    
    class Node {
        int key;
        Node prev;
        Node next;
        
        Node(int key) {
            this.key = key;
        }
    }
    
    Node search(int key) {
        Node node = head;
        while (node != null && node.key != key) {
            node = node.next;
        }
        return node;
    }
}
```

### 链表的插入

下面的代码将一个新元素插入到链表的前端：

```
void insert(int key) {
    Node newNode = new Node(key);
    newNode.next = head;
    if (head != null) {
        head.prev = newNode;
    }
    head = newNode;
    newNode.prev = null;
}
```

### 链表的删除

delete 方法将一个元素 x 从链表中移除，该过程要求给定一个指向 x 的指针，然后通过修改一些指针，将 x 删除出该链表。delete 的运行时间为 O(1)，但如果要删除具有给定关键字的元素，则最坏情况下需要的时间为 Θ(n)，因为需要先调用 search 找到该元素。

```
void delete(Node node) {
    if (node.prev != null) {
        node.prev.next = node.next;
    } else {
        head = node.next;
    }
    if (node.next != null) {
        node.next.prev = node.prev;
    }
}

void delete(int key) {
    Node node = search(key);
    if (node != null) {
        delete(node);
    }
}
```
