## 矩阵链乘法

给定一个n个矩阵的序列(矩阵链) <A1, A2, ..., An>，我们希望计算它们的乘积 A1A2...An。为了计算矩阵链乘积，我们可以先用括号明确计算次序，然后利用标准的矩阵相乘算法进行计算。由于矩阵乘法满足结合律，因此任何加括号的方法都会得到相同的计算结果。我们称有如下性质的矩阵乘积链为完全括号化的(fully parenthesized)：它是单一矩阵，或者是两个完全括号化的矩阵链的积，且已外加括号。例如，如果矩阵链为<A1, A2, A3, A4>，则共有5种完全括号化的矩阵乘积链：
1. (A1(A2(A3A4)))
2. (A1((A2A3)A4))
3. ((A1A2)(A3A4)
4. ((A1(A2A3))A4)
5. (((A1A2)A3)A4)

对矩阵链加括号的方式会对乘积运算的代价产生巨大影响，我们先来分析两个矩阵相乘的代价。下面是两个矩阵相乘的标准乘法。

```java
public int[][] matrixMultiply(int[][] a, int[][] b) {
    int aRows = a.length;
    int aColumns = a[0].length;
    int bRows = b.length;
    int bColumns = b[0].length;
    if (aColumns != bRows) {
        throw new IllegalArgumentException("incompatible dimensions");
    } else {
        int[][] c = new int[aRows][bColumns];
        for (int i = 0; i < aRows; i++) {
            for (int j = 0; j < bColumns; j++) {
                c[i][j] = 0;
                for (int k = 0; k < aColumns; k++) {
                    c[i][j] += a[i][k] + b[k][j];
                }
            }
        }
        return c;
    }
}
```

两个矩阵A和B只有相容(compatible)，即A的列数等于B的行数时，才能相乘。如果A是p x q的矩阵，B是q x r的矩阵，那么乘积C是p x r的矩阵。计算C所需时间由p * q * r决定。

我们以矩阵链<A1, A2, A3>相乘为例，来说明不同的加括号方案会导致不同的计算代价。假设三个矩阵的规模分别为10 x 100, 100 x 5和5 x 50。如果按((A1A2)A3)的顺序计算，为计算A1A2，需要做10 * 100 * 5 = 5000次标量乘法，再与A3相乘又需要做10 * 5 * 50 = 2500次乘法，共需7500次标量乘法。如果按(A1(A2A3))的顺序，计算A2A3需100 * 5 * 50 = 25000次乘法，A1再与之相乘又需10 * 100 * 50 = 50000次标量乘法，共需75000次标量乘法。因此，按第一种顺序计算矩阵链乘积要比第二种顺序快10倍。

矩阵链乘法问题(matrix-chain multiplication problem)可描述如下：给定n个矩阵的链<A1, A2, ..., An>，矩阵Ai的规模为p[i - 1] x p[i] (1 <= i <= n)，求完全括号化方案，使得计算乘积A1A2...An所需标量乘法次数最少。

注意，求解矩阵链乘法问题并不是要真正进行矩阵相乘运算，我们的目标只是确定代价最低的计算顺序。确定最优计算顺序所花费的时间通常要比随后真正进行矩阵相乘所节省的时间(例如仅进行7500次标量乘法而不是75000次)要少。

令P(n)表示可供选择的括号化方案的数量。当n = 1时，由于只有一个矩阵，因此只有一种完全括号化方案。当n >= 2时，完全括号化方案的矩阵乘积可描述为两个完全括号化的部分积相乘的形式，而两个部分积的划分点在第k个矩阵和第k + 1个矩阵之间，k为1, 2, ..., n - 1中的任意一个值。因此，我们可以得到如下递归公式：

![](../../assets/images/part4/matrix-chain-multiplication.png)

括号化方案的数量与n呈指数关系，通过暴力搜索穷尽所有可能的括号化方案来寻找最优方案，是一个糟糕的策略。

### 应用动态规划方法

下面用动态规划方法来求解矩阵链的最优括号化方案，还是按照动态规划的4个步骤进行：

1. 刻画一个最优解的结构特征。

2. 递归地定义最优解的值。

3. 计算最优解的值，通常采用自底向上的方法。

4. 利用计算出的信息构造一个最优解。

我们按顺序进行这几个步骤，清楚地展示针对本问题每个步骤应该怎么做。

#### 步骤1：最优括号化方案的结构特征

动态规划方法的第一步是寻找最优子结构，然后就可以利用这种子结构从子问题的最优解构造出原问题的最优解。在矩阵链乘法问题中，此步骤的做法如下所述。为方便起见，我们用符号Ai..j(i <= j)表示AiAi+1...Aj乘积的结果矩阵。可以看出，如果问题是非平凡的，即i < j，那么为了对AiAi+1...Aj进行括号化，我们就必须在某个Ak和Ak+1之间将矩阵链划分开(k为i <= k < j之间的整数)。也就是说，对某个整数k，我们首先计算矩阵Ai..k和Ak+1..j，然后再计算它们的乘积得到最终结果Ai..j。此方案的计算代价等于矩阵Ai..k的计算代价，加上矩阵Ak+1..j的计算代价，再加上两者相乘的计算代价。

#### 步骤2：一个递归求解方案

下面用子问题的最优解来递归地定义原问题最优解的代价。对矩阵链乘法问题，我们可以将对所有1 <= i <= j <= n确定AiAi+1...Aj的最小代价括号化方案作为子问题，令m[i][j]表示计算矩阵Aij所需标量乘法次数的最小值，那么，原问题的最优解——计算A1..n所需的最低代价就是m[1][n]。

我们可以递归定义m[i][j]如下。对于i = j时的平凡问题，矩阵链只包含唯一的矩阵Ai..i = Ai，因此不需要做任何标量乘法运算。所以对所有i = 1, 2, ..., n, m[i][j] = 0。若i < j，我们利用步骤1中得到的最优子结构来计算m[i][j]。我们假设AiAi+1..Aj的最优括号化方案的分割点在矩阵Ak和Ak+1之间，其中i <= k < j。那么，m[i][j]就等于计算Ai..k和Ak+1..j的代价加上两者相乘的代价的最小值。由于矩阵Ai的大小为p[i - 1] x p[i]，易知Ai..k与Ak+1..j相乘的代价为p[i - 1] * p[k] * p[j]次标量乘法运算。因此，我们得到：

m[i][j] = m[i][k] + m[k + 1][j] + p[i - 1] * p[k] * [pj]

此递归公式假定最优分割点k是已知的，但实际上我们是不知道的。不过，k只有j - i种可能的取值，即k = i, i+1, ..., j-1。由于最优分割点必在其中，我们只需检查所有可能情况，找到最优解即可。因此，AiAi+1...Aj最小代价括号化方案的递归求解公式变为：

![](../../assets/images/part4/matrix-chain-multiplication2.png)

m[i][j]的值给出了子问题最优解的代价，但它并未提供足够的信息来构造最优解，为此，我们用s[i][j]保存AiAi+1..Aj最优括号化方案的分割点位置k，即使得m[i][j] = m[i][k] + m[k + 1][j] + p[i - 1] * p[k] * p[j]成立的k的值。

#### 步骤3：计算最优代价

现在，我们可以很容易地基于以上的递归公式写出一个递归算法，来计算A1A2...An相乘的最小代价m[1][n]。

```java
public int recursiveMatrixChain(int[] p, int i, int j) {
    if (i == j) {
        return 0;
    }
    int min = Integer.MAX_VALUE;
    for (int k = i; k < j; k++) {
        min = Math.min(min, recursiveMatrixChain(p, i, k) + recursiveMatrixChain(p, k + 1, j) + p[i - 1] * p[k] * p[j]);
    }
    return min;
}
```

注意到，我们需要求解的不同子问题的数目是相对较少的，每对满足1 <= i <= j <= n的i和j对应一个唯一的子问题，共有Θ(n ^ 2)个。递归算法会在递归调用树的不同分支中多次遇到同一个子问题，这种子问题重叠的性质是动态规划的另一个标识(第一个标识是最优子结构)。

我们采用自底向上的表格法替代递归式来计算最优代价。为了实现自底向上方法，我们必须确定计算m[i][j]时需要访问哪些其它表项。上述公式显示，j - i + 1个矩阵链相乘的最优计算代价m[i][j]只依赖于那些少于j - i + 1个矩阵链相乘的最优计算代价。也就是说，对k = i, i+1, ..., j-1，矩阵Ai..k是k - i + 1 < j - i + 1个矩阵的积，矩阵Ak+1...j是j - k < j - i + 1个矩阵的积。因此，算法应该按长度递增的顺序求解矩阵链括号化问题，并按对应的顺序填写表m。对矩阵链AiAi+1...Aj最优括号化的子问题，我们认为其规模为链的长度j - i + 1。

```java
public int bottomUpMatrixChain(int[] p) {
    int n = p.length - 1;
    int[][] m = new int[n + 1][n + 1];
    for (int i = 1; i <= n; i++) {
        m[i][i] = 0;
    }
    for (int len = 2; len <= n; len++) {
        for (int i = 1; i <= n - len + 1; i++) {
            int j = i + len - 1;
            int min = Integer.MAX_VALUE;
            for (int k = i; k < j; k++) {
                min = Math.min(min, m[i][k] + m[k + 1][j] + p[i - 1] * p[k] * p[j]);
            }
            m[i][j] = min;
        }
    }
    return m[1][n];
}

public Object[] extendedBottomUpMatrixChain(int[] p) {
    int n = p.length - 1;
    int[][] m = new int[n + 1][n + 1];
    int[][] s = new int[n + 1][n + 1];
    for (int i = 1; i <= n; i++) {
        m[i][i] = 0;
    }
    for (int len = 2; len <= n; len++) {
        for (int i = 1; i <= n - len + 1; i++) {
            int j = i + len - 1;
            int min = Integer.MAX_VALUE;
            for (int k = i; k < j; k++) {
                if (min > m[i][k] + m[k + 1][j] + p[i - 1] * p[k] * p[j]) {
                    min = m[i][k] + m[k + 1][j] + p[i - 1] * p[k] * p[j];
                    s[i][j] = k;
                }
            }
            m[i][j] = min;
        }
    }
    return new Object[]{m[1][n], s};
}
```

#### 步骤4：构造最优解

虽然extendedBottomUpMatrixChain求出了计算矩阵链乘积所需的最少标量乘法运算次数，但它并未直接指出如何进行这种最优代价的矩阵链乘法计算。表s记录了构造最优解所需的信息，每个表项s[i][j]记录了一个k值，指出AiAi+1...Aj的最优括号化方案的分割点应在Ak和Ak+1之间。因此，我们知道A1..n的最优计算方案中最后一次矩阵乘法运算应该是A1..s[1][n]As[1][n]..n。我们可以用相同的方法递归地求出更早的矩阵乘法的具体计算过程，因为s[1, s[1][n]]指出了计算A1..s[1][n]时应进行的最后一次矩阵乘法运算，s[s[1][n]+1][n]指出了计算As[1][n]+1..n时应进行的最后一次矩阵乘法运算。

```java
public void printMatrixChain(int[][] s, int i, int j) {
    if (i == j) {
        System.out.print("A" + i);
    } else {
        System.out.print("(");
        printMatrixChain(s, i, s[i][j]);
        printMatrixChain(s, s[i][j] + 1, j);
        System.out.print(")");
    }
}
```
