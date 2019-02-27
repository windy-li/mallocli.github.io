## Insertion Sort

### Direct insertion sort

Insertion sort is an efficient algorithm for sorting a small number of elements. Insertion sort works the way many people sort a hand of playing cards. We start with an empty left hand and the cards face down on the table. We then remove one card at a time from the table and insert it into the correct position in the left hand. To find the correct position for a card, we compare it with each of the cards already in the hand, from right to left. At all times, the cards held in the left hand are sorted.

![](../../assets/images/part1/insertion-sort.png)

![](../../assets/images/part1/insertion-sort.gif)

Below is the implementation of insertion sort, the algorithm sorts the input numbers in place: it rearranges the numbers within the array, with at most a constant number of them stored outside the array at any time. The input array contains the sorted output sequence when the insertion sort procedure is finished.

```
void insertionSort(int[] arr) {
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

In insertion sort, the best case occurs if the array is already sorted.  For each j = 1, 2, 3, ..., n-1, we can find that arr[i] <= key when i has its initial value of j-1. Each element only compares one time, the running time is O(n). If the array is in reverse sorted order, that is, in decreasing order—the worst case results. We must compare each element arr[i] with each element in the entire sorted subarray
