```java
// key-value pair를 가진 각 item을 뜻한다. (hash table에 집어넣을)
class Entry<K, V> {
    int hash;
    K key; V value;

    public Entry(K key, V value) {
        this.key = key;
        this.value = value;
        // .hashcode(): 해시 알고리즘에 의해 생성된 Integer 값. Entry 객체에 cache 해놓자.
        this.hash = key.hashCode();
    }

    // Object의 equals method를 override하는 것이 아님.
    public boolean equals(Entry <K, V> other) {
        // hash가 다르다면 무조건 다른 값이므로 (.hashcode()으로 생성한 값)
        if (hash != other.hash) return false;
        return key.equals(other.key);
    }

    @Override
    public String toString() {
        return key + " => " + value;
    }
}

@SuppressWarnings("unchecked")
public class HashTableSeperateChaining<K, V> implements Iterable<K> {
    private static final int DEFAULT_CAPACITY = 3;
    private static final double DEFAULT_LOAD_FACTOR = 0.75;

    // table size가 얼마나 커질 수 있는지
    private double maxLoadFactor;

    // threshold -> 언제 resize해야하는지 알려줌
    private int capacity, threshold, size = 0;
    private LinkedList<Entry<K, V>>[] table;

    public HashTableSeperateChaining(int capacity) {
        this(capacity, DEFAULT_LOAD_FACTOR);
    }

    public HashTableSeperateChaining(int capacity, double maxLoadFactor) {
        if (capacity < 0) {
            throw new IllegalArgumentException("Illegal capacity!");
        }
        if (maxLoadFactor <= 0 || Double.isNaN(maxLoadFactor) || Double.isInfinite(maxLoadFactor)) {
            throw new IllegalArgumentException("Illegal maxLoadFactor");
        }

        this.maxLoadFactor = maxLoadFactor;
        this.capacity = Math.max(DEFAULT_CAPACITY, capacity);
        threshold = (int) (this.capacity * maxLoadFactor);
        table = new LinkedList[this.capacity];
    }

    // hash table안에 있는 element의 개수를 return
    public int size() {
        return size;
    }

    // hash value를 index로 변환.
    // 기본적으로 음수를 제거하고, [0, capacity) 의 domain을 가지는 hash value로 mapping 한다.
    private int normalizeIndex(int keyHash) {
        return (keyHash & 0x7FFFFFFF) % capacity;
    }

    // hash table의 모든 content를 clear
    public void clear() {
        Arrays.fill(table, null);
        size = 0;
    }

    public boolean containsKey(K key) {
        return hasKey(key);
    }

    // hash table안에 key가 있는지 확인
    public boolean hasKey(K key) {
        int bucketIndex = normalizeIndex(key.hashCode());
        return bucketSeekEntry(bucketIndex, key) != null;
    }

    public V put(K key, V value) {
        return insert(key, value);
    }

    public V add(K key, V value) {
        return insert(key, value);
    }

    public V insert(K key, V value) {
        if (key == null) throw new IllegalArgumentException("Null key");
        Entry<K, V> newEntry = new Entry<>(key, value);
        int bucketIndex = normalizeIndex(newEntry.hash);
        return bucketInsertEntry(bucketIndex, newEntry);
    }

    // key의 value를
    // value가 null일 경우 null을 return하고, key가 존재하지 않을 경우에도 null을 return
    public V get(K key) {
        if (key == null) {
            return null;
        }
        int bucketIndex = normalizeIndex(key.hashCode());
        Entry<K, V> entry = bucketSeekEntry(bucketIndex, key);
        if (entry != null) return entry.value;
        return null;
    }

    // 주어진 bucket의 entry를 삭제함.
    private V bucketRemoveEntry(int bucketIndex, K key) {
        // entry를 찾고
        Entry<K, V> entry = bucketSeekEntry(bucketIndex, key);

        // entry가 있으면
        if (entry != null) {
            // table에서 해당 bucket을 찾고 -> remove -> size 줄이기.
            LinkedList<Entry<K, V>> links = table[bucketIndex];
            links.remove(entry);
            --size;
            return entry.value;
        } else {
            return null;
        }
    }

    // 주어진 bucket에 entry를 삽입
    // 만약 이미 bucket에 값이 존재한다면 update해준다.
    private V bucketInsertEntry(int bucketIndex, Entry<K, V> entry) {
        LinkedList<Entry<K, V>> bucket = table[bucketIndex];
        if (bucket == null) {
            table[bucketIndex] = bucket = new LinkedList<>();
        }

        Entry<K, V> existentEntry = bucketSeekEntry(bucketIndex, entry.key);
        if (existentEntry == null) {
            bucket.add(entry);
            if (++size > threshold) resizeTable();
            return null;  // previous entry가 없었다는 것을 null로 알려준다.
        } else {
            V oldVal = existentEntry.value;
            existentEntry.value = entry.value;
            return oldVal;
        }
    }

    // 주어진 bucket의 특정 entry를 찾아서 반환함. 없다면 null return
    private Entry<K, V> bucketSeekEntry(int bucketIndex, K key) {
        if (key == null) return null;
        LinkedList<Entry<K, V>> bucket = table[bucketIndex];

        if (bucket == null) return null;
        for (Entry<K, V> entry: bucket) {
            if (entry.key.equals(key)) return entry;
        }
        return null;
    }


    // entry가 담긴 bucket의 table을 resize 해준다.
    private void resizeTable() {
        // capacity를 2배 늘리고 그에 맞는 새 table을 생성 (linkedlist로 이루어진)
        capacity *= 2;
        threshold = (int) (capacity * maxLoadFactor);

        LinkedList<Entry<K, V>>[] newTable = new LinkedList[capacity];

        // 기존의 table을 loop하면서 기존 table안에 있던 entry들 새 table로 rehash
        for (int i = 0; i < table.length; i++) {
            if (table[i] != null) {
                // 각 table의 bucket (linkedList) loop
                for (Entry<K, V> entry : table[i]) {
                    int bucketIndex = normalizeIndex(entry.hash);
                    LinkedList<Entry<K, V>> bucket = newTable[bucketIndex];
                    if (bucket == null) newTable[bucketIndex] = bucket = new LinkedList<>();
                    bucket.add(entry);
                }

                // Memory leak을 방지하고 Garbage collector
                table[i].clear();
                table[i] = null;
            }
        }
        this.table = newTable;
    }

    // hash table에 들어있는 key 값들을 모두 list에 담아 return
    public List<K> keys() {
        List<K> keys = new ArrayList<>(size());
        for (LinkedList<Entry<K, V>> bucket: table) {
            if (bucket != null) {
                for (Entry<K, V> entry: bucket) {
                    keys.add(entry.key);
                }
            }
        }
        return keys;
    }

    public List<V> values() {
        List<V> values = new ArrayList<>(size());
        for (LinkedList<Entry<K, V>> bucket: table) {
            if (bucket != null) {
                for (Entry<K, V> entry: bucket) {
                    values.add(entry.value);
                }
            }
        }
        return values;
    }

    // map안의 key를 iterate하는 iterator를 return
    @Override
    public Iterator<K> iterator() {
        final int elemCount = size();
        return new Iterator<K>() {
            int bucketIndex = 0;
            Iterator<Entry<K, V>> bucketIter = (table[0] == null) ? null : table[0].iterator();

            @Override
            public boolean hasNext() {
                // iterating 도중 item이 추가되거나 제거됨.
                if (elemCount != size) throw new ConcurrentModificationException();

                // iterator가 없거나, 현재 iterator가 비어있으면
                if (bucketIter == null || !bucketIter.hasNext()) {
                    // valid한 iterator가 나올 때까지 다음 bucket을 찾는다
                    while (++bucketIndex < capacity) {
                        if (table[bucketIndex] != null) {
                            // iterator가 실제로 element를 가지고 있는지 확인
                            Iterator<Entry<K, V>> nextIter = table[bucketIndex].iterator();
                            if (nextIter.hasNext()) {
                                bucketIter = nextIter;
                                break;
                            }
                        }
                    }

                }
                return bucketIndex < capacity;
            }

            @Override
            public K next() {
                return bucketIter.next().key;
            }

            @Override
            public void remove() {
                throw new UnsupportedOperationException();
            }
        };
    }
}
```
