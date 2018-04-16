## 二叉搜索树

### 什么是二叉搜索树

顾名思义，二叉搜索树是以一棵二叉树的形式来组织的。如下图所示，这样一棵树可是使用一个链表数据结构来表示，其中每个节点是一个对象。除了key和卫星数据之外，每个节点还包含属性left、right和parent，它们分别指向节点的左孩子、右孩子和父结点。如果某个孩子节点或父结点不存在，则相应属性为null。根节点是树中唯一父指针为null的节点。

![](./assets/images/part3/binary-search-tree.png)

```java
public class BinarySearchTree {
    private Node root;
    
    private class Node {
        int key;
        Node parent;
        Node left;
        Node right;
        
        Node(int key) {
            this.key = key;
        }
    }
}
```

二叉搜索树中的关键字总是以满足二叉搜索树性质的方式存储的：

设x是二叉搜索树中的一个结点，如果y是x左子树中的一个结点，那么y.key <= x.key。如果y是x右子树中的一个结点，那么y.key >= x.key。

二叉搜索树性质允许我们通过一个简单的递归算法来按序输出二叉搜索树中的所有关键字，这种算法称为中序遍历(inorder tree walk)。这样命名的原因是输出的子树根的关键字位于其左子树的关键字和右子树的关键字之间。类似地，先序遍历(preorder tree walk)输出的根的关键字在其左右子树关键字之前，而后序遍历(postorder tree walk)输出的根的关键字在其左右子树的关键字之后。

```java
public void inorderTreeWalk(Node node) {
    if (node != null) {
        inorderTreeWalk(node.left);
        System.out.println(node);
        inorderTreeWalk(node.right);
    }
}
```

遍历一棵有n个结点的二叉搜索树需要耗费Θ(n)的时间，因为初次调用后，对于树中的每个结点这个过程恰好要自己调用两次：一次是它的左孩子，一次是它的右孩子。

### 查询二叉搜索树

我们经常需要查找一个存储在二叉搜索树中的关键字，除了search之外，二叉搜索树还能支持诸如minimum, maximum, successor和predecessor的查询操作。在一棵高度为h的二叉搜索树上，可以在O(h)时间内完成每个操作。

#### 查找

使用下面的方法在一棵二叉搜索树中查找一个具有给定关键字的结点。

```java
public Node search(int key) {
    Node node = root;
    while (node != null && node.key != key) {
        if (key < node.key) {
            node = node.left;
        } else {
            node = node.right;
        }
    }
    return node;
}
```

这个过程从树根开始，并沿着这棵树的一条简单路径向下进行。对于遇到的每个节点node，比较关键字key和node.key，如果两个关键字相等，查找就终止。如果key小于node.key，查找在node的左子树中继续，因为二叉搜索树性质蕴含了key不可能被存储在右子树中。对称地，如果key大于node.key，查找在右子树中继续。从树根开始递归期间遇到的节点就形成了一条向下的简单路径，所以search的运行时间是O(h)，其中h是这棵树的高度。

#### 最大关键字元素和最小关键字元素

通过从树根开始沿着left孩子指针直到遇到一个null,我们总能在一棵二叉搜索树中找到以给定节点x为根的最小节点。

```java
public Node minimum(Node node) {
    while (node.left != null) {
        node = node.left;
    }
    return node;
}
```

二叉搜索树的性质保证了minimum是正确的。如果结点x没有左子树，那么由于x右子树中的每个关键字都至少大于或等于x.key，则以x为根的子树中的最小关键字是x.key。如果结点x有左子树，那么由于其右子树中没有关键字小于x.key，且在左子树中的每个关键字不大于x.key，则以x为根的子树的最小关键字一定在以x.left为根的子树中。

maximum的代码是对称的：

```java
public Node maximum(Node node) {
    while (node.right != null) {
        node = node.right;
    }
    return node;
}
```

这两个方法在一棵高度为h的树上均能在O(h)时间内执行完，因为与search一样，它们所遇到的结点均形成了一条从树根向下的简单路径。

#### 后继和前驱

给定一棵二叉搜索树中的一个结点，有时候需要按中序遍历的次序查找它的后继。如果所有的关键字互不相同，则一个结点x的后继是大于x.key的最小关键字的节点。一棵二叉搜索树的结构允许我们通过没有任何关键字的比较来确定一个结点的后继。

```java
public Node successor(Node node) {
    if (node.right != null) {
        return minimum(node.right);
    }
    Node parent = node.parent;
    while (parent != null && node == parent.right) {
        node = parent;
        parent = parent.parent;
    }
    return parent;
}
```

successor的代码分两种情况：如果结点x的右子树非空，那么x的后继恰好是x右子树中的最左结点，通过调用minimum可以得到。如果结点x的右子树为空并有一个后继y，那么y就是x的有左孩子的最底层祖先，并且它也是x的一个祖先。

在一棵高度为h的树上，successor的运行时间为O(h)，因为该过程或者遵从一条简单路径沿树向上或者遵从一条简单路径沿树向下。predecessor和successor是对称的，其运行时间也是O(h)。

```java
public Node predecessor(Node node) {
    if (node.left != null) {
        return maximum(node.left);
    }
    Node parent = node.parent;
    while (parent != null && node == parent.left) {
        node = parent;
        parent = parent.parent;
    }
    return parent;
}
```

即使关键字非全不相同，我们仍然定义任何节点x的后继和前驱为分别调用successor和predecessor所返回的结点。

### 插入和删除

插入和删除操作会引起二叉搜索树表示的动态集合的变化，一定要修改数据结构来反映这个变化，但修改要保持二叉搜索树的性质的成立。

#### 插入

```java
public void insert(int key) {
    Node newNode = new Node(key);
    Node parent = null;
    Node trailingPointer = root;
    while (trailingPointer != null) {
        parent = trailingPointer;
        if (key < trailingPointer.key) {
            trailingPointer = trailingPointer.left;
        } else {
            trailingPointer = trailingPointer.right;
        }
    }
    newNode.parent = parent;
    if (parent == null) {
        root = newNode;
    } else if (key < parent.key) {
        parent.left = newNode;
    } else {
        parent.right = newNode;
    }
}
```

insert从树根开始，指针trailingPointer记录了一条向下的简单路径，该过程保持parent作为trailingPointer的父结点。while循环使得这两个指针沿树向下移动，向左或向右移动取决于key和trailingPointer.key的比较，直到trailingPointer变为null。这个null占据的位置就是输入项newNode要放置的地方。

与其它搜索树上的原始操作一样，insert在一棵高度为h的树上运行时间为O(h)。

#### 删除

从一棵二叉搜索树上删除一个结点z的整个策略分为三种基本情况，但只有一种情况有点棘手。

1. 如果z没有孩子结点，那么只是简单地将它删除，并修改它的父节点，用null作为孩子来替换z。

2. 如果z只有一个孩子，那么将这个孩子提升到树中z的位置上，并修改z的父结点，用z的孩子来替换z。

3. 如果z有两个孩子，那么找z的后继y（一定在z的右子树上），并让y占据树中z的位置。z的原来右子树部分成为y的新的右子树，并且z的左子树成为y的新的左子树。这种情况稍显麻烦，因为还与y是否为z的右孩子相关。

从一棵二叉搜索树中删除一个给定的节点z，考虑下图显示的4种情况，它与前面概括的三种情况有些不同。

![](./assets/images/part3/binary-search-tree2.png)

1. 如果z没有左孩子，那么用其右孩子来替换z，这个右孩子可以是null，也可以不是。当z的右孩子是null时，此时这种情况归为z没有孩子结点的情形。当z的右孩子非null时，这种情况就是z仅有一个孩子结点的情形，该孩子是其右孩子。

2. 如果z仅有一个孩子且为其左孩子，那么用其左孩子来替换z。

3. 否则，z既有一个左孩子又有一个右孩子。我们要查找z的后继y，这个后继位于z的右子树中并且没有左孩子。现在需要将y移出原来的位置进行拼接，并替换树中的z。

4. 如果y是z的右孩子，那么用y替换z，并仅留下y的右孩子。

5. 否则，y位于z的右子树中但不是z的右孩子。在这种情况下，先用y的右孩子替换y，然后再用y替换z。

为了在二叉搜索树中移动子树，定义一个子过程transplant，它用另一棵子树替换一棵子树并成为其父结点的孩子结点。当transplant用一棵以v为根的子树来替换一棵以u为根的子树时，结点u的父结点就变为结点v的父结点，并且最后v成为u的父结点的相应孩子。

transplant并没有处理v.left和v.right的更新，这些更新都由transplant的调用者来负责。

```java
public void delete(int key) {
    Node node = search(key);
    if (node == null) {
        return;
    }
    if (node.left == null) {
        transplant(node.right, node);
    } else if (node.right == null) {
        transplant(node.left, node);
    } else {
        Node successor = minimum(node.right);
        if (successor.panrent != node) {
            transplant(successor.right, successor);
            successor.right = node.right;
            successor.right.parent = successor;
        }
        transplant(successor, node);
        successor.left = node.left;
        successor.left.parent = successor;
    }
}

private void transplant(Node src, Node dest) {
    if (dest.parent == null) {
        root = src;
    } else if (dest == dest.parent.left) {
        dest.parent.left = src;
    } else {
        dest.parent.right = src;
    }
    if (src != null) {
        src.parent = dest.parent;
    }
}
```

除了调用minimum之外，delete的每一行，包括调用transplant，都只花费常数时间，因此，在一棵高度为h的树上，delete的运行时间为O(h)。
