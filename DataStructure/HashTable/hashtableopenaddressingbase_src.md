```java
// Hash Table: Hash Collision을 Open Addressing으로 해결할 때의 base case
@SuppressWarnings("unchecked")
public abstract class HashTableOpenAddressingBase<K, V> implements Iterable<K> {

    // table에 얼마나 차있는지 알려주는 지표 (factor)
    protected double loadFactor;
    protected int capacity, threshold, modificationCount;

    // tombstone 포함해서 이용된 bucket의 수
    protected int usedBucket;
    // 순수하게 (k,v) pair가 담긴 bucket의 수
    protected int keyCount;

    // (k, v) pair 의 array
    protected K[] keys;
    protected V[] values;

    // 지워진 자리라는 걸 알려주는 TOMBSTONE
    protected final K TOMBSTONE = (K) (new Object());

    private static final int DEFAULT_CAPACITY = 7;
    private static final double DEFAULT_LOAD_FACTOR = 0.65;

    protected HashTableOpenAddressingBase() {
        this(DEFAULT_CAPACITY, DEFAULT_LOAD_FACTOR);
    }

    protected HashTableOpenAddressingBase(int capacity) {
        this(capacity, DEFAULT_LOAD_FACTOR);
    }

    protected HashTableOpenAddressingBase(int capacity, double loadFactor) {
        if (capacity <= 0) throw new IllegalArgumentException("Illegal capacity: " + capacity);

        if (loadFactor <= 0 || Double.isNaN(loadFactor) || Double.isInfinite(loadFactor))
            throw new IllegalArgumentException("Illegal loadFactor: " + loadFactor);

        this.loadFactor = loadFactor;
        this.capacity = Math.max(DEFAULT_CAPACITY, capacity);
        adjustCapacity();
        threshold = (int) (this.capacity * loadFactor);

        // array 자리 생성.
        keys = (K[]) new Object[this.capacity];
        values = (V[]) new Object[this.capacity];
    }

    // 실제로 probing을 어떻게 구현할 건지에 따라 상속하여 구현하면 됨.
    // open addressing scheme으로 무엇을 선택하는지에 따라 구현이 달라짐.
    protected abstract void setUpProbing(K key);

    protected abstract int probe(int x);

    // hash table의 capacity를 늘려주는 작업.
    // hash table의 size는 probing function의 funcionality에 따라 달라지기 때문에 상속해서 구현해야한다.
    protected abstract void adjustCapacity();

    // hash table의 capacity를 늘여준다.
    protected void increaseCapacity() {
        capacity = (2 * capacity) + 1;
    }

    public void clear() {
        for (int i = 0; i < capacity; i++) {
            keys[i] = null;
            values[i] = null;
        }
        keyCount = usedBucket = 0;
        modificationCount++;
    }

    // hash table안에 실제로 들어있는 key의 개수를 반환한다.
    public int size() {
        return keyCount;
    }

    // hash table의 capacity를 return
    public int getCapacity() {
        return capacity;
    }

    // hash table의 key들을 list에 담아서 return
    public List<K> keys() {
        List<K> hashtableKeys = new ArrayList<>(size());
        for (int i = 0; i < capacity; i++) {
            if (keys[i] != null && keys[i] != TOMBSTONE) hashtableKeys.add(keys[i]);
        }
        return hashtableKeys;
    }

    // hash table의 모든 value들을 list에 담아 return
    public List<V> values() {
        List<V> hashtableValues = new ArrayList<>(size());
        for (int i = 0; i < capacity; i++) {
            if (values[i] != null && values[i] != TOMBSTONE) hashtableValues.add(values[i]);
        }
        return hashtableValues;
    }

    // hash-table의 size를 2배로 늘인다.
    protected void resizeTable() {
        increaseCapacity();
        adjustCapacity();

        threshold = (int) (capacity * loadFactor);

        K[] oldKeyTable = (K[]) new Object[capacity];
        V[] oldValueTalbe = (V[]) new Object[capacity];

        // key table pointer swap을 수행
        K[] keyTableTmp = keys;
        keys = oldKeyTable;
        oldKeyTable = keyTableTmp;

        // value table pointer swap을 수행
        V[] valueTableTmp = values;
        values = oldValueTalbe;
        oldValueTalbe = valueTableTmp;

        // key count와 bucket used를 reset; 모든 (k, v)를 새 hash-table에 re-insert할 것이므로
        keyCount = usedBucket = 0;

        for (int i = 0; i < oldKeyTable.length; i++) {
            if (oldKeyTable[i] != null && oldKeyTable[i] != TOMBSTONE) {

            }
            oldKeyTable[i] = null;
            oldValueTalbe[i] = null;
        }
    }

    // hash value를 index로 convert한다.
    // 이 과정은 근본적으로 음수를 제거하고, hash value를 [0, capacity)의 domain을 가진 정수로 mapping 함.
    protected final int normalizeIndex(int keyHash) {
        return (keyHash & 0x7FFFFFFF) % capacity;
    }

    // a와 b의 최대공약수 (GCD - Greatest Common Denominator)를 찾는다
    protected static final int gcd(int a, int b) {
        if (b == 0) return b;
        return gcd(b, a % b);
    }

    // (k, v) pair를 hash table에 insert.
    // 이미 있는 (k, v) pair라면 값을 update 해준다.
    public V insert(K key, V value) {
        if (key == null) throw new IllegalArgumentException("Null key");
        if (usedBucket >= threshold) resizeTable();

        setUpProbing(key);

        final int offset = normalizeIndex(key.hashCode());

        // i = 현재 probing 중인 index
        // j = 처음 만난 tombstone의 자리 (안만났으면 -1)
        for (int i = 0, j = -1, x = 1; ; i = normalizeIndex(offset + probe(x++))) {
            // 현재 조회한 slot이 이전에 삭제된 곳인가. (tombstone 기록장이 기록이 되어있으면 추가 안함)
            if (keys[i] == TOMBSTONE) {
                if (j == -1) j = i;
                // current cell이 이미 key를 가지고 있음.
            } else if (keys[i] != null) {
                // 우리가 찾는 key가 맞으면 update, 아니면 건너뛴다.
                if (keys[i].equals(key)) {
                    V oldValue = values[i];
                    // 맞는데, 우리가 tombstone을 지나오지 않았으면 그 자리인 i에 값 update
                    if (j == -1) {
                        values[i] = value;
                        // key값 맞는데 우리가 tombstone을 한 개라도 지나왔으면 i자리는 tombstone으로 채워주고, j자리 (tombstone이 있던 곳)에 값 insert
                    } else {
                        keys[i] = TOMBSTONE;
                        values[i] = null;
                        keys[j] = key;
                        values[j] = value;
                    }
                    modificationCount++;
                    return oldValue;
                }

                // current cell이 null이라서 insertion/update가 발생할 수 있음.
            } else {
                // tombstone을 거쳐간 적이 없음.
                if (j == -1) {
                    usedBucket++;
                    keyCount++;
                    keys[i] = key;
                    values[i] = value;

                    // 이전에 tombstone을 거쳐간 기록이 있다.
                    // 현재 찾은 자리인 i에 새 element를 insert하기 보다는, tombstone이 세워진 첫번째 자리에 insert해주자.
                } else {
                    keyCount++;
                    keys[j] = key;
                    values[j] = value;
                }

                modificationCount++;
                return null;
            }
        }
    }

    // 주어진 key값이 hash table에 존재하는지 확인
    public boolean hasKey(K key) {
        if (key == null) throw new IllegalArgumentException("Null key");

        setUpProbing(key);
        final int offset = normalizeIndex(key.hashCode());

        // original hsah
        for (int i = offset, j = -1, x = 1; ; i = normalizeIndex(offset + probe(x++))) {
            // tombstone은 무시하지만, 첫번째 tombstone의 위치는 기록해두어야 함 (optimization을 위해)
            if (keys[i] == TOMBSTONE) {
                if (j == -1) j = i;

                // non-null key를 찾았다면, 우리가 찾는 key인지 확인해줌.
            } else if (keys[i] != null) {
                // 찾는 key가 hash table에 있었다!
                if (keys[i].equals(key)) {

                    // 만약 tombstone을 찍은적이 있다면 최적화를 위해 return 하기 전에 tombstone과 자리를 바꿔주자.
                    if (j != -1) {
                        keys[j] = keys[i];
                        values[j] = values[i];
                        keys[i] = TOMBSTONE;
                        values[i] = null;
                    }
                    return true;
                }

                // hash table 안에 key가 없었음.
            } else return false;
        }
    }

    // 주어진 key의 value 값을 가져온다.
    public V get(K key) {
        if (key == null) throw new IllegalArgumentException("Null key");

        setUpProbing(key);
        final int offset = normalizeIndex(key.hashCode());

        for (int i = offset, j = -1, x = 1; ; i = normalizeIndex(offset + probe(x++))) {
            if (keys[i] == TOMBSTONE) {
                if (j == -1) j = i;
            } else if (keys[i] != null) {
                if (keys[i].equals(key)) {
                    if (j != -1) {
                        keys[j] = keys[i];
                        values[j] = values[i];
                        keys[i] = TOMBSTONE;
                        values[i] = null;
                        return values[j];
                    } else {
                        return null;
                    }
                }
            }
        }
    }

    // 주어진 key 값을 가지는 (k,v) pair를 hash table에서 제거한다.
    public V remove(K key) {
        if (key == null) throw new IllegalArgumentException("Null Key");

        setUpProbing(key);
        final int offset = normalizeIndex(key.hashCode());

        for (int i = offset, j = -1, x = 1; ; i = normalizeIndex(offset + probe(x++))) {
            if (keys[i] == TOMBSTONE) continue;

            if (keys[i] == null) return null;

            if (keys[i].equals(key)) {
                keyCount--;
                modificationCount++;
                V oldValue = values[i];
                keys[i] = TOMBSTONE;
                values[i] = null;
                return oldValue;
            }
        }
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();

        sb.append("{");
        for (int i = 0; i < capacity; i++) {
            if (keys[i] != null && keys[i] != TOMBSTONE) sb.append(keys[i] + " => " + values[i] + ", ");
        }
        sb.append("}");

        return sb.toString();
    }

    @Override
    public Iterator<K> iterator() {
        // iteration 시작되기 전에 지금껏 hash table에 일어난 modification number를 기록한다
        // iterate하는 동안 이 modification count는 변해서는 안된다.
        final int MODIFICATION_COUNT = modificationCount;

        return new Iterator<K>() {
            int index, keysLeft = keyCount;

            @Override
            public boolean hasNext() {
                // table의 content가 이미 alter되었다.
                if (MODIFICATION_COUNT != modificationCount) throw new ConcurrentModificationException();
                return keysLeft != 0;
            }

            @Override
            public K next() {
                while (keys[index] == null || keys[index] == TOMBSTONE) index++;
                keysLeft--;
                return keys[index++];
            }

            @Override
            public void remove() {
                throw new UnsupportedOperationException();
            }
        };
    }
}
```
