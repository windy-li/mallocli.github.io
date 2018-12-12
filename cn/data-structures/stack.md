## 栈

在栈中，被删除的是最近插入的元素：栈实现的是一种后进先出 (last-in, first-out, LIFO) 策略。

栈上的插入操作称为压入 (push)，而无元素参数的删除操作称为弹出 (pop)。这两个名字使人联想到现实中的栈，比如餐馆里装有弹簧的摞盘子的栈。盘子从栈中弹出的次序刚好同它们被压入的次序相反，这是因为只有最上面的盘子才能被取下来。

可以用一个数组 arr[0...n-1] 来实现一个最多可容纳 n 个元素的栈。该数组有一个属性 top，指向最新插入的元素。栈中包含的元素为 arr[0...top]，其中 arr[0] 是栈底元素，arr[top] 是栈顶元素。当 top = 0 时，栈中不包含任何元素，即栈是空的。当 top = n - 1 时，栈是满的。

![](../../assets/images/data-structures/stack.png)

栈的数组实现。只有出现在浅灰色格子里的才是栈元素。（a）栈有 4 个元素，栈顶元素为 9。（b）调用 push(17) 和 push(3) 后的栈。（c）调用 pop() 并返回最后压入的元素 3 的栈。虽然元素 3 仍在数组里，但它已不在栈内了，此时在栈顶的是元素 17。

```java
public class Stack {
    int top;
    int[] keys;
    
    Stack(int capacity) {
        top = -1;
        keys = new int[capacity];
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

栈的 push 和 pop 操作的执行时间都是 O(1)。
