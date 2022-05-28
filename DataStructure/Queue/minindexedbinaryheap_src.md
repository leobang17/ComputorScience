```java
// Degree가 2인 Binary heap.
// MinIndexedDHeap을 상속해서 degree가 2인 생성자를 만든다.
public class MInIndexedBinaryHeap<T extends Comparable<T>> extends MinIndexedDHeap<T>{
    public MInIndexedBinaryHeap(int maxSize) {
        super(2, maxSize);
    }
}
```
