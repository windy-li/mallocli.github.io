## 堆排序

与归并排序一样，但不同于插入排序的是，堆排序的时间复杂度是 O(n * lgn)。而与插入排序相同，但不同于归并排序的是，堆排序同样具有空间原址性：任何时候都只需要常数个额外的元素空间存储临时数据。因此，堆排序是集合了插入排序和归并排序优点的一种排序算法。

堆排序引入了另一种算法设计技巧，使用了一种称为“堆”的数据结构。堆不仅用在堆排序上，而且它还可以构造一种有效的优先队列。

虽然“堆”这一词源自堆排序，但是目前它已经被引申为“垃圾收集存储机制”，例如在 Java 语言中所定义的。在这里，我们使用的堆不是垃圾收集存储，而是堆数据结构。

### 堆

（二叉）堆是一个数组，它可以被看成是一个近似的完全二叉树，树上的每一个节点对应数组中的一个元素。除了最底层外，该树是完全充满的，而且是从左向右填充。表示堆的数组 arr 包括两个属性：length 给出数组元素的个数，heapSize 表示有多少个堆元素存储在该数组中。也就是说，虽然 arr[0...arr.length-1] 可能都存有数据，但只有 arr[0...heapSize-1] 中存放的是堆的有效元素，这里 0 <= heapSize <= arr.length。树的根节点是 arr[0]。给定一个节点的下标 i，我们很容易计算得到它的父节点、左孩子和右孩子的下标。

```java
int parent(int i) {
    return (i - 1) / 2;
}

int left(int i) {
    return 2 * i + 1;
}

int right(int i) {
    return 2 * i + 2;
}
```

![](../assets/images/sorting-and-order-statistics/heap-sort.png)

上图是以（a）二叉树和（b）数组形式展现的一个最大堆。每个结点圆圈内部的数字是它所存储的数据。结点上方的数字是它在数组中相应的下标。数组上方和下方的连线显示的是父-子关系：父结点总是在它的孩子结点的左边。该树的高度为 3，下标为 4（值为8）的结点的高度为 1。

二叉堆可以分为两种形式：最大堆和最小堆。在这两种堆中，节点的值都要满足堆的性质，但一些细节定义则有所差异。在最大堆中，最大堆性质是指除了根以外的所有节点 i 都要满足：

```
arr[parent(i)] >= arr[i]
```

也就是说，某个节点的值至多与其父节点一样大。因此，堆的最大元素存放在根节点中。并且，在任一子树中，该树所包含的所有节点的值都不大于该树根节点的值。最小堆的组织方式正好相反：最小堆性质是指除了个根节点以外所有节点 i 都有：

```
arr[parent(i)] <= arr[i]
```

在堆排序算法中，我们使用的是最大堆。最小堆通常用于构造优先队列。

如果把堆看成是一棵树，我们定义一个堆中的节点的高度就为该节点到叶节点最长简单路径上边的数目，进而我们可以把堆的高度定义为根节点的高度。既然一个包含 n 个元素的堆可以看做一棵完全二叉树，那么该堆的高度是 Θ(lgn)。我们会发现，堆结构上的一些基本操作的运行时间至多与堆的高度成正比，即时间复杂度为 O(lgn)。

* void maxHeapify(int i) 时间复杂度为 O(lgn)，它是维护最大堆性质的关键。

* void buildMaxHeap(int[] arr) 具有线性时间复杂度，功能是从无序的输入数组中构造一个最大堆。

* void heapSort(int[] arr) 时间复杂度为 O(n * lgn)，功能是对一个数组进行原址排序。

* void insert(int key)、int extractMax()、void increaseKey(int i, int newKey) 和 int maximum() 时间复杂度为 O(lgn)，功能是利用堆实现一个优先队列。

### 维护堆的性质

maxHeapify 是维护最大堆性质的关键，它的输入为一个下标 i。在调用 maxHeapify 的时候，我们假定根节点为 left(i) 和 right(i) 的二叉树都是最大堆，但这时 arr[i] 有可能小于其孩子，这样就违背了最大堆的性质。maxHeapify 通过让 arr[i] 的值在最大堆中逐级下降，从而使得以下标 i 为根节点的子树重新遵循最大堆的性质。

```java
void maxHeapify(int[] arr, int i) {
    int l = left(i);
    int r = right(i);
    int largest = i;
    if (l < heapSize && arr[l] > arr[i]) {
        largest = l;
    }
    if (r < heapSize && arr[r] > arr[largest]) {
        largest = r;
    }
    if (largest != i) {
        Util.swap(arr, largest, i);
        maxHeapify(arr, largest);
    }
}
```

下图展示了 maxHeapify 的执行过程。在程序的每一步中，从 arr[i]、arr[left(i)] 和 arr[right(i)] 中选出最大的，并将其下标存储在 largest 中。如果 arr[i] 是最大的，那么以 i 为根结点的子树已经是最大堆，程序结束。否则，最大元素是 i 的某个孩子结点，交换 arr[i] 和 arr[largest] 的值，从而使 i 及其孩子都满足最大堆的性质。在交换后，下标为 largest 的结点的值是原来的 arr[i]，于是以该结点为根结点的子树又有可能会违反最大堆的性质，因此，需要对该子树递归调用 maxHeapify。

![](../assets/images/sorting-and-order-statistics/heap-sort2.png)

对于一棵以 i 为根结点、大小为 n 的子树， maxHeapify 的时间代价包括：调整 arr[i]、arr[left(i)] 和 arr[right(i)] 的关系的时间代价 Θ(1)，加上在一棵以 i 为根结点的子树上运行 maxHeapify 的时间代价（这里假设递归调用会发生），而 maxHeapify 最多运行的次数就是树高，因此，对于一个树高为 h 的结点来说， maxHeapify 的时间复杂度是 O(h)。

maxHeapify 的代码效率较高，但递归调用可能除外，它可能使某些编译器产生低效的代码，可以使用循环控制结构取代递归，重写 maxHeapify 如下：

```java
void maxHeapify(int[] arr, int i) {
    while (true) {
        int l = left(i);
        int r = right(i);
        int largest = i;
        if (l < heapSize && arr[l] > arr[i]) {
            largest = l;
        }
        if (r < heapSize && arr[r] > arr[largest]) {
            largest = r;
        }
        if (largest != i) {
            Util.swap(arr, i, largest);
            i = largest;
        } else {
            return;
        }
    }
}
```

### 建堆

我们可以使用自底向上的方式利用 maxHeapify 把一个长度为 n 的数组转化为最大堆，子数组 arr[n/2 ... n-1] 的元素都是树的叶节点，每个叶节点都可以看成只包含一个元素的堆。

```java
void buildMaxHeap(int[] arr) {
    int n = arr.length;
    heapSize = n;
    for (int i = n / 2 - 1; i >= 0; i--) {
        maxHeapify(arr, i);
    }
}
```

我们可以在线性时间内，把一个无序数组构造成一个最大堆。下图是 buildMaxHeap 的操作过程。

![](../assets/images/sorting-and-order-statistics/heap-sort3.png)

### 堆排序算法

初始时候，堆排序算法利用 buildMaxHeap 将输入数组 arr[0...n-1] 建成最大堆，其中 n = arr.length。因为数组中的最大元素总在根节点 arr[0] 中，通过把它与 arr[n - 1] 进行互换，我们可以让该元素放到正确的位置。这时候，我们从堆中去掉节点 n - 1，这一操作可以通过减少 heapSize 的值来实现，剩余的节点中，原来根的孩子节点仍然是最大堆，而新的根节点可能会违背最大堆的性质。为了维护最大堆的性质，我们要做的是调用 maxHeapify(arr, 0)，从而在 arr[0...n - 2] 上构造一个新的最大堆。堆排序算法不断重复这一过程，直到堆的大小从 n 降到 1。

```java
void heapSort(int[] arr) {
    buildMaxHeap(arr);
    for (int i = arr.length - 1; i > 0; i--) {
        Util.swap(arr, 0, i);
        heapSize--;
        maxHeapify(arr, 0);
    }
}
```

heapSort 的时间复杂度是 O(n * lgn)，因为每次调用 buildMaxHeap 的时间复杂度是 O(n)，而 n - 1 次调用 maxHeapify，每次的时间是 O(lgn)。

下图是 heapSort 的运行过程。（a）用 buildMaxHeap 构造得到的最大堆。（b）~（j）每次调用 maxHeapify 后得到的最大堆，并标识当次的 i 值，其中，仅仅浅色阴影的结点被保留在堆中。（k）最终数组的排序结果。

![](../assets/images/sorting-and-order-statistics/heap-sort4.png)

### 优先队列

堆排序是一个优秀的算法，但在实际应用中，快速排序的性能一般会优于堆排序。尽管如此，堆这一数据结构仍然有很多应用，下面我们将介绍堆的一个常见应用：作为高效的优先队列。和堆一样，优先队列也有两种形式：最大优先队列和最小优先队列。在这里，我们关注于如何基于最大堆实现最大优先队列。

```java
class MaxPriorityQueue {
    int heapSize;
    int[] arr;
    
    MaxPriorityQueue(int capacity) {
        heapSize = 0;
        arr = new int[capacity];
    }
}
```

优先队列（priority queue）是一种用来维护由一组元素构成的集合 S 的数据结构，其中的每一个元素都有一个相关的值，称为关键字（key）。一个最大优先队列支持以下操作：

* void insert(int key) 把元素 x 插入到集合 S 中，这一操作等价于 S = S ∪ {x}。

* int maximum() 返回 S 中具有最大关键字的元素。

* int extractMax() 去掉并返回 S 中具有最大关键字的元素。

* void increaseKey(int i, int newKey) 将下标为 i 的元素关键字值增加到 newKey。这里假设 newKey 不小于该元素的原关键字值。

下面的过程可以在 Θ(1) 的时间内实现 maximum 操作。

```java
int maximum() {
    return arr[0];
}
```

extractMax 的时间复杂度为 O(lgn)，因为除了时间复杂度为 O(lgn) 的 maxHeapify 以外，它的其它操作都是常数阶的。

```java
int extractMax() {
    if (heapSize == 0) {
        throw new RuntimeException("heap underflow");
    }
    int max = arr[0];
    Util.swap(arr, 0, heapSize - 1);
    heapSize--;
    maxHeapify(0);
    return max;
}
```

在优先队列中，我们希望增加关键字的元素由对应的数组下标 i 来标识。首先需要将元素 arr[i] 的关键字更新为新值，因为增大 arr[i] 的关键字可能会违反最大堆的性质，所以可以采用类似插入排序的方式，在从当前结点到根结点的路径上，为新增的关键字寻找恰当的插入位置。在 increaseKey 的操作过程中，当前元素会不断地与其父结点进行比较，如果当前结点的关键字较大，则当前结点与父结点进行交换。这一过程会不断重复，直到当前结点的关键字小于其父结点时终止，因为此时已经重新符合了最大堆的性质。下图显示了 increaseKey 的操作过程，在包含 n 个元素的堆上， increaseKey 的时间复杂度是 O(lgn)，因为做了关键字更新的结点到根结点的路径长度为 O(lgn)。

```java
void increaseKey(int i, int key) {
    if (key < arr[i]) {
        throw new RuntimeException("new key is less than current key");
    }
    while (parent(i) >= 0 && arr[parent(i)] < key) {
        arr[i] = arr[parent(i)];
        i = parent(i);
    }
    arr[i] = key;
}
```

![](../assets/images/sorting-and-order-statistics/heap-sort5.png)

（a）图中的最大堆，下标为 i 的结点以深色阴影表示。（b）该结点的关键字增加到 15。（c）经过 while 循环的第一次迭代，该结点与其父结点交换关键字，同时下标 i 的指示上移到其父结点。（d）经过再一次迭代后得到的最大堆，此时， arr[parent(i)] >= arr[i]，最大堆的性质成立，程序终止。

insert 首先通过增加一个关键字为 -∞ 的叶结点来扩展最大堆，然后调用 increaseKey 为新结点设置对应的关键字，同时保持最大堆的性质。在包含 n 个元素的堆上， insert 的运行时间为 O(lgn)。

```java
void insert(int key) {
    if (heapSize == arr.length) {
        throw new RuntimeException("heap overflow");
    }
    heapSize++;
    arr[heapSize - 1] = Integer.MIN_VALUE;
    increaseKey(heapSize - 1, key);
}
```

总之，在一个包含 n 个元素的堆中，所有优先队列的操作都可以在 O(lgn) 时间内完成。
