## 栈和队列

栈和队列都是动态集合，在栈中，被删除的是最近插入的元素：栈实现的是一种后进先出 (last-in, first-out, LIFO) 策略。类似地，在队列 (queue) 中，被删除的总是在集合中存在时间最长的那个元素：队列实现的是一种先进先出 (first-in, first-out, FIFO) 策略。

### 栈

栈上的插入操作称为压入 (push)，而无元素参数的删除操作称为弹出 (pop)。这两个名字使人联想到现实中的栈，比如餐馆里装有弹簧的摞盘子的栈，盘子从栈中弹出的次序刚好同它们被压入的次序相反，这是因为只有最上面的盘子才能被取下来。

可以用一个数组 arr[0...n-1] 来实现一个最多可容纳 n 个元素的栈。该数组有一个属性 top，指向最新插入的元素。栈中包含的元素为 arr[0...top]，其中 arr[0] 是栈底元素，arr[top] 是栈顶元素。当 top = -1 时，栈中不包含任何元素，即栈是空的。当 top = n - 1 时，栈是满的。

![](../assets/images/data-structures/stack.png)

栈的数组实现。只有出现在浅灰色格子里的才是栈元素。（a）栈有 4 个元素，栈顶元素为 9。（b）调用 push(17) 和 push(3) 后的栈。（c）调用 pop() 并返回最后压入的元素 3 的栈。虽然元素 3 仍在数组里，但它已不在栈内了，此时在栈顶的是元素 17。

```java
public class Stack {
    int[] keys;
    int top;
    
    Stack(int capacity) {
        keys = new int[capacity];
        top = -1;
    }
    
    boolean isEmpty() {
        return top == -1;
    }
    
    boolean isFull() {
        return top == keys.length - 1;
    }
    
    void push(int key) {
        if (isFull()) {
            throw new RuntimeException("stack overflow");
        }
        keys[++top] = key;
    }
    
    int pop() {
        if (isEmpty()) {
            throw new RuntimeException("stack underflow");
        }
        return keys[top--];
    }
}
```

### 队列

队列上的插入操作称为入队 (enqueue)，删除操作称为出队 (dequeue)。正如栈的 pop 操作一样，dequeue 操作也没有元素参数。队列的先进先出特性类似于收银台前排队等待结账的一排顾客。队列有队头 (head) 和队尾 (tail)，当有一个元素入队时，它被放在队尾的位置，就像一个新到来的顾客排在队伍的末端一样，而出队的元素则总是在队头的那个，就像排在队伍前面等待最久的那个顾客一样。

![](../assets/images/data-structures/queue.png)

利用数组 arr[0...11] 实现一个队列。只有出现在浅灰色格子里的才是队列的元素。（a）列队包含 5 个元素，位于 arr[6...10]。（b）依次调用 enqueue(17)、enqueue(3) 和 enqueue(5) 后队列的构成。（c）在调用 dequeue() 并返回原队头的关键字值 15 后，队列的构成，此时新的队头元素为 6。

可以用数组 arr[0...n-1] 来实现一个最多容纳 n - 1 个元素的队列。该队列有一个属性 head 指向队头元素，而属性 tail 指向下一个新元素将要插入的位置。队列中的元素存放在位置 head, head + 1, head + 2, ..., tail - 1，并在最后的位置环绕，感觉好像位置 0 紧邻在位置 n - 1 后面行成一个环。当 head = tail 时，队列为空，初始时有 head = tail = 0，当 head = tail + 1 或者 head = 0 且 tail = n - 1 时，队列是满的。

```java
public class Queue {
    int[] keys;
    int head;
    int tail;
    
    Queue(int capacity) {
        keys = new int[capacity + 1];
        head = 0;
        tail = 0;
    }
    
    boolean isEmpty() {
        return head == tail;
    }
    
    boolean isFull() {
        return (head == tail + 1) || (head == 0 && tail == keys.length - 1);
    }
    
    void enqueue(int key) {
        if (isFull()) {
            throw new RuntimeException("queue overflow");
        }
        keys[tail] = key;
        if (tail == keys.length - 1) {
            tail = 0;
        } else {
            tail++;
        }
    }
    
    int dequeue() {
        if (isEmpty()) {
            throw new RuntimeException("queue underflow");
        }
        int res = keys[head];
        if (head == keys.length - 1) {
            head = 0;
        } else {
            head++;
        }
        return res;
    }
}
```

队列的 enqueue 和 dequeue 操作的执行时间都为 O(1)。
