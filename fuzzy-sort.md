## 对区间的模糊排序

考虑这样的一种排序问题：我们无法准确知道待排序的数字是什么，但对于每一个数，我们知道它属于实数轴上的某个区间。也就是说，我们得到了n个形如[ai, bj]的闭区间，其中ai <= bj。我们的目标是实现这些区间的模糊排序，即对于j = 1, 2, 3 ..., n，生成一个区间的排列<i1, i2, i3, ..., in>，且存在cj属于[ai, bj]，满足c1 <= c2 <= ... <= cn。

下面是对n个区间的模糊排序的随机算法，同时它也能利用区间的重叠性质来改善时间性能，当区间重叠越来越多的时候，区间的模糊排序问题会变得越来越容易。

```java
public class FuzzySort {
    private class Interval {
        int left;
        int right;
        
        Interval(int left, int right) {
            this.left = left;
            this.right = right;
        }
    }
    
    public void fuzzySort(Interval[] intervals, int p, int r) {
        if (p < r) {
            int[] q = partition(intervals, p, r);
            fuzzySort(intervals, p, q[0] - 1);
            fuzzySort(intervals, q[1] + 1, r);
        }
    }
    
    private int[] partition(Interval[] intervals, int p, int r) {
        Interval pivot = new Interval(intervals[r].left, intervals[r].right);
        int i = p;
        int j = p;
        int k = r;
        while (j < k) {
            if (intervals[j].right <= pivot.left) {
                swap(intervals, i, j);
                i++;
                j++;
            } else if (intervals[j].left >= pivot.right) {
                swap(intervals, j, k);
                k--;
            } else {
                pivot.left = Math.max(intervals[j].left, pivot.left);
                pivot.right = Math.min(intervals[j].right, pivot.right);
                j++;
            }
        }
        return new int[]{i, k};
    }
    
    private void swap(Interval[] intervals, int i, int j) {
        Interval temp = intervals[i];
        intervals[i] = intervals[j];
        intervals[j] = temp;
    }
}
```

在一般情况下，fuzzySort的期望运行时间是Θ(n * lgn)，但是，当所有区间都有重叠的时候（也就是说，存在一个值x，对所有的i，都有x属于[ai, bi]），算法的期望运行时间为Θ(n)。fuzzySort算法不必显式地检查这种情况，而是随着重叠情况的增加，算法的性能自然地提高。
