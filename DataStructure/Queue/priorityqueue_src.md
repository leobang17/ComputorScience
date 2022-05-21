```java
public class PriorityQueue <T extends Comparable<T>> {
    // heap 안의 element 개수
    private int heapSize = 0;

    // heap의 internal capacity
    private int heapCapacity = 0;

    // heap안의 element를 track할 dynamic list
    private List<T> heap = null;

    // removal을 log(n)으로 줄여주기 위한 value - index를 묶어주는 hashmap
    private Map<T, TreeSet<Integer>> map = new HashMap<>();

    // 우선순의 큐를 생성하고 초가화 (capacity는 1)
    public PriorityQueue() {
        this(1);
    }

    // initial capacity로 우선순위 큐를 생성
    public PriorityQueue(int size) {
        heap = new ArrayList<>(size);
    }

    // 이미 있는 List를 가지고 heapify - O(n)하여 우선순위 큐를 생성한다
    public PriorityQueue(T[] elems) {
        heapSize = heapCapacity = elems.length;
        heap = new ArrayList<T>(heapCapacity);

        // 모든 element를 heap에 담는다.
        for (int i = 0; i < heapSize; i ++) {
            mapAdd(elems[i], i);
            heap.add(elems[i]);
        }

        // heapify 과정 - O(n)
        for (int i = Math.max(0, (heapSize / 2) - 1); i >= 0; i --) {
            sink(i);
        }
    }

    // 우선순위 큐 생성. O(nlog(n))
    public PriorityQueue(Collection<T> elems) {
        this(elems.size());
        for (T elem: elems) add(elem);
    }

    // heap을 clear - O(n)
    public void clear() {
        for (int i = 0; i < heapCapacity; i ++) {
            heap.set(i, null);
        }
        heapSize = 0;
        map.clear();
    }

    public int size() {
        return heapSize;
    }

    // 우선순위 큐에서 가장 낮은 우선순위를 가지는 값을 return
    // 우선순위 큐가 비었다면 null을 return 한다.
    public T peak() {
        if (heap.isEmpty()) return null;
        return heap.get(0);
    }

    // heap의 root를 삭제 - O(log(n))
    public T poll() {
        return removeAt(0);
    }

    // element가 heap에 들어있는지 확인 - O(1)
    public boolean contains(T elem) {
        if (elem == null) return false;
        return map.containsKey(elem);
    }

    // 우선순위 큐에 element를 삽입 - O(log(n))
    public void add(T elem) {
        if (elem == null) throw new IllegalArgumentException();

        if (heapSize < heapCapacity) {
            heap.set(heapSize, elem);
        } else {
            heap.add(elem);
            heapCapacity++;
        }

        mapAdd(elem, heapSize);

        swim(heapSize);
        heapSize++;
    }

    // node i <= node j 인지 테스트. - O(1)
    private boolean less(int i, int j) {
        T node1 = heap.get(i);
        T node2 = heap.get(j);
        return node1.compareTo(node2) <= 0;
    }

    // Bottom up swim - O(log(n))
    private void swim(int k) {
        // 다음 parent node의 index
        int parent = (k - 1) / 2;

        // root에 닿거나 parent보다 작을때까지 swim
        while (k > 0 && less(k, parent)) {
            // k를 parent와 swap
            swap(parent, k);
            k = parent;

            // next parent node의 index
            parent = (k - 1) / 2;
        }
    }

    // Top down sink - O(log(n))
    private void sink(int k) {
        while (true) {
            int left = 2 * k + 1; // left node
            int right = 2 * k + 2; // Right node
            int smallest = left; // left가 두 children 중 작은 node라고 가정

            // left 혹은 right 둘 중 누가 더 작은지 확인 -> right가 더 작으면 작은 쪽이 right에 가도록 함.
            if (right < heapSize && less(right, left)) smallest = right;

            // tree의 바깥으로 나가거나, k를 더 이상 sink할 수 없게되면 break
            if (left >= heapSize || less(k, smallest)) break;

            swap(smallest, k);
            k = smallest;
        }
    }

    // 두 node를 swap. i와 j는 valid하다고 가정 - O(1)
    private void swap(int i, int j) {
        T i_elem = heap.get(i);
        T j_elem = heap.get(j);

        heap.set(i, j_elem);
        heap.set(j, i_elem);

        mapSwap(i_elem, j_elem, i, j);
    }

    // heap의 특정 element를 제거 - O(log(n))
    public boolean remove(T elem) {
        if (elem == null) return false;

        Integer index = mapGet(elem);
        if (index != null) removeAt(index);
        return index != null;
    }

    // 특정 index의 node를 제거. O(log(n))
    private T removeAt(int i) {
        if (heap.isEmpty()) return null;

        heapSize--;
        T removedData = heap.get(i);
        swap(i, heapSize);

        // 값을 지운다
        heap.set(heapSize, null);
        mapRemove(removedData, heapSize);

        // 마지막 element를 지운다
        if (i == heapSize) return removedData;

        T elem = heap.get(i);

        // element를 sink.
        sink(i);

        // sinking이 안된다면 swim
        if (heap.get(i).equals(elem)) swim(i);

        return removedData;
    }

    // heap이 min heap인지 재귀적으로 check.
    public boolean isMinHeap(int k) {
        // heap의 bound 바깥에 있다면 true를 return
        if (k >= heapSize) return true;

        int left = 2 * k + 1;
        int right = 2 * k + 2;

        // 현재 노드인 k가 양쪽 자식 노드보다 작은 것을 확인.
        // invalid한 heap일 경우 false를 return
        if (left < heapSize && !less(k, left)) return false;
        if (right < heapSize && !less(k, right)) return false;

        // 자식 노드를 재귀
        return isMinHeap(left) && isMinHeap(right);
    }

    // node value를 map의 index로 추가.
    private void mapAdd(T value, int index) {
        TreeSet<Integer> set = map.get(value);

        // map에 새 값을 insert
        if (set == null) {
            set = new TreeSet<>();
            set.add(index);
            map.put(value, set);

            // map안에 이미 값이 존재하는 경우
        } else {
            set.add(index);
        }
    }

    // 주어진 값의 index를 없앤다 - O(log(n))
    private void mapRemove(T value, int index) {
        TreeSet<Integer> set = map.get(value);
        set.remove(index);
        if (set.size() == 0) map.remove(value);
    }

    // 주어진 값의 index position을 가져옴. -> 값의 index가 앖다면 heap의 가장 높은 index가 return된다.
    private Integer mapGet(T value) {
        TreeSet<Integer> set = map.get(value);
        if (set != null) return set.last();
        return null;
    }

    // map안에서 두 node의 index를 swap
    private void mapSwap(T val1, T val2, int val1Index, int val2Index) {
        Set<Integer> set1 = map.get(val1);
        Set<Integer> set2 = map.get(val2);

        set1.remove(val1Index);
        set2.remove(val2Index);

        set1.add(val2Index);
        set2.add(val1Index);
    }

    @Override
    public String toString() {
        return heap.toString();
    }
}
```
