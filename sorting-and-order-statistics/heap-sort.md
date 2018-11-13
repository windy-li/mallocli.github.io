## 堆排序

与归并排序一样，但不同于插入排序的是，堆排序的时间复杂度是 O(n * lgn)。而与插入排序相同，但不同于归并排序的是，堆排序同样具有空间原址性：任何时候都只需要常数个额外的元素空间存储临时数据。因此，堆排序是集合了插入排序和归并排序优点的一种排序算法。

堆排序引入了另一种算法设计技巧，使用了一种称为“堆”的数据结构。堆不仅用在堆排序上，而且它还可以构造一种有效的优先队列。

虽然“堆”这一词源自堆排序，但是目前它已经被引申为“垃圾收集存储机制”，例如在 Java 语言中所定义的。在这里，我们使用的堆不是垃圾收集存储，而是堆数据结构。

### 堆

（二叉）堆是一个数组，它可以被看成是一个近似的完全二叉树，树上的每一个节点对应数组中的一个元素。除了最底层外，该树是完全充满的，而且是从左向右填充。表示堆的数组 arr 包括两个属性：length 给出数组元素的个数，heapSize 表示有多少个堆元素存储在该数组中。也就是说，虽然 arr[0...arr.length-1] 可能都存有数据，但只有 arr[0...heapSize-1] 中存放的是堆的有效元素，这里 0 <= heapSize <= arr.length。树的根节点是 arr[0]。给定一个节点的下标 i，我们很容易计算得到它的父节点、左孩子和右孩子的下标。

```
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

二叉堆可以分为两种形式：最大堆和最小堆。在这两种堆中，节点的值都要满足堆的性质，但一些细节定义则有所差异。在最大堆中，最大堆性质是指除了根以外的所有节点 i 都要满足：

arr[parent(i)] >= arr[i]

也就是说，某个节点的值至多与其父节点一样大。因此，堆的最大元素存放在根节点中。并且，在任一子树中，该树所包含的所有节点的值都不大于该树根节点的值。最小堆的组织方式正好相反：最小堆性质是指除了个根节点以外所有节点 i 都有：

arr[parent(i)] <= arr[i]

在堆排序算法中，我们使用的是最大堆。最小堆通常用于构造优先队列。

如果把堆看成是一棵树，我们定义一个堆中的节点的高度就为该节点到叶节点最长简单路径上边的数目，进而我们可以把堆的高度定义为根节点的高度。既然一个包含 n 个元素的堆可以看做一棵完全二叉树，那么该堆的高度是 Θ(lgn)。我们会发现，堆结构上的一些基本操作的运行时间至多与堆的高度成正比，即时间复杂度为 O(lgn)。

* void maxHeapify(int i) 时间复杂度为 O(lgn)，它是维护最大堆性质的关键。

* void buildMaxHeap(int[] arr) 具有线性时间复杂度，功能是从无序的输入数组中构造一个最大堆。

* void heapSort(int[] arr) 时间复杂度为 O(n * lgn)，功能是对一个数组进行原址排序。

* void insert(int key)、int extractMax()、void increaseKey(int i, int newKey) 和 int minimum() 时间复杂度为 O(lgn)，功能是利用堆实现一个优先队列。

### 维护堆的性质

maxHeapify 是维护最大堆性质的关键，它的输入为一个下标 i。在调用 maxHeapify 的时候，我们假定根节点为 left(i) 和 right(i) 的二叉树都是最大堆，但这时 arr[i] 有可能小于其孩子，这样就违背了最大堆的性质。maxHeapify 通过让 arr[i] 的值在最大堆中逐级下降，从而使得以下标 i 为根节点的子树重新遵循最大堆的性质。

```
void maxHeapify(int i) {
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
        swap(arr, largest, i);
        maxHeapify(largest);
    }
}

void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
```

在程序执行的每一步中，从 arr[i]、arr[left(i)] 和 arr[right(i)] 中选出最大的，并将其下标存储在 largest 中。如果 arr[i] 是最大的，那么以 i 为根节点的子树已经是最大堆，程序结束。否则，最大元素是 i 的某个孩子节点，则交换 arr[i] 和 arr[largest] 的值，从而使 i 及其孩子都满足最大堆的性质。在交换后，下标为 largest 的节点的值是原来的 arr[i]，于是以该节点为根的子树又可能会违反最大堆性质。因此，需要对该子树递归调用 maxHeapify。

maxHeapify 的代码效率较高，但递归调用可能除外，它可能使某些编译器产生低效的代码，可以使用循环控制结构取代递归，重写 maxHeapify 如下：

```
void maxHeapify(int[] arr, int i) {
    while (true) {
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
            i = largest;
        } else {
            return;
        }
    }
}
```

### 建堆

我们可以使用自底向上的方式利用 maxHeapify 把一个长度为 n 的数组转化为最大堆，子数组 arr[n/2 ... n-1] 的元素都是树的叶节点。每个叶节点都可以看成只包含一个元素的堆。

```
void buildMaxHeap(int[] arr) {
    int n = arr.length;
    heapSize = n;
    for (int i = n / 2 - 1; i >= 0; i--) {
        maxHeapify(arr, i);
    }
}
```

### 堆排序算法

初始时候，堆排序算法利用 buildMaxHeap 将输入数组 arr[0...n-1] 建成最大堆，其中 n = arr.length。因为数组中的最大元素总在根节点 arr[0] 中，通过把它与 arr[n - 1] 进行互换，我们可以让该元素放到正确的位置。这时候，我们从堆中去掉节点 n - 1，这一操作可以通过减少 heapSize 的值来实现，剩余的节点中，原来根的孩子节点仍然是最大堆，而新的根节点可能会违背最大堆的性质。为了维护最大堆的性质，我们要做的是调用 maxHeapify(arr, 0)，从而在 arr[0...n - 2] 上构造一个新的最大堆。堆排序算法不断重复这一过程，直到堆的大小从 n 降到 1。

```
void heapSort(int[] arr) {
    buildMaxHeap(arr);
    for (int i = arr.length - 1; i >= 1; i--) {
        swap(arr, 0, i);
        heapSize--;
        maxHeapify(arr, 0);
    }
}
```

heapSort 的时间复杂度是 O(n * lgn)，因为每次调用 buildMaxHeap 的时间复杂度是 O(n)，而 n - 1 次调用 maxHeapify，每次的时间是 O(lgn)。
