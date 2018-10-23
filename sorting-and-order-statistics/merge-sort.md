## 归并排序

许多有用的算法在结构上是递归的，为了解决一个给定的问题，算法一次或多次递归地调用自身以解决紧密相关的若干子问题。这些算法典型地遵循分治法的思想：将原问题分解为若干个规模较小但类似于原问题的子问题，递归地求解这些子问题，然后再合并这些子问题的解来建立原问题的解。

分治模式在每层递归时都有三个步骤：

1. 分解原问题为若干子问题，这些子问题是原问题的规模较小的实例。
2. 解决这些子问题：递归地求解各子问题。然而，若子问题的规模足够小，则直接求解。
3. 合并这些子问题的解构成原问题的解。

归并排序算法完全遵循分治模式，直观上其操作过程如下：

1. 分解：分解待排序的n个元素的序列成各具n/2个元素的两个子序列。
2. 解决：使用归并排序递归地排序两个子序列。
3. 合并：合并两个已排序的子序列以产生已排序的答案。

当待排序的序列长度为1时，递归开始回升，在这种情况下不要做任何工作，因为长度为1的每个序列都已排好序。

归并排序算法的关键步骤是“合并”步骤中两个已排序序列的合并，它按以下方式工作。假设桌子上有两堆牌，每堆都已排好序，最小的牌在顶上，我们希望把这两堆牌合并成单一的排好序的输出堆。我们每次从两堆牌顶中选择较小的一张，将该牌从其堆中移开（该堆的顶上将露出一张新牌）并将该牌放置到输出堆。重复这个步骤，直到一个输入堆为空，这时，我们就可以简单地将该输入堆的每张牌放置到输出堆中。

为了避免在每个基本步骤中检查是否有堆为空，在每个堆的底部放置一张哨兵牌，它包含一个特殊值∞，每当显示一张值为∞的牌，它不可能为较小的牌，除非两个堆都已显露其哨兵牌。但是，一旦发生这种情况，所有非哨兵牌都已被放置到输出堆。因为我们事先知道刚好r-p+1张牌将被放置到输出堆，所以一旦已执行r-p+1个基本步骤，算法就可以停止。

```java
public void mergeSort(int[] arr, int p, int r) {
    if (p < r) {
        int q = (p + r) / 2;
        mergeSort(arr, p, q);
        mergeSort(arr, q + 1, r);
        merge(arr, p, q, r);
    }
}

private void merge(int[] arr, int p, int q, int r) {
    int n1 = q - p + 1;
    int n2 = r - q;
    int[] left = new int[n1 + 1];
    int[] right = new int[n2 + 1];
    for (int i = 0; i < n1; i++) {
        left[i] = arr[p + i];
    }
    left[n1] = Integer.MAX_VALUE;
    for (int i = 0; i < n2; i++) {
        right[i] = arr[q + 1 + i];
    }
    right[n2] = Integer.MAX_VALUE;
    int i = 0, j = 0;
    for (int k = p; k <= r; k++) {
        if (left[i] <= right[j]) {
            arr[k] = left[i++];
        } else {
            arr[k] = right[j++];
        }
    }
}
```

在合并步骤中，如果不使用哨兵，那么一旦数组left或right所有元素均被复制回数组arr，就立刻停止，然后把另一个数组的剩余部分复制回arr。

```java
private void merge(int[] arr, int p, int r) {
    int n1 = q - p + 1;
    int n2 = r - q;
    int[] left = new int[n1];
    int[] right = new int[n2];
    for (int i = 0; i < n1; i++) {
        left[i] = arr[p + i];
    }
    for (int i = 0; i < n2; i++) {
        right[i] = arr[q + 1 + i];
    }
    int i = 0, j = 0, k = p;
    while (i < n1 && j < n2) {
        if (left[i] <= right[j]) {
            arr[k++] = left[i++];
        } else {
            arr[k++] = right[j++];
        }
    }
    if (i == n1) {
        while (j < n2) {
            arr[k++] = right[j++];
        }
    } else {
        while (i < n1) {
            arr[k++] = left[i++];
        }
    }
}
```
