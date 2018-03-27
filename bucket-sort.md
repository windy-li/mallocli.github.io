## 桶排序

桶排序假设输入数据服从均匀分布，平均情况下它的时间代价为O(n)。与计数排序类似，因为对输入数据作了某种假设，桶排序的速度也很快。具体来说，计数排序假设输入数据都属于一个小区间内的整数，而桶排序则假设输入是由一个随机过程产生，该过程将元素均匀、独立地分布在[0, 1)区间上。

桶排序将[0, 1)区间划分为n个相同大小的子区间，或称为桶。然后，将n个输入数分别放到各个桶中。因为输入数据是均匀、独立地分布在[0, 1)区间上，所以一般不会出现很多数落在同一个桶中的情况。为了得到输出结果，我们先对每个桶中的数进行排序，然后遍历每个桶，按照次序把各个桶中的元素列出来即可。

在桶排序的代码中，我们假设输入是一个包含n个元素的数组arr，且每个元素arr[i]满足0 <= arr[i] < 1。此外，算法还需要一个临时数组b[0...n - 1]来存放链表。

```java
public void bucketSort(double[] arr) {
    int n = arr.length;
    LinkedList<Double>[] buckets = new LinkedList[n];
    for (int i = 0; i < n; i++) {
        buckets[i] = new LinkedList<>();
    }
    for (int i = 0; i < n; i++) {
        buckets[(int) (n * arr[i])].add(arr[i]);
    }
    int pointer = 0;
    for (int i = 0; i < n; i++) {
        LinkedList<Double> list = buckets[i];
        insertionSort(list);
        for (int j = 0; j < list.size(); j++) {
            arr[pointer++] = list.get(j);
        }
    }
}

private void insertionSort(LinkedList<Double> list) {
    for (int j = 1; j < list.size(); j++) {
        double key = list.get(j);
        int i = j - 1;
        while (i >= 0 && list.get(i) > key) {
            list.set(i + 1, list.get(i));
            i--;
        }
        list.set(i + 1, key);
    }
}
```

即使输入数据不服从均匀分布，桶排序也仍然可以在线性时间内完成，只要输入数据满足下列性质：所有桶的大小的平方和与总的元素数呈线性关系。
