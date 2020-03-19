# Comparable 和 Comparator

Comparable 和 Comparator 是 Java 的两个接口。从名字看起来很相似，实际的功能也都是为了`排序`。那这两个有什么区别呢？

## 1. Comparable

Comparable 的源码非常简单，源码只定义了一个方法compareTo(T o).

```java
publicinterface Comparable<T> {
    int compareTo(T t);
}
```

如果一个类实现了Comparable这个接口，`这个类就具有了比较功能`。所以一般是实体类继承Comparable。
（domain包下的类，用于描述一个对象的类）

如果一个继承了Comparable这个接口，那么它需要干一件事：重写compareTo，`告诉编译器比较规则`。如果说我实现了People类：

```java
//注意Comparable后面最好带上类类型<People>
class People implements Comparable<People>{
    private String sex;
    private int age;
    private String name;

    People(String sex, int age, String name){
        this.sex = sex;
        this.age = age;
        this.name = name;
    }

    //节约篇幅，这里申请get(),set(),toString()方法

    @Override
    public int compareTo(People o) {
        //优先以性别排序
        int flag = this.sex.compareTo(o.getSex());
        //性别相同再以年龄排序
        if(flag == 0){
            flag = this.age - o.getAge();
            //年龄相同再以姓名排序
            if(flag == 0){
                flag = this.name.compareTo(o.getName());
            }
        }
        return flag;
    }
}
```

上面的例子中，People有三个字段：sex, age, name；并重写了compareTo()，比较规则为：`先比较性别，如果性别相同，再比较年龄，如果再相同，再比较姓名；这些字段以升序的进行排序`。

如果对于比较规则这么写还感到迷惑，没关系，我们先看下去，后面会讲。

一个类实现了Comparable接口，就可以实现`多个对象排序`，或者`两个类进行比价`。现在来测试一下数据：

```java
    public static void main(String[] args) {
        //排序
        ArrayList<People> peopleArrayList = new ArrayList<>();
        peopleArrayList.add(new People("男", 21, "a"));
        peopleArrayList.add(new People("男", 21, "a"));
        peopleArrayList.add(new People("男", 22, "a"));
        peopleArrayList.add(new People("男", 21, "b"));
        peopleArrayList.add(new People("男", 22, "a"));
        peopleArrayList.add(new People("女", 21, "a"));
        peopleArrayList.add(new People("女", 21, "b"));

        //因为People内部定义了比较方法，所以可以使用Collections进行排序。
        Collections.sort(peopleArrayList);
        for(People p : peopleArrayList){
            System.out.println(p);
        }

        /*
        结果为：先按性别排序，再按年龄排序，再按姓名排序：

        People{sex='男', age=21, name='a'}
        People{sex='男', age=21, name='a'}
        People{sex='男', age=21, name='b'}
        People{sex='男', age=22, name='a'}
        People{sex='男', age=22, name='a'}
        People{sex='女', age=21, name='a'}
        People{sex='女', age=21, name='b'}
        People{sex='男', age=21, name='a'}
        People{sex='男', age=21, name='a'}
        People{sex='男', age=21, name='b'}
        People{sex='男', age=22, name='a'}
        People{sex='男', age=22, name='a'}
        */

        //比较
        People p1 = new People("男", 21, "b");
        People p2 = new People("男", 21, "a");
        System.out.println(p1.compareTo(p2));

        /*
        结果：b的排序值比a要大。
        1
        */
```

## 2. Comparator

Comparator的源码：

```java
publicinterface Comparator<T> {
    int compare(T o1, T o2);
    boolean equals(Object obj);
}
```

第一个方法 compare(T o1, T o2) 的返回值可能为负数，零或者正数，代表的意思是第一个对象小于、等于或者大于第二个对象。

第二个方法 equals(Object obj) 需要传入一个 Object 作为参数，并判断该 Object 是否和 Comparator 保持一致。

什么时候用Comparator好呢？

