## 最大子数组问题

在分治策略中，我们递归地求解一个问题，在每层递归中应用如下三个步骤：

1. 分解(Divide)步骤将问题划分为一些子问题，子问题的形式与原问题一样，只是规模更小。
2. 解决(Conquer)步骤递归地求解出子问题。如果子问题的规模足够小，则停止递归，直接求解。
3. 合并(Combine)步骤将子问题的解组合成原问题的解。

当子问题足够大，需要递归求解时，我们称之为递归情况(recursive case)，当子问题变得足够小，不再需要递归时，我们说递归已经“触底”，进入了基本情况(base case)。有时，除了与原问题形式完全一样的规模更小的子问题外，还需要求解与原问题不完全一样的子问题，我们将这些子问题的求解看成合并步骤的一部分。

我们把某数组的和最大的非空连续子数组称为最大连续子数组(maximum subarray)，通常说“一个最大子数组”而不是“最大子数组”，因为可能有多个子数组达到最大和。只有当数组中包含负数时，最大子数组问题才有意义，如果所有数组元素都是非负的，最大子数组问题没有任何难度，因为整个数组的和肯定是最大的。

![](./assets/images/part1/maximum-subarray1.png)

使用暴力法求解最大子数组问题，需要考虑数组的每一个连续子数组，选出其中和最大者。

```java
bruteForceMaximumSubarray(int[] arr) {
    int n = arr.length;
    int left = 0;
    int right = 0;
    int max = Integer.MIN_VALUE;
    for (int i = 0; i < n; i++) {
        int sum = 0;
        for (int j = i; j < n; j++) {
            sum += arr[j];
            if (sum > max) {
                max = sum;
                left = i;
                right = j;
            }
        }
    }
    return new int[]{start, end, max};
}
```

暴力求解法穷举了所有可能的子数组，运行时间为O(n^2)。

我们来思考如何使用分治法来求解最大子数组问题。假定我们要寻找子数组arr[low...high]的最大子数组。使用分治技术意味着我们要将子数组划分为两个规模尽可能相等的子数组，也就是说，找到子数组的中央位置，比如mid，然后考虑求解两个子数组arr[low...mid]和arr[mid + 1...high]。如图所示，arr[low...high]的任何连续子数组arr[i...j]所处的位置必然是以下三种情况之一：

* 完全位于子数组arr[low...mid]中，因此low <= i <= j <= mid。
* 完全位于子数组arr[mid + 1, high]中，因此mid < i <= j <= high。
* 跨越了中点，因此low <= i <= mid < j <= high。

![](./assets/images/part1/maximum-subarray2.png)

因此，arr[low...high]的一个最大子数组所处的位置必然是这三种情况之一。我们可以递归地求解arr[low...mid]和arr[mid + 1...high]，因为这两个问题仍然是最大子数组问题，只是规模更小。因此，剩下的工作就是寻找跨越中点的最大子数组，然后在三种情况中选择和最大者。

```java
public int[] divideAndConquerMaximumSubarray(int[] arr, int low, int high) {
    if (low == high) {
        return new int[]{low, high, arr[low]};
    } else {
        int mid = (low + high) / 2;
        int[] left = divideAndConquerMaximumSubarray(arr, low, mid);
        int[] right = divideAndConquerMaximumSubarray(arr, mid + 1, high);
        int[] cross = maxCrossingSubarray(arr, low, mid, high);
        int leftSum = left[2];
        int rightSum = right[2];
        int crossSum = cross[2];
        if (leftSum >= rightSum && leftSum >= crossSum) {
            return left;
        } else if (rightSum >= leftSum && rightSum >= crossSum) {
            return right;
        } else {
            return cross;
        }
    }
}

private int[] maxCrossingSubarray(int[] arr, int low, int mid, int high) {
    int sum = 0;
    int leftSum = Integer.MAX_VALUE;
    int leftIndex = mid;
    for (int i = mid; i >= low; i--) {
        sum += arr[i];
        if (sum > leftSum) {
            leftSum = sum;
            leftIndex = i;
        }
    }
    sum = 0;
    int rightSum = Integer.MAX_VALUE;
    int rightIndex = mid + 1;
    for (int i = mid + 1; i <= high; i++) {
        sum += arr[i];
        if (sum > rightSum) {
            rightSum = sum;
            rightIndex = i;
        }
    }
    return new int[]{leftIndex, rightIndex, leftSum + rightSum};
}
```

还可以使用如下思想为最大子数组问题设计一个非递归的、线性时间的算法。从数组的左边界开始，由左至右处理，记录到目前为止已经处理过的最大子数组。若已知arr[0...j]的最大子数组，基于如下性质将解扩展为arr[0...j+1]的最大子数组：arr[0...j+1]的最大子数组要么是arr[0...j]的最大子数组，要么是某个子数组arr[i...j+1] (其中1 < = i <= j+1)。在已知arr[0...j]的最大子数组的情况下，可以在线性时间内找出形如arr[i...j+1]的最大子数组。

```java
public int[] linearTimeMaximumSubarray(int[] arr) {
    int left = 0, right = 0;
    int max = Integer.MAX_VALUE;
    int sum = 0;
    for (int i = 0; i < arr.length; i++) {
        sum += arr[i];
        if (sum > max) {
            max = sum;
            right = i;
        }
        if (sum < 0) {
            sum = 0;
            left = i + 1;
        }
    }
    return new int[]{left, right, max};
}
```
