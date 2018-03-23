## 选择排序

选择排序首先找出数组中的最小元素并将其与数组中的第一个元素进行交换，接着，找出数组中的次小元素并将
其与数组的第二个元素进行交换。对数组的前n-1项按该方式继续。

```java
public class SelectionSort {
    public void selectionSort(int[] arr) {
        int n = arr.length;
        for (int i = 0; i < n - 1; i++) {
            int min = i;
            for (int j = i + 1; j < n - 1; j++) {
                if (arr[j] < arr[min]) {
                    min = j;
                }
            }
            swap(arr, min, i);
        }
    }
    
    private void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}
```
