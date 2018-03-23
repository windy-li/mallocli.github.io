## 归并排序

许多有用的算法在结构上是递归的，为了解决一个给定的问题，算法一次或多次递归地调用自身以解决紧密相关的若干子问题。这些算法典型地遵循分治法的思想：将原问题分解为若干个规模较小但类似于原问题的子问题，递归地求解这些子问题，然后再合并这些子问题的解来建立原问题的解。

分治模式在每层递归时都有三个步骤：

分解原问题为若干子问题，这些子问题是原问题的规模较小的实例。

解决这些子问题：递归地求解各子问题。然而，若子问题的规模足够小，则直接求解。

合并这些子问题的解成原问题的解。

归并排序算法完全遵循分治模式，直观上其操作过程如下：

分解：分解待排序的n个元素的序列成各具n/2个元素的两个子序列。

解决：使用归并排序递归地排序两个子序列。

合并：合并两个已排序的子序列以产生已排序的答案。

当待排序的序列长度为1时，递归开始回升，在这种情况下不要做任何工作，因为长度为1的每个序列都已排好序。

```java
public class MergeSort {
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
                arr[p] = left[i++];
            } else {
                arr[p] = right[j++];
            }
        }
    }
}
```
