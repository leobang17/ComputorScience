```java
public class UnionFind {
    // union find안의 element 수
    private int size;

    // 각 component의 size를 track 하기 위함
    private int[] sz;

    // id[i] 는 i의 parent를 가리킨다. id[i] == i 라면 i가 root node다.
    private int[] id;

    // union find의 component 개수를 track
    private int numComponents;

    public UnionFind(int size) {
        if (size <= 0)
            throw new IllegalArgumentException("Size <= 0 is not allowed");

        this.size = numComponents = size;
        sz = new int[size];
        id = new int[size];

        for (int i = 0; i < size; i ++) {
            id[i] = i; // self root로 link
            sz[i] = 1; // 각 component는 생성시 size가 1
        }
    }

    // p가 어떤 component/set에 속하는지 찾는다. - a(n) amortized constant time
    public int find(int p) {
        // component/set의 root을 찾는다.
        int root = p;
        while (root != id[root]) root = id[root];

        // root을 향하는 path를 compress -> "path compression"
        while (p != root) {
            int next = id[p];
            id[p] = root;
            p = next;
        }

        return root;
    }

    // 'p'와 'q'가 같은 component/set에 있는지 return
    public boolean connected(int p, int q) {
        return find(p) == find(q);
    }

    // 'p'가 속하는 component/set의 size를 return
    public int componentSize(int p) {
        return sz[find(p)];
    }

    // 이 UnionFind/Disjoint set의 element 개수를 return
    public int components() {
        return this.numComponents;
    }

    // 'p'와 'q'가 담긴 components/sets을 unify
    public void unify(int p, int q) {
        int root1 = find(p);
        int root2 = find(q);

        // 이미 같은 group일 경우 return
        if (root1 == root2) return;

        // 작은 걸 큰 걸로 merge
        if (sz[root1] < sz[root2]) {
            sz[root1] += sz[root2];
            id[root1] = root2;
        } else {
            sz[root2] += sz[root1];
            id[root2] = root1;
        }

        // component/set의 개수가 1개 줄어든다 (merge 했으므로)
        numComponents--;
    }
}
```
