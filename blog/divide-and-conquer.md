## Divide-and-conquer

We can choose from a wide range of algorithm design techniques. For insertion
sort, we used an incremental approach: having sorted the subarray A[0...j-1],
we inserted the single element A[j] into its proper place, yielding the sorted
subarray A[0...j].

In this section, we examine an alternative design approach, known as “divide-and-conquer”.
We’ll use divide-and-conquer to design a sorting algorithm whose worst-case
running time is much less than that of insertion sort.

### The divide-and-conquer approach

Many useful algorithms are recursive in structure: to solve a given problem, they
call themselves recursively one or more times to deal with closely related subproblems.
These algorithms typically follow a divide-and-conquer approach: they
break the problem into several subproblems that are similar to the original problem
but smaller in size, solve the subproblems recursively, and then combine these
solutions to create a solution to the original problem.

The divide-and-conquer paradigm involves three steps at each level of the recursion:

* **Divide** the problem into a number of subproblems that are smaller instances of the
  same problem.

*  **Conquer** the subproblems by solving them recursively. If the subproblem sizes are
   small enough, however, just solve the subproblems in a straightforward manner.

* **Combine** the solutions to the subproblems into the solution for the original problem.

The **_merge sort_** algorithm closely follows the divide-and-conquer paradigm.
Intuitively, it operates as follows.

* **Divide:** Divide the n-element sequence to be sorted into two subsequences of n/2
  elements each.

* **Conquer:** Sort the two subsequences recursively using merge sort.

* **Combine:** Merge the two sorted subsequences to produce the sorted answer.

The recursion “bottoms out” when the sequence to be sorted has length 1, in which
case there is no work to be done, since every sequence of length 1 is already in
sorted order.

The key operation of the merge sort algorithm is the merging of two sorted
sequences in the “combine” step. We merge by calling an auxiliary procedure
MERGE(A, p, q, r), where A is an array and p, q, and r are indices into the array
such that p <= q < r. The procedure assumes that the subarrays A[p...q] and
A[q...r] are in sorted order. It merges them to form a single sorted subarray
that replaces the current subarray A[P...r].

Our MERGE procedure takes time Θ(n), where n = r - p + 1 is the total
number of elements being merged, and it works as follows. Returning to our
card-playing motif, suppose we have two piles of cards face up on a table. Each pile is
sorted, with the smallest cards on top. We wish to merge the two piles into a single
sorted output pile, which is to be face down on the table. Our basic step consists
of choosing the smaller of the two cards on top of the face-up piles, removing it
from its pile (which exposes a new top card), and placing this card face down onto
the output pile. We repeat this step until one input pile is empty, at which time
we just take the remaining input pile and place it face down onto the output pile.
Computationally, each basic step takes constant time, since we are comparing just
the two top cards. Since we perform at most n basic steps, merging takes Θ(n) time.

The following code implements the above idea, but with an additional
twist that avoids having to check whether either pile is empty in each basic step.
We place on the bottom of each pile a sentinel card, which contains a special value
that we use to simplify our code. Here, we use Integer.MAX_VALUE as the sentinel value, so that
whenever a card with Integer.MAX_VALUE is exposed, it cannot be the smaller card unless both piles
have their sentinel cards exposed. But once that happens, all the nonsentinel cards
have already been placed onto the output pile. Since we know in advance that
exactly r - p + 1 cards will be placed onto the output pile, we can stop once we
have performed that many basic steps.

```java
public void mergeSort(int[] A, int p, int q, int r) {
    if (p < r) {
        int q = (p + r) / 2;
        mergeSort(A, p, q);
        mergeSort(A, q + 1, r);
        merge(A, p, q, r);
    }
}

private void merge(int[] A, int p, int q, int r) {
    int n1 = q - p + 1;
    int n2 = r - q;
    int[] left = new int[n1 + 1];
    int[] right = new int[n2 + 1];
    for (int i = 0; i < n1; i++) {
        left[i] = A[p + i];
    }
    left[n1] = Integer.MAX_VALUE;
    for (int i = 0; i < n2; i++) {
        right[i] = A[q + 1 + i];
    }
    right[n2] = Integer.MAX_VALUE;
    int i = 0, j = 0;
    for (int k = p; k <= r; k++) {
        if (left[i] <= right[j]) {
            A[k] = left[i++];
        } else {
            A[k] = right[j++];
        }
    }
}
```

### Analyzing divide-and-conquer algorithms

When an algorithm contains a recursive call to itself, we can often describe its
running time by a **_recurrence equation_** or **_recurrence_**, which describes the overall
running time on a problem of size n in terms of the running time on smaller inputs.
We can then use mathematical tools to solve the recurrence and provide bounds on
the performance of the algorithm.

![](../assets/img/foundations/merge-sort.png)

The operation of merge sort on the array A = {5, 2, 4, 7, 1, 3, 2, 6}. The lengths of the
sorted sequences being merged increase as the algorithm progresses from bottom to top.









