## d叉堆

d叉堆与二叉堆类似，但一个例外是其中的每个非叶节点有d个孩子，而不是2个。

```java
public class DaryHeap {
    private int d;
    private int[] arr;
    private int heapSize;
    
    public DaryHeap(int capacity, int d) {
        this.d = d;
        arr = new int[capacity];
        heapSize = 0;
    }
    
    public int extractMax(){
        if (heapSize < 1) {
            throw new RuntimeException("heap underflow");
        }
        int max = arr[0];
        arr[0] = arr[heapSize - 1];
        heapSize--;
        maxHeapify(0);
        return max;
    }
    
    private void maxHeapify(int i) {
        while (true) {
            int l = left(i);
            int r = right(i);
            int largest;
            for (int j = l; j <= r; j++) {
                if (j < heapSize && arr[j] > arr[largest]) {
                    largest = j;
                }
            }
            if (largest != i) {
                swap(arr, largest, i);
                i = largest;
            } else {
                return;
            }
        }
    }
    
    private void parent(int i) {
        return (i - 1) / d;
    }
    
    private int left(int i) {
        return d * i + 1;
    }
    
    private int right(int i) {
        return d * i + d;
    }
    
    private int child(int i, int k) {
        return d * i + 1 + k;
    }
    
    private void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
    
    public void increaseKey(int i, int newKey) {
        if (newKey < arr[i]) {
            throw new IllegalArgumentException("new key is smaller than current key");
        }
        while (i > 0 && arr[parent(i) < newKey]) {
            arr[i] = arr[parent(i)];
            i = parent(i);
        }
        arr[i] = newKey;
    }
    
    public void insert(int key) {
        if (heapSize == arr.length) {
            throw new RuntimeException("heap overflow");
        }
        heapSize++;
        arr[heapSize - 1] = Integer.MIN_VALUE;
        increaseKey(heapSize - 1, key);
    }
}
```
