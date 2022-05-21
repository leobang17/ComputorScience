```java
public class Stack <T> implements Iterable <T> {
    // Java.util에서 제공하는 LinkedList (양방향 - doubly)
    private LinkedList<T> list = new LinkedList<>();

    // 빈 stack 생성자
    public Stack() {}

    // inital element와 함께 stack 생성
    public Stack(T firstElem) {
        push(firstElem);
    }

    // stack의 element 수를 return
    public int size() {
        return list.size();
    }

    // stack이 비었는지 확인
    public boolean isEmpty() {
        return size() == 0;
    }

    // stack에 push
    public void push(T elem) {
        list.addLast(elem);
    }

    // stack의 마지막을 pop. 없을 경우 throw error
    public T pop() {
        if (isEmpty())
            throw new EmptyStackException();
        return list.removeLast();
    }

    // stack의 마지막 element를 꺼내본다.
    public T peek() {
        if (isEmpty()) throw new EmptyStackException();
        return list.peekLast();
    }

    @Override
    public Iterator<T> iterator() {
        return list.iterator();
    }
}
```
