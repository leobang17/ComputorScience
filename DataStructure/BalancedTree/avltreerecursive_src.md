```java
public class AVLTreeRecursive<T extends Comparable<T>> {
    public class Node {
        // Balance Factor
        public int bf;

        public T value;

        // tree에서 이 node의 높이
        public int height;

        public Node left, right;

        public Node(T value) {
            this.value = value;
        }
    }

    // AVL tree의 root node
    public Node root;

    // tree에 있는 node의 갯수
    private int nodeCount = 0;

    public int height() {
        if (root == null) return 0;
        return root.height;
    }

    public int size() {
        return nodeCount;
    }

    public boolean isEmpty() {
        return size() == 0;
    }

    public boolean contains(T value) {
        return contains(root, value);
    }

    // 재귀적인 contains 메서드
    private boolean contains(Node node, T value) {
        if (node == null) return false;

        // 주어진 value와 node의 value를 비교
        int cmp = value.compareTo(node.value);

        // 왼쪽 subtree로 간다.
        if (cmp < 0) return contains(node.left, value);

        // 오른쪽 subtree로 간다
        if(cmp > 0) return contains(node.right, value);

        // 값을 찾음 (cmp == 0)
        return true;
    }

    // AVL tree애 값을 추가. - O(log(n))
    public boolean insert(T value) {
        if (value == null) return false;
        if (!contains(root, value)) {
            root = insert(root, value);
            nodeCount++;
            return true;
        }
        return false;
    }

    private Node insert(Node node, T value) {
        if (node == null) return new Node(value);

        int cmp = value.compareTo(node.value);

        // 왼쪽 subtree에 값을 insert
        if (cmp < 0) {
            node.left = insert(node.left, value);
            // 오른쪽 subtree에 값을 insert
        } else {
            node.right = insert(node.right, value);
        }

        // balance factor와 height 값을 update
        update(node);

        // tree를 re-balancing 한다.
        return balance(node);
    }

    private void update(Node node) {
        int leftNodeHeight = (node.left == null) ? -1 : node.left.height;
        int rightNodeHeight = (node.right == null) ? -1 : node.right.height;

        // node의 height를 update
        node.height = 1 + Math.max(leftNodeHeight, rightNodeHeight);

        // node의 balance factor를 udpate
        node.bf = rightNodeHeight - leftNodeHeight;
    }


    // balance factor가 2 혹은 -2일 때 node를 re-balance 한다.
    private Node balance(Node node) {
        // Left heavy Subtree
        if (node.bf == -2) {
            // left-left case
            if (node.left.bf <= 0) {
                return leftLeftCase(node);
                // left-right case
            } else {
                return leftRightCase(node);
            }
            // Right heavy Subtree
        } else if (node.bf == +2) {
            // right-right case
            if (node.right.bf >= 0) {
                return rightRightCase(node);
                // right-left case
            } else {
                return rightLeftCase(node);
            }
        }

        // balance factor가 -1, 0, 1 중 하나이므로 good
        return node;
    }

    private Node leftLeftCase(Node node) {
        return rightRotation(node);
    }

    private Node leftRightCase(Node node) {
        node.left = leftRotation(node.left);
        return leftLeftCase(node);
    }

    private Node rightLeftCase(Node node) {
        return leftRotation(node);
    }

    private Node rightRightCase(Node node) {
        node.right = rightRotation(node.right);
        return rightRightCase(node);
    }

    private Node leftRotation(Node node) {
        Node newParent = node.right;
        node.right = newParent.left;
        newParent.left = node;
        update(node);
        update(newParent);
        return newParent;
    }

    private Node rightRotation(Node node) {
        Node newParent = node.left;
        node.left = newParent.right;
        newParent.right = node;
        update(node);
        update(newParent);
        return newParent;
    }

    // Binary Tree에서 value를 찾고, 있으면 remove해준다. - O(log(n))
    public boolean remove(T elem) {
        if (elem == null) {
            return false;
        }

        if (contains(root, elem)) {
            root = remove(root, elem);
            nodeCount--;
            return true;
        }
        return false;
    }

    // AVL Tree에서 값을 삭제
    private Node remove(Node node, T elem) {
        if (node == null) {
            return null;
        }

        int cmp = elem.compareTo(node.value);

        // 찾는 값이 현재보다 작으므로 왼쪽 subtree로 간다
        if (cmp < 0) {
            node.left = remove(node.left, elem);
        } else if (cmp > 0) {
            node.right = remove(node.right, elem);

            // 우리가 지우고자 하는 것을 찾음.
        } else {
            // right subtree만 있거나, subtree가 없을 경우.
            // 그냥 right child랑 swap만 하면 된다 .
            if (node.left == null) {
                return node.right;
            } else if (node.right == null) {
                return node.left;

                // 좌우 subtree가 모두 존재할 경우, 둘 중 하나를 정하는데
                // left를 정할 경우, left subtree에서 가장 큰 값을 가지는 node와 바꾸거나,
                // right를 정할 경우, right subtree에서 가장 작은 값을 가지는 node와 바꾸면 된다.
            } else {
                // 왼쪽 subtree에서 바꾸기로 한다.
                if (node.left.height > node.right.height) {
                    T successorValue = findMax(node.left);
                    node.value = successorValue;
                    node.left = remove(node.left, successorValue);
                    // 오른쪽 subtree에서 바꾸기로 한다.
                } else {
                    T successorValue = findMin(node.right);
                    node.value = successorValue;
                    node.right = remove(node.right, successorValue);
                }
            }
        }

        update(node);

        return balance(node);
    }

    private T findMin(Node node) {
        while (node.left != null) {
            node = node.left;
        }
        return node.value;
    }

    private T findMax(Node node) {
        while (node.right != null) {
            node = node.right;
        }
        return node.value;
    }

    // 모든 left child node가 그 부모보다 작음을 확인하고,
    // 모든 right child node가 그 부모보다 큼을 확인한다.
    public boolean validateBSTInvarient(Node node) {
        if (node == null) return true;

        T val = node.value;
        boolean isValid = true;
        if (node.left != null) isValid = isValid && node.left.value.compareTo(val) < 0;
        if (node.right != null) isValid = isValid && node.right.value.compareTo(val) > 0;

        return isValid && validateBSTInvarient(node.left) && validateBSTInvarient(node.right);
    }


}
```
