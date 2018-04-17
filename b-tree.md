## B树

B树是为磁盘或其他直接存取的辅助存储设备而设计的一种平衡搜索树。B树类似于红黑树，但它们在降低磁盘I/O方面要更好一些。许多数据库系统使用B树或B树的变种来存储信息。

B树与红黑树的不同之处在于B树的结点可以有很多孩子，从数个到数千个。也就是说，一个B树的“分支因子”可以相当大，尽管它们通常依赖于所使用的磁盘单元的特性。B树类似于红黑树，就是每棵含有n个结点的B树的高度为O(lgn)。然而，一棵B树的严格高度可能比一棵红黑树的高度要小许多，这是因为它的分支因子，也就是表示高度的对数的底数可以非常大。因此，我们也可以使用B树在时间O(lgn)内完成一些动态集合的操作。

B树以一种自然的方式推广了二叉搜索树。如果B树的一个内部结点x包含x.n个关键字，那么结点x就有x.n + 1个孩子。结点x中的关键字就是分隔点，它把结点x中所处理的关键字的属性分隔为x.n + 1个子域，每个子域都由x的一个孩子处理。当在一棵B树中查找一个关键字时，基于对存储在x中的x.n个关键字的比较，做出一个x.n + 1路的选择。叶结点的结构与内部结点不同，后面将讨论这些差别。

下图给出了一棵关键字为英语中辅音字母的B树。一个内部结点x包含x.n个关键字以及x.n + 1个孩子，所有叶结点处于树中相同的深度，浅阴影的结点是在查找字母R时检查过的结点。

![](./assets/images/part3/b-tree.png)

### B树的定义

为了简单起见，我们假定，就像二叉搜索树和红黑树一样，任何和关键字相联系的“卫星数据”(satellite information)将与关键字一样存放在同一个结点中，当一个关键字从一个结点移动到另一个结点时，无论是与关键字相联系的卫星数据，还是指向卫星数据的指针，都会随着关键字一起移动。一个常见的B树变种，称为B+树，它把所有的卫星数据都存储在叶结点中，内部结点只存放关键字和孩子指针，因此最大化了内部结点的分支因子。

一棵B树是具有以下性质的有根树：

1. 每个结点x有下面属性：
* n,当前存储在结点x中的关键字个数。
* n个关键字本身keys[0], keys[1], ..., keys[n - 1]，以非降序存放，使得keys[0] <= keys[1] <= ... <= keys[n - 1]。
* isLeaf，一个布尔值，如果x是叶结点，则为true，如果x为内部结点，则为false。

2. 每个内部结点x还包含n + 1个指向其孩子的指针children[0], children[1], ..., children[n]，叶结点没有孩子。

3. 关键字keys[i]对存储在各子树中的关键字范围加以分割：如果k[i]为任意一个存储在以children[i]为根的子树中的关键字，那么 k[0] <= keys[0] <= k[1] <= ... <= keys[n - 1] <= k[n]

4. 每个叶结点具有相同的深度，即树的高度h。

5. 每个结点所包含的关键字个数有上界和下界。用一个被称为B树的最小度数(minimum degree)的固定整数t >= 2来表示这些界：
* 除了根结点以外的每个结点必须至少有t - 1个关键字。因此，除了根结点以外的每个内部结点至少有t个孩子。如果树非空，根结点至少有一个关键字。
* 每个结点至多可包含2 * t - 1个关键字。因此，一个内部结点至多可有2 * t个孩子。当一个结点恰好有2t - 1个关键字时，称该结点是满的(full)。t = 1时的B树是最简单的，每个内部结点有2个、3个或4个孩子，即一棵2-3-4树。然而，在实际中，t的值越大，B树的高度就越小。

```java
public class BTree {
    private Node root;
    private int t;
    private static final int DEFAULT_DEGREE = 2;

    private class Node {
        int n;
        boolean isLeaf;
        String[] keys;
        Node[] children;
        
        Node() {
            n = 0;
            isLeaf = true;
            keys = new String[2 * t - 1];
            children = new Node[2 * t];
        }
    }
    
    public BTree() {
        t = DEFAULT_DEGREE;
        root = new Node();
    }
    
    public BTree(int degree) {
        if (degree < 2) {
            t = DEFAULT_DEGREE;
        } else {
            t = degree;
        }
        root = new Node();
    }
}
```

#### B树的高度

如果n >= 1，那么对任意一棵包含n个关键字、高度为h、最小度数t >= 2的B树T，有

![](./assets/images/part3/b-tree2.png)

### B树上的基本操作

#### 搜索B树

搜索一棵B树和搜索一棵二叉搜索树很相似，只是在每个结点所做的不是二叉或者“两路”分支选择，而是根据结点的孩子数做多路分支选择。更严格地说，对每个内部结点x，做的是一个x.n + 1路的分支选择。

```java
public Object[] search(Node root, String key) {
    int i = 0;
    while (i < root.n && key.compareTo(root.keys[i]) > 0) {
        i++;
    }
    if (i < root.n && key.equals(root.keys[i])) {
        return new Object[]{root, i};
    } else {
        if (root.isLeaf) {
            return new Object[]{null, null};
        } else {
            return search(root.children[i], key);
        }
    }
}
```

就像二叉搜索树的搜索过程一样，在递归过程中所遇到的结点构成了一条从树根向下的简单路径。因此，由search过程访问的磁盘页面数为O(h) = O(Math.log(n) / Math.log(t))，其中h为B树的高度，n为B树中所含的关键字个数。由于x.n < 2t，所以while循环在每个结点中花费的时间为O(t)，总的CPU时间为O(t * Math.log(n) / Math.log(t))。

#### 向B树中插入一个关键字

B树中插入一个关键字要比二叉搜索树中插入一个关键字复杂得多。像二叉搜索树一样，要查找插入新关键字的叶结点的位置。然而，在B树中，不能简单地创建一个新的叶结点，然后将其插入，因为这样得到的树将不再是合法的B树。相反，我们是将新的关键字插入一个已经存在的叶结点上。由于不能将关键字插入一个满的叶结点，故引入一个操作，将一个满的结点y(有2t - 1个关键字)按其中间关键字(median key)y.keys[i]分裂(split)为两个各含t - 1个关键字的结点。中间关键字被提升到y的父结点，以标识两棵新树的划分点。但是如果y的父结点也是满的，就必须在插入新的关键字之前将其分裂，最终满结点的分裂会沿着树向上传播。

与一棵二叉搜索树一样，可以在从树根到叶子这个单程向下过程中将一个新的关键字插入B树中。为了做到这一点，我们并不是等到找出插入过程中实际要分裂的满结点才做分裂。相反，当沿着树往下查找新的关键字所属位置时，就分裂沿途遇到的每个满结点(包括叶结点本身)。因此，每当要分列一个满结点y时，就能确保它的父结点不是满的。

#### 分裂B树中的结点

splitChild的输入是一个非满的内部结点x和一个使x.children[i]为x的满子结点的下标i。该过程把这个子结点分裂成两个，并调整x，使之包含多出来的孩子。要分列一个满的根，首先要让根成为一个新的空根结点的孩子，这样才能使用splitChild，树的高度因此增加1。分裂是树长高的唯一途径。

```java
private void splitChild(Node x, int i) {
    Node y = x.children[i];
    Node z = new Node();
    z.isLeaf = y.isLeaf;
    z.n = t - 1;
    y.n = t - 1;
    for (int j = 0; j < t - 1; j++) {
        z.keys[j] = y.keys[t + j];
        y.keys[t + j] = null;
    }
    if (!y.isLeaf) {
        for (int j = 0; j < t; j++) {
            z.children[j] = y.children[t + j];
            y.children[t + j] = null;
        }
    }
    for (int j = x.n; j >= i + 1; j--) {
        x.children[j + 1] = x.children[j];
    }
    x.children[i + 1] = z;
    for (int j = x.n - 1; j >= i; j--) {
        x.keys[j + 1] = x.keys[j];
    }
    x.keys[i] = y.keys[t - 1];
    y.keys[t - 1] = null;
    x.n = x.n + 1;
}
```

下图显示了这个过程。满结点y = x.children[i]按照其中间关键字S进行分裂，S被提升到y的父结点x。y中的那些大于中间关键字的关键字够置于一个新的结点z中，它成为x的一个新的孩子。

![](./assets/images/part3/b-tree3.png)

splitChild以直接的“剪贴”方式工作。这里x是被分裂的结点，y是x的第i个孩子。开始时，结点y有2t个孩子(2t - 1个关键字)，在分裂后减少至t个孩子(t - 1个关键字)。结点z取走y的t个最大的孩子(t - 1个关键字)，并且z成为x的新孩子，它在x的孩子列表中仅位于y之后，y的中间关键字上升到x中，成为分隔y和z的关键字。

#### 沿树单程下行方式向B树插入关键字

insert利用splitChild来保证递归始终不会降至一个满结点上。

```java
public void insert(String key) {
    Node r = root;
    if (r.n == 2 * t - 1) {
        Node newRoot = new Node();
        root = newRoot;
        newRoot.isLeaf = false;
        newRoot.children[0] = r;
        splitChild(newRoot, 0);
        insertNonFull(newRoot, key);
    } else {
        insertNonFull(r, key);
    }
}
```

当根结点为满时，原来的根结点被分裂，一个新的结点s(有两个孩子)称为根。对根进行分裂是增加B树高度的唯一途径，下图说明了这种情况。与二叉搜索树不同，B树高度的增加发生在顶部而不是底部。insert通过调用insertNonFull完成将关键字key插入以非满的根结点为根的树中。insertNonFull在需要时沿树向下递归，在必要时通过调用splitChild来保证任何时刻它所递归处理的结点都是非满的。

![](./assets/images/part3/b-tree4.png)

辅助的递归过程insertNonFull将关键字插入结点x，要求假定在调用该过程时x是非满的。操作insert和递归操作insertNonFull保证了这个假设成立。

```java
private void insertNonFull(Node node, String key) {
    int i = node.n - 1;
    if (node.isLeaf) {
        while (i >= 0 && key.compareTo(node.keys[i]) < 0) {
            node.keys[i + 1] = node.keys[i];
            i--;
        }
        i++;
        node.keys[i] = key;
        node.n = node.n + 1;
    } else {
        while (i >= 0 && key.compareTo(node.keys[i]) < 0) {
            i--;
        }
        i++;
        if (node.children[i].n == 2 * t - 1) {
            splitChild(node, i);
            if (key.compareTo(node.keys[i]) > 0) {
                i++;
            }
        }
        insertNonFull(node.children[i], key);
    }
}
```

下图演示了向B树中插入关键字：

![](./assets/images/part3/b-tree5.png)

这棵B树的最小度数t为3，所以一个结点至多可包含5个关键字。在插入过程中被修改的结点由浅阴影标记。(a)这个例子初始时的树。(b)向初始树中插入B后的结果；这是一个对叶结点的简单插入。(c)将Q插入前一棵树中的结果。结点RSTUV被分裂为两个分别包含RS和UV的结点，关键字T被提升到根中，Q被插入两半的最左边(RS结点)。(d)将L插入前一棵树中的结果。由于根结点是满的，所以它立即被分裂，同时B树的高度增加1，然后L被插入包含JK的叶结点中。(e)将F插入前一棵树的结果。在将F插入两半的最右边(DE结点)之前，结点ABCDE会进行分裂。

### 从B树中删除关键字

B树上的删除操作与插入操作类似，只是略微复杂一点，因为可以从任意一个结点中删除一个关键字，而不仅仅是叶结点，而且当从一个内部结点删除一个关键字时，还要重新安排这个结点的孩子。与插入操作一样，必须防止因删除操作而导致树的结构违反B树性质。就像必须保证一个结点不会因为插入而变得太大一样，必须保证一个结点不会在删除期间变得太小(根结点除外，因为它允许有比最少关键字数t - 1还少的关键字个数)。一个简单插入算法，如果插入关键字的路径上结点满，可能需要向上回溯，与此类似，一个简单删除算法，当要删除关键字的路径上结点(非根)有最少的关键字个数时，也可能需要向上回溯。

delete从以x为根的子树中删除关键字k。我们设计的这个过程必须保证无论何时，结点x递归调用自身时，x中关键字个数至少为最小度数t。注意到，这个条件要求比通常B树中的最少关键字多一个以上，使得有时在递归下降至结点之前，需要把一个关键字移到子结点中。这个加强的条件允许在一趟下降过程中，就可以将一个关键字从树中删除，无需任何向上回溯(有一个例外，后面会解释)。对下面的B树上删除操作的规定应当这样理解，如果根结点x成为一个不包含任何关键字的内部结点，那么x就要被删除，x的唯一孩子x.children[0]成为树的新根，从而树的高度降低1，同时也维持树根必须包含至少一个关键字的性质(除非树是空的)。

现在我们来简要介绍删除操作是如何工作的，下图描绘了从B树中删除关键字的各种情况。

![](./assets/images/part3/b-tree6.png)

1. 如果关键字k在结点x中，并且x是叶结点，则从x中删除k。

2. 如果关键字k在结点x中，并且x是内部结点，则做以下操作：
* a. 如果结点x中前于k的子结点y至少包含t个关键字，则找出k在以y为根的子树的前驱k'。递归地删除k'，并在x中用k'代替k(找到k'并删除它可在沿树下降的单过程中完成)。
* b. 对称地，如果y有少于t个关键字，则检查结点x中后于k的子结点z。如果z至少有t个关键字，则找出k在以z为根的子树中的后继k'。递归地删除k'，并在x中用k'代替k(找到k'并删除它可在沿树下降的单过程中完成)。
* c. 否则，如果y和z都只含有t - 1个关键字，则将k和z的全部合并进y，这样x就是去了k和指向x的指针，并且y现在包含2t - 1个关键字。然后释放z并递归地从y中删除k。

3. 如果关键字k当前不在内部结点x中，则确定必包含k的子树的根x.children[i] (如果k确实在树中)。如果x.children[i]只有t - 1个关键字，必须执行步骤3a或3b来保证降至一个至少包含t个关键字的结点。然后，通过对x的某个合适的子结点进行递归而结束。
* a. 如果x.children[i]只含有t - 1个关键字，但是它的一个相邻的兄弟至少包含t个关键字，则将x中的某一个关键字降至x.children[i]中，将x.children[i]的相邻左兄弟或右兄弟的一个关键字升至x，将该兄弟中相应的孩子指针移到x.children[i]中，这样就使得x.children[i]增加了一个额外的关键字。
* b. 如果x.children[i]以及x.children[i]的所有相邻兄弟都只包含t - 1个关键字，则将x.children[i]与一个兄弟合并，即将x的一个关键字移至新合并的结点，使之成为该结点的中间关键字。

由于一棵B树的大部分关键字都在叶结点中，我们可以预期在实际中，删除操作最经常用于从叶结点删除关键字。这样delete过程只要沿树下降一趟即可，不需要向上回溯。然而，当要删除某个内部结点的关键字时，该过程也要沿树下降一趟，但可能还要返回删除了关键字的那个结点，以用其前驱或后继来取代被删除的关键字(情况2a和情况2b)。
