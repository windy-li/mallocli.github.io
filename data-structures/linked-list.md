## 链表

链表 (linked list) 是一种这样的数据结构，其中的各对象按线性顺序排列。数组的线性顺序是由数组下标决定的，然而与数组不同的是，链表的顺序是由各个对象里的指针决定的。

如下图所示，双向链表 (doubly linked list) 的每个元素都是一个对象，每个对象有一个关键字 key 和两个指针：next 和 prev。对象中还可以包含其它的辅助数据（或称卫星数据）。设 x 为链表的一个元素，x.next 指向它在链表中的后继元素，x.prev 指向它的前驱元素。如果 x.prev == null，则元素 x 没有前驱，因此是链表的第一个元素，即链表的头。如果 x.next == null，则元素 x 没有后继，因此是链表的最后一个元素，即链表的尾。属性 head 指向链表的第一个元素，如果 head == null，则链表为空。

![](../assets/images/data-structures/linked-list.png)

（a）表示动态集合 {1, 4, 9, 16} 的双向链表。链表中的每个元素都是一个对象，拥有关键字和指向前后对象的指针（用箭头表示）。表尾的 next 属性和表头的 prev 属性都是 null，用一个斜杠表示。属性 head 指向表头元素。（b）在执行 insert(25) 之后，链表以关键字为 25 的新对象作为新的表头。（c）随后调用 delete(4) 的结果。

链表可以有多种形式，它可以是单链接的或双链接的，可以是已排序的或未排序的，可以是循环的或非循环的。如果一个链表是单链接 (singly linked) 的，则省略每个元素的 prev 指针。如果链表是已排序 (sorted) 的，则链表的线性顺序与链表元素中关键字的线性顺序一致，据此，最小的元素就是表头元素，而最大的元素就是表尾元素。如果链表是未排序的 (unsorted)，则各元素可以可以以任何顺序出现。在循环链表 (circular list) 中，表头元素的 prev 指针指向表尾元素，而表尾元素的 next 指针指向表头元素，我们可以把循环链表想象成一个各元素组成的圆环。在余下的部分，我们所处理的链表都是未排序的且是双链接的。

```java
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
}
```

### 链表的搜索

可以采用简单的线性搜索方法，用于查找链表中第一个关键字为 k 的元素，并返回指向该元素的指针，如果链表中没有关键字为 k 的元素，则返回 null。要搜索一个有 n 个对象的链表，在最坏情况下运行时间为 Θ(n)，因为可能需要搜索整个链表。

```java
Node search(int key) {
    Node node = head;
    while (node != null && node.key != key) {
        node = node.next;
    }
    return node;
}
```

### 链表的插入

下面的代码将一个新元素插入到链表的前端：

```java
void insert(int key) {
    Node node = new Node(key);
    node.prev = null;
    node.next = head;
    if (head != null) {
        head.prev = node;
    }
    head = node;
}
```

### 链表的删除

delete 方法将一个元素 x 从链表中移除，该过程要求给定一个指向 x 的指针，然后通过修改一些指针，将 x 删除出该链表。delete 的运行时间为 O(1)，但如果要删除具有给定关键字的元素，则最坏情况下需要的时间为 Θ(n)，因为需要先调用 search 找到该元素。

```java
void delete(int key) {
    Node node = search(key);
    if (node != null) {
        delete(node);
    }
}

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
```

### 链表的反转

链表的反转就是交换所有的 prev 和 next 指针，head 从原来的头部转移到尾部。下面是反转的的实现，其时间复杂度为 O(n)，除存储链表本身所需的空间外，只使用固定大小的额外存储空间。

```java
void reverse() {
    Node temp = null;
    Node current = head;
    while (current != null) {
        temp = current.prev;
        current.prev = current.next;
        current.next = temp;
        current = current.prev;
    }
    if (temp != null) {
        head = temp.prev;
    }
}
```

### 单向链表

单向链表的 search 和 insert 操作运行时间和双向链表一样，分别是 O(n) 和 O(1)，但单链表的删除操作比双链表慢，双链表只需简单的修改被删除结点的 prev 和 next 指针就能将该结点从链表中删除，运行时间为 O(1)，而单链表需要从 head 开始，沿着 next 指针找到结点所在的位置，通过修改该结点前驱的 next 指针才能将其删除，运行时间为 O(n)。

```java
public class SinglyLinkedList {
    Node head;

    class Node {
        int key;
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

    void insert(int key) {
        Node node = new Node(key);
        node.next = head;
        head = node;
    }

    void delete(Node node) {
        if (node == head) {
            head = head.next;
            return;
        }
        Node prev = head;
        while (prev.next != node) {
            prev = prev.next;
        }
        prev.next = node.next;
    }
    
    void delete(int key) {
        Node node = search(key);
        if (node != null) {
            delete(node);
        }
    }

    void reverse() {
        Node prev = null;
        Node current = head;
        Node next = null;
        while (current != null) {
            next = current.next;
            current.next = prev;
            prev = current;
            current = next;
        }
        head = prev;
    }
}
```
