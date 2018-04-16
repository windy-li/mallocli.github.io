## 随机排列数组

很多随机算法通过对给定的输入变换排列以使输入随机化，这里，我们将讨论两种随机化方法。不失一般性，假设给定一个数组arr，包含元素1到n，我们的目标是构造这个数组的一个随机排列。

一个通常的做法是为数组的每个元素arr[i]赋一个随机的优先级p[i]，然后根据优先级对数组arr中的元素进行排序。例如，如果初始数组arr = [1, 2, 3, 4]，随机选择的优先级p = [36, 3, 62, 19]，则将产生一个数组[2, 4, 1, 3]，因为第2个优先级最小，接下来是第4个，然后第1个，最后第3个。

```java
public void permuteBySorting(int[] arr) {
    int n = arr.length;
    int[] p = new int[n];
    Random r = new Random();
    for (int i = 0; i < n; i++) {
        p[i] = r.nextInt(n * n * n);
    }
    sort(arr, p);
}

private void sort(int[] arr, int[] p) {
    for (int j = 1; j < arr.length; j++) {
        int key = p[j];
        int i = j - 1;
        while (i >= 0 && p[i] > key) {
            p[i + 1] = p[i];
            i--;
        }
        p[i + 1] = key;
        swap(arr, i + 1, j);
    }
}

private void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
```

选取1 ~ n^3范围之间的随机数是为了让p中所有优先级尽可能唯一。

这个过程中最耗时的步骤是sort方法，如果使用比较排序，排序至少要花费n * lgn的时间，使用归并排序或快速排序等可以达到这个下界。排序以后，如果p[i]是第j个最小的优先级，那么arr[i]将会出现在输出位置j上。用这种方式，我们得到了一个随机均匀排列，即该过程等可能地产生数字1~n的每一种排列。

产生随机排列的一个更好的方法是原址排列给定数组：

```java
public void randomizeInPlace(int[] arr) {
    int n = arr.length;
    Random r = new Random();
    for (int i = 0; i < n; i++) {
        swap(arr, i, i + r.nextInt(n - 1 - i + 1));
    }
}
```

在进行第i次迭代时，元素arr[i]是从arr[i]到arr[n - 1]中随机选取的，第i次迭代以后，arr[i]将不再改变。
