## 顺序统计量

在一个由 n 个元素组成的集合中，第 i 个顺序统计量 (order statistic) 是该集合中第 i 小的元素。例如，在一个元素集合中，最小值是指第 1 个顺序统计量 (i = 1)，最大值是第 n 个顺序统计量 (i = n)。

这里我们将讨论从一个由 n 个互异的元素构成的集合中选择第 i 个顺序统计量的问题，为了方便起见，假设元素都是互异的，但实际上，我们所做的都可以推广到集合中包含重复元素的情形。

### 最大值和最小值

在一个有 n 个元素的集合中，需要做多少次比较才能确定其最小元素呢？我们可以很容易地给出 n - 1 次比较这个上界：依次遍历集合中的每个元素，并记录下当前最小元素。

```java
int minimum(int[] arr) {
    int min = arr[0];
    for (int i = 1; i < arr.length; i++) {
        if (min > arr[i]) {
            min = arr[i];
        }
    }
    return min;
}
```

当然，最大值也可以通过 n - 1 次比较找出来。

这是我们能得到的最好结果吗？是的，对于确定最小值问题，我们可以得到其下界就是 n - 1 次比较。对于任意一个确定最小值的算法，可以把它看成是各元素之间进行的一场锦标赛，每次比较都是锦标赛中的一场比赛，两个元素中较小的获胜。需要注意的是，除了最终获胜者以外，每个元素至少要输掉一场比赛。因此，我们得出结论：为了确定最小值，必须要做 n - 1 次比较。因此，从所执行的比较次数来看，算法 minimum 是最优的。

### 同时找到最大值和最小值

在某些应用中，我们必须要找出一个包含 n 个元素的集合中的最大值和最小值。用渐进最优的 Θ(n) 次比较，在 n 个元素中同时找到最小值和最大值的方法是显然的：只要分别独立地找出最小值和最大值，这各需要 n - 1 次比较，共 2n - 2 次比较。

事实上，我们只需要最多 3 * Math.floor(n / 2) 次比较就可以同时找到最小值和最大值。具体的方法是记录已知的最小值和最大值，但我们并不是将每一个输入元素与当前的最小值和最大值进行比较，这样做的代价是每个元素需要 2 次比较，而是对输入元素成对地进行处理。首先，我们将一对输入元素互相进行比较，然后把较小的与当前最小值比较，把较大的与当前最大值进行比较。这样，对每 2 个元素共需 3 次比较。

如何设定已知的最小值和最大值的初始值依赖于 n 是奇数还是偶数。如果 n 是奇数，我们就将最小值和最大值的初始值都设置为第一个元素的值，然后成对地处理余下的元素。如果 n 是偶数，就对前两个元素做一次比较，以决定最小值和最大值的初始值，然后成对地处理余下的元素。

如果 n 是奇数，那么总共进行 3 * Math.floor(n / 2) 次比较，如果 n 是偶数，则是先进行一次初始比较，然后进行 3 * (n - 2) / 2 次比较，共 3 * n / 2 - 2 次比较。因此，不管是哪一种情况，总的比较次数至多是 3 * Math.floor(n / 2)。

```java
int minimumAndMaximum(int[] arr) {
    int n = arr.length;
    int min, max, start;
    if (n % 2 == 1) {
        min = max = arr[0];
        start = 1;
    } else {
        if (arr[0] < arr[1]) {
            min = arr[0];
            max = arr[1];
        } else {
            min = arr[1];
            max = arr[0];
        }
        start = 2;
    }
    for (int i = start; i < n; i += 2) {
        if (arr[i] < arr[i + 1]) {
            if (min > arr[i]) {
                min = arr[i];
            }
            if (max < arr[i + 1]) {
                max = arr[i + 1];
            }
        } else {
            if (min > arr[i + 1]) {
                min = arr[i + 1];
            }
            if (max < arr[i]) {
                max = arr[i];
            }
        }
    }
    return new int[]{min, max};
}
```

### 期望为线性时间的选择算法

一般选择问题看起来要比找最小值这样的简单问题更难，但令人吃惊的是，这两个问题的渐进运行时间却是相同的：Θ(n)。

下面的 randomizedSelect 算法是以快速排序算法为模型的，与快速排序一样，我们仍然将输入数组进行递归划分，但与快速排序不同的是，快速排序会递归处理划分的两边，而 randomizedSelect 只处理划分的一边。这一差异会在性能分析中体现出来：快速排序的期望运行时间是 Θ(n * lgn)，而 randomizedSelect 的期望运行时间是 Θ(n)。这里，假设输入元素都是互异的。

randomizedSelect 利用了快速排序的 randomizedPartition 过程，与 randomizedQuickSort 一样，因为它的部分行为是由随机数生成器的输出决定的，所以它也是个随机算法。下面的代码返回数组 arr[p...r] 中第 i 小的元素。

```java
int randomizedSelect(int[] arr, int p, int r, int i) {
    if (p == r) {
        return arr[p];
    }
    int q = randomizedPartition(arr, p, r);
    int k = q - p + 1;
    if (i == k) {
        return arr[q];
    } else if (i < k) {
        return randomizedSelect(arr, p, q - 1, i);
    } else {
        return randomizedSelect(arr, q + 1, r, i - k);
    }
}

int randomizedPartition(int[] arr, int p, int r) {
    int i = Util.randomInt(p, r + 1);
    Util.swap(arr, i, r);
    return partition(arr, p, r);
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

下面是 randomizedSelect 的基于循环的版本：

```java
int iterativeRandomizedSelect(int[] arr, int p, int r, int i) {
    while (p < r) {
        int q = randomizedPartition(arr, p, r);
        int k = q - p + 1;
        if (i == k) {
            return arr[q];
        } else if (i < k) {
            r = q - 1;
        } else {
            p = q + 1;
            i -= k;
        }
    }
    return arr[p];
}
```

randomizedSelect 的最坏情况运行时间是 Θ(n^2)，即使是找出最小元素也是如此，因为在每次划分时可能极不走运地总是按余下的元素中最大的来划分，而划分操作需要 Θ(n) 时间。该算法有线性的期望运行时间，又因为它是随机化的，所以不存在一个特定的会导致其最坏情况发生的输入数据。

### 最坏情况为线性时间的选择算法

现在我们来看一个最坏情况运行时间为 O(n) 的选择算法。像 randomizedSelect 一样，select 算法通过对输入数组的递归划分来找出所需元素，但是，在该算法中能够保证得到对数组的一个好的划分。select 使用的也是来自快速排序的确定性划分算法 partition，但做了修改，把划分的主元也作为输入参数。

通过执行以下步骤，算法 select 可以确定一个有 n > 1 个不同元素的输入数组中第 i 小的元素（如果 n = 1，则 select 只返回它的唯一元素作为第 i 小的元素）

1. 将输入数组的 n 个元素划分为 Math.floor(n / 5) 组，每组 5 个元素，且至多只有一组由剩下的 n mod 5 个元素组成。

2. 寻找这 Math.ceil(n / 5) 组中每一组的中位数：首先对每组元素进行插入排序，然后确定每组有序元素的中位数。

3. 对第 2 步中找出的 Math.ceil(n / 5) 个中位数，递归调用 select 以找出其中位数 x（如果有偶数个中位数，为了方便约定 x 是较小的中位数）。

4. 利用修改过的 partition 版本，按中位数的中位数 x 对输入数组进行划分。让 k 比划分的低区中的元素数目多 1，因此 x 是第 k 小的元素，并且有 n - k 个元素在划分的高区。

5. 如果 i = k，则返回 x。如果 i < k，则在低区递归调用 select 来找出第 i 小的元素，如果 i > k，则在高区递归查找第 i - k 小的元素。

```java
int select(int[] arr, int p, int r, int i) {
    if (p == r) {
        return arr[p];
    }
    int n = r - p + 1;
    int group = (n + 4) / 5;
    int[] medians = new int[group];
    for (int j = 0; j < group; j++) {
        int start = p + j * 5;
        int end = (start + 5 > r) ? r : (start + 5);
        medians[j] = getMedian(arr, start, end);
    }
    int medOfMed = select(medians, 0, group - 1, (group - 1) / 2 + 1);
    int q = partition(arr, p, r, medOfMed);
    int k = q - p + 1;
    if (i == k) {
        return arr[q];
    } else if (i < k) {
        return select(arr, p, q - 1, i);
    } else {
        return select(arr, q + 1, r, i - k);
    }
}

int getMedian(int[] arr, int start, int end) {
    for (int j = start + 1; j <= end; j++) {
        int key = arr[j];
        int i = j - 1;
        while (i >= start && arr[i] > key) {
            arr[i + 1] = arr[i];
            i--;
        }
        arr[i + 1] = key;
    }
    return arr[(start + end) / 2];
}
```

![](../assets/images/sorting-and-order-statistics/order-statistic.png)

对算法 select 的分析。所有 n 个元素都由圆圈来表示，并且每一组的 5 个元素在同一列上。其中，每组的中位数用白色圈表示，而中位数的中位数 x 也被标识出来（当查找偶数个元素的中位数时，使用较小的中位数）。箭头从较大的元素指向较小的元素，从图中可以看出，在 x 的右边，每一个包含 5 个元素的组中有 3 个元素大于 x。在 x 的左边，每一个包含 5 个元素的组中有 3 个元素小于 x。大于 x 的元素的背景以阴影来显示。

与比较排序一样，select 和 randomizedSelect 也是通过元素间的比较来确定它们之间的相对次序的。我们知道在[比较模型](lower-bounds-for-sorting.md)中，即使是在平均情况下，排序仍然需要 Ω(n * lgn) 时间。线性时间排序算法（[计数排序](counting-sort.md)、[基数排序](radix-sort.md)、[桶排序](bucket-sort.md)）在输入上作了一些假设，从而突破了 Ω(n * lgn) 的下界，相反，这里的线性时间选择算法不需要任何关于输入的假设，它们不受限于 Ω(n * lgn) 的下界约束，因为它们没有使用排序就解决了问题。
