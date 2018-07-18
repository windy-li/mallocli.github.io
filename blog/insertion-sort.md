## 插入排序

插入排序的工作方式就像许多人排序一手扑克牌，开始时，我们的左手为空并且待排序的牌都在桌子上，然后，我们每次从桌子上拿走一张牌并将它插入左手中正确的位置。
为了找到一张牌的正确位置，我们从右到左将它与已在手中的每张牌进行比较，如图所示，拿在手中的牌总是排好序的。

![](../assets/images/part1/insertion-sort.png)

使用插入排序为一列数字进行排序的过程：

![](../assets/images/part1/insertion-sort.gif)

具体算法描述如下：

1. 从第一个元素开始，该元素可以认为已经被排序
2. 取出下一个元素，在已经排序的元素序列中从后向前扫描
3. 如果该元素（已排序）大于新元素，将该元素移到下一位置
4. 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置
5. 将新元素插入到该位置后
重复步骤2~5

用程序实现如下：

```java
public void insertionSort(int[] arr) {
    for (int j = 1; j < arr.length; j++) {
        int key = arr[j];
        int i = j - 1;
        while (i >= 0 && arr[i] > key) {
            arr[i + 1] = arr[i];
            i--;
        }
        arr[i + 1] = key;
    }
}
```

还可以把插入排序表示为如下的一个递归过程：为了排序数组的前n项，我们递归地排序前n-1项，然后把第n项插入已排序的前n-1个元素中。

```java
public void recursiveInsertionSort(int[] arr, int n) {
    if (n > 0) {
        recursiveInsertionSort(arr, n - 1);
        int key = arr[n];
        int i = n - 1;
        while (i >= 0 && arr[i] > key) {
            arr[i + 1] = arr[i];
            i--;
        }
        arr[i + 1] = key;
    }
}
```
