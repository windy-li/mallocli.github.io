## 最长公共子序列

一个给定序列的子序列，就是将给定序列中零个或多个元素去掉之后得到的结果。其形式化定义如下：给定一个序列 X = <x<sub>1</sub>, x<sub>2</sub>, ..., x<sub>n</sub>>，另一个序列 Z = <z<sub>1</sub>, z<sub>2</sub>, ..., z<sub>k</sub>> 满足如下条件时称为 X 的子序列（subsequence），即存在一个严格递增的 X 的下标序列 <i<sub>1</sub>, i<sub>2</sub>, ..., i<sub>k</sub>>，对所有 j = 1, 2, ..., k，满足 x[i<sub>k</sub>] = z[j]。例如，Z = <B, C, D, B> 是 X = <A, B, C, B, D, A, E> 的子序列，对应的下标序列为 <2, 3, 5, 7>。

给定两个序列 X 和 Y，如果 Z 既是 X 的子序列，也是 Y 的子序列，我们称它是 X 和 Y 的公共子序列（common subsequence）。例如，如果 X = <A, B, C, B, D, A, B>，Y = <B, D, C, A, B, A>，那么序列 <B, C, A> 就是 X 和 Y 的公共子序列。但它不是 X 和 Y 的最长公共子序列（LCS），因为它的长度为 3，而 <B, C, B, A> 也是 X 和 Y 的公共子序列，其长度为 4。<B, C, B, A> 是 X 和 Y 的最长公共子序列，<B, D, A, B> 也是，因为 X 和 Y 不存在长度大于 5 等于 5 的公共子序列。

最长公共子序列问题（longest-common-subsequence problem）就是给定两个序列 X = <x[1], x[2], ..., x[m]> 和 Y = <y[1], y[2], ..., y[n]>，求 X 和 Y 长度最长的公共子序列。

### 步骤 1：刻画最长公共子序列的特征

如果用暴力搜索方法求解 LCS 问题，就要穷举 X 的所有子序列，对每个子序列检查它是否也是 Y 的子序列，记录找到的最长子序列。X 的每个子序列对应 X 的下标集合 {1, 2, ..., m} 的一个子集，所以 X 有 2^m 个子序列，因此暴力方法的运行时间为指数阶，对较长的序列是不实用的。

但是，LCS 问题具有最优子结构性质。子问题的自然分类对应两个输入序列的“前缀”对，前缀的严谨定义如下：给定一个序列 X = <x[1], x[2], ..., x[m]>，对 i = 0, 1, ..., m，定义 X 的第 i 前缀为 X[i] = <x[1], x[2], ..., x[i]>。例如，若 X = <A, B, C, B, D, A, B>，则 X[4] = <A, B, C, B>，X[0] 为空串。

定理（LCS 的最优子结构）：令 X = <x[1], x[2], ..., x[m]> 和 Y = <y[1], y[2], ..., y[n]> 为两个序列，Z = <z[1], z[2], ..., z[k]> 为 X 和 Y 的任意 LCS。

```
1. 如果 x[m] = y[n]，则 z[k] = x[m] = y[n] 且 Z[k-1] 是 X[m-1] 和 Y[n-1] 的一个 LCS。

2. 如果 x[m] != y[n]，那么 z[k] != x[m] 意味着 Z 是 X[m-1] 和 Y 的一个 LCS。

3. 如果 x[m] != y[n]，那么 z[k] != y[n] 意味着 Z 是 X 和 Y[n-1] 的一个 LCS。
```

证明：（1）如果 z[k] != x[m]，那么可以将 x[m]= y[n] 追加到 Z 的末尾，得到 X 和 Y 的一个长度为 k + 1 的公共子序列，与 Z 是 X 和 Y 的最长公共子序列的假设矛盾。因此，必然有 z[k] = x[m] = y[n]。这样，前缀 Z[k-1] 是 X[m-1] 和 Y[n-1] 的一个长度为 k - 1 的公共子序列。我们希望证明它是一个 LCS。利用反证法，假设存在 X[m-1] 和 Y[n-1] 的一个长度大于 k - 1 的公共子序列 W，则将 x[m] = y[n] 追加到 W 的末尾会得到 X 和 Y 的一个长度大于 k 的公共子序列，矛盾。  

（2）如果 z[k] != x[m]，那么 Z 是 X[m-1] 和 Y 的一个公共子序列。如果存在 X[m-1] 和 Y 的一个长度大于 k 的公共子序列 W，那么 W 也是 X[m] 和 Y 的公共子序列，与 Z 是 X 和 Y 的最长公共子序列的假设矛盾。  

（3）与情况（2）对称。

上述定理告诉我们，两个序列的 LCS 包含两个序列的前缀的 LCS。因此，LCS 问题具有最优子结构性质。

### 步骤 2：一个递归解

上述定理意味着，在求 X = <x[1], x[2], ..., x[m]> 和 Y = <y[1], y[2], ..., y[n]> 的一个 LCS 时，我们需要求解一个或两个子问题。如果 x[m] = y[n]，我们应该求解 X[m-1] 和 Y[n-1] 的一个 LCS，将 x[m] = y[n] 追加到这个 LCS 的末尾，就得到 X 和 Y 的一个 LCS。如果 x[m] != y[n]，我们必须求解两个子问题：求 X[m-1] 和 Y 的一个 LCS 与 X 和 Y[n-1] 的一个 LCS，两个 LCS 较长者即为 X 和 Y 的一个 LCS。由于这些情况覆盖了所有可能性，因此我们知道必然有一个子问题的最优解出现在 X 和 Y 的 LCS 中。

我们很容易看出 LCS 问题的重叠子问题性质。为了求 X 和 Y 的一个 LCS，我们可能需要求 X 和 Y[n-1] 的一个 LCS 及 X[m-1] 和 Y 的一个 LCS。但这几个子问题都包含求解 X[m-1] 和 Y[n-1] 的 LCS 的子子问题，很多其它子问题也都共享子子问题。

设计 LCS 问题的递归算法首先要建立最优解的递归式。我们定义 c[i][j] 表示 X[i] 和 Y[j] 的 LCS 的长度，如果 i = 0 或 j = 0，即一个序列长度为 0，那么 LCS 的长度为 0。根据 LCS 问题的最优子结构性质，可得如下公式：

![](../assets/images/part4/longest-common-subsequence.png)

观察到在递归公式中，我们通过限制条件限制了需要求解哪些子问题。当 x[i] = y[j] 时，我们可以而且应该求解子问题：X[i-1] 和 Y[j-1] 的一个 LCS。否则，应该求解两个子问题：X[i] 和 Y[j-1] 的一个 LCS 及 X[i-1] 和 Y 的一个 LCS。

### 步骤 3：计算 LCS 的长度

根据以上的递归公式，我们可以很容易地写出一个指数时间的递归算法来求解两个序列的 LCS 的长度。

```java
int recursive(char[] x, char[] y, int m, int n) {
    if (m == 0 || n == 0) {
        return 0;
    } else if (x[m - 1] == y[n - 1]) {
        return recursive(x, y, m - 1, n - 1) + 1;
    } else {
        return Math.max(recursive(x, y, m - 1, n), recursive(x, y, m, n - 1));
    }
}
```

但是，由于 LCS 问题只有 Θ(mn) 个不同的子问题，我们可以用动态规划方法自底向上地计算。

```java
int bottomUp(char[] x, char[] y) {
    int m = x.length;
    int n = y.length;
    int[][] c = new int[m + 1][n + 1];
    for (int i = 0; i <= m; i++) {
        c[i][0] = 0;
    }
    for (int j = 0; j <= n; j++) {
        c[0][j] = 0;
    }
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (x[i - 1] == y[j - 1]) {
                c[i][j] = c[i - 1][j - 1] + 1;
            } else {
                c[i][j] = Math.max(c[i - 1][j], c[i][j - 1]);
            }
        }
    }
    return c[m][n];
}
```

bottomUp 接受两个序列 X = <x[1], x[2], ..., x[m]> 和 Y = <y[1], y[2], ..., y[n]> 为输入，它将 c[i][j] 的值保存在表 c[0..m][0..n] 中，并按行主次序（row-major order）计算表项（即首先由左至右计算 c 的第一行，然后计算第二行，依此类推），c[m][n] 保存了 X 和 Y 的 LCS 长度。下面的 extendedBottomUp 过程还维护一个表 b[1..m][1..n]，用来帮助构造最优解。b[i][j] 指向的表项对应计算 c[i][j] 时所选择的子问题最优解。

```java
static final int LEFT = 1;
static final int TOP = 2;
static final int TURN = 3;

Object[] extendedBottomUp(char[] x, char[] y) {
    int m = x.length;
    int n = y.length;
    int[][] c = new int[m + 1][n + 1];
    int[][] b = new int[m + 1][n + 1];
    for (int i = 0; i <= m; i++) {
        c[i][0] = 0;
    }
    for (int j = 0; j <= n; j++) {
        c[0][j] = 0;
    }
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (x[i - 1] == y[j - 1]) {
                c[i][j] = c[i - 1][j - 1] + 1;
                b[i][j] = TURN;
            } else {
                if (c[i - 1][j] >= c[i][j - 1]) {
                    c[i][j] = c[i - 1][j];
                    b[i][j] = TOP;
                } else {
                    c[i][j] = c[i][j - 1];
                    b[i][j] = LEFT;
                }
            }
        }
    }
    return new Object[]{c[m][n], b};
}
```

下图显示了 extendedBottomUp 对输入序列 X = <A, B, C, B, D, A, B> 和 Y = <B, D, C, A, B, A> 生成的结果。过程的运行时间为 Θ(mn)，因为每个表项的计算时间为 Θ(1)。

![](../assets/images/part4/longest-common-subsequence1.png)

图中第 i 行和第 j 列的方格包含了 c[i][j] 的值和 b[i][j] 记录的箭头。表项 c[7][6]（表的右下角）中的 4 即为 X 和 Y 的一个 LCS <B, C, B, A> 的长度。对所有 i, j > 0，表项 c[i][j] 仅依赖于是否 x[i] = y[j] 以及 c[i-1][j]、c[i][j-1] 和 c[i-1][j-1] 的值，这些值都会在 c[i][j] 之前计算出来。为了构造 LCS 中的元素，从右下角开始沿着 b[i][j] 的箭头前进即可，如图中阴影方格序列。阴影序列中每个“↖”对应的表项表示 x[i] = y[j] 是 LCS 的一个元素。

### 步骤 4：构造 LCS

我们可以用 extendedBottomUp 返回的表 b 快速构造 X = <x[1], x[2], ..., x[m]> 和 Y = <y[1], y[2], ..., y[n]> 的 LCS，只需简单地从 b[m][n] 开始，并按箭头方向追踪下去即可。当在表项 b[i][j] 中遇到一个“↖”时，意味着 x[i] = y[j] 是 LCS 的一个元素。按照这种方法，我们可以按逆序依次构造出 LCS 的所有元素。

```java
void constructSolution(int[][] b, char[] x, int i, int j) {
    if (i == 0 || j == 0) {
        return;
    }
    if (b[i][j] == TURN) {
        constructSolution(b, x, i - 1, j - 1);
        System.out.print(x[i - 1]);
    } else {
        if (b[i][j] == TOP) {
            constructSolution(b, x, i - 1, j);
        } else {
            constructSolution(b, x, i, j - 1);
        }
    }
}
```

对于上图中的表 b，constructSolution 会打印出 BCBA，过程的运行时间为 O(m + n)，因为每次递归调用 i 和 j 至少有一个会减少 1。

### 算法改进

对于 extendedBottomUp 过程，我们完全可以去掉表 b，每个 c[i][j] 项只依赖于表 c 中其它三项：c[i-1][j]、c[i][j-1] 和 c[i-1][j-1]。给定 c[i][j] 的值，我们可以在 O(1) 时间内判断出在计算 c[i][j] 时使用了这三项中的哪一项。因此，我们可以用一个类似 constructSolution 的过程在 O(m + n) 时间内完成重构 LCS 的工作，而且不必使用表 b。但是，虽然这种方法节省了 Θ(mn) 的空间，但计算 LCS 所需的辅助空间并未渐近减少，因为无论如何表 c 都需要 Θ(mn) 的空间。

不过，bottomUp 过程的空间需求是可以渐近减少的，因为无论在任何时刻它只需要表 c 中的两行：当前正在计算的一行和前一行。如果我们只需计算 LCS 的长度，这一改进是有效的，但如果要重构 LCS 中的元素，这么小的表空间所保存的信息不足以在 O(m + n) 时间内完成重构工作。

