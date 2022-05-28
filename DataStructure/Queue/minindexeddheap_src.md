```java
// Degree가 D인 heap tree.
public class MinIndexedDHeap<T extends Comparable<T>> {
    // heap안에 얼마나 element가 있는지
    private int sz;

    // heap이 담을 수 있는 최대 element 개수
    private final int N;

    // tree의 degree 중 최대 degree (차수 - 자식 노드의 수)
    private final int D;

    // 각 node의 child/parent index를 확인하기 위한 array
    private final int[] child, parent;

    // Position Map (pm)
    // Key Index (ki)를 pq에서 어느 index에 위치해있는지 mapping 해줌
    public final int[] pm;

    // Inverse Map (im)
    // priority queue의 index 위치를 통해 Key Index 값을 구할 수 있다.
    // im 과 pm은 inversed 되어있다. (bi-directoinal로 서로가 서로를 조회할 수 있게 하기 위한 것)
    // pm[im[i]] = im[pm[i]] = i
    public final int[] im;

    // key와 관련된 value들. 이 array는 Key Index (ki) 를 통해 indexing이 가능하다.
    public final Object[] values;

    // 차수가 D인 heap tree를 maxSize로 초기화
    public MinIndexedDHeap(int degree, int maxSize) {
        if (maxSize <= 0) throw new IllegalArgumentException("maxSize <= 0");

        D = Math.max(2, degree);
        N = Math.max(D + 1, maxSize);

        im = new int[N];
        pm = new int[N];
        child = new int[N];
        parent = new int[N];
        values = new Object[N];

        for (int i = 0; i < N; i++) {
            parent[i] = (i - 1) / D;
            child[i] = i * D + 1;
            pm[i] = im[i] = -1;
        }
    }

    public int size() {
        return sz;
    }

    public boolean isEmpty() {
        return size() == 0;
    }

    public boolean contains(int ki) {
        keyInBoundsOrThrow(ki);
        return pm[ki] != -1;
    }

    public int peekMinKeyIndex() {
        isNotEmptyOrThrow();
        return im[0];
    }

    public int pollMinKeyIndex() {
        int minKi = peekMinKeyIndex();
        delete(minKi);
        return minKi;
    }

    @SuppressWarnings("unchecked")
    public T peekMinValue() {
        isNotEmptyOrThrow();
        return (T) values[im[0]];
    }

    public T pollMinValeu() {
        T minValue = peekMinValue();
        delete(peekMinKeyIndex());
        return minValue;
    }

    public void insert(int ki, T value) {
        if (contains(ki)) throw new IllegalArgumentException("index already exists; received: " + ki);
        valueNotNullOrThrow(value);
        pm[ki] = sz;
        im[sz] = ki;
        values[ki] = value;
        swim(sz++);
    }

    @SuppressWarnings("unchecked")
    public T valueOf(int ki) {
        keyExistsOrThrow(ki);
        return (T) values[ki];
    }

    @SuppressWarnings("unchecked")
    public T delete(int ki) {
        keyExistsOrThrow(ki);
        final int i = pm[ki];
        swap(i, --sz);
        sink(i);
        swim(i);
        T value = (T) values[ki];
        values[ki] = null;
        pm[ki] = -1;
        im[sz] = -1;
        return value;
    }

    @SuppressWarnings("unchecked")
    public T update(int ki, T value) {
        keyExistsAndValueNotNullOrThrow(ki, value);
        final int i = pm[ki];
        T oldValue = (T) values[ki];
        values[ki] = value;
        sink(i);
        swim(i);
        return oldValue;
    }

    // 'ki' - 'value' 의 value를 줄인다.
    public void decrease(int ki, T value) {
        keyExistsAndValueNotNullOrThrow(ki, value);
        if (less(value, values[ki])) {
            values[ki] = value;
            swim(pm[ki]);
        }
    }

    // 'ki' - 'value' 의 value를 증가시킨다.
    public void increase(int ki, T value) {
        keyExistsAndValueNotNullOrThrow(ki, value);
        if (less(values[ki], value)) {
            values[ki] = value;
            sink(pm[ki]);
        }
    }

    private void sink(int i) {
        for (int j = minChild(i); j != -1; ) {
            swap(i, j);
            i = j;
            j = minChild(i);
        }
    }

    private void swim(int i) {
        while (less(i, parent[i])) {
            swap(i, parent[i]);
            i = parent[i];
        }
    }

    private int minChild(int i) {
        int index = -1, from = child[i], to = Math.min(sz, from + D);
        for (int j = from; j < to; j++) {
            if (less(j, i)) index = i = j;
        }
        return index;
    }

    private void swap(int i, int j) {
        pm[im[j]] = i;
        pm[im[i]] = j;
        int tmp = im[i];
        im[i] = im[j];
        im[j] = tmp;
    }

    private boolean less(int i, int j) {
        return ((Comparable<? super T>) values[im[i]]).compareTo((T) values[im[j]]) < 0;
    }

    private boolean less(Object obj1, Object obj2) {
        return ((Comparable<? super T>) obj1).compareTo((T) obj2) < 0;
    }

    private void isNotEmptyOrThrow() {
        if (isEmpty()) throw new NoSuchElementException("Priority Queue underflow");
    }

    private void keyExistsAndValueNotNullOrThrow(int ki, Object value) {
        keyExistsOrThrow(ki);
        valueNotNullOrThrow(value);
    }

    private void keyExistsOrThrow(int ki) {
        if (!contains(ki)) throw new NoSuchElementException("Index does not exist; received: " + ki);
    }

    private void valueNotNullOrThrow(Object value) {
        if (value == null) throw new IllegalArgumentException("value cannot be null");
    }

    private void keyInBoundsOrThrow(int ki) {
        if (ki < 0 || ki >= N) {
            throw new IllegalArgumentException("Key index out of bounds; received: " + ki);
        }
    }

    public boolean isMinHeap() {
        return isMinHeap(0);
    }

    private boolean isMinHeap(int i) {
        int from = child[i], to = Math.min(sz, from + D);
        for (int j = from; j < to; j++) {
            if(!less(i, j)) return false;
            if (!isMinHeap(j)) return false;
        }

        return true;
    }
}
```
