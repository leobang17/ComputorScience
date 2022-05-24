```java
// Hash Table: Hash collision 을 open addressing으로 해결하는 방식 (probing function이 quadratic)
// 이 implementation에서 probing function으로는: H(k, x) = h(k) + f(x) mod 2^n을 이용한다.
@SuppressWarnings("unchecked")
public class HashTableQuadraticProbing<K, V> extends HashTableOpenAddressingBase<K, V> {
    public HashTableQuadraticProbing() {
        super();
    }

    public HashTableQuadraticProbing(int capacity) {
        super(capacity);
    }

    public HashTableQuadraticProbing(int capacity, double loadFactor) {
        super(capacity, loadFactor);
    }

    //
    private static int nextPowerOfTwo(int n) {
        return Integer.highestOneBit(n) << 1;
    }

    // Quadratic Probing에는 setup probing이 필요하지 않음.
    @Override
    protected void setUpProbing(K key) {}

    @Override
    protected int probe(int x) {
        // Quadratic probing function (x^2 + x) / 2
        return (x * x + x) >> 1;
    }

    @Override
    protected void adjustCapacity() {
        int pow2 = Integer.highestOneBit(capacity);
        if (capacity == pow2) return;
        increaseCapacity();
    }

    @Override
    protected void increaseCapacity() {
        capacity = nextPowerOfTwo(capacity);
    }
}
```
