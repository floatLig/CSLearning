
# 1. LinkedList： 链表-栈-队列

## 1.1 队列 / 栈 常用API

LinkedList可以用作`链表、栈、队列`；其常用的方法为：

|          |    增加    |     删除     |  查看元素  |
| :------: | :--------: | :----------: | :--------: |
| **队列** | addFirst() | removeLast() | getFirst() |
|  **栈**  |   push()   |    pop()     |   peek()   |

## 1.2 线程不安全

LinkedList`不是线程安全的`，如果想使LinkedList变成线程安全的，可以调用静态类Collections类中的synchronizedList方法：

```java
List list=Collections.synchronizedList(new LinkedList(...));
```

# 2. 源码分析

## 2.1 底层结构

LinkedList的底层结构是链表。核心属性有`Node<E> first`和`和Node<E> last`。 Node是`内部类`，详细代码为：

```java
    //属性，指向链表头节点
    transient Node<E> first;
    //属性，指向链表尾结点
    transient Node<E> last;

    //内部类，Node结点
    private static class Node<E> {
        //值
        E item;
        //双向链表
        Node<E> next;
        Node<E> prev;

        //没有无参构造函数，初始Node必须初始化三个参数
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

## 2.2 构造函数

无参构造函数

```java
    public LinkedList() {
    }
```

参数为Collection的构造函数

```java
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```

addAll()的代码为：

```java
    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }

    public boolean addAll(int index, Collection<? extends E> c) {
        //检查参数index
        checkPositionIndex(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        //检查参数c
        if (numNew == 0)
            return false;

        //得到插入位置的前驱节点和后继节点，通过判断，正确初始化pred和succ
        Node<E> pred, succ;
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }

        //插入结点，如果有需要，重置first和last
        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }
```

## 2.3 在链表尾（last）加入元素

```java
    public boolean add(E e) {
        linkLast(e);
        return true;
    }

    public void addLast(E e) {
        linkLast(e);
    }

    public boolean offer(E e) {
        return add(e);
    }

    public boolean offerLast(E e) {
        addLast(e);
        return true;
    }
```

核心的linkLast()如下：

```java
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        //重置last指向的位置
        last = newNode;
        //如果有必要，也需要重置first指向的位置
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }

```

## 2.4 在链表头 / 栈 / 队列 加入元素

在链表头 / 队列 加入元素

```java
    public void addFirst(E e) {
        linkFirst(e);
    }

    public boolean offerFirst(E e) {
        addFirst(e);
        return true;
    }
```

向 栈 加入元素

```java
    public void push(E e) {
        addFirst(e);
    }
```

核心的linkFirst()如下：

```java
    private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }
```

其他：

```java

```

## 2.5 链表尾 / 队列 删除元素

```java
    public E removeLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }

    public E pollLast() {
        final Node<E> l = last;
        return (l == null) ? null : unlinkLast(l);
    }
```

核心的unlinkLast如下：

```java
    private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev;
        l.item = null;
        l.prev = null; // help GC
        last = prev;
        if (prev == null)
            first = null;
        else
            prev.next = null;
        size--;
        modCount++;
        return element;
    }
```

## 2.6 链表头 / 栈 删除元素

```java
    //如果linkList用作栈，弹出使用pop()，不用其他，这样子可以明确linkList的作用是栈
    public E pop() {
        return removeFirst();
    }

    //一般不要写remove()容易混，写成removeFirst()好一点
    public E remove() {
        return removeFirst();
    }

    public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }

    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }

    public E pollFirst() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }
```

核心的unlinkFirst()为：

```java
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
```

## 2.7 获取头元素

```java
    public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }

    public E peekFirst() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
     }

     public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }
```

## 2.8 获取尾元素

```java
    public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
    }

    public E peekLast() {
        final Node<E> l = last;
        return (l == null) ? null : l.item;
    }
```

## 2.9 参数为index / Object

```java
    public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }

    public boolean contains(Object o) {
        return indexOf(o) != -1;
    }

    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
```

# 测试用例

```java
public static void main(String[] args) {
        LinkedList<String> linkedList = new LinkedList<>();

        //增加
        linkedList.add("a");
        linkedList.add("b");
        linkedList.add("c");
        System.out.println(linkedList);
        //print: [a, b, c]

        //删除
        //具有栈的功能：String first = linkedList.pop();
        String first = linkedList.removeFirst();
        System.out.println("被移除的第一个元素：" + first);
        //print: 被移除的第一个元素：a

        String last = linkedList.removeLast();
        System.out.println("被移除的最后一个元素" + last);
        //print: 被移除的最后一个元素c

        //删除所有的元素
        linkedList.clear();

        //添加元素
        //在列表开头插入元素
        linkedList.push("b");
        linkedList.addFirst("a");
        //在列表结尾插入元素
        linkedList.addLast("c");
        linkedList.add("d");

        System.out.println(linkedList);
        //print: [a, b, c, d]

        if (!linkedList.isEmpty()) {
            System.out.println("获取linkedList的第一个元素"+ linkedList.getFirst());
            System.out.println("获取linkedList的最后一个元素"+ linkedList.getLast());
            //print: 获取linkedList的第一个元素a
            //print: 获取linkedList的最后一个元素d
        }

    }
```