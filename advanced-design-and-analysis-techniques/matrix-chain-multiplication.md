## 矩阵链乘法

给定一个 n 个矩阵的序列（矩阵链） <A<sub>1</sub>, A<sub>2</sub>, ..., A<sub>n</sub>>，我们希望计算它们的乘积 A<sub>1</sub>A<sub>2</sub>...A<sub>n</sub>。为了计算矩阵链乘积，我们可以先用括号明确计算次序，然后利用标准的矩阵相乘算法进行计算。由于矩阵乘法满足结合律，因此任何加括号的方法都会得到相同的计算结果。我们称有如下性质的矩阵乘积链为完全括号化的（fully parenthesized）：它是单一矩阵，或者是两个完全括号化的矩阵链的积，且已外加括号。例如，如果矩阵链为 <A<sub>1</sub>, A<sub>2</sub>, A<sub>3</sub>, A<sub>4</sub>>，则共有 5 种完全括号化的矩阵乘积链：

<center>(A<sub>1</sub>(A<sub>2</sub>(A<sub>3</sub>A<sub>4</sub>)))</center>  
<center>(A<sub>1</sub>((A<sub>2</sub>A<sub>3</sub>)A<sub>4</sub>))</center>  
<center>((A<sub>1</sub>A<sub>2</sub>)(A<sub>3</sub>A<sub>4</sub>)</center>  
<center>((A<sub>1</sub>(A<sub>2</sub>A<sub>3</sub>))A<sub>4</sub>)</center>  
<center>(((A<sub>1</sub>A<sub>2</sub>)A<sub>3</sub>)A<sub>4</sub>)</center>  

对矩阵链加括号的方式会对乘积运算的代价产生巨大影响，我们先来分析两个矩阵相乘的代价。下面是两个矩阵相乘的标准乘法。

```java
int[][] matrixMultiply(int[][] a, int[][] b) {
    int aRows = a.length;
    int aColumns = a[0].length;
    int bRows = b.length;
    int bColumns = b[0].length;
    if (aColumns != bRows) {
        throw new RuntimeException("Incompatible dimensions");
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

两个矩阵 A 和 B 只有相容（compatible），即 A 的列数等于 B 的行数时，才能相乘。如果 A 是 p × q 的矩阵，B 是 q × r 的矩阵，那么乘积 C 是 p × r 的矩阵。计算 C 所需时间由 matrixMultiply 最内层 for 循环的标量乘法次数决定，即 pqr。

我们以矩阵链 <A<sub>1</sub>, A<sub>2</sub>, A<sub>3</sub>> 相乘为例，来说明不同的加括号方案会导致不同的计算代价。假设三个矩阵的规模分别为 10 × 100, 100 × 5 和 5 × 50。如果按 ((A<sub>1</sub>A<sub>2</sub>)A<sub>3</sub>) 的顺序计算，为计算 A<sub>1</sub>A<sub>2</sub>，需要做 10 * 100 * 5 = 5000 次标量乘法，再与 A<sub>3</sub> 相乘又需要做 10 * 5 * 50 = 2500 次乘法，共需 7500 次标量乘法。如果按 (A<sub>1</sub>(A<sub>2</sub>A<sub>3</sub>)) 的顺序，计算 A<sub>2</sub>A<sub>3</sub> 需 100 * 5 * 50 = 25000 次乘法，A<sub>1</sub> 再与之相乘又需 10 * 100 * 50 = 50000 次标量乘法，共需 75000 次标量乘法。因此，按第一种顺序计算矩阵链乘积要比第二种顺序快 10 倍。

矩阵链乘法问题（matrix-chain multiplication problem）可描述如下：给定 n 个矩阵的链 <A<sub>1</sub>, A<sub>2</sub>, ..., A<sub>n</sub>>，矩阵 A<sub>i</sub> 的规模为 p<sub>i-1</sub> × p<sub>i</sub> (1 <= i <= n)，求完全括号化方案，使得计算乘积 A<sub>1</sub>A<sub>2</sub>...A<sub>n</sub> 所需标量乘法次数最少。

注意，求解矩阵链乘法问题并不是要真正进行矩阵相乘运算，我们的目标只是确定代价最低的计算顺序。确定最优计算顺序所花费的时间通常要比随后真正进行矩阵相乘所节省的时间（例如仅进行 7500 次标量乘法而不是 75000 次）要少。

### 计算括号化方案的数量

在用动态规划方法求解矩阵链乘法问题之前，我们先说服自己——穷举所有可能的括号化方案不会产生一个高效的算法。对于一个 n 个矩阵的链，令 P(n) 表示可供选择的括号化方案的数量。当 n = 1 时，由于只有一个矩阵，因此只有一种完全括号化方案。当 n >= 2 时，完全括号化方案的矩阵乘积可描述为两个完全括号化的部分积相乘的形式，而两个部分积的划分点在第 k 个矩阵和第 k + 1 个矩阵之间，k 为 1, 2, ..., n-1 中的任意一个值。因此，我们可以得到如下递归公式：

![](../assets/images/part4/matrix-chain-multiplication.png)

括号化方案的数量与 n 呈指数关系，通过暴力搜索穷尽所有可能的括号化方案来寻找最优方案，是一个糟糕的策略。

### 应用动态规划方法

下面用动态规划方法来求解矩阵链的最优括号化方案，还是按照动态规划的 4 个步骤进行：

1. 刻画一个最优解的结构特征。

2. 递归地定义最优解的值。

3. 计算最优解的值，通常采用自底向上的方法。

4. 利用计算出的信息构造一个最优解。

我们按顺序进行这几个步骤，清楚地展示针对本问题每个步骤应该怎么做。

#### 步骤 1：最优括号化方案的结构特征

动态规划方法的第一步是寻找最优子结构，然后就可以利用这种子结构从子问题的最优解构造出原问题的最优解。在矩阵链乘法问题中，此步骤的做法如下所述。为方便起见，我们用符号 A<sub>i...j</sub> (i <= j) 表示 A<sub>i</sub>A<sub>i+1</sub>...A<sub>j</sub> 乘积的结果矩阵。可以看出，如果问题是非平凡的，即 i < j，那么为了对 A<sub>i</sub>A<sub>i+1</sub>...A<sub>j</sub> 进行括号化，我们就必须在某个 A<sub>k</sub> 和 A<sub>k+1</sub> 之间将矩阵链划分开（k 为 [i, j) 之间的整数）。也就是说，对某个整数 k，我们首先计算矩阵 A<sub>i...k</sub> 和 A<sub>k+1...j</sub>，然后再计算它们的乘积得到最终结果 A<sub>i...j</sub>。此方案的计算代价等于矩阵 A<sub>i...k</sub> 的计算代价，加上矩阵 A<sub>k+1...j</sub> 的计算代价，再加上两者相乘的计算代价。

下面我们给出本问题的最优子结构。假设 A<sub>i</sub>A<sub>i+1</sub>...A<sub>j</sub> 的最优括号化方案的分割点在 A<sub>k</sub> 和 A<sub>k+1</sub> 之间，那么，继续对“前缀”子链 A<sub>i</sub>A<sub>i+1</sub>...A<sub>k</sub> 进行括号化时，我们应该直接采用独立求解它时所得的最优方案。这样做的原因是什么呢？如果不采用独立求解 A<sub>i</sub>A<sub>i+1</sub>...A<sub>k</sub> 所得的最优方案来对它进行括号化，那么可以将此最优解代入 A<sub>i</sub>A<sub>i+1</sub>...A<sub>j</sub> 的最优解中，代替原来对子链 A<sub>i</sub>A<sub>i+1</sub>...A<sub>k</sub> 进行括号化的方案（因为原来的方案比 A<sub>i</sub>A<sub>i+1</sub>...A<sub>k</sub> 最优解的代价更高），显然，这样得到的解比 A<sub>i</sub>A<sub>i+1</sub>...A<sub>j</sub> 原来的最优解代价更低，产生矛盾。对子链 A<sub>k+1</sub>A<sub>k+2</sub>...A<sub>j</sub>，我们有相似的结论：在原问题 A<sub>i</sub>A<sub>i+1</sub>..A<sub>j</sub> 的最优括号化方案中，对子链 A<sub>k+1</sub>A<sub>k+2</sub>...A<sub>j</sub> 进行括号化的方法，就是它自身的最优括号化方案。

现在我们展示如何利用最优子结构性质从子问题的最优解构造原问题的最优解。我们已经看到，一个非平凡的矩阵链乘法问题实例的任何解都需要划分链，而任何最优解都是由子问题实例的最优解构成的。因此，为了构造一个矩阵链乘法问题实例的最优解，我们可以将问题划分为两个子问题（A<sub>i</sub>A<sub>i+1</sub>...A<sub>k</sub> 和 A<sub>k+1</sub>A<sub>k+2</sub>...A<sub>j</sub> 的最优括号化问题），求出子问题实例的最优解，然后将子问题的最优解组合起来。我们必须保证在确定分割点时，已经考察了所有可能的划分点，这样就可以保证不会遗漏最优解。

#### 步骤 2：一个递归求解方案

下面用子问题的最优解来递归地定义原问题最优解的代价。对矩阵链乘法问题，我们可以将对所有 1 <= i <= j <= n 确定 A<sub>i</sub>A<sub>i+1</sub>...A<sub>j</sub> 的最小代价括号化方案作为子问题，令 m[i][j] 表示计算矩阵 A<sub>i,j</sub> 所需标量乘法次数的最小值，那么，原问题的最优解——计算 A<sub>1...n</sub> 所需的最低代价就是 m[1][n]。

我们可以递归定义 m[i][j] 如下。对于 i = j 时的平凡问题，矩阵链只包含唯一的矩阵 A<sub>i...i</sub> = A<sub>i</sub>，因此不需要做任何标量乘法运算。所以对所有 i = 1, 2, ..., n, m[i][j] = 0。若 i < j，我们利用步骤 1 中得到的最优子结构来计算 m[i][j]。我们假设 A<sub>i</sub>A<sub>i+1</sub>...A<sub>j</sub> 的最优括号化方案的分割点在矩阵 A<sub>k</sub> 和 A<sub>k+1</sub> 之间，其中 i <= k < j。那么，m[i][j] 就等于计算 A<sub>i...k</sub> 和 A<sub>k+1...j</sub> 的代价加上两者相乘的代价的最小值。由于矩阵 A<sub>i</sub> 的大小为 p[i-1] × p[i]，易知 A<sub>i...k</sub> 与 A<sub>k+1...j</sub> 相乘的代价为 p[i-1] * p[k] * p[j] 次标量乘法运算。因此，我们得到：

```
m[i][j] = m[i][k] + m[k+1][j] + p[i-1] * p[k] * [pj]
```

此递归公式假定最优分割点 k 是已知的，但实际上我们是不知道的。不过，k 只有 j - i 种可能的取值，即 k = i, i+1, ..., j-1。由于最优分割点必在其中，我们只需检查所有可能情况，找到最优解即可。因此，A<sub>i</sub><sub>i+1</sub>...A<sub>j</sub> 最小代价括号化方案的递归求解公式变为：

![](../assets/images/part4/matrix-chain-multiplication2.png)

m[i][j] 的值给出了子问题最优解的代价，但它并未提供足够的信息来构造最优解，为此，我们用 s[i][j] 保存 A<sub>i</sub>A<sub>i+1</sub>..A<sub>j</sub> 最优括号化方案的分割点位置 k，即使得 m[i][j] = m[i][k] + m[k+1][j] + p[i-1] * p[k] * p[j] 成立的 k 的值。

#### 步骤 3：计算最优代价

现在，我们可以很容易地基于以上的递归公式写出一个递归算法，来计算 A<sub>1</sub>A<sub>2</sub>...A<sub>n</sub> 相乘的最小代价 m[1][n]，此递归算法是指数时间的，并不比检查所有括号化方案的暴力搜索方法更好。

```java
int recursive(int[] p, int i, int j) {
    if (i == j) {
        return 0;
    }
    int min = Integer.MAX_VALUE;
    for (int k = i; k < j; k++) {
        min = Math.min(min, recursive(p, i, k) + recursive(p, k + 1, j) + p[i - 1] * p[k] * p[j]);
    }
    return min;
}
```

注意到，我们需要求解的不同子问题的数目是相对较少的，每对满足 1 <= i <= j <= n 的 i 和 j 对应一个唯一的子问题，共有 Θ(n<sup>2</sup>) 个。递归算法会在递归调用树的不同分支中多次遇到同一个子问题，这种子问题重叠的性质是动态规划的另一个标识（第一个标识是最优子结构）。

我们采用自底向上的表格法替代递归式来计算最优代价。为了实现自底向上方法，我们必须确定计算 m[i][j] 时需要访问哪些其它表项。上述公式显示，j - i + 1 个矩阵链相乘的最优计算代价 m[i][j] 只依赖于那些少于 j - i + 1 个矩阵链相乘的最优计算代价。也就是说，对 k = i, i+1, ..., j-1，矩阵 A<sub>i...k</sub> 是 k - i + 1 < j - i + 1 个矩阵的积，矩阵 A<sub>k+1...j</sub> 是 j - k < j - i + 1 个矩阵的积。因此，算法应该按长度递增的顺序求解矩阵链括号化问题，并按对应的顺序填写表 m。对矩阵链 A<sub>i</sub>A<sub>i+1</sub>...A<sub>j</sub> 最优括号化的子问题，我们认为其规模为链的长度 j - i + 1。

下面的 bottomUp 过程实现了自底向上表格法，此过程假定矩阵 A<sub>i</sub> 的规模为 p<sub>i-1</sub> × p<sub>i</sub> (i = 1, 2, ..., n)。它的输入是一个序列 p = <p<sub>0</sub>, p<sub>1</sub>, ..., p<sub>n</sub>>，其长度为 p.length = n + 1。过程用一个辅助表 m[1...n][1...n] 来保存代价 m[i][j]，用另一个辅助表 s[1...n-1][2...n] 记录最优值 m[i][j] 对应的分割点 k，我们就可以用表 s 构造最优解。

```java
int bottomUp(int[] p) {
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
```

算法首先对所有 i = 1, 2, ..., n 计算 m[i][i] = 0（长度为 1 的链的最小计算代价）。接着在第一个 for 循环的第一个循环步中，利用递归公式对所有 i = 1, 2, ..., n-1 计算 m[i][i+1]（长度 len = 2 的链的最小计算代价），在第二个循环步中，算法对所有 i = 1, 2, ..., n-2 计算 m[i][i+2]（长度 len = 3 的链的最小计算代价），依此类推。在每个循环步中，计算代价 m[i][j] 时仅依赖于已经计算出的表项 m[i][k] 和 m[k+1, j]。

简单分析 bottomUp 的循环嵌套结构，可以看到算法的运行时间为 O(n<sup>3</sup>)。循环嵌套的深度为三层，每层的循环变量（len，i 和 k）最多取 n - 1 个值。算法还需要 Θ(n<sup>2</sup>) 的内存空间来保存表 m 和 s，因此，bottomUp 比穷举所有可能的括号化方案来寻找最优解的指数阶算法要高效得多。

#### 步骤 4：构造最优解

虽然 bottomUp 求出了计算矩阵链乘积所需的最少标量乘法运算次数，但它并未直接指出如何进行这种最优代价的矩阵链乘法计算。表 s[1...n-1][2...n] 记录了构造最优解所需的信息，每个表项 s[i][j] 记录了一个 k 值，指出 A<sub>i</sub><sub>i+1</sub>...A<sub>j</sub> 的最优括号化方案的分割点应在 A<sub>k</sub> 和 A<sub>k+1</sub> 之间。因此，我们知道 A<sub>1...n</sub> 的最优计算方案中最后一次矩阵乘法运算应该是 A<sub>1...s[1][n]</sub>A<sub>s[1][n]+1...n</sub>。我们可以用相同的方法递归地求出更早的矩阵乘法的具体计算过程，因为 s[1][s[1][n]] 指出了计算 A<sub>1...s[1][n]</sub> 时应进行的最后一次矩阵乘法运算，s[s[1][n]+1][n] 指出了计算 A<sub>s[1][n]+1...n</sub> 时应进行的最后一次矩阵乘法运算。

```java
Object[] extendedBottomUp(int[] p) {
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

void constructSolution(int[][] s, int i, int j) {
    if (i == j) {
        System.out.print("A" + i);
    } else {
        System.out.print("(");
        constructSolution(s, i, s[i][j]);
        constructSolution(s, s[i][j] + 1, j);
        System.out.print(")");
    }
}
```
