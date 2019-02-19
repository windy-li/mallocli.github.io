## 最长公共子序列

一个给定序列的子序列，就是将给定序列中零个或多个元素去掉之后得到的结果。其形式化定义如下：给定一个序列 X = <x[1], x[2], ..., x[n]>，另一个序列 Z = <z[1], z[2], ..., z[k]> 满足如下条件时称为 X 的子序列（subsequence），即存在一个严格递增的 X 的下标序列 <i1, i2, ..., ik>，对所有 j = 1, 2, ..., k，满足 x[i[k]] = z[j]。例如，Z = <B, C, D, B> 是 X = <A, B, C, B, D, A, E> 的子序列，对应的下标序列为 <2, 3, 5, 7>。

给定两个序列 X 和 Y，如果 Z 既是 X 的子序列，也是 Y 的子序列，我们称它是 X 和 Y 的公共子序列（common subsequence）。例如，如果 X = <A, B, C, B, D, A, B>，Y = <B, D, C, A, B, A>，那么序列 <B, C, A> 就是 X 和 Y 的公共子序列。但它不是 X 和 Y 的最长公共子序列（LCS），因为它的长度为 3，而 <B, C, B, A> 也是 X 和 Y 的公共子序列，其长度为 4。<B, C, B, A> 是 X 和 Y 的最长公共子序列，<B, D, A, B> 也是，因为 X 和 Y 不存在长度大于 5 等于 5 的公共子序列。

最长公共子序列问题（longest-common-subsequence problem）就是给定两个序列 X = <x[1], x[2], ..., x[m]> 和 Y = <y[1], y[2], ..., y[n]>，求 X 和 Y 长度最长的公共子序列。

#### 步骤 1：刻画最长公共子序列的特征

如果用暴力搜索方法求解 LCS 问题，就要穷举 X 的所有子序列，对每个子序列检查它是否也是 Y 的子序列，记录找到的最长子序列。X 的每个子序列对应 X 的下标集合 {1, 2, ..., m} 的一个子集，所以 X 有 2^m 个子序列，因此暴力方法的运行时间为指数阶，对较长的序列是不实用的。

但是，LCS 问题具有最优子结构性质。子问题的自然分类对应两个输入序列的“前缀”对，前缀的严谨定义如下：给定一个序列 X = <x[1], x[2], ..., x[m]>，对 i = 0, 1, ..., m，定义 X 的第 i 前缀为 X[i] = <x[1], x[2], ..., x[i]>。例如，若 X = <A, B, C, B, D, A, B>，则 X[4] = <A, B, C, B>，X[0] 为空串。

LCS 的最优子结构：令 X = <x[1], x[2], ..., x[m]> 和 Y = <y[1], y[2], ..., y[n]> 为两个序列，Z = <z[1], z[2], ..., z[k]> 为 X 和 Y 的任意 LCS。

1. 如果 x[m] = y[n]，则 z[k] = x[m] = y[n] 且 Z[k-1] 是 X[m-1] 和 Y[n-1] 的一个 LCS。

2. 如果 x[m] != y[n]，那么 z[k] != x[m] 意味着 Z 是 X[m-1] 和 Y 的一个 LCS。

3. 如果 x[m] != y[n]，那么 z[k] != y[n] 意味着 Z 是 X 和 Y[n-1] 的一个 LCS。

#### 步骤 2：一个递归解

上述定理意味着，在求 X = <x[1], x[2], ..., x[m]> 和 Y = <y[1], y[2], ..., y[n]> 的一个 LCS 时，我们需要求解一个或两个子问题。如果 x[m] = y[n]，我们应该求解 X[m-1] 和 Y[n-1] 的一个 LCS。将 x[m] = y[n] 追加到这个 LCS 的末尾，就得到 X 和 Y 的一个 LCS。如果 x[m] != y[n]，我们必须求解两个子问题：求 X[m-1] 和 Y 的一个 LCS 与 X 和 Y[n-1] 的一个 LCS，两个 LCS 较长者即为 X 和 Y 的一个 LCS。由于这些情况覆盖了所有可能性，因此我们知道必然有一个子问题的最优解出现在 X 和 Y 的 LCS 中。

我们很容易看出 LCS 问题的重叠子问题性质。为了求 X 和 Y 的一个 LCS，我们可能需要求 X 和 Y[n-1] 的一个 LCS 及 X[m-1] 和 Y 的一个 LCS。但这几个子问题都包含求解 X[m-1] 和 Y[n-1] 的 LCS 的子子问题，很多其它子问题也都共享子子问题。

设计 LCS 问题的递归算法首先要建立最优解的递归式。我们定义 c[i][j] 表示 X[i] 和 Y[j] 的 LCS 的长度，如果 i = 0 或 j = 0，即一个序列长度为 0，那么 LCS 的长度为 0。根据 LCS 问题的最优子结构性质，可得如下公式：

![](../assets/images/part4/longest-common-subsequence.png)

观察到在递归公式中，我们通过限制条件限制了需要求解哪些问题。当 x[i] = y[j] 时，我们可以而且应该求解子问题 X[i-1] 和 Y[j-1] 的一个 LCS。否则，应该求解两个子问题：X[i] 和 Y[j-1] 的一个 LCS 及 X[i-1] 和 Y 的一个 LCS。

#### 步骤 3：计算 LCS 的长度

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

但是，由于 LCS 问题只有 Θ(m * n) 个不同的子问题，我们可以用动态规划方法自底向上计算。

```java
static final int LEFT = 1;
static final int TOP = 2;
static final int TURN = 3;

int bottomUp(char[] x, char[] y) {
    int m = x.length;
    int n = y.length;
    int[][] L = new int[m + 1][n + 1];
    for (int i = 0; i <= m; i++) {
        L[i][0] = 0;
    }
    for (int j = 0; j <= n; j++) {
        L[0][j] = 0;
    }
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (x[i - 1] == y[j - 1]) {
                L[i][j] = L[i - 1][j - 1] + 1;
            } else {
                L[i][j] = Math.max(L[i - 1][j], L[i][j - 1]);
            }
        }
    }
    return L[m][n];
}

Object[] extendedBottomUp(char[] x, char[] y) {
    int m = x.length;
    int n = y.length;
    int[][] L = new int[m + 1][n + 1];
    int[][] b = new int[m + 1][n + 1];
    for (int i = 0; i <= m; i++) {
        L[i][0] = 0;
    }
    for (int j = 0; j <= n; j++) {
        L[0][j] = 0;
    }
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (x[i - 1] == y[j - 1]) {
                L[i][j] = L[i - 1][j - 1] + 1;
                b[i][j] = TURN;
            } else {
                if (L[i - 1][j] >= L[i][j - 1]) {
                    L[i][j] = L[i - 1][j];
                    b[i][j] = TOP;
                } else {
                    L[i][j] = L[i][j - 1];
                    b[i][j] = LEFT;
                }
            }
        }
    }
    return new Object[]{L[m][n], b};
}
```

#### 构造 LCS

我们可以用 extendedBottomUp 返回的表 b 快速构造 X = <x[1], x[2], ..., x[m]> 和 Y = <y[1], y[2], ..., y[n]> 的 LCS，只需简单地从 b[m][n] 开始，并按箭头方向追踪下去即可。当在表项中遇到 TURN 时，意味着 x[i] = y[j] 是 LCS 的一个元素。按照这种方法，我们可以逆序依次构造出 LCS 的所有元素。

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
