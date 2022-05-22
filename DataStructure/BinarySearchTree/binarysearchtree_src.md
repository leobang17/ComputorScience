```java
// 어떤 것이든, comparable한 것을 type으로 가진다.
public class BinarySearchTree<T extends Comparable<T>> {
    // BST의 node 개수를 track
    private int nodeCount = 0;

    // 이 BST는 rooted tree이므로 root node를 관리한다.
    private Node root = null;

    // node reference를 가지고 있는 internal node class.
    private class Node {
        T data;
        Node left, right;

        public Node(Node left, Node right, T elem) {
            this.data = elem;
            this.left = left;
            this.right = right;
        }
    }

    // BST의 node 개수 return
    public int size() {
        return nodeCount;
    }

    // 비었는지
    public boolean isEmpty() {
        return size() == 0;
    }

    // Binary tree에 element 삽입. 성공했을 경우 true 반환
    public boolean add(T elem) {
        // 값이 이미 binary tree에 존재하는지 확인 -> 있으면 무시한다.
        if (contains(elem)) {
            return false;
        } else {
            root = add(root, elem);
            nodeCount ++;
            return true;
        }
    }

    private Node add(Node node, T elem) {
        // Base case: leaf node (말단 노드)를 발견
        if (node == null) {
            node = new Node(null, null, elem);
        } else {
            // element를 삽입할 subtree를 고른다.
            if (elem.compareTo(node.data) < 0) {
                node.left = add(node.left, elem);
            } else {
                node.right = add(node.right, elem);
            }
        }

        return node;
    }

    // 값이 존재한다면, binary tree안에서 제거함. - O(n)
    public boolean remove(T elem) {
        // 지우기 전에 지우려는 node 가 존재하는지 확인
        if (contains(elem)) {
            root = remove(root, elem);
            nodeCount --;
            return true;
        }

        return false;
    }

    private Node remove(Node node, T elem) {
        if (node == null) return null;

        int cmp = elem.compareTo(node.data);

        if (cmp < 0) {
            node.left = remove(node.left, elem);
        } else if (cmp > 0) {
            node.right = remove(node.right, elem);

            // 지우고자 하는 node를 찾음.
        } else {
            // right subtree만 존재하거나, subtree가 존재하지 않는 경우.
            // 이런 경우에는 제거하려는 node의 right child랑 자리만 swap 하면 된다. (없는 경우 null이 들어가니까 아무래도 상관x)
            if (node.left == null) {
                Node rightChild = node.right;
                node.data = null;
                node = null;

                return rightChild;

                // left subtree만 존재하거나, subtree가 존재하지 않는 경우.
                // 이런 경우에는 제거하려는 node의 left child와 자리만 swap하면 된다. (없는 경우 null이 들어가니까 아무래도 상관x)
            } else if (node.right == null) {
                Node leftChild = node.left;

                node.data = null;
                node = null;

                return leftChild;

                // left와 right subtree가 모두 존재하는 경우
                // 1. left subtree의 가장 큰 값을 가지는 node
                // 2. right subtree의 가장 작은 값을 가지는 node
                // 둘 중 하나가 successor가 되어야 한다. -> 여기서는 2번을 따른다.
            } else {
                // right subtree의 가장 왼쪽 값을 찾자.
                Node tmp = findMin(node.right);

                // data를 swap
                node.data = tmp.data;

                // right subtree로 가서 왼쪽의 가장 큰 node (tmp에 찾은 것)를 삭제함
                node.right = remove(node.right, tmp.data);
            }
        }

        return node;
    }

    private Node findMin(Node node) {
        while (node.left != null) {
            node = node.left;
        }
        return node;
    }

    private Node findMax(Node node) {
        while (node.right != null) {
            node = node.right;
        }
        return node;
    }

    // element가 tree안에 존재하면 true를 반환.
    public boolean contains(T elem) {
        return contains(root, elem);
    }

    // tree 안에 element가 존재하는지 찾기위한 recursive method
    private boolean contains(Node node, T elem) {
        // Base case: bottom에 도달함. -> 값을 찾지 못한 것이므로 false
        if (node == null) return false;

        int cmp = elem.compareTo(node.data);

        // 찾는 값이 현재보다 작으므로 왼쪽 subtree로 내려가기.
        if (cmp < 0) return contains(node.left, elem);

        // 찾는 값이 현재보다 크므로 오른쪽 subtree로 내려가기.
        else if (cmp > 0) return contains(node.right, elem);

        // cmp == 0 이므로 우리가 찾는 값이 나옴 -> true 반환.
        else return true;
    }

    // tree의 높이를 구해주는 재귀 메서드.
    public int height(Node node) {
        if (node == null) return 0;
        return Math.max(height(node.left), height(node.right)) + 1;
    }

    // TreeTraveralOrder (enum) 값에 따라 iterator를 반환해준다.
    // Preorder, Inorder, Postorder, Levelorder
    public Iterator<T> traverse(TreeTraversalOrder order) {
        switch (order) {
            case PRE_ORDER:
                return preOrderTraversal();
            case IN_ORDER:
                return inOrderTraversal();
            case POST_ORDER:
                return postOrderTraversal();
            case LEVEL_ORDER:
                return levelOrderTraversal();
            default:
                return null;
        }
    }

    // tree를 pre order로 순회하는 iterator를 반환
    public Iterator<T> preOrderTraversal() {
        final int expectedNodeCount = nodeCount;
        final Stack<Node> stack = new Stack<>();
        stack.push(root);

        return new Iterator<T>() {
            @Override
            public boolean hasNext() {
                if (expectedNodeCount != nodeCount) throw new ConcurrentModificationException();
                return root != null && !stack.isEmpty();
            }

            @Override
            public T next() {
                if (expectedNodeCount != nodeCount) throw new ConcurrentModificationException();
                Node node = stack.pop();
                if (node.right != null) stack.push(node.right);
                if (node.left != null) stack.push(node.left);
                return node.data;
            }

            @Override
            public void remove() {
                throw new UnsupportedOperationException();
            }
        };
    }

    public Iterator<T> inOrderTraversal() {
        final int expectedNodeCount = nodeCount;
        final Stack<Node> stack = new Stack<>();
        stack.push(root);

        return new Iterator<T>() {
            Node trav = root;

            @Override
            public boolean hasNext() {
                if (expectedNodeCount != nodeCount) throw new ConcurrentModificationException();
                return root != null && !stack.isEmpty();
            }

            @Override
            public T next() {
                if (expectedNodeCount != nodeCount) throw new ConcurrentModificationException();
                // Dig left
                while (trav != null && trav.left != null) {
                    stack.push(trav.left);
                    trav = trav.left;
                }

                Node node = stack.pop();

                // 오른쪽으로 한번 더 감
                if (node.right != null) {
                    stack.push(node.right);
                    trav = node.right;
                }

                return node.data;
            }

            @Override
            public void remove() {
                throw new UnsupportedOperationException();
            }
        };
    }

    public Iterator<T> postOrderTraversal() {
        final int expectedNodeCount = nodeCount;
        final Stack<Node> stack1 = new Stack<>();
        final Stack<Node> stack2 = new Stack<>();
        stack1.push(root);
        while (!stack1.isEmpty()) {
            Node node = stack1.pop();
            if (node != null) {
                stack2.push(node);
                if (node.left != null) stack1.push(node.left);
                if (node.right != null) stack1.push(node.right);
            }
        }

        return new Iterator<T>() {
            @Override
            public boolean hasNext() {
                if (expectedNodeCount != nodeCount) throw new ConcurrentModificationException();
                return root != null && !stack2.isEmpty();
            }

            @Override
            public T next() {
                if (expectedNodeCount != nodeCount) throw new ConcurrentModificationException();
                return stack2.pop().data;
            }

            @Override
            public void remove() {
                throw new UnsupportedOperationException();
            }
        };
    }

    public Iterator<T> levelOrderTraversal() {
        final int expectedNodeCount = nodeCount;
        final java.util.Queue<Node> queue = new LinkedList<>();
        queue.offer(root);

        return new Iterator<T>() {
            @Override
            public boolean hasNext() {
                if (expectedNodeCount != nodeCount) throw new ConcurrentModificationException();
                return root != null && !queue.isEmpty();
            }

            @Override
            public T next() {
                if (expectedNodeCount != nodeCount) throw new ConcurrentModificationException();
                Node node = queue.poll();
                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
                return node.data;
            }

            @Override
            public void remove() {
                throw new UnsupportedOperationException();
            }
        };
    }
}
```
