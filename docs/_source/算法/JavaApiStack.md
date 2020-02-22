# Stack

## 1. 简介

LinkedListed、Stack都可以实现栈的功能。

与LinkedListed不同的是，`Stack是继承Vector，底层是数组结构，且线程同步`。

```java
class Stack<E> extends Vector<E>
```

## 2. 源码分析

### 2.1 push

```java
    public E push(E item) {
        //调用Vector的addElement(E obj)
        addElement(item);

        return item;
    }
```

这里注意，addElement(E obj)加了`synchronized`，是线程同步的。

addElement(E obj)方法`是Vector的方法`。

```java
    public synchronized void addElement(E obj) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = obj;
    }

    private void ensureCapacityHelper(int minCapacity) {
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //这里的capacityIncrement，是创建实例Vector指定的；如果没有指定，则默认为0
        //可以看到，如果没有指定capacityIncrement，扩容为原来的2倍
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }
```

### 2.2 pop

```java
    public synchronized E pop() {
        E       obj;
        int     len = size();

        obj = peek();
        removeElementAt(len - 1);

        return obj;
    }
```

这里也是加了`线程同步`。调用的removeElement(int index)`是Vector的方法`。

```java
    public synchronized void removeElementAt(int index) {
        modCount++;
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        else if (index < 0) {
            throw new ArrayIndexOutOfBoundsException(index);
        }
        int j = elementCount - index - 1;
        if (j > 0) {
            System.arraycopy(elementData, index + 1, elementData, index, j);
        }
        elementCount--;
        elementData[elementCount] = null; /* to let gc do its work */
    }
```

### 2.3 peek

```java
    public synchronized E peek() {
        int     len = size();

        if (len == 0)
            throw new EmptyStackException();
        return elementAt(len - 1);
    }
```

这里也是加了线程同步。