## 用于不相交集合的数据结构

一些应用涉及将n个不同的元素分成一组不相交的集合，这些应用经常要进行两种特别的操作：寻找包含给定元素的唯一集合和合并两个集合。

用一个对象表示一个集合的每个元素，设x表示一个对象，我们希望支持以下三个操作：

* makeSet(x)建立一个新的集合，它的唯一成员(因而是代表)是x。因为各个集合是不相交的，故x不会出现在别的某个集合中。

* union(x, y)将包含x和y的两个动态集合(表示为S[x]和S[y])合并成一个新的集合，即这两个集合的并集。假定在操作之前这两个集合是不相交的。虽然union的很多实现中特别地选择S[x]或S[y]的代表作为新的代表，然而结果集的代表可以是S[x] ∪ S[y]中的任何成员。由于我们要求各个集合不相交，故要消除原有的集合S[x]和S[y]，即把它们从S中删除。实际上，我们经常把其中一个集合的元素并入另一个集合中，来替代删除操作。

* findSet(x)返回一个指针，这个指针指向包含x的(唯一)集合的代表。

### 不相交集合的操作

一个**不相交结合数据结构(disjoint-set data structure)** 维护了一个不相交动态集的集合S = {S[1], S[2], ..., S[k]}。我们用一个代表(representative)来标识每个集合，它是这个集合的某个成员。在一些应用中，我们不关心哪个成员被用来作为代表，仅仅关心的是2次查询动态集合的代表中，如果这些查询没有修改动态集合，则这两次查询应该得到相同的答案。其它一些应用可能会需要一个预先说明的规则来选择代表，比如选择这个集合中最小的成员(当然假设集合中的元素能被比较次序)。

### 不相交集合的链表表示

下图给出了一个实现不相交集数据结构的简单方法：每个集合用一个自己的链表来表示。每个集合的对象包含head属性和tail属性，head属性指向表的第一个对象，tail属性指向表的最后一个对象。链表的每个对象都包含一个集合成员、一个指向链表中下一个对象的指针和一个指回到集合对象的指针。在每个链表中，对象可以以任意的次序出现。代表是链表中第一个对象的集合成员。

用这种链表表示，makeSet操作和findSet操作是非常方便的，只需O(1)的时间。要执行makeSet(x)操作，我们需要创建一个只有x对象的新的链表。对于findSet(x)，仅沿着x对象的返回指针返回到集合对象，然后返回head指向对象的成员。例如，在下图(a)中，findSet(g)的调用将返回f。

![](../assets/images/part3/disjoint-set.png)

在使用链表集合表示的实现中，union操作的最简单实现明显比makeSet或findSet花费的时间多。如图(b)所示，我们通过把y所在的链表拼接到x所在的链表实现了union(x, y)。x所在的链表的代表成为结果集的代表。利用x所在链表的tail指针，可以迅速地找到拼接y所在的链表的位置。因为y所在的链表的所有成员加入了x所在的链表，此时可以删除y所在的链表的集合对象。遗憾的是，对于y所在链表的每个对象，我们必须更新指向集合对象的指针，这将花费的时间与y所在链表长度呈线性关系。

```java
public class DisjointSet {
    private Node[] nodes;
    private int count;

    private class Node {
        int id;
        Node next;
        Set set;

        Node(int id) {
            this.id = id;
            set = new Set(this);
        }
    }

    private class Set {
        Node head;
        Node tail;

        Set(Node node) {
            head = node;
            tail = node;
        }
    }

    public DisjointSet(int n) {
        nodes = new Node[n];
        count = n;
        for (int i = 0; i < n; i++) {
            nodes[i] = new Node(i);
        }
    }

    public void union(int x, int y) {
        link(nodes[x].set, nodes[y].set);
    }

    private void link(Set x, Set y) {
        x.tail.next = y.head;
        Node z = y.head;
        while (z.next != null) {
            z.set = x;
        }
        x.tail = y.tail;
        count--;
    }

    public int getCount() {
        return count;
    }

    public boolean isConnected(int x, int y) {
        return nodes[x].set == nodes[y].set;
    }
}

```

#### 一种加权合并的启发式策略

在最坏情况下，上面给出的union过程的每次调用平均需要Θ(n)的时间，这是因为需要把一个较长的表拼接到一个较短的表上，此时必须对较长表的每个成员更新其指向集合对象的指针。现在换一种做法，假设每个表中还包含了表的长度(这是很容易维护的)以及拼接次序可以任意的话，我们总是把较短的表拼接到较长的表中。使用这种简单的加权合并启发式策略(weighted-union heuristic)，如果两个集合都有Ω(n)个成员，则单个的union操作仍然需要Ω(n)的时间。然而一个具有m个makeSet、union和findSet操作的序列(其中有n个是makeSet操作)需要耗费O(m + n * lgn)的时间。

整个m个操作的序列所需的时间很容易求出。每个makeSet和findSet操作需要O(1)时间，它们的总数为O(m)，所以整个序列的总时间是O(m + n * lgn)。

### 不相交集合森林

在一个不相交集合更快的实现中，我们使用有根树来标识集合，树中每个结点包含一个成员，每棵树代表一个集合。在一个不相交集合森林(disjoint-set forest)中，每个成员仅指向它的父结点。每棵树的根包含集合的代表，并且是其自己的父结点。正如我们将要看到的那样，虽然使用这种表示的直接算法并不比使用链表表示的算法快，但通过引入两种启发式策略(“按秩合并”和“路径压缩”)，我们能得到一个渐进最优的不相交集合数据结构。

![](../assets/images/part3/disjoint-set2.png)

我们执行以下三种不相交集合操作：makeSet操作简单地创建一棵只有一个结点的树，findSet操作通过沿着指向父结点的指针找到树的根。这一通向根结点的简单路径上所访问的结点构成了查找路径(find path)。union操作如图(b)所示，使得一棵树的根指向另外一棵树的根。

#### 改进运行时间的启发式策略

到目前为止，我们还没有对使用链表的实现作出改进。一个包含n - 1个union操作的序列可以构造出一棵恰好含有n个结点的线性链的树。然而，通过使用两种启发式策略，我们能获得一个几乎与总的操作数m呈线性关系的运行时间。

第一种启发式策略是按秩合并(union by rank)，它类似于链表表示中使用的加权合并启发式策略。显而易见的做法是，使具有较少结点的树的根指向具有较多结点的树的根。这里并不显式地记录每个结点为根的子树的大小，而是采用一种易于分析的方法。对于每个结点，维护一个秩，它表示该结点高度的一个上界。在使用按秩合并策略的union操作中，我们可以让具有较小秩的根指向具有较大秩的根。

第二种启发式策略是路径压缩(path compression)，也相当简单和高效。如下图所示，在findSet操作中，使用这种策略可以使查找路径中的每个结点直接指向根。路径压缩并不改变任何结点的秩。

![](../assets/images/part3/disjoint-set3.png)

上图表示操作findSet过程中的路径压缩。箭头和根结点的自环被略去了。(a)在执行findSet(a)之前代表一个集合的树。三角形代表一棵子树，其根为图中示出的结点。每个结点有一个指向父结点的指针。(b)在执行findSet(a)之后的同一个集合，现在在查找路径上每个结点都直接指向了根。

为了使用按秩合并的启发式策略实现一个不相交集合森林，我们必须记录下秩的变化情况。对每个结点x，维护一个整数值x.rank，它代表x的高度(从x到某一后代叶结点的最长简单路径上边的数目)的一个上界。当makeSet创建一个单元素集合时，这个树上的单结点有一个为0的初始秩。每一个findSet操作不改变任何秩。union操作有两种情况，取决于两棵树的根是否有相同的秩。如果根没有相同的秩，就让较大秩的根成为较小秩的根的父结点，但秩本身保持不变。另一种情况是两个根有相同的秩时，任意选择两个根中的一个作为父结点，并使它的秩加1。

```java
public class DisjointSetForest {
    private Node[] nodes;
    private int count;

    private class Node {
        int id;
        Node parent;
        int rank;

        Node(int id) {
            this.id = id;
            parent = this;
            rank = 0;
        }
    }

    public DisjointSetForest(int n) {
        nodes = new Node[n];
        count = n;
        for (int i = 0; i < n; i++) {
            nodes[i] = new Node(i);
        }
    }

    public void union(int x, int y) {
        link(findSet(nodes[x]), findSet(nodes[y]));
    }

    private Node findSet(Node node) {
        while (node != node.parent) {
            node.parent = node.parent.parent;
            node = node.parent;
        }
        return node;
    }

    private void link(Node x, Node y) {
        if (x.rank > y.rank) {
            y.parent = x;
        } else {
            x.parent = y;
            if (x.rank == y.rank) {
                y.rank++;
            }
        }
        count--;
    }

    public int getCount() {
        return count;
    }
}
```

findSet过程是一种两趟方法(two-pass method)：当它递归时，第一趟沿着查找路径向上直到找到根，当递归回溯时，第二趟沿着搜索树向下更新每个结点，使其直接指向根。
