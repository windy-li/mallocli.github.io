## 快速排序

对于包含n个数的输入数组来说，快速排序是一种最坏情况时间复杂度为Θ(n ^ 2)的排序算法。虽然最坏情况时间复杂度很差，但是快速排序通常是实际排序应用中最好的选择，因为它的平均性能非常好：它的期望运行时间为Θ(n * lgn)，而且Θ(n * lgn)中隐含的常数因子非常小。

### 快速排序的描述

与归并排序一样，快速排序也使用了分治法的思想，下面是对一个典型的子数组arr[p...r]进行快速排序的三步分治过程：

1. 分解：数组arr[p...r]被划分为两个（可能为空）的子数组arr[p...q - 1]和arr[q + 1...r]，使得arr[p...q - 1]中的每个元素都小于等于arr[q]，arr[q + 1...r]中的每个元素都大于arr[q]。其中，计算数组下标q也是划分过程的一部分。
2. 解决：通过递归调用快速排序，对子数组arr[p...q - 1]和arr[q + 1...r]进行排序。
3. 合并：因为子数组都是原址排序的，所以不需要合并操作，数组arr[p...r]已经有序。

下面的程序实现快速排序：

```java
public void quickSort(int[] arr, int p, int r) {
    if (p < r) {
        int q = partition(arr, p, r);
        quickSort(arr, p, q + 1);
        quickSort(arr, q + 1, r);
    }
}

private int partition(int[] arr, int p, int r) {
    int i = p - 1;
    for (int j = p; j <= r - 1; j++) {
        if (arr[j] <= arr[r]) {
            i++;
            swap(arr, i, j);
        }
    }
    swap(arr, i + 1, r);
    return i + 1;
}

private void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
```

为了排序一个数组arr的全部元素，初始调用的是quickSort(arr, 0, arr.length - 1);

算法的关键步骤是partition过程，它实现了对子数组arr[p...r]的原址重排。

partition总是选择一个元素arr[r]作为主元(pivot element)，并围绕它来划分数组arr[p...r]。

在partition的最后两行中，通过将主元与最左的大于主元的元素进行交换，就可以将主元移动到它在数组中的正确位置上，并返回主元的新下标。

partition在子数组arr[p...r]上的时间复杂度是Θ(n)，其中n = r - p + 1。

### 快速排序的性能

快速排序的运行时间依赖于划分是否平衡，而平衡与否又依赖于用于划分的元素。如果划分是平衡的，那么快速排序的性能与归并排序一样。如果划分是不平衡的，那么快速排序的性能就接近于插入排序了。

#### 最坏情况划分

当划分产生的两个子问题分别包含了n - 1和0个元素时，快速排序的最坏情况发生了。如果在算法的每一层递归上，划分都是最大程度不平衡的，那么算法的时间复杂度就是Θ(n ^ 2)。也就是说，在最快情况下，快速排序算法的运行时间并不比插入排序更好。此外，当输入数组已经完全有序时，快速排序的时间复杂度仍然是Θ(n ^ 2)，而在同样情况下，插入排序的时间复杂度为O(n)。

#### 最好情况划分

在可能的最平衡的划分中，partition得到的两个子问题的规模都不大于n/2，这是因为其中的一个规模为Math.floor(n / 2)，而另一个规模为Math.ceil(n / 2) - 1。在这种情况下，快速排序的性能非常好，时间复杂度为O(n * lgn)。

#### 平衡的划分

快速排序的平均运行时间更接近于其最好情况，而非最坏情况。例如，假设划分算法总是产生9:1的划分，乍一看，这种划分是很不平衡的，但快速排序的运行时间是O(n * lgn)，与恰好在在中间划分的渐进运行时间是一样的。实际上，即使是99:1的划分，其时间复杂度仍然是O(n * lgn)。事实上，任何一种常数比例的划分都会产生深度为Θ(lgn)的递归树，其中每一层的代价都是O(n)。因此，只要划分是常数比例的，算法的运行时间总是O(n * lgn)。

#### 对于平均情况的直观观察

当好和差的划分交替出现时，快速排序的时间复杂度与全是好的划分时一样，仍然是O(n * lgn)，区别只是O符号中隐含的常数因子要略大一点。

### 快速排序的随机化版本

在讨论快速排序的平均情况性能的时候，我们的前提假设是：输入数据的所有排列都是等概率的。但是在实际工程中，这个假设并不会总是成立。有时我们可以通过在算法中引入随机性，从而使得算法对于所有的输入都能获得较好的期望性能。很多人都选择随机化版本的快速排序作为大数据输入情况下的排序算法。

与始终采用arr[r]作为主元的方法不同，随机抽样是从子数组arr[p...r]中随机选择一个元素作为主元，为了达到这一目的，首先将arr[r]与从arr[p...r]中随机选出的一个元素交换。通过对序列p...r的随机取样，我们可以保证主元元素arr[r]是等概率地从子数组的r-p+1个元素中选取的。因为主元是随机选取的，我们期望在平均情况下，对输入数组的划分是比较平衡的。

对quickSort和partition的改动非常小，在新的划分程序中，我们只是在真正进行划分前进行一次交换：

```java
public void randomizedQuickSort(int[] arr, int p, int r) {
    if (p < r) {
        int q = randomizedPartition(arr, p, r);
        randomizedQuickSort(arr, p, q);
        randomizedQuickSort(arr, q + 1, r);
    }
}

private int randomizedPartition(int[] arr, int p, int r) {
    int i = p + (int) (Math.random() * (r - p + 1));
    swap(arr, i, r);
    return partition(arr, p, r);
}

private void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
```

### Hoare划分

前面给出的partition算法并不是其最初的版本，下面给出的是最早由C. R. Hoare所设计的快速排序及划分算法：

```java
public void hoareQuickSort(int[] arr, int p, int r) {
    if (p < r) {
        int q = hoarePartition(arr, p, r);
        hoareQuickSort(arr, p, q);
        hoareQuickSort(arr, q + 1, r);
    }
}

private int hoarePartition(int[] arr, int p, int r) {
    int x = arr[p];
    int i = p - 1;
    int j = r + 1;
    while (true) {
        do {
            i++;
        } while (arr[i] < x);
        do {
            j--;
        } while (arr[j] > x);
        if (i < j) {
            swap(arr, i, j);
        } else {
            return j;
        }
    }
}
```

下标i和j可以使我们不会访问在子数组arr[p...r]以外的数组元素，当hoarePartition结束时，它的返回值j满足p <= j < r，当hoarePartition结束时，arr[p...j]中的每一个元素都小于或等于arr[j + 1...r]中的元素。

在前面的partition过程中，主元（原来存储在arr[r]中）是与它所划分的两个分区分离的。与之对应，在hoarePartition中，主元（原来存储在arr[p]中）是存在于分区arr[p...j]或arr[j + 1...r]中的。因为有p <= j < r，所以这一划分总是非平凡的。

### 针对相同元素值的快速排序

在随机化快速排序的分析中，我们假设输入元素的值是互异的，下面我们看看如果这一假设不成立会出现什么情况。

如果所有输入元素都相同，那么随机化快速排序的每次划分步骤都会产生一个不平衡的划分，n-1个元素会全部被划分到数组的左边，因此，快速排序的运行时间是Θ(n ^ 2)。partition返回一个数组下标q，使得arr[p...q-1]中的每一个元素都小于或等于arr[q]，而arr[q+1...r]中的每个元素都大于arr[q]。修改partition代码来构造一个新的partition，它排列arr[p...r]的元素，返回值是两个数组下标q和t，其中p <= q <= t <= r，且有：
* arr[q...t]中的所有元素都相等。
* arr[p...q - 1]中的每个元素都小于arr[q]。
* arr[t + 1...r]中的每个元素都大于arr[q]。

与partition类似，新构造的partition的时间复杂度是Θ(r - p)，只有分区内的元素互不相同的时候才做递归调用。

### 快速排序的栈深度

朴素quickSort算法包含了两个对其自身的递归调用。在调用partition后，quickSort分别调用了左边的子数组和右边的子数组。quickSort第二个递归调用并不是必须的，我们可以用一个循环控制结构来替代它。这一技术称为尾递归，好的编译器都提供这一功能。考虑下面这个版本的快速排序，它模拟了尾递归情况。

```java
public void tailRecursiveQuickSort(int[] arr, int p, int r) {
    while (p < r) {
        int q = partition(arr, p, r);
        tailRecursiveQuickSort(arr, p, q - 1);
        p = q + 1;
    }
}
```

编译器通常使用栈来存储递归执行过程中的相关信息，包括每一次递归调用的参数等。最新调用的信息存在栈的顶部，而第一次调用的信息存在栈的底部。当一个过程被调用时，其相关信息被压入栈中，当它结束时，其信息被弹出。因为我们假设数组参数是用指针来指示的，所以每次过程调用只需要O(1)的栈空间。栈深度是在一次计算中会用到的栈空间的最大值。

通过修改tailRecursiveQuickSort的代码，使其最坏情况下栈深度是Θ(lgn)，并且能够保持O(n * lgn)的期望时间复杂度。

```java
public void modifiedTailRecursiveQuickSort(int[] arr, int p, int r) {
    while (p < r) {
        int q = partition(arr, p, r);
        if (q < (r - p) / 2) {
            modifiedTailRecursiveQuickSort(arr, p, q - 1);
            p = q + 1;
        } else {
            modifiedTailRecursiveQuickSort(arr, q + 1, r);
            r = q - 1;
        }
    }
}
```

### 三数取中划分

一种改进randomizedQuickSort的方法是在划分时，要从子数组中更细致地选择作为主元的元素（而不是简单地随机选择）。常用的做法是三数取中：从子数组中随机选出三个元素，取其中位数作为主元。

```java
private int medianOfThreePartition(int[] arr, int p, int r) {
    int a = p + (int) (Math.random() * (r - p + 1));
    int b = p + (int) (Math.random() * (r - p + 1));
    int c = p + (int) (Math.random() * (r - p + 1));
    int m = medianOfThree(arr, a, b, c);
    swap(arr, m, r);
    return partition(arr, p, r);
}

private int medianOfThree(int[] arr, int a, int b, int c) {
    return arr[a] < arr[b] ? (arr[b] < arr[c] ? b : arr[a] < arr[c] ? c : a)
        : arr[b] > arr[c] ? b : arr[a] > arr[c] ? c : a;
}
```

对于快速排序而言，三数取中法只影响其时间复杂度的常数因子。
