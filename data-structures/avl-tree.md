## AVL 树

AVL 树是一种高度平衡的（height balanced）二叉搜索树：对每一个结点，它的左子树与右子树的高度差至多为 1。

![](../assets/images/data-structures/avl-tree1.jpg)

上图是一棵合法的 AVL 树，因为对每个结点来说，其左子树和右子树的高度差都不超过 1。

![](../assets/images/data-structures/avl-tree2.jpg)

上图不是一棵合法的 AVL 树，因为对结点 12 和 8 来说，它们的左右子树高度差都超过了 1。

对于一棵高度为 h 的普通二叉搜索树来说，它的 search、predecessor、successor、minimum、maximum、insert 和 delete 操作的时间复杂度均为 O(h)。因此，如果搜索树的高度较低时，这些集合操作会执行得比较快。然而，如果树的形状比较偏斜，高度达到了 O(n)，这些集合操作的时间复杂度就退化到了 O(n)。AVL 树是许多平衡搜索树中的一种，可以保证每次插入和删除操作之后，树的高度 h 一直都能维持在 O(lgn)。

要实现一棵 AVL 树，需要在每个结点内维护一个额外的属性 height，代表该结点的高度。要在一棵 AVL 树中插入一个结点，首先以二叉搜索树的顺序把该结点放在适当的位置上，此时，这棵树可能就不再是高度平衡的，具体来说，某些结点的左子树和右子树的高度差可能会到 2。为了让树重新保持平衡，我们引入一个 balance 操作，输入一棵以 x 为根的子树，其左子树和右子树都是高度平衡的，而且它们的高度差至多是 2，即丨x.right.height - x.left.height丨<= 2，将这棵以 x 为根的子树转变为高度平衡的。

```java
public class AVLTree {
    Node root;

    class Node {
        int key;
        int height;
        Node left;
        Node right;

        Node(int key) {
            this.key = key;
            height = 1;
        }
    }

    int getHeight(Node node) {
        if (node == null) {
            return 0;
        } else {
            return node.height;
        }
    }

    int getDifference(Node node) {
        if (node == null) {
            return 0;
        } else {
            return getHeight(node.left) - getHeight(node.right);
        }
    }

    void balance(Node node) {
        while (Math.abs(getDifference(node)) > 1) {
            if (getDifference(node) > 0) {
                rightRotate(node);
            } else {
                leftRotate(node);
            }
            balance(node.left);
            balance(node.right);
        }
    }

    Node rightRotate(Node p) {
        Node l = p.left;
        p.left = l.right;
        l.right = p;
        p.height = Math.max(getHeight(p.left), getHeight(p.right)) + 1;
        l.height = Math.max(getHeight(l.left), getHeight(l.right)) + 1;
        return l;
    }

    Node leftRotate(Node p) {
        Node r = p.right;
        p.right = r.left;
        r.left = p;
        p.height = Math.max(getHeight(p.left), getHeight(p.right)) + 1;
        r.height = Math.max(getHeight(r.left), getHeight(r.right)) + 1;
        return r;
    }

    Node insert(Node node, int key) {
        // Perform the normal BST insertion
        if (node == null) {
            return new Node(key);
        }
        if (key < node.key) {
            node.left = insert(node.left, key);
        } else if (key > node.key) {
            node.right = insert(node.right, key);
        } else {
            // Duplicated keys not allowed
            return node;
        }

        // Update height of this ancestor node
        node.height = Math.max(getHeight(node.left), getHeight(node.right)) + 1;

        // Get the balance factor of this ancestor node to check whether this node became unbalanced
        int balance = getDifference(node);

        // If this node becomes unbalanced, then there are 4 cases

        // Left Left Case
        if (balance > 1 && key < node.left.key) {
            return rightRotate(node);
        }

        // Right Right Case
        if (balance < -1 && key > node.right.key) {
            return leftRotate(node);
        }

        // Left Right Case
        if (balance > 1 && key > node.left.key) {
            node.left = leftRotate(node.left);
            return rightRotate(node);
        }

        // Right Left Case
        if (balance < -1 && key < node.right.key) {
            node.right = rightRotate(node.right);
            return leftRotate(node);
        }

        // Return the (unchanged) node pointer
        return node;
    }
}
```

