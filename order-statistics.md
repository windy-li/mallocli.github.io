### 顺序统计量

在一个由n个元素组成的集合中，第i个顺序统计量(order statistic)是该集合中第i小的元素。例如，在一个元素集合中，最小值是指第1个顺序统计量(i = 1)，最大值是第n个顺序统计量(i = n)。

这里我们将讨论从一个由n个互异的元素构成的集合中选择第i个顺序统计量的问题，为了方便起见，假设元素都是互异的，但实际上，我们所做的都可以推广到集合中包含重复元素的情形。

### 最大值和最小值

在一个有n个元素的集合中，需要做多少次比较才能确定其最小元素呢？我们可以很容易地给出n-1次比较这个上界：依次遍历集合中的每个元素，并记录下当前最小元素。

```java
public int minimum(int[] arr) {
    int min = arr[0];
    for (int i = 1; i < arr.length; i++) {
        if (min > arr[i]) {
            min = arr[i];
        }
    }
    return min;
}
```

当然，最大值也可以通过n-1次比较找出来。

这是我们能得到的最好结果吗？是的，对于确定最小值问题，我们可以得到其下界就是n-1次比较。对于任意一个确定最小值的算法，可以把它看成是各元素之间进行的一场锦标赛，每次比较都是锦标赛中的一场比赛，两个元素中较小的获胜。需要注意的是，除了最终获胜者以外，每个元素至少要输掉一场比赛。因此，我们得出结论：为了确定最小值，必须要做n-1次比较。因此，从所执行的比较次数来看，算法minimum是最优的。

### 同时找到最大值和最小值

在某些应用中，我们必须要找出一个包含n个元素的集合中的最大值和最小值。用渐进最优的Θ(n)次比较，在n个元素中同时找到最小值和最大值的方法是显然的：只要分别独立地找出最小值和最大值，这各需要n-1次比较，共2n - 2次比较。

事实上，我们只需要最多3 * Math.floor(n / 2)次比较就可以同时找到最小值和最大值。具体的方法是记录已知的最小值和最大值，但我们并不是将每一个输入元素与当前的最小值和最大值进行比较，这样做的代价是每个元素需要2次比较，而是对输入元素成对地进行处理。首先，我们将一对输入元素互相进行比较，然后把较小的与当前最小值比较，把较大的与当前最大值进行比较。这样，对每2个元素共需3次比较、

如何设定已知的最小值和最大值的初始值依赖于n是奇数还是偶数。如果n是奇数，我们就将最小值和最大值的初始值都设置为第一个元素的值，然后成对地处理余下的元素。如果n是偶数，就对前两个元素做一次比较，以决定最小值和最大值的初始值，然后与n是奇数时的情形一样，成对地处理余下的元素。

如果n是奇数，那么总共进行3 * Math.floor(n / 2)次比较，如果n是偶数，则是先进行一次初始比较，然后进行3 * (n - 2) / 2次比较，共3 * n / 2 - 2次比较。因此，不管是哪一种情况，总的比较次数至多是3 * Math.floor(n / 2)。

```java
public int minimumAndMaximum(int[] arr) {
    int n =arr.length;
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

下面的randomizedSelect算法是以快速排序算法为模型的，与快速排序一样，我们仍然将输入数组进行递归划分，但与快速排序不同的是，快速排序会递归处理划分的两边，而randomizedSelect只处理划分的一边。这一差异会在性能分析中体现出来：快速排序的期望运行时间是Θ(n * lgn)，而randomizedSelect的期望运行时间是Θ(n)。这里，假设输入元素都是互异的。

randomizedSelect利用了快速排序的randomizedPartition过程，与randomizedQuickSort一样，因为它的部分行为是由随机数生成器的输出决定的，所以它也是个随机算法。下面的代码返回数组arr[p...r]中第i小的元素。

```java
public int randomizedSelect(int[] arr, int p, int r, int i) {
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

private int randomizedPartition(int[] arr, int p, int r) {
    int i = p + (int) (Math.random() * (r - p + 1));
    swap(arr, i, r);
    return partition(arr, p, r);
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

下面是randomizedSelect的基于循环的版本：

```java
public int iterativeRandomizedSelect(int[] arr, int i) {
    int p = 0;
    int n = arr.length;
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

### 最坏情况为线性时间的选择算法

现在我们来看一个最坏情况为线性时间O(n)的选择算法。像randomizedSelect一样，select算法通过对输入数组的递归划分来找出所需元素，但是，在该算法中能够保证得到对数组的一个号的划分。select使用的也是来自快速排序的确定性划分算法partition，但做了修改，把划分的主元也作为输入参数。

通过执行以下步骤，算法select可以确定一个有n > 1个不同元素的输入数组中第i小的元素（如果n=1，则select只返回它的唯一输入数值作为第i小的元素）

1. 将输入数组的n个元素划分为Math.floor(n / 5)组，每组5个元素，且至多只有一组由剩下的n mod 5个元素组成。

2. 寻找这Math.ceil(n / 5)组中每一组的中位数：首先对每组元素进行插入排序，然后确定每组有序元素的中位数。

3. 对第2步中找出的Math.ceil(n / 5)个中位数，递归调用select以找出其中位数x（如果有偶数个中位数，为了方便约定x是较小的中位数）。

4. 利用修改过的partition版本，按中位数的中位数x对输入数组进行划分。让k比划分的低区中的元素数目多1，因此x是第k小的元素，并且有n - k个元素在划分的高区。

5. 如果i == k，则返回x。如果i < k，则在低区递归调用select来找出第i小的元素，如果i > k，则在高区递归查找第i - k小的元素。

```java
public int select(int[] arr, int p, int r, int x) {
    if (p >= r) {
        return arr[p];
    }
    int n = r - p + 1;
    int group = (n + 4) / 5;
    int[] medians = new int[group];
    for (int i = 0; i < group; i++) {
        int start = p + i * 5;
        int end = (start + 5 > r) ? r : (start + 5);
        medians[i] = getMedian(arr, start, end);
    }
    int medOfMed = select(medians, 0, group - 1, (group - 1) / 2 + 1);
    int q = partition(arr, p, r, medOfMed);
    int k = q - p + 1;
    if (x == k) {
        return arr[q];
    } else if (x < k) {
        return select(arr, p, q - 1, x);
    } else {
        return select(arr, q + 1, r, x - k);
    }
}

private int getMedian(int[] arr, int start, int end) {
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

private int partition(int[] arr, int p, int r, int pivot) {
    int i = p - 1;
    int k = 0;
    for (int j = p; j <= r; j++) {
        if (arr[j] < pivot) {
            i++;
            swap(arr, i, j);
        }
        if (arr[j] == pivot) {
            k = j;
        }
    }
    swap(arr, i + 1, k);
    return i + 1;
}

private void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i]= arr[j];
    arr[j] = temp;
}
```
