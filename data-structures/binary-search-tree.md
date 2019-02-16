## 二叉搜索树

### 什么是二叉搜索树

顾名思义，二叉搜索树是以一棵二叉树的形式来组织的。如下图所示，这样一棵树可是使用一个链表数据结构来表示，其中每个节点是一个对象。除了 key 和卫星数据之外，每个节点还包含属性 left、right 和 parent，它们分别指向节点的左孩子、右孩子和父结点。如果某个孩子节点或父结点不存在，则相应属性为 null。根节点是树中唯一父指针为 null 的节点。

![](../assets/images/part3/binary-search-tree.png)

```java
public class BinarySearchTree {
    Node root;
    
    class Node {
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

> 设 x 是二叉搜索树中的一个结点，如果 y 是 x 左子树中的一个结点，那么 y.key <= x.key。如果 y 是 x 右子树中的一个结点，那么 y.key >= x.key。即树中任意结点的关键字，总是大于等于左孩子，小于等于右孩子。

#### 二叉搜索树的遍历

二叉搜索树性质允许我们通过一个简单的递归算法来按序输出二叉搜索树中的所有关键字，这种算法称为中序遍历 (inorder tree walk)。这样命名的原因是输出的子树根的关键字位于其左子树的关键字和右子树的关键字之间。类似地，先序遍历 (preorder tree walk) 输出的根的关键字在其左右子树关键字之前，而后序遍历 (postorder tree walk) 输出的根的关键字在其左右子树的关键字之后。

```java
void inorderTreeWalk(Node node) {
    if (node != null) {
        inorderTreeWalk(node.left);
        System.out.println(node);
        inorderTreeWalk(node.right);
    }
}
```

遍历一棵有 n 个结点的二叉搜索树需要耗费 Θ(n) 的时间，因为初次调用后，对于树中的每个结点这个过程恰好要自己调用两次：一次是它的左孩子，一次是它的右孩子。

使用栈作为辅助数据结构，可以实现中序遍历的非递归算法。

```java
void nonRecursiveInorderTreeWalk(Node node) {
    Stack<Node> stack = new Stack<>();
    while (node != null || !stack.isEmpty()) {
        // Reach the left most node of the current node
        while (node != null) {
            // Place pointer to a tree node on the stack before traversing the node's left subtree
            stack.push(node);
            node = node.left;
        }
        
        // Current must be null at this point
        node = stack.pop();
        
        System.out.println(node);
        
        // We have visited the node and it's left subtree, now, it's right subtree's turn
        node = node.right;
    }
}
```

实际上，不使用栈也能实现非递归的中序遍历。

```java
void MorrisTraversal(Node node) {
    while (node != null) {
        if (node.left == null) {
            System.out.println(node);
            node = node.right;
        } else {
            // Find the inorder predecessor of current
            Node pre = node.left;
            while (pre.right != null && pre.right != node) {
                pre = pre.right;
            }

            // Make current as right child of its inorder predecessor
            if (pre.right == null) {
                pre.right = node;
                node = node.left;
            } else {
                // Revert the changes made in "if" part to restore the original tree, 
                // fix the right child of predecessor
                pre.right = null;
                System.out.println(node);
                node = node.right;
            }
        }
    }
}
```

### 查询二叉搜索树

我们经常需要查找一个存储在二叉搜索树中的关键字，除了 search 之外，二叉搜索树还能支持诸如 minimum, maximum, successor 和 predecessor 的查询操作。在一棵高度为 h 的二叉搜索树上，可以在 O(h) 时间内完成每个操作。

#### 查找

使用下面的方法在一棵二叉搜索树中查找一个具有给定关键字的结点。

```java
Node search(int key) {
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

这个过程从树根开始，并沿着这棵树的一条简单路径向下进行。对于遇到的每个节点 node，比较关键字 key 和 node.key，如果两个关键字相等，查找就终止。如果 key 小于 node.key，查找在 node 的左子树中继续，因为二叉搜索树性质蕴含了 key 不可能被存储在右子树中。对称地，如果 key 大于 node.key，查找在右子树中继续。从树根开始递归期间遇到的节点就形成了一条向下的简单路径，所以 search 的运行时间是 O(h)，其中 h 是这棵树的高度。

#### 最大关键字元素和最小关键字元素

通过从树根开始沿着 left 孩子指针直到遇到一个 null，我们总能在一棵二叉搜索树中找到以给定节点 x 为根的最小节点。

```java
Node minimum(Node node) {
    while (node.left != null) {
        node = node.left;
    }
    return node;
}
```

二叉搜索树的性质保证了 minimum 是正确的。如果结点 x 没有左子树，那么由于 x 右子树中的每个关键字都至少大于或等于 x.key，则以 x 为根的子树中的最小关键字是 x.key。如果结点 x 有左子树，那么由于其右子树中没有关键字小于 x.key，且在左子树中的每个关键字不大于 x.key，则以 x 为根的子树的最小关键字一定在以 x.left 为根的子树中。

maximum 的代码是对称的：

```java
Node maximum(Node node) {
    while (node.right != null) {
        node = node.right;
    }
    return node;
}
```

这两个方法在一棵高度为 h 的树上均能在 O(h) 时间内执行完，因为与 search 一样，它们所遇到的结点均形成了一条从树根向下的简单路径。

#### 后继和前驱

给定一棵二叉搜索树中的一个结点，有时候需要按中序遍历的次序查找它的后继。如果所有的关键字互不相同，则一个结点 x 的后继是大于 x.key 的最小关键字的节点。一棵二叉搜索树的结构允许我们通过没有任何关键字的比较来确定一个结点的后继。

```java
Node successor(Node node) {
    if (node.right != null) {
        return minimum(node.right);
    }
    Node p = node.parent;
    while (p != null && node == p.right) {
        node = p;
        p = p.parent;
    }
    return p;
}
```

![](../assets/images/part3/binary-search-tree2.png)

successor 的代码分两种情况：如果结点 x 的右子树非空，那么 x 的后继恰好是 x 右子树中的最左结点，通过调用 minimum 可以得到，例如，在上图中，15 的后继是 17。如果结点 x 的右子树为空并有一个后继 y，那么 y 就是 x 的有左孩子的第一个祖先，并且 x 在它的左子树上，例如，在上图中，13 的后继是 15。

在一棵高度为 h 的树上，successor 的运行时间为 O(h)，因为该过程或者遵从一条简单路径沿树向上或者遵从一条简单路径沿树向下。predecessor 和 successor 是对称的，其运行时间也是 O(h)。

```java
Node predecessor(Node node) {
    if (node.left != null) {
        return maximum(node.left);
    }
    Node p = node.parent;
    while (p != null && node == p.left) {
        node = p;
        p = p.parent;
    }
    return p;
}
```

即使关键字非全不相同，我们仍然定义任何节点 x 的后继和前驱为分别调用 successor 和 predecessor 所返回的结点。

### 插入和删除

插入和删除操作会引起二叉搜索树表示的动态集合的变化，一定要修改数据结构来反映这个变化，但修改要保持二叉搜索树的性质的成立。

#### 插入

```java
void insert(int key) {
    Node node = new Node(key);
    Node p = null;
    Node temp = root;
    while (temp != null) {
        p = temp;
        if (key < temp.key) {
            temp = temp.left;
        } else {
            temp = temp.right;
        }
    }
    node.parent = p;
    if (p == null) {
        root = node;
    } else if (key < p.key) {
        p.left = node;
    } else {
        p.right = node;
    }
}
```

insert 从树根开始，指针 temp 记录了一条向下的简单路径，该过程保持 p 作为 temp 的父结点。while 循环使得这两个指针沿树向下移动，向左或向右移动取决于 key 和 temp.key 的比较，直到 temp 变为 null。这个 null 占据的位置就是输入项新结点要放置的地方。

与其它搜索树上的原始操作一样，insert 在一棵高度为 h 的树上运行时间为 O(h)。

#### 带有相同关键字的二叉搜索树

相同关键字给二叉搜索树的实现带来了问题，将 n 个具有相同关键字的数据插入到一棵初始为空的二叉搜索树中，由于 if (key < temp.key) 始终为 false，故所有的元素都会插入到唯一的叶结点的右边，这样就形成了一个长度为 n 的链表，插入、查找和删除操作的时间复杂度都退化到了 O(n)。通过在 if (key < temp.key) 之前测试 if (key == temp.key)，如果相等，可以根据下面的策略之一来改进：

1. 在结点 node 设置一个布尔标志 node.b，并根据 node.b 的值，置 node 为 node.left 或 node.right。当插入一个与 node 关键字相同的结点时，每次访问 node 时交替地置 node.b 为 false 或 true。

2. 在 node 处设置一个与 node 关键字相同的结点列表，并将新结点插入到该列表中。

3. 随机地置 node 为 node.left 或 node.right。

#### 删除

从一棵二叉搜索树上删除一个结点 z 的整个策略分为三种基本情况，但只有一种情况有点棘手。

1. 如果 z 没有孩子结点，那么只是简单地将它删除，并修改它的父节点，用 null 作为孩子来替换 z。

2. 如果 z 只有一个孩子，那么将这个孩子提升到树中 z 的位置上，并修改 z 的父结点，用 z 的孩子来替换 z。

3. 如果 z 有两个孩子，那么找 z 的后继 y（一定在 z 的右子树上），并让 y 占据树中 z 的位置。z 的原来右子树部分成为 y 的新的右子树，并且 z 的左子树成为 y 的新的左子树。这种情况稍显麻烦，因为还与 y 是否为 z 的右孩子相关。

从一棵二叉搜索树中删除一个给定的节点 z，考虑下图显示的 4 种情况，它与前面概括的 3 种情况有些不同。

![](../assets/images/part3/binary-search-tree3.png)

1. 如果 z 没有左孩子，那么用其右孩子来替换 z，这个右孩子可以是 null，也可以不是。当 z 的右孩子是 null 时，此时这种情况归为 z 没有孩子结点的情形。当 z 的右孩子非 null 时，这种情况就是 z 仅有一个孩子结点的情形，该孩子是其右孩子。

2. 如果 z 仅有一个孩子且为其左孩子，那么用其左孩子来替换 z。

3. 否则，z 既有一个左孩子又有一个右孩子。我们要查找 z 的后继 y，这个后继位于 z 的右子树中并且没有左孩子。现在需要将 y 移出原来的位置进行拼接，并替换树中的 z。

4. 如果 y 是 z 的右孩子，那么用 y 替换 z，并仅留下 y 的右孩子。

5. 否则，y 位于 z 的右子树中但不是 z 的右孩子。在这种情况下，先用 y 的右孩子替换 y，然后再用 y 替换 z。

为了在二叉搜索树中移动子树，定义一个子过程 transplant，它用另一棵子树替换一棵子树并成为其父结点的孩子结点。当 transplant 用一棵以 src 为根的子树来替换一棵以 dest 为根的子树时，结点 dest 的父结点就变为结点 src 的父结点，并且最后 src 成为 dest 的父结点的相应孩子。

transplant 并没有处理 src.left 和 src.right 的更新，这些更新都由 transplant 的调用者来负责。

```java
void delete(int key) {
    Node node = search(key);
    if (node == null) {
        return;
    }
    if (node.left == null) {
        transplant(node.right, node);
    } else if (node.right == null) {
        transplant(node.left, node);
    } else {
        Node suc = minimum(node.right);
        if (suc.panrent != node) {
            transplant(suc.right, suc);
            suc.right = node.right;
            suc.right.parent = suc;
        }
        transplant(suc, node);
        suc.left = node.left;
        suc.left.parent = suc;
    }
}

void transplant(Node src, Node dest) {
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

除了调用 minimum 之外，delete 的每一行，包括调用 transplant，都只花费常数时间，因此，在一棵高度为 h 的树上，delete 的运行时间为 O(h)。
