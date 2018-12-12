## 平均排序

假设我们不是要完全排序一个数组，而只是要求数组中的元素在平均情况下是升序的，更准确的说，如果对所有的 i = 0, 1, 2, 3, ..., n - 1 - k，有 arr[i] <= arr[i + k] 成立，我们就称一个包含 n 个元素的数组 arr 是 k 排序的（k-sorted）。

下面的算法能在 O(n * lg(n / k)) 时间内对一个包含 n 个元素的数组进行 k 排序，当 k 是一个常数时，对包含 n 个元素的数组进行 k 排序需要 Ω(n * lgn) 的时间。

```java
void kSort(int[] arr, int k, int p, int r) {
    if (p < r && r - p >= k) {
        int q = partition(arr, p, r);
        kSort(arr, k, p, q - 1);
        kSort(arr, k, q + 1, r);
    }
}

int partition(int[] arr, int p, int r) {
    int i = p - 1;
    for (int j = p; j <= r - 1; j++) {
        if (arr[j] <= arr[r]) {
            i++;
            Util.swap(arr, i, j);
        }
    }
    Util.swap(arr, i + 1, r);
    return i + 1;
}
```
