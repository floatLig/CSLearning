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

Comparator的`核心`源码：

```java
    //
    publicinterface Comparator<T> {
        int compare(T o1, T o2);
        boolean equals(Object obj);
    }
```

第一个方法 compare(T o1, T o2) 的返回值可能为负数，零或者正数，代表的意思是第一个对象小于、等于或者大于第二个对象。

第二个方法 equals(Object obj) 需要传入一个 Object 作为参数，并判断该 Object 是否和 Comparator 保持一致。

什么时候用Comparator好呢？

`我们想让类保持它的原貌，不想主动实现 Comparable 接口，但我们又需要它们之间进行比较`，这个时候我们可以定义一个`比较器`来继承Comparator。

原类可以保持不变：

```java
class People implements Comparable<People>{
    private String sex;
    private int age;
    private String name;
    
    //节约篇幅，这里申请get(),set(),toString()方法
}
```

定义一个比较器PeopleComparator继承Comparator接口

```java
class PeopleComparator implements Comparator<People> {
    @Override
    public int compare(People o1, People o2) {
        //优先以性别排序
        int flag = o1.getSex().compareTo(o2.getSex());
        //性别相同再以年龄排序
        if(flag == 0){
            flag = o1.getAge() - o2.getAge();
            //年龄相同再以姓名排序
            if(flag == 0){
                flag = o1.getName().compareTo(o2.getName());
            }
        }
        return flag;
    }
}
```

再来看看测试类：

```java
        peopleArrayList.sort(new PeopleComparator());
        for(People p : peopleArrayList){
            System.out.println(p);
        }

        //运行结果同之前的一样

        //比较
        System.out.println(new PeopleComparator().compare(p1, p2));
        
        //运行结果：1
```

## 使用Comparable 还是 Comparator

通过上面的两个例子可以比较出 Comparable 和 Comparator 两者之间的区别：

一个类实现了 Comparable 接口，意味着该类的对象可以直接进行比较（排序），但`比较（排序）的规则就固定了`。而且

一个类如果想要`保持原样`，又需要`进行不同方式的比较（排序`），就可以`不同的定制比较器`（实现 Comparator 接口）。

Comparable 接口在 java.lang 包下，而 Comparator 接口在 java.util 包下，算不上是亲兄弟，但可以称得上是表（堂）兄弟。

举个不恰当的例子。我想从洛阳出发去北京看长城，体验一下好汉的感觉，要么坐飞机，要么坐高铁；但如果是孙悟空的话，翻个筋斗就到了。我和孙悟空之间有什么区别呢？孙悟空`自己实现了 Comparable 接口`（他那年代也没有飞机和高铁，没得选），而我`可以借助 Comparator 接口`（现代化的交通工具）。

总而言之，如果对象的排序需要基于自然顺序，请选择 Comparable，如果需要按照对象的不同属性进行排序，请选择 Comparator。

## 参考链接

- [鸭血粉丝:一文搞懂Comparable和Comparator](https://mp.weixin.qq.com/s?src=11&timestamp=1584607862&ver=2225&signature=dCv-jvJlNI*yVNS4RWneh75Lc4J2buLzlWFTNfDy7RKl2cEYeDETWL7buW2H4slgSPjSCaV6GBHGdUfTAm9zUOk2Cv0hDZ-oQgfIW*jYmu4pNmX58yYIB8FGt0i2gUNc&new=1)