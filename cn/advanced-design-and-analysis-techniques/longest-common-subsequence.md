## 最长公共子序列

一个给定序列的子序列，就是将给定序列中零个或多个元素去掉之后得到的结果。其形式化定义如下：给定一个序列X = <x[1], x[2], ..., x[n]>，另一个序列Z = <z[1], z[2], ..., z[k]>满足如下条件时称为X的子序列(subsequence)，即存在一个严格递增的X的下标序列<i1, i2, ..., ik>，对所有j = 1, 2, ..., k，满足x[i[k]] = z[j]。例如，Z = <B, C, D, B>是X = <A, B, C, B, D, A, E>的子序列，对应的下标序列为<2, 3, 5, 7>。

给定两个序列X和Y，如果Z既是X的子序列，也是Y的子序列，我们称它是X和Y的公共子序列(common subsequence)。例如，如果X = <A, B, C, B, D, A, B>, Y = <B, D, C, A, B, A>，那么序列<B, C, A>就是X和Y的公共子序列。但它不是X和Y的最长公共子序列(LCS)，因为它的长度为3，而<B, C, B, A>也是X和Y的公共子序列，其长度为4。<B, C, B, A>是X和Y的最长公共子序列，<B, D, A, B>也是，因为X和Y不存在长度大于5等于5的公共子序列。

**最长公共子序列问题(longest-common-subsequence problem)**给定两个序列X = <x[1], x[2], ..., x[m]>和Y = <y[1], y[2], ..., y[n]>，求X和Y长度最长的公共子序列。

#### 步骤1：刻画最长公共子序列的特征

如果用暴力搜索方法求解LCS问题，就要穷举X的所有子序列，对每个子序列检查它是否也是Y的子序列，记录找到的最长子序列。X的每个子序列对应X的下标集合{1, 2, ..., m}的一个子集，所以X有2 ^ m个子序列，因此暴力方法的运行时间为指数阶，对较长的序列是不实用的。

但是，LCS问题具有最优子结构性质。子问题的自然分类对应两个输入序列的“前缀”对，前缀的严谨定义如下：给定一个序列X = <x[1], x[2], ..., x[m]>，对i = 0, 1, ..., m，定义X的第i前缀为X[i] = <x[1], x[2], ..., x[i]>。例如，若X = <A, B, C, B, D, A, B>，则X[4] = <A, B, C, B>，X[0]为空串。

LCS的最优子结构：令X = <x[1], x[2], ..., x[m]>和Y = <y[1], y[2], ..., y[n]>为两个序列，Z = <z[1], z[2], ..., z[k]>为X和Y的任意LCS。
1. 如果x[m] == y[n]，则z[k] == x[m] == y[n]且Z[k-1]是X[m-1]和Y[n-1]的一个LCS。
2. 如果x[m] != y[n]，那么z[k] != x[m]意味着Z是X[m-1]和Y的一个LCS。
3. 如果x[m] != y[n]，那么z[k] != y[n]意味着Z是X和Y[n-1]的一个LCS。

#### 步骤2：一个递归解

上述定理意味着，在求X = <x[1], x[2], ..., x[m]>和Y = <y[1], y[2], ..., y[n]>的一个LCS时，我们需要求解一个或两个子问题。如果x[m] == y[n]，我们应该求解X[m-1]和Y[n-1]的一个LCS。将x[m] = y[n]追加到这个LCS的末尾，就得到X和Y的一个LCS。如果x[m] != y[n],我们必须求解两个子问题：求X[m-1]和Y的一个LCS与X和Y[n-1]的一个LCS，两个LCS较长者即为X和Y的一个LCS。由于这些情况覆盖了所有可能性，因此我们知道必然有一个子问题的最优解出现在X和Y的LCS中。

我们很容易看出LCS问题的重叠子问题性质。为了求X和Y的一个LCS，我们可能需要求X和Y[n-1]的一个LCS及X[m-1]和Y的一个LCS。但这几个子问题都包含求解X[m-1]和Y[n-1]的LCS的子子问题，很多其它子问题也都共享子子问题。

设计LCS问题的递归算法首先要建立最优解的递归式。我们定义c[i][j]表示X[i]和Y[j]的LCS的长度，如果i == 0或j == 0，即一个序列长度为0，那么LCS的长度为0。根据LCS问题的最优子结构性质，可得如下公式：

![](../../assets/images/part4/longest-common-subsequence.png)

观察到在递归公式中，我们通过限制条件限制了需要求解哪些问题。当x[i] == y[j]时，我们可以而且应该求解子问题X[i-1]和Y[j-1]的一个LCS。否则，应该求解两个子问题：X[i]和Y[j-1]的一个LCS及X[i-1]和Y的一个LCS。

#### 步骤3：计算LCS的长度

根据以上的递归公式，我们可以很容易地写出一个指数时间的递归算法来求解两个序列的LCS的长度。

```java
public int recursiveLcs(char[] x, char[] y, int m, int n) {
    if (m == 0 || n == 0) {
        return 0;
    } else if (x[m - 1] == y[n - 1]) {
        return recursiveLcs(x, y, m - 1, n - 1) + 1;
    } else {
        return Math.max(recursiveLcs(x, y, m - 1, n), recursiveLcs(x, y, m, n - 1));
    }
}
```

但是，由于LCS问题只有Θ(m * n)个不同的子问题，我们可以用动态规划方法自底向上计算。

```java
private static final int LEFT = 1;
private static final int TOP = 2;
private static final int TURN = 3;

public int bottomUpLcs(char[] x, char[] y) {
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

public Object[] extendedBottomUpLcs(char[] x, char[] y) {
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

#### 构造LCS

我们可以用extendedBottomUpLcs返回的表b快速构造X = <x[1], x[2], ..., x[m]>和Y = <y[1], y[2], ..., y[n]>的LCS，只需简单地从b[m][n]开始，并按箭头方向追踪下去即可。当在表项中遇到TURN时，意味着x[i] == y[j]是LCS的一个元素。按照这种方法，我们可以逆序依次构造出LCS的所有元素。

```java
public void constructSolution(int[][] b, char[] x, int i, int j) {
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
