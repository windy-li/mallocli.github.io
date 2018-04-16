## 红黑树

对于一棵高度为h的普通二叉搜索树来说，它的search、predecessor、successor、minimum、maximum、insert和delete操作的时间复杂度均为O(h)。因此，如果搜索树的高度较低时，这些集合操作会执行得比较快。然而，如果树的高度较高时，这些集合操作可能并不比在链表上执行得快。红黑树(red black tree)是许多平衡搜索树中的一种，可以保证在最坏情况下基本动态集合操作的时间复杂度为O(lgn)。

### 红黑树的性质

红黑树是一棵二叉搜索树，它在每个结点上增加了一个存储位来表示结点的颜色，可以是RED或BLACK。通过对任何一条从根到叶子的简单路径上各个结点的颜色进行约束，红黑树确保没有一条路径会比其它路径长出2倍，因而是近似于平衡的。

树中的每个结点包含5个属性：color、key、left、right和parent。如果一个结点没有子结点或父结点，则该结点相应指针属性的值为nil。我们可以把这些nil视为指向二叉搜索树的叶结点（外部结点）的指针，而把带关键字的结点视为树的内部结点。

```java
public class RedBlackTree {
    private Node root;
    private Node nil;
    private static final boolean RED = true;
    private static final boolean BLACK = false;
    
    private class Node {
        int key;
        Node parent;
        Node left;
        Node right;
        boolean color;
        
        Node(int key) {
            this.key = key;
        }
    }
    
    public RedBlackTree() {
        nil = new Node(-1);
        nil.color = BLACK;
        root = nil;
    }
}
```

一棵红黑树是满足下面红黑性质的二叉搜索树：

1. 每个结点或是红色的，或是黑色的。

2. 根结点是黑色的。

3. 每个叶结点(nil)是黑色的。

4. 如果一个结点是红色的，则它的两个子结点都是黑色的。

5. 对每个结点，从该结点到其所有后代叶结点的简单路径上，均包含相同数目的黑色结点。

为了便于处理红黑树代码中的边界条件，使用一个哨兵来代表nil。对于一棵红黑树，哨兵nil是一个与树中普通结点有相同属性的对象，它的color属性为BLACK，而其它属性parent、left、right和key可以为任意值。

![](./assets/images/part3/red-black-tree.png)

使用哨兵后，就可以将结点x的nil孩子视为一个普通结点，其父结点为x。尽管可以为树内的每一个nil新增一个不同的哨兵结点，使得每个nil的父结点都有这样的良定义，但这种做法会浪费空间。取而代之的是，使用一个哨兵nil来代表所有的nil：根结点的父结点和所有的叶结点。哨兵的属性parent、left、right和key的取值并不重要。

![](./assets/images/part3/red-black-tree2.png)

从某个结点x出发（不包含该结点）到达一个叶结点的任意一条简单路径上的黑色结点个数称为该结点的黑高(black height)。根据性质5，黑高的概念是明确定义的，因为从该结点出发的所有下降到其叶结点的简单路径的黑结点个数都相同。于是定义红黑树的黑高为其根结点的黑高。一棵有n个内部结点的红黑树的高度至多为2 * lg(n + 1)。

### 旋转

搜索树操作insert和delete在包含n个关键字的红黑树上，运行花费时间为O(lgn)。由于这两个操作对树做了修改，结果可能违反红黑性质。为了维护这些性质，必须要改变树中某些结点的颜色以及指针结构。

指针结构的修改是通过旋转(rotate)来完成的，这是一种能保持二叉搜索树性质的搜索树局部操作。下图给出了两种旋转：左旋和右旋。

![](./assets/images/part3/red-black-tree3.png)

```java
private void leftRotate(Node p) {
    Node r = p.right;
    p.right = r.left;
    p.right.parent = p;
    r.parent = p.parent;
    if (p.parent == nil) {
        root = r;
    } else if (p == p.parent.left) {
        p.parent.left = r;
    } else {
        p.parent.right = r;
    }
    r.left = p;
    r.left.parent = r;
}

private void rightRotate(Node p) {
    Node l = p.left;
    p.left = l.right;
    p.left.parent = p;
    l.parent = p.parent;
    if (p.parent == nil) {
        root = l;
    } else if (p == p.parent.right) {
        p.parent.right = l;
    } else {
        p.parent.left = l;
    }
    l.right = p;
    l.right.parent = l;
}
```

下图给出了一个leftRotate操作修改二叉搜索树的例子，rightRotate操作的代码是对称的。leftRotate和rightRotate都在O(1)时间内运行完成，在旋转操作中，只有指针改变，其它所有属性都保持不变。

![](./assets/images/part3/red-black-tree4.png)

### 插入

我们可以在O(lgn)的时间内完成向一棵含n个结点的红黑树中插入一个新结点。为了做到这一点，利用二叉搜索树略作修改的insert版本来将结点z插入树内，就好像一棵普通的二叉搜索树一样，然后将z着为红色。为了保证红黑性质能继续保持，我们调用一个辅助程序insertFixUp来对结点重新着色并旋转。

```java
public void insert(int key) {
    Node newNode = new Node(key);
    Node parent = nil;
    Node trailingPointer = root;
    while (trailingPointer != nil) {
        parent = trailingPointer;
        if (key < trailingPointer.key) {
            trailingPointer = trailingPointer.left;
        } else {
            trailingPointer = trailingPointer.right;
        }
    }
    newNode.parent = parent;
    if (parent == nil) {
        root = newNode;
    } else if (key < parent.key) {
        parent.left = newNode;
    } else {
        parent.right = newNode;
    }
    newNode.left = nil;
    newNode.right = nil;
    newNode.color = RED;
    insertFixUp(newNode);
}

private void insertFixUp(Node node) {
    while (node.parent.color == RED) {
        if (node.parent == node.parent.parent.left) {
            Node uncle = node.parent.parent.right;
            if (uncle.color == RED) {
                node.parent.color = BLACK;
                uncle.color = BLACK;
                node.parent.parent.color = RED;
                node = node.parent.parent;
            } else {
                if (node == node.parent.right) {
                    node = node.parent;
                    leftRotate(node);
                }
                node.parent.color = BLACK;
                node.parent.parent.color = RED;
                rightRotate(node.parent.parent);
            }
        } else {
            Node uncle = node.parent.parent.left;
            if (uncle.color == RED) {
                node.parent.color = BLACK;
                uncle.color = BLACK;
                node.parent.parent.color = RED;
                node = node.parent.parent;
            } else {
                if (node == node.parent.left) {
                    node = node.parent;
                    rightRotate(node);
                }
                node.parent.color = BLACK;
                node.parent.parent.color = RED;
                leftRotate(node.parent.parent);
            }
        }
    }
    root.color = BLACK;
}
```

为了理解insertFixUp如何工作，把代码分为三个主要的步骤。首先，要确定当结点z被插入并着为红色后，红黑性质中有哪些不能继续保持。其次，应分析while循环的总目标。最后，要分析while循环体中的三种情况，看它们是如何完成目标的。下图给出一个范例，显示在一棵红黑树上insertFixUp如何操作。

![](./assets/images/part3/red-black-tree5.png)

在调用insertFixUp操作时，哪些红黑性质可能会被破坏呢？性质1和性质3继续成立，因为新插入的红结点两个子结点都是哨兵nil。性质5，即从一个指定结点开始的每条简单路径上的黑结点的个数都是相等的，也会成立，因为结点z代替了(黑色)哨兵，并且结点z本身是有哨兵孩子的红结点。这样来看，仅可能被破坏的就是性质2和性质4，即根结点需要为黑色以及一个红结点不能有红孩子。这两个性质可能被破坏是因为z被着为红色。如果z是根结点，则破坏了性质2，如果z的父结点是红结点，则破坏了性质4.

while循环在每次迭代的开头保持下列三个部分的不变式：

1. 结点z是红结点。

2. 如果z.parent是根结点，则z.parent是黑结点。

3. 如果有任何红黑性质被破坏，则至多只有一条被破坏，或是性质2，或是性质4。如果性质2被破坏，其原因为z是根结点且是红结点。如果性质4被破坏，其原因为z和z.parent都是红结点。

需要考虑while循环中的6种情况，而其中三种与另外三种是对称的。这取决于z的父结点z.parent是z的祖父结点z.parent.parent的左孩子还是右孩子。根据循环不变式，如果z.parent是根结点，那么z.parent是黑色的，可知结点z.parent.parent存在。因为只有在z.parent是红色时才进入一次循环迭代，所以z.parent不可能是根结点，因此z.parent.parent存在。

情况1和情况2、情况3的区别在于z父亲的兄弟结点(或称为叔结点)的颜色不同。

**情况1：z的叔结点y是红色的。**

这种情况在z.parent和y都是红色时发生。因为z.parent.parent是黑色的，所以将z.parent和y都着为黑色，以此解决z和z.parent都是红色的问题，将z.parent.parent着为红色以保持性质5，然后把z.parent.parent作为新结点z来重复while循环。指针z在树中上移两层。

![](./assets/images/part3/red-black-tree6.png)

**情况2：z的叔结点y是黑色的且z是一个右孩子。**

**情况3：z的叔结点y是黑色的且z是一个左孩子。**

在情况2和情况3中，z的叔结点y是黑色的，通过z是z.parent的右孩子还是左孩子来区别这两种情况。在情况2中，结点z是它的父结点的右孩子，可以立即使用一个左旋来将此情形转变为情况3，此时结点z为左孩子。因为z和z.parent都是红色的，所以该旋转对结点的黑高和性质5都无影响。无论是直接进入情况2，还是通过情况3进入情况2，z的叔结点y总是黑色的，因为否则就要执行情况1。此外，z.parent.parent一定存在。在情况3中，改变某些结点的颜色并做一次右旋，以保持性质5，这样，由于一行中不再有两个红色结点，所有的处理到此完毕。因为此时z.parent是黑色的，所以无需再执行while循环。

![](./assets/images/part3/red-black-tree7.png)

### 删除

与n个结点的红黑树上的其它操作一样，删除一个结点要花费O(lgn)的时间。与插入操作相比，删除操作要稍微复杂些。

从一棵红黑树中删除结点是基于普通二叉搜索树的delete过程。首先，需要特别设计一个子程序transplant，并将其应用到红黑树上。

```java
private void transplant(Node src, Node dest) {
    if (dest.parent == nil) {
        root = src;
    } else if (dest == dest.parent.left) {
        dest.parent.left = src;
    } else {
        dest.parent.right = src;
    }
    src.parent = dest.parent;
}
```

红黑树的delete和普通二叉搜索树的delete类似，只是多了几行代码。多出的几行代码记录结点y的踪迹，y有可能导致红黑性质的破坏。当想要删除结点z，且此时z的子结点少于2个时，z从树中删除，并让y成为z。当z有两个子结点时，y应该是z的后继，并且y将移至树中的z位置。在结点被移除或者在树中移动之前，必须记住y的颜色，并且记录结点x的踪迹，将x移至树中y原来的位置，因为结点x也可能会引起红黑性质的破坏。删除结点z后，需要调用一个辅助过程deleteFixUp，该过程通过改变颜色和执行旋转来恢复红黑性质。

```java
public void delete(int key) {
    Node node = search(key);
    if (node == nil) {
        return;
    }
    boolean deletedColor = node.color;
    Node replacer;
    if (node.left == nil) {
        replacer = node.right;
        transplant(node.right, node);
    } else if (node.right == nil) {
        replacer = node.left;
        transplant(node.left, node);
    } else {
        Node successor = minimum(node.right);
        deletedColor = successor.color;
        replacer = successor.right;
        if (successor.parent != node) {
            transplant(successor.right, successor);
            successor.right = node.right;
            successor.right.parent = successor;
        }
        transplant(successor, node);
        successor.left = node.left;
        successor.left.parent = successor;
        successor.color = node.color;
    }
    if (deletedColor == BLACK) {
        deleteFixUp(replacer);
    }
}

private void deleteFixUp(Node node) {
    while (node != nil && node.color == BLACK) {
        if (node == node.parent.left) {
            Node sibling = node.parent.right;
            if (sibling.color == RED) {
                sibling.color = BLACK;
                node.parent.color = RED;
                leftRotate(node.parent);
                sibling = node.parent.right;
            }
            if (sibling.left.color == BLACK && sibling.right.color == BLACK) {
                sibling.color = RED;
                node = node.parent;
            } else {
                if (sibling.right.color == BLACK) {
                    sibling.left.color = BLACK;
                    sibling.color = RED;
                    rightRotate(sibling);
                    sibling = node.parent.right;
                }
                sibling.color = node.parent.color;
                node.parent.color = BLACK;
                sibling.right.color = BLACK;
                leftRotate(node.parent);
                node = root;
            }
        } else {
            Node sibling = node.parent.left;
            if (sibling.color == RED) {
                sibling.color = BLACK;
                node.parent.color = RED;
                rightRotate(node.parent);
                sibling = node.parent.left;
            }
            if (sibling.left.color == BLACK && sibling.right.color == BLACK) {
                sibling.color = RED;
                node = node.parent;
            } else {
                if (sibling.left.color == BLACK) {
                    sibling.right.color = BLACK;
                    sibling.color = RED;
                    leftRotate(sibling);
                    sibling = node.parent.left;
                }
                sibling.color = node.parent.color;
                node.parent.color = BLACK;
                sibling.left.color = BLACK;
                rightRotate(node.parent);
                node = root;
            }
        }
    }
    node.color = BLACK;
}
```

insertFixUp恢复性质1、性质2和性质4。while循环的目标是将额外的黑色沿树上移，直到：

1. x指向红黑结点，此时将x着为单个黑色。
2. x指向根结点，此时可以简单地移除额外的黑色。
3. 执行适当的旋转和重新着色，退出循环。

在while循环中，x总是指向一个具有双重黑色的非根结点。由于结点x是双重黑色的，故w不可能是nil，因为否则，从x.parent至单黑色叶子w的简单路径上的黑结点个数就会小于从x.parent到x的简单路径上的黑结点个数。

下图给出了代码中的4种情况。

![](./assets/images/part3/red-black-tree8.png)

**情况1：x的兄弟结点w是红色的。**

情况1发生在结点x的兄弟结点w为红色时。因为w必须有黑色子结点，所以可以改变w和x.parent的颜色，然后对x.parent做一次左旋而不违反红黑树的任何性质。现在，x的新兄弟结点是旋转之前w的某个子结点，其颜色为黑色。这样，就将情况1转化为情况2、3或4处理。

当结点w为黑色时，属于情况2、3和4，这些情况是由w的子结点的颜色来区分的。

**情况2：x的兄弟结点w是黑色的，而且w的两个子结点都是黑色的。**

在情况2中，w的两个子结点都是黑色的。因为w也是黑色的，所以从x和w上去掉一重黑色，使得x只有一重黑色而w为红色。为了补偿从x和w中去掉的一重黑色，在原来是红色或黑色的x.parent上新增一重额外的黑色。通过将x.parent作为新结点x来重复while循环。注意到，如果通过情况1进入到情况2，则新结点x是红黑色的，因为原来的x.parent是红色的。因此，新结点x的color属性为RED，并且在测试循环条件后终止。然后，将新结点x着为单一黑色。

**情况3：x的兄弟结点w是黑色的，w的左孩子是红色的，w的右孩子是黑色的。**

情况3发生在w为黑色且其左孩子为红色、右孩子为黑色时。可以交换w和其左孩子w.left的颜色，然后对w进行右旋而不违反红黑树的任何性质。现在x的新兄弟结点w是一个有红色右孩子的黑色结点，这样我们就将情况3转换成了情况4.

**情况4：x的兄弟结点w是黑色的，且w的右孩子是红色的。**

情况4发生在结点x的兄弟结点w为黑色且w的右孩子为红色时。通过进行某些颜色修改并对x.parent做一次左旋，可以去掉x的额外黑色，从而使得它变为单重黑色，而且不破坏红黑树的任何性质。将x设置为根后，当while循环测试其循环条件时，循环终止。

delete的运行时间是怎样的呢？因为含有n个结点的红黑树的高度为O(lgn)，不调用deleteFixUp时该过程总时间代价为O(lgn)。在deleteFixUp后，情况1、3和4在各执行常数次数的颜色改变和至多3次旋转后便终止。情况2是while循环可以重复执行的唯一情况，然后指针x沿树上升至多O(lgn)次，且不执行任何旋转。所以，deleteFixUp要花费O(lgn)时间，做至多3次旋转。因此，delete运行的总时间为O(lgn)。
