## Insertion sort

Insertion sort is an efficient algorithm for sorting a small
number of elements. Insertion sort works the way many people sort a hand of
playing cards. We start with an empty left hand and the cards face down on the
table. We then remove one card at a time from the table and insert it into the
correct position in the left hand. To find the correct position for a card, we compare
it with each of the cards already in the hand, from right to left. At all times,
the cards held in the left hand are sorted, and these cards were originally the 
top cards of the pile on the table.

![](../assets/img/foundations/insertion-sort.png)

![](../assets/img/foundations/insertion-sort2.png)

The operation of INSERTION-SORT on the array A = {5, 2, 4, 6, 1, 3}. Array indices
appear above the rectangles, and values stored in the array positions appear within the rectangles.

```java
public void insertionSort(int[] A) {
    for (int j = 1; j < A.length; j++) {
        int key = A[j];
        int i = j - 1;
        while (i >= 0 && A[i] > key) {
            A[i + 1] = A[i];
            i--;
        }
        A[i + 1] = key;
    }
}
```

The index j indicates the "current card" being inserted into the hand. At the beginning
of each iteration of the for loop, which is indexed by j, the subarray consisting
of elements A[0...j-1] constitutes the currently sorted hand, and the remaining
subarray A[j+1...n-1] corresponds to the pile of cards still on the table. In fact,
elements A[0...j-1] are the elements originally in positions 0 through j-1, but
now in sorted order.

Below is the recursive version.

```java
public void recursiveInsertionSort(int[] A, int n) {
    if (n > 0) {
        recursiveInsertionSort(A, n - 1);
        int key = A[n];
        int i = n - 1;
        while (i >= 0 && A[i] > key) {
            A[i + 1] = A[i];
            i--;
        }
        A[i + 1] = key;
    }
}
```
