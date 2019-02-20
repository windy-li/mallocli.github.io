## 快速排序

对于包含 n 个数的输入数组来说，快速排序是一种最坏情况时间复杂度为 Θ(n<sup>2</sup>) 的排序算法。虽然最坏情况时间复杂度很差，但是快速排序通常是实际排序应用中最好的选择，因为它的平均性能非常好：它的期望运行时间为 Θ(n lgn)，而且 Θ(n lgn) 中隐含的常数因子非常小。另外，它还能进行原址排序。

### 快速排序的描述

与归并排序一样，快速排序也使用了分治法的思想，下面是对一个典型的子数组 arr[p...r] 进行快速排序的三步分治过程：

1. 分解：数组 arr[p...r] 被划分为两个（可能为空）的子数组 arr[p...q-1] 和 arr[q+1...r]，使得 arr[p...q-1] 中的每个元素都小于等于 arr[q]，arr[q+1...r] 中的每个元素都大于 arr[q]。其中，计算数组下标 q 也是划分过程的一部分。

2. 解决：通过递归调用快速排序，对子数组 arr[p...q-1] 和 arr[q+1...r] 进行排序。

3. 合并：因为子数组都是原址排序的，所以不需要合并操作，数组 arr[p...r] 已经有序。

下面的程序实现快速排序：

```java
void quickSort(int[] arr, int p, int r) {
    if (p < r) {
        int q = partition(arr, p, r);
        quickSort(arr, p, q - 1);
        quickSort(arr, q + 1, r);
    }
}

int partition(int[] arr, int p, int r) {
    int i = p - 1;
    for (int j = p; j < r; j++) {
        if (arr[j] <= arr[r]) {
            i++;
            Util.swap(arr, i, j);
        }
    }
    Util.swap(arr, i + 1, r);
    return i + 1;
}
```

算法的关键步骤是 partition 过程，它实现了对子数组 arr[p...r] 的原址重排。下图显示了 partition 如何在一个包含 8 个元素的数组上进行操作的过程。partition 总是选择一个元素 x = arr[r] 作为主元（pivot element），并围绕它来划分数组 arr[p...r]。随着程序的执行，数组被划分为 4 个（可能有空的）区域，for 循环的每一轮迭代的开始，每一个区域都满足一定的性质，对于任意数组下标 k，有：

1. 若 p <= k <= i，则 arr[k] <= x。

2. 若 i + 1 <= k <= j - 1，则 arr[k] > x。

3. 若 k == r，则 arr[k] == x。

但是上述三种情况没有覆盖下标 j 到 r - 1，对应位置的值与主元之间也不存在特定的大小关系。

![](../assets/images/sorting-and-order-statistics/quick-sort.png)

上图是在一个样例数组上的 partition 操作过程，数组项 arr[r] 是主元 x，浅阴影部分的数组元素都在划分的第一部分，其值都不大于 x，深阴影部分的元素都在划分的第二部分，其值都大于 x。无阴影的元素则是还未分入这两个部分中的任意一个。最后的白色元素就是主元 x。（a）初始的数组和变量设置，数组元素均未被放入前两个部分中的任何一个。（b）2 与它自身进行交换，并被放入了元素值较小的那个部分。（c）~（d）8 和 7 被添加到元素值较大的那个部分中。（e）1 和 8 进行交换，数值较小的部分规模增加。（f）数值 3 和 7 进行交换，数值较小的部分规模增加。（g）~（h）5 和 6 被包含进较大部分，循环结束。（i）通过将主元与最左的大于主元的元素进行交换，就可以将主元移动到它在数组中的正确位置上，这样主元就位于两个部分之间，并返回主元的新下标。

![](../assets/images/sorting-and-order-statistics/quick-sort2.png)

在子数组 arr[p...r] 上，partition 维护了上图所示的 4 个区域。arr[p...i] 区间内的所有值都小于等于 x，arr[i+1...j-1] 区间内的所有值都大于 x。子数组 arr[j...r-1] 中的值可能属于任何一种情况。

partition 在子数组 arr[p...r] 上的时间复杂度是 Θ(n)，其中 n = r - p + 1。

### 快速排序的性能

快速排序的运行时间依赖于划分是否平衡，而平衡与否又依赖于用于划分的元素。如果划分是平衡的，那么快速排序的性能与归并排序一样。如果划分是不平衡的，那么快速排序的性能就接近于插入排序了。

#### 最坏情况划分

当划分产生的两个子问题分别包含了 n - 1 和 0 个元素时，快速排序的最坏情况发生了。如果在算法的每一层递归上，划分都是最大程度不平衡的，那么算法的时间复杂度就是 Θ(n<sup>2</sup>)。也就是说，在最坏情况下，快速排序算法的运行时间并不比插入排序更好。此外，当输入数组已经完全有序时，快速排序的时间复杂度仍然是 Θ(n<sup>2</sup>)，而在同样情况下，插入排序的时间复杂度为 O(n)。

#### 最好情况划分

在可能的最平衡的划分中，partition 得到的两个子问题的规模都不大于 n / 2，这是因为其中的一个规模为 floor(n / 2)，而另一个规模为 ceil(n / 2) - 1。在这种情况下，快速排序的性能非常好，时间复杂度为 O(n lgn)。

#### 平衡的划分

快速排序的平均运行时间更接近于其最好情况，而非最坏情况。例如，假设划分算法总是产生 9 : 1 的划分，乍一看，这种划分是很不平衡的，此时，我们得到的快速排序时间复杂度的递归式为：

```
T(n) = T(9n/10) + T(n/10) + cn
```

这里，我们显式地写出了 Θ(n) 项中所隐藏的常数 c，下图显示了这一递归调用所对应的递归树。注意，树中每一层的代价都是 cn，直到在深度 log<sub>10</sub><sup>n</sup> = Θ(lgn) 处达到递归的边界条件时为止，之后每层代价至多为 cn，递归在深度为 log<sub>10/9</sub><sup>n</sup> = Θ(lgn) 处终止。因此，快速排序的总代价为 O(n lgn)，与恰好在中间划分的渐近运行时间是一样的。实际上，即使是 99 : 1 的划分，其时间复杂度仍然是 O(n lgn)。事实上，任何一种常数比例的划分都会产生深度为 Θ(lgn) 的递归树，其中每一层的代价都是 O(n)。因此，只要划分是常数比例的，算法的运行时间总是 O(n lgn)。

![](../assets/images/sorting-and-order-statistics/quick-sort3.png)

上图是 quickSort 的一棵递归树，其中 partition 总是产生 9 : 1 的划分，该树的时间复杂度为 O(n lgn)。每个结点的值表示子问题的规模，每一层的代价显示在最右边，每一层的代价包含了 Θ(n) 项中隐含的常数 c。

#### 对于平均情况的直观观察

为了对快速排序的各种随机情况有一个清楚的认识，我们需要对遇到的各种输入的出现频率做出假设。快速排序的行为依赖于输入数组中元素的相对顺序，而不是某些特定值本身。

当对一个随机输入的数组运行快速排序时，想要像前面非形式化分析中所假设的那样，在每一层上都有同样的划分是不太可能的，我们预期某些划分会比较平衡，而另一些则会很不平衡。在平均情况下，partition 所产生的划分同时混合有好和差的划分，此时，在与 partition 平均情况执行过程所对应所对应的递归树中，好和差的划分是随机分布的。基于直觉，假设好和差的划分交替出现在树的各层上，并且好的划分是最好情况划分，而差的划分是最坏情况划分。下图显示出了递归树的连续两层上的划分情况。在根结点处，划分的代价为 n，划分产生的两个子数组的大小为 n - 1 和 0，即最坏情况。在下一层，大小为 n - 1 的子数组按最好情况划分成大小分别为 (n - 1) / 2 - 1 和 (n - 1) / 2 的子数组。在这里，我们假设大小为 0 的子数组的边界条件代价为 1.

在一个差的划分后面接着一个好的划分，这种组合产生出三个子数组，大小分别为 0、(n - 1) / 2 - 1 和 (n - 1) / 2，这一组合的划分代价为 Θ(n) + Θ(n - 1) = Θ(n)，该代价并不比上述固定比例的划分更差。在下图（b）中，一层划分就产生出大小为 (n - 1) / 2 的两个子数组，划分代价为 Θ(n)。但是，后者的划分是平衡的！从直观上看，差划分的代价 Θ(n - 1) 可以被吸收到好划分的代价 Θ(n) 中去，而得到的划分结果也是好的。因此，当好和差的划分交替出现时，快速排序的时间复杂度与全市好的划分时一样，仍然是 O(n lgn)，区别只是 O 符号中隐含的常数因子要略大一些。

![](../assets/images/sorting-and-order-statistics/quick-sort4.png)

（a）一棵快速排序递归树的两层。在根结点这一层的划分代价是 n，产生了一个坏的划分：两个子数组的大小分别为 0 和 n - 1。对大小为 n - 1 的子数组的划分代价为 n - 1，并产生了一个好的划分：大小分别为 (n - 1) / 2 - 1 和 (n - 1) / 2 的子数组。（b）一棵非常平衡的递归树中的一层。在两棵树中，椭圆阴影所示的子问题的划分代价都是 Θ(n)。可以看出，（a）中以矩形阴影显示的待解决子问题的规模并不大于（b）中对应的待解决子问题。

### 快速排序的迭代版本

只需要一个辅助的栈，我们可以就将递归版本的快速排序转化为迭代版本的，下面是它的实现。

```java
void iterativeQuickSort(int[] arr, int p, int r) {
    int[] stack = new int[r - p + 1];
    int top = -1;
    stack[++top] = p;
    stack[++top] = r;
    while (top >= 0) {
        r = stack[top--];
        p = stack[top--];
        int q = partition(arr, p, r);
        if (q - p > 1) {
            stack[++top] = p;
            stack[++top] = q - 1;
        }
        if (r - q > 1) {
            stack[++top] = q + 1;
            stack[++top] = r;
        }
    }
}
```

### 快速排序的随机化版本

在讨论快速排序的平均情况性能的时候，我们的前提假设是：输入数据的所有排列都是等概率的。但是在实际工程中，这个假设并不会总是成立。有时我们可以通过在算法中引入随机性，从而使得算法对于所有的输入都能获得较好的期望性能。很多人都选择随机化版本的快速排序作为大数据输入情况下的排序算法。

与始终采用 arr[r] 作为主元的方法不同，随机抽样是从子数组 arr[p...r] 中随机选择一个元素作为主元，为了达到这一目的，首先将 arr[r] 与从 arr[p...r] 中随机选出的一个元素交换。通过对序列 p...r 的随机取样，我们可以保证主元元素 arr[r] 是等概率地从子数组的 r - p + 1 个元素中选取的。因为主元是随机选取的，我们期望在平均情况下，对输入数组的划分是比较平衡的。

对 quickSort 和 partition 的改动非常小，在新的划分程序中，我们只是在真正进行划分前进行一次交换：

```java
void randomizedQuickSort(int[] arr, int p, int r) {
    if (p < r) {
        int q = randomizedPartition(arr, p, r);
        randomizedQuickSort(arr, p, q - 1);
        randomizedQuickSort(arr, q + 1, r);
    }
}

int randomizedPartition(int[] arr, int p, int r) {
    int i = Util.randomInt(p, r + 1);
    Util.swap(arr, i, r);
    return partition(arr, p, r);
}
```

### 快速排序的栈深度

朴素 quickSort 算法包含了两个对其自身的递归调用。在调用 partition 后，quickSort 分别调用了左边的子数组和右边的子数组。quickSort 第二个递归调用并不是必须的，我们可以用一个循环控制结构来替代它。这一技术称为尾递归，好的编译器都提供这一功能。考虑下面这个版本的快速排序，它模拟了尾递归情况。

```java
void tailRecursiveQuickSort(int[] arr, int p, int r) {
    while (p < r) {
        int q = partition(arr, p, r);
        tailRecursiveQuickSort(arr, p, q - 1);
        p = q + 1;
    }
}
```

编译器通常使用栈来存储递归执行过程中的相关信息，包括每一次递归调用的参数等。最新调用的信息存在栈的顶部，而第一次调用的信息存在栈的底部。当一个过程被调用时，其相关信息被压入栈中，当它结束时，其信息被弹出。因为我们假设数组参数是用指针来指示的，所以每次过程调用只需要 O(1) 的栈空间。栈深度是在一次计算中会用到的栈空间的最大值。

通过修改 tailRecursiveQuickSort 的代码，使其最坏情况下栈深度是 Θ(lgn)，并且能够保持 O(n lgn) 的期望时间复杂度。

```java
void modifiedTailRecursiveQuickSort(int[] arr, int p, int r) {
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

### 针对相同元素值的快速排序（3 路快排）

在随机化快速排序的分析中，我们假设输入元素的值是互异的，下面我们看看如果这一假设不成立会出现什么情况。

如果所有输入元素都相同，那么随机化快速排序的每次划分步骤都会产生一个不平衡的划分，n - 1 个元素会全部被划分到数组的左边，因此，快速排序的运行时间是 Θ(n<sup>2</sup>)。原始版本的 partition 返回一个数组下标 q，使得 arr[p...q-1] 中的每一个元素都小于或等于 arr[q]，而 arr[q+1...r] 中的每个元素都大于 arr[q]。修改 partition 代码来构造一个新的 partition，它排列 arr[p...r] 的元素，返回值是两个数组下标 q 和 t，其中 p <= q <= t <= r，且有：

* arr[p...q-1] 中的每个元素都小于 arr[q]。

* arr[q...t] 中的所有元素都相等。

* arr[t+1...r] 中的每个元素都大于 arr[q]。

与 partition 类似，新构造的 partition 的时间复杂度是 Θ(r - p)，只有分区内的元素互不相同的时候才做递归调用。

下面的代码实现了 3 路快排，pivot 为 arr[r]，循环过程中始终保持 arr[p...lt] 小于 pivot，arr[gt...r] 大于 pivot，arr[lt+1...gt-1] 等于 pivot，partition返回 lt 和 gt，之后只需对 arr[p...lt] 和 arr[gt...r] 两部分进行递归调用即可。

```java
void threeWayQuickSort(int[] arr, int p, int r) {
    if (p < r) {
        int[] res = threeWayPartition(arr, p, r);
        threeWayQuickSort(arr, p, res[0]);
        threeWayQuickSort(arr, res[1], r);
    }
}

int[] threeWayPartition(int[] arr, int p, int r) {
    int lt = p - 1;
    int gt = r;
    int i = p;
    while (i < gt) {
        if (arr[i] < arr[r]) {
            lt++;
            Util.swap(arr, i, lt);
            i++;
        } else if (arr[i] > arr[r]) {
            gt--;
            Util.swap(arr, i, gt);
        } else {
            i++;
        }
    }
    Util.swap(arr, r, gt);
    return new int[]{lt, gt + 1};
}
```
