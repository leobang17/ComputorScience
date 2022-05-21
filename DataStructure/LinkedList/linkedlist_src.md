```java
public class DoublyLinkedList<T> implements Iterable <T> {

    private int size = 0;
    private Node <T> head = null;
    private Node<T> tail = null;

    // Internal node class to represent data
    private class Node<T> {
        T data;
        Node<T> prev, next;
        public Node(T data, Node<T> prev, Node<T> next) {
            this.data = data;
            this.next = next;
            this.prev = prev;
        }

        @Override
        public String toString() {
            return data.toString();
        }
    }

    // Empty this linked list, O(n)
    public void clear() {
        Node<T> trav = head;
        while (trav != null) {
            Node<T> next = trav.next;
            trav.prev = trav.next = null;
            trav.data = null;
            trav = next;
        }
        head = tail = trav = null;
        size = 0;
    }

    // Return the size of this linked list.
    public int size() {
        return this.size;
    }

    // Is this linked list empty?
    public boolean isEmpty() {
        return size() == 0;
    }

    // Add an element to the tail of the linked list - O(1)
    public void add(T elem) {
        addLast(elem);
    }

    // linked list의 맨 앞 node에 추가. - O(1)
    public void addFirst(T elem) {
        // the linked list is empty
        if (isEmpty()) {
            head = tail = new Node<T>(elem, null, null);
        } else {
            head.prev = new Node<T>(elem, null, head);
            head = head.prev;
        }
        size ++;
    }

    // linked list의 마지막에 node를 추가 - O(1)
    public void addLast(T elem) {
        if (isEmpty()) {
            head = tail = new Node<>(elem, null, null);
        } else {
            tail.next = new Node<>(elem, tail, null);
            tail = tail.next;
        }
        size ++;
    }

    // head node를 반환 (존재한다면) - O(1)
    public T peekFirst() {
        if (isEmpty()) throw new RuntimeException("Empty List");
        return head.data;
    }

    // tail node의 데이터를 반환 (존재한다면) - O(1)
    public T peekLast() {
        if (isEmpty()) throw new RuntimeException("Empty List");
        return tail.data;
    }

    // linked list의 head 값을 제거 - O(1)
    public T removeFirst() {
        if (isEmpty()) throw new RuntimeException("EmptyList");

        // head의 data를 추출하고 head pointer를 한칸 뒤로 이동
        T data = head.data;
        head = head.next;
        --size;

        // list가 empty라면 tail 역시 null 처리 (head는 1개라면 앞에서 null 처리 됨)
        if(isEmpty()) tail = null;

            // prev node의 메모리를 지워준다.
        else head.prev = null;

        // 삭제된 첫번째 노드의 data를 return
        return data;
    }

    public T removeLast() {
        if (isEmpty()) throw new RuntimeException("EmptyList");

        T data = tail.data;
        tail = tail.prev;
        --size;

        if (isEmpty()) head = null;
            // 메모리 지워주기
        else tail.next = null;

        return data;
    }

    // linked list에서 특정 node를 삭제한다 - O(1)
    private T remove(Node<T> node) {
        // 제거하려고 건넨 node가 head나 tail 일 경우
        if (node.prev == null) return removeFirst();
        if (node.next == null) return removeLast();

        // pointer를 조정해놓는다. (다음노드의 이전 pointer는 삭제할 노드의 이전 pointer로, 이전 node의 다음 pointer는 삭제할 노드의 다음 pointer로)
        node.next.prev = node.prev;
        node.prev.next = node.next;

        // return할 data를 임시로 저장해 놓음
        T data = node.data;

        // Memory cleanup
        node.data = null;
        node = node.prev = node.next = null;

        --size;
        return data;
    }

    // 특정 index의 node를 삭제 - O(n)
    public T removeAt(int index) {
        // index가 유효한지 검증
        if(index < 0 || index >= size) throw new IllegalArgumentException();

        int i;
        Node<T> trav;

        // list의 처음부터 검색
        if (index < size / 2) {
            for (i = 0, trav = head; i != index; i ++)
                trav = trav.next;
            // list의 뒤부터 검색
        } else {
            for (i = size - 1, trav = tail; i != index; i --)
                trav = trav.prev;
        }

        return remove(trav);
    }

    // linked list의 특정 값을 제거 - O(n)
    public boolean remove(Object obj) {
        Node<T> trav = head;

        // null을 searching할 수도
        if (obj == null) {
            for (trav = head; trav != null; trav = trav.next) {
                if (trav.data == null) {
                    remove(trav);
                    return true;
                }
            }
            // non null object를 search
        } else {
            for (trav = head; trav != null; trav = trav.next) {
                if (obj.equals(trav.data)) {
                    remove(trav);
                    return true;
                }
            }
        }
        return false;
    }

    // linked list안에서 특정 value가 어떤 index에 있는지 찾기 - O(n)
    public int indexOf(Object obj) {
        int index = 0;
        Node<T> trav = head;

        if (obj == null) {
            for (trav = head; trav != null; trav = trav.next, index ++) {
                if (trav.data == null) {
                    return index;
                }
            }
        } else {
            for (trav = head; trav != null; trav = trav.next, index ++) {
                if (obj.equals(trav.data)) {
                    return index;
                }
            }
        }

        return -1;
    }

    // linkedlist안에 특정 value가 있는지 확인
    public boolean contains(Object obj) {
        return indexOf(obj) != -1;
    }

    @Override
    public Iterator<T> iterator() {
        return null;
    }
}
```
