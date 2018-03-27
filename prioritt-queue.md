## 优先队列

堆排序是一个优秀的算法，但是在实际应用中，快速排序的性能一般会优于堆排序。尽管如此，堆这一数据结构仍然有很多应用，比如优先队列。和堆一样，优先队列也有两种形式：最大优先队列和最小优先队列，这里，我们关注于如何基于最大堆实现最大优先队列。

```java
public class MaxPriorityQueue {
    private int heapSize;
    private int[] arr;
    
    public MaxPriorityQueue(int capacity) {
        heapSize = 0;
        arr = new int[capacity];
    }
}
```

maximum在O(1)时间内返回堆中最大的元素。

```java
public int maximumn() {
    return arr[0];
}
```

extractMax的时间复杂度是O(lgn)，它的作用是返回堆中最大的元素并重新维持最大堆性质。

```java
public int extractMax() {
    if (heapSize < 1) {
        throw new RuntimeException("heap underflow");
    }
    int max = arr[0];
    arr[0] = arr[heapSize - 1];
    heapSize--;
    maxHeapify(0);
    return max;
}

private void maxHeapify(int i) {
    int l = left(i);
    int r = right(i);
    int largest;
    if (l < heapSize && arr[l] > arr[i]) {
        largest = l;
    } else {
        largest = i;
    }
    if (r < heapSize && arr[r] > arr[largest]) {
        largest = r;
    }
    if (largest != i) {
        swap(arr, i, largest);
        maxHeapify(largest);
    }
}

private int parent(int i) {
    return (i - 1) / 2;
}

private int left(int i) {
    return 2 * i + 1;
}

private int right(int i) {
    return 2 * i + 2;
}

private void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
```

在优先队列中，我们希望增加关键字的优先队列元素由对应的数组下标i来标识，这一操作需要首先将元素arr[i]的关键字更新为新值。因为增大arr[i]的关键字可能会违反最大堆的性质，所以，采用类似于插入排序的方式，在从当前节点到根结点的路径上，为新增的关键字寻找恰当的插入位置。当前节点会不断地与其父结点进行比较，如果当前元素的关键字较大，则当前元素与其父节点进行交换。这一过程会不断地重复，直到当前元素的关键字小于其父结点时终止，因此此时已经重新符合了最大对的性质。

```java
public void increaseKey(int i, int newKey) {
    if (newKey < arr[i]) {
        throw new IllegalArgumentException("new key is smaller than current key");
    }
    arr[i] = newKey;
    while (i > 0 && arr[parent(i) < arr[i]]) {
        swap(arr, i, parent(i));
        i = parent(i);
    }
}
```

在while循环内的swap操作，需要通过三次赋值来完成，可以利用insertion sort内循环部分的思想，只用一次赋值就完成这一操作：

```java
public void increaseKey(int i, int newKey) {
    if (newKey < arr[i]) {
        throw new IllegalArgumentException("new key is smaller than current key");
    }
    while (i > 0 && arr[parent(i) < newKey]) {
        arr[i] = arr[parent(i)];
        i = parent(i);
    }
    arr[i] = newKey;
}
```

最大堆的插入操作首先通过增加一个关键字为Integer.MIN_VALUE的叶节点来扩展最大堆，然后调用increaseKey为新结点设置对应的关键字，同时保持最大堆的性质。

```java
public void insert(int key) {
    if (heapSize == arr.length) {
        throw new RuntimeException("heap overflow");
    }
    heapSize++;
    arr[heapSize - 1] = Integer.MIN_VALUE;
    increaseKey(heapSize - 1, key);
}
```

在包含n个元素的堆上，insert的运行时间为O(lgn)。

总之，在一个包含n个元素的堆中，所有优先队列的操作都可以在O(lgn)时间内完成。
