
# ArrayList - 数组

`ArrayList的底层结构是一个数组。`

## 1. 源码分析

### 1.1 核心结构

ArrayList的底层结构是一个数组。

```java
transient Object[] elementData;
```

### 1.2 构造函数

ArrayList有三种方式来初始化，构造方法源码如下：

1. 无参构造函数

```java
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
    *   默认构造函数，构造一个空列表(无参数构造)
    */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```

这里需要注意：以`无参数`构造方法创建 ArrayList 时，实际上`初始化赋值的是一个空数组`。当真正对数组`进行添加元素`操作时，才`真正分配容量`( 即向数组中添加第一个元素时: add(Element) ，数组容量扩为10 )。

2. 参数为int的构造函数

```java
    /**
     * 带初始容量参数的构造函数。（用户自己指定容量）
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```

3. 参数为Collection的构造函数

//TODO: Collection

```java
    /**
    *构造包含指定collection元素的列表，将这些元素复制到ArrayList中
    */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

### 1.3 add

```java
    public boolean add(E e) {
        //判断是否要扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```

add(E e)方法调用了ensureCapacityInternal(int minCapacity)，如果需要扩容，将在这个函数进行处理。

```java
    private void ensureCapacityInternal(int minCapacity) {
        //判断是否要扩容；如果是初始化后添加第一个元素，则容量设置为10
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
```

调用了calculateCapacity(Object[] elementData, int minCapacity)方法。`minCapacity为`调用add(E e)`增加一个元素后`,`elementData[]的最小容量`。

calculateCapacity(Object[] elementData, int minCapacity)计算更加适合elementData[]的minCapcity。

```java
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    private static final int DEFAULT_CAPACITY = 10;

    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            //当ArrayList初始化后，add第一个元素，容量为DEFAULT_CAPACITY = 10
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
```

ensureExplicitCapacity(int minCapacity)用刚刚计算出来的minCapacity判断是否扩容

```java
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            //扩容
            grow(minCapacity);
    }
```

判断特殊情况，否则扩容为`1.5倍`。

```java

    @Native public static final int   MAX_VALUE = 0x7fffffff;
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //扩容为1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //特殊情况进行处理
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        //调用native方法进行复制
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

特殊情况进行处理：

```java
    //Integer.MAX_VALUE:
    @Native public static final int   MAX_VALUE = 0x7fffffff;

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

### 1.4 remove

```java
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```

remove()比较直观，这里的`elementData[--size] = null;`注意一下。

为什么要置为null呢？这里是为了避免`对象游离`。

Java 的垃圾收集策略是`回收所有无法被访问的对象的内存`。在我们对 remove() 的实现中，被弹出的元素的`引用`仍然存在于数组中。这个元素实际上已经是一个孤儿了——它永远也不会再被访问了，但 Java 的垃圾收集器没法知道这一点，除非该引用被覆盖。即使用例已经不再需要这个元素了，数组中的引用仍然可以让它继续存在。这种情况（保存一个不需要的对象的引用）称为游离。在这里，避免对象游离很容易，只需将被弹出的数组元素的值设为 null 即可，这将覆盖无用的引用并使系统可以在用例使用完后回收它的内存。

### 1.5 size

```java
    public int size() {
        return size;
    }
```

size()的源码很简单，要注意的问题是：

- java 中的 `length 属性`是针对`数组`说的,比如说你声明了一个数组,想知道这个数组的长度则用到了 `length 这个属性`.
- java 中的 `length()` 方法是针对`字符串`说的,如果想看这个字符串的长度则用到 length() 这个方法.
- java 中的 `size()` 方法是针对`泛型集合`说的,如果想看这个泛型有多少个元素,就调用此方法来查看!

### 1.6 System.arraycopy() 和 Arrays.copyOf()

Arrays.copyOf()中调用了System.arraycopy()

```java
    public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```

System.arraycopy()是一个native方法【用C、C++实现】

```java
    //src:源数组;srcPos:源数组中的起始位置;
    //desc：目标数组；destPos：目标数组中的起始位置；
    //length：要复制的数组元素的数量；
    public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

**区别：**

1. arraycopy() 需要目标数组，将原数组拷贝到你自己定义的数组里或者原数组，而且可以选择拷贝的起点和长度以及放入新数组中的位置
2. copyOf() 是系统自动在内部新建一个数组，并返回该数组。

### 1.7 ensureCapacity()

ArrayList 源码中有一个 ensureCapacity 方法不知道大家注意到没有，这个方法 ArrayList 内部没有被调用过，所以很显然是提供给用户调用的，那么这个方法有什么作用呢？

```java
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            //扩容
            grow(minCapacity);
    }

```

`最好在 add 大量元素之前用 ensureCapacity 方法，以减少增量重新分配的次数。`

### 1.8 索引异常抛出

`要注意index索引不要越界`。如果index索引越界，则会抛出异常：

```java
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

## 2. 测试用例

```java
    java.util.List<String> list = new ArrayList<>();
        list.add("a");
        list.add("b");
        list.add("c");
        //重写了toString方法，可以输出所有数据
        System.out.println(list);
        //print: [a, b, c]

        //在c和d之间添加一个“zzl”
        list.add(3, "zzl");
        System.out.println(list);
        //print: [a, b, c, zzl]

        //移除元素
        String removeElement = list.remove(0);
        System.out.println("被移除的元素" + removeElement);
        //print: 被移除的元素a
        System.out.println(list);
        //print: [b, c, zzl]

        //替换元素
        String setElement = list.set(2, "demo");
        System.out.println(list);
        //print: [b, c, demo]

        //遍历的方式
        for (String s : list) {
            System.out.print(s);
        }

        Iterator<String> iterator = list.iterator();
        while (iterator.hasNext()){
            System.out.print(iterator.next());
        }

        //注意数组下标IndexOutOfBoundsException
        //System.out.println(list.get(10));
```
