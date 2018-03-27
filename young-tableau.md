## Young氏矩阵

在一个m * n的Young氏矩阵中，每一行的数据都是从左到右排序的，每一列的数据都是从上到下排序的。Young氏矩阵也会存在一些值为无穷大(INF)的数据项，表示哪些不存在的元素。因此，Young氏矩阵可以存储r <= m * n个有限的数。

对于一个m * n的Young氏矩阵Y来说，如果Y[0][0] = INF，则Y为空，如果Y[m][n] < INF，则Y为满（即包含m * n个元素）。

```java
public class YoungTableau {
    private int m;
    private int n;
    private int[][] mat;
    private static final int INF = Integer.MAX_VALUE;
    
    public YoungTableau(int m, int n) {
        mat = new int[m][n];
        this.m = m;
        this.n = n;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                mat[i][j] = INF;
            }
        }
    }
    
    public boolean isEmpty() {
        return mat[0][0] = INF;
    }
    
    public boolean isFull() {
        return mat[m - 1][n - 1] < INF;
    }
}
```

下面是一个在m * n的Young氏矩阵上复杂度为O(m + n)的extractMin算法实现。考虑使用一个递归过程，它使用类似于maxHeapify的方式，把一个规模为m * n的问题分解为规模为(m - 1) * n或者m * (n - 1)的子问题。

```java
public int extractMin() {
    if (isEmpty()) {
        throw new RuntimeException("Young tableau underflow");
    }
    int ret = mat[0][0];
    mat[0][0] = INF;
    youngify(0, 0);
    return ret;
}

private void youngify(int i, int j) {
    int downKey = (i < n - 1) ? mat[i + 1][j] : INF;
    int rightKey = (j < m - 1) ? mat[i][j + 1] : INF;
    if (downKey == INF && rightKey == INF) {
        return;
    }
    if (downKey < rightKey) {
        mat[i][j] = downKey;
        mat[i + 1][j] = INF;
        youngify(i + 1, j);
    } else {
        mat[i][j] = rightKey;
        mat[i][j + 1] = INF;
        youngify(i, j + 1);
    }
}
```

可以在O(m + n)时间内，将一个新元素插入到一个未满的m * n的杨氏矩阵中。

```java
public void insert(int key) {
    if (isFull()) {
        throw new RuntimeException("Young tableau overflow");
    }
    decreaseKey(m - 1, n - 1, key);
}

private void decreaseKey(int i, int j, int key) {
    if (key > mat[i][j]) {
        throw new IllegalArgumentException("new key is larger than current key");
    }
    int topKey = (i > 0) ? mat[i - 1][j] : Integer.MIN_VALUE;
    int leftKey = (j > 0) ? mat[i][j - 1] : Integer.MIN_VALUE;
    if (key >= topKey && key >= leftKey) {
        mat[i][j] = key;
        return;
    }
    if (topKey > leftKey) {
        mat[i][j] = topKey;
        mat[i - 1][j] = key;
        decreaseKey(i - 1, j, key);
    } else {
        mat[i][j] = leftKey;
        mat[i][j - 1] = key;
        decreaseKey(i, j - 1, key);
    }
}
```

contains方法的时间复杂度为O(m + n)，它可以用来判断一个给定的数是否存储在m * n的Young氏矩阵中。

```java
public boolean contains(int key) {
    int row = 0;
    int col = n - 1;
    while (col >= 0 && row <= n - 1) {
        int val = mat[row][col];
        if (key == val) {
            return true;
        } else if (key < val) {
            col--;
        } else {
            row++;
        }
    }
    return false;
}
```
