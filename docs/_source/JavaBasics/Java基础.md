## 一、数据类型

## 参考资料

[cyc-Notes](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E5%9F%BA%E7%A1%80)

### 基本类型

|  类型   |  位数  | 字节数 |  默认值  |  包装类   |
| :-----: | :----: | :----: | :------: | :-------: |
|  byte   |   8    |   1    |    0     |   Byte    |
|  char   |   16   |   2    | '\u0000' | Character |
| boolean | 不确定 | 不确定 |  false   |  Boolean  |
|  short  |   16   |   2    |    0     |   Short   |
|   int   |   32   |   4    |    0     |  Integer  |
|  long   |   64   |   8    |    0     |   Long    |
|  float  |   32   |   4    |   0.0f   |   Float   |
| double  |   64   |   8    |   0.0    |  Double   |

- boolean 只有两个值：true、false，可以使用 1 bit 来存储，但是具体大小没有明确规定。JVM 会在编译时期将 boolean 类型的数据转换为 `int`，使用 1 来表示 true，0 表示 false。JVM 支持 `boolean 数组`，但是是通`过读写 byte 数组`来实现的。

### 包装类型

基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用`自动装箱与拆箱`完成。

```java
Integer x = 2;  //装箱 调用了Integer.valueOf(2)
int y = x;      //拆箱 调用了x.intValue()
```

### 缓存池

Integer是一个对象，但是使用Integer都是创建一个新的对象吗？不是的。Integer对高频数字采用了缓存池设计。让我们先来看两个实现的区别：

new Integer(123) 与 Integer.valueOf(123) 的区别在于：

- new Integer(123) 每次都会新建一个对象；
- Integer.valueOf(123) 会`使用缓存池中的对象`，多次调用会取得同一个对象的引用。

```java
Integer x1 = new Integer(100);      //new,创建新的对象，不适用缓存池中的对象
Integer x2 = 100;                   //自动装箱，使用缓冲池的对象
System.out.println(x1 == x2);       //false,比较的是地址值
System.out.println(x1.equals(x2)); //true，equals的代码：return value == ((Integer)obj).intValue();

Integer y1 = Integer.valueOf(123);
Integer y2 = Integer.valueOf(123);
System.out.println(y1 == y2);       //true,有缓存
System.out.println(y1.equals(y2));  //true

Integer z1 = Integer.valueOf(129);
Integer z2 = Integer.valueOf(129);
System.out.println(z1 == z2);       //false 缓存池默认范围为-128~127
System.out.println(z1.equals(z2));  //true
```

我们来看一下自动装箱 valueOf()的源代码

```java
    static final int low = -128;
    static final int high;

    public static Integer valueOf(int i) {
        //在-128到high的范围内
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            //返回缓存池的对象。-IntegerCache.low == 128（缓存池对象的存放采用补码的顺序）
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

Integer 缓存池的大小默认为 -128~127。

```java
        static {
            // high value may be configured by property
            int h = 127;
            //JVM参数，可以自己设置
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }
```

编译器会在自动装箱过程调用 valueOf() 方法，因此`多个值相同且值在缓存池范围内`，那么就会引用`相同的对象`。

```java
Integer m = 123;
Integer n = 123;
System.out.println(m == n); // true
```

基本类型对应的缓冲池如下：

- boolean values true and false
- all byte values
- short values between -128 and 127
- int values between -128 and 127
- char in the range \u0000 to \u007F 【ASCII，\u0000 ~ \u007F为各种控制字符，详情查看[ASCII](https://zh.wikipedia.org/wiki/ASCII)】

在使用这些基本类型对应的包装类型时，如果该数值范围在缓冲池范围内，就可以直接使用缓冲池中的对象。

在 jdk 1.8 所有的数值类缓冲池中，Integer 的缓冲池 IntegerCache 很特殊，这个缓冲池的下界是 - 128，`上界默认是 127`，但是这个上界是可调的，在启动 jvm 的时候，通过 `-XX:AutoBoxCacheMax=<size>`来指定这个缓冲池的大小，该选项在 JVM 初始化的时候会设定一个名为 java.lang.IntegerCache.high 系统属性，然后 IntegerCache 初始化的时候就会读取该系统属性来决定上界。

## 二、String

> [GitHub - kangjianwei/OpenJDK-11: OpenJDK-11源码，可供阅读学习使用。](https://github.com/kangjianwei/LearningJDK)  
> [JDK源码阅读顺序](https://blog.csdn.net/qq_21033663/article/details/79571506)

### 概览

String被声明为final，因此它是不可被继承。（Integer等包装类也是不可以被继承的）

在Java 8中，String内部使用char数组存储数据。

```java
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0
```

在Java 9 之后，String类的实现改用byte数组存储字符串，同时使用coder来标识使用哪种编码。

```java
    /* 以字节形式存储String中的char，即存储码元
     *
     * 如果是纯英文字符，则采用压缩存储，一个byte代表一个char。
     * 出现汉字等符号后，汉字可占多个byte，且一个英文字符也将占有2个byte。
     *
     * windows上使用小端法存字符串。
     * 如果输入是：String s = "\u56DB\u6761\uD869\uDEA5"; // "四条𪚥"，"𪚥"在UTF16中占4个字节
     * 则value中存储（十六进制）：[DB, 56, 61, 67, 69, D8, A5, DE]
     */
    @Stable
    private final byte[] value;
    
    // 当前字符串的编码：LATIN1(0)或UTF16(1)
    private final byte coder;
```

value数组被声明为final，这意味着value数组初始化之后就不能再引用其他数组了。【final必须赋初值，或者通过构造参数赋值】

### 不可变的好处

1. **可以缓存hash值**

因为String的hash值经常使用，例如String用作HashMap的key。不可变的特性可以使得hash值不可变，因此只需要进行一次计算。

2. **String Pool的需要**

如果一个String对象被创建过了，那么就会从`String Pool`中取得引用。只有String是不可变的，才可能使用String Pool。

![java基础01.png](../../_img/java基础01.png)

3. **安全性**

String经常作为参数，String不可变性可以保证参数的不可变。例如在作为网络连接参数的情况下，如果String是可变的，那么在网络连接过程中，改变String的一方以为现在连接的是其他主机，而实际情况却不一定是。

4. **线程安全**

String不可变性天生具备线程安全，可以在多个线程中安全的使用。

### String，StringBuffer 和 StringBuilder

1. **可变性**

- String不可变
- StringBuffer 和 StringBuilder可变

2. **线程安全**

- String不可变，因此是线程安全的
- StringBuilder `不是`线程安全的
- StringBuffer 是线程安全的，内部使用`synchronized`进程同步

StringBuffer的append()方法。

```java
    @Override
    public synchronized StringBuffer append(Object obj) {
        toStringCache = null;
        super.append(String.valueOf(obj));
        return this;
    }
```

### String Poll

字符串常量池（String Pool）保存着所有`字符串字面量`（literal strings），这些字面量在编译时期就确定。不仅如此，还可以使用String的intern()方法在运行过程中将字符串添加到String Pool中。

当一个字符串调用intern()方法时，如果String Pool中已经存在一个字符串和该字符串值相等（使用equal()方法确定），那么就会返回String Poll中字符串的引用；否则，就会在String Pool中添加一个新的字符串，并返回这个新字符串的引用。

```java
        String str1 = "1";  //这种字面量的形式创建字符串，会自动地将字符串放入 String Pool 中。
        String str2 = "1";
        String str3 = new String("1");
        String str4 = new String("1");
        String str5 = str1.intern();    //先在内存中查找有没有相等的字符串，如果有，返回字符串的引用
        String str6 = str1;
        System.out.println(str1 == str2);   //true
        System.out.println(str1 == str3);   //false
        System.out.println(str1 == str4);   //false
        System.out.println(str1 == str5);   //true
        System.out.println(str1 == str6);   //true
```

### new String("abc")

会创建多少个对象？

如果String Poll里面没有“abc”，就创建两个对象：String Poll中创建“abc”【字面量，`在编译期就已经创建`】，然后创建String对象，它的value指向String Poll的“abc”地址。

以下是 String 构造函数的源码，可以看到，在将一个字符串对象作为另一个字符串对象的构造函数参数时，并不会完全复制 value 数组内容，而是都会指向同一个 value 数组。

```java
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```

## 三、运算

### 参数传递

Java是值传递，但是，如果`参数是对象`的话，传递的值是`地址`，因此会造成原对象值的改变。

**这里需要注意**：假设传递的参数是Integer，Integer的源码中，value值被限定为final。

```java
private final int value;
```

所以如果参数是Integer的话，值是不会被修改的（因为final确保value值是不会被修改的）。

在下面的代码中，如果在函数中修改Integer的值，由于Integer的自动装箱,实际上 `i = 100`中的`100`会被自动装箱成`new Integer(100)`，`i的地址值将指向新的对象`，而不是原来的对象。

```java
public void changeInteger(Integer i){
    i = 100; // i = new Integer(100);
}
```

### 隐式类型转换

低精度的可以自动转化为高精度，但是高精度不能自动转化为低精度（因为可能会精度丢失）

因为字面量 1 是 int 类型，它比 short 类型精度要高，因此不能隐式地将 int 类型向下转型为 short 类型。

```java
short s1 = 1;
// s1 = s1 + 1;
```

但是使用 += 或者 ++ 运算符会执行隐式类型转换。

```java
s1 += 1;
s1++;
```

上面的语句相当于将 s1 + 1 的计算结果进行了向下转型：

```java
s1 = (short) (s1 + 1);
```

### switch

switch支持的条件判断的类型：`char, byte, short, int, Character, Byte, Short, Integer, String, or an enum`

```java
String s = "a";
switch (s) {
    case "a":
        System.out.println("aaa");
        break;
    case "b":
        System.out.println("bbb");
        break;
}
```

## 四、关键字

### final

1. **数据**

声明的数据为常量，可以是编译时常量，也可以是运行时被初始化后不能改变的常量。

- 对于`基本类型`，final使`数值不变`；
- 对于`引用类型`，final使`引用不变`，也就不能引用其他对象，但是被引用的对象本身是可以修改的.

```java
final int x = 1;
// x = 2;  // cannot assign value to final variable 'x'
final A y = new A();
y.a = 1;
```

2. **方法**

声明方法**不能被子类重写**。

private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。

3. **类**

声明类**不允许被继承**。

### static

#### 1. 静态变量

静态变量：又称为类变量，也就是说这个变量属于类的，类所有的实例都共享静态变量，可以直接通过类名来访问它。静态变量在内存中只存在一份。
实例变量：每创建一个实例就会产生一个实例变量，它与该实例同生共死。

```java
public class A {

    private int x;         // 实例变量
    private static int y;  // 静态变量

    public static void main(String[] args) {
        // int x = A.x;  // Non-static field 'x' cannot be referenced from a static context
        A a = new A();
        int x = a.x;
        int y = A.y;
    }
}
```

#### 2. 静态方法

静态方法在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法。

```java
public abstract class A {
    public static void func1(){
    }
    // public abstract static void func2();  // Illegal combination of modifiers: 'abstract' and 'static'
}
```

只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字，`因此这两个关键字与具体对象关联`,静态字段在类加载阶段已经创建了，但是类实例只有在被new时，才创建。

```java
public class A {

    private static int x;
    private int y;

    public static void func1(){
        int a = x;
        // int b = y;  // Non-static field 'y' cannot be referenced from a static context
        // int b = this.y;     // 'A.this' cannot be referenced from a static context
    }
}
```

#### 3. 静态语句块

静态语句块在类初始化时运行一次。

```java
public class A {
    static {
        System.out.println("123");
    }

    public static void main(String[] args) {
        A a1 = new A();
        A a2 = new A();
    }
}
```

```
123
```

#### 4. 静态内部类

非静态内部类依赖于外部类的实例，也就是说需要先创建外部类实例，才能用这个实例去创建非静态内部类。而静态内部类不需要。

```java
public class OuterClass {

    class InnerClass {
    }

    static class StaticInnerClass {
    }

    public static void main(String[] args) {
        // InnerClass innerClass = new InnerClass(); // 'OuterClass.this' cannot be referenced from a static context
        OuterClass outerClass = new OuterClass();
        InnerClass innerClass = outerClass.new InnerClass();
        StaticInnerClass staticInnerClass = new StaticInnerClass();
    }
}
```

静态内部类不能访问外部类的非静态的变量和方法。

#### 5. 初始化顺序

1. 在类加载阶段，`static final`，`static`会被初始化。staitc final 初始化时，直接就赋值了，final初始化时，初始化为null，或者是默认值。
2. 再加载静态代码块的内容。
3. 如果有类进行实例化，先对实例化属性，代码块内容；然后再执行构造函数的内容。

想要实例化子类对象时，看看执行顺序是什么样的吧。（注释后面的数值为顺序）

```java
class Father {

    static{
        staticFatherA = "staticFatherA block front";            //3. 静态代码块，按照从上到下的顺序
    }
    {
        fatherA = "fatherA block front";                        //11
    }

    Father(){
        System.out.println("father");                           //14. 属性赋值完后，执行构造函数
    }

    static final String FINAL_STATIC_STR = "finalStaticStr";    //1. 静态常量变量 初始化值为"finalStaticStr"
    private static String staticFatherA = "staticFatherA";      //2. 静态变量 初始化值为null  4. staticFatherA 值变为“staticFatherA”
    private String fatherA = "fatherA";                         //10. 当实例化一个对象时，先从父类的属性开始，初始值为null.  12.

    static{
        staticFatherA = "staticFatherA block after";            //5. 静态代码块
    }
    {
        fatherA = "interFatherA block after";                   //13
    }
}

class Child extends Father {

    static{
        staticChildA = "staticChildA block front";              //7. 静态代码块，从上往下执行
    }
    {
        childA = "childA block front";                          //16.
    }

    private static String staticChildA = "staticChildA";        //6. 子类的静态代码块，初始值为null  8.
    private String childA = "childA";                           //15. 初始值为null      17.

    static{
        staticChildA = "staticA block after";                   //9.
    }
    {
        childA = "childA block after";                          //18.
    }

    Child(){
        System.out.println("Child");                            //19. 构造函数
    }

    public static void main(String[] args) {
        Father a = new Child();
    }
}
```

这里有一点要强调一下:

```java
    static{
        staticFatherA = "staticFatherA block front";            
    }

    private static String staticFatherA = "staticFatherA";     

    static{
        staticFatherA = "staticFatherA block after";            
    }
```

上面代码的顺序等同于【`因为staticFatherA是static,不是static final，所以初始值为null`】

```java
    private static String staticFatherA = null;     

    static{
        staticFatherA = "staticFatherA block front";   
        staticFatherA = "staticFatherA";   
        staticFatherA = "staticFatherA block after";            
    }
```

存在继承的情况下，初始化顺序为：

- 父类（静态变量、静态语句块）
- 子类（静态变量、静态语句块）
- 父类（实例变量、普通语句块）
- 父类（构造函数）
- 子类（实例变量、普通语句块）
- 子类（构造函数）

## 五、Object 通用方法

### 概览

### equals()

- 对于基本类型，==判断两个值是否相等，基本类型没有equals()方法
- 对于引用类型，==判断两个变量是否引用同一个对象（引用的地址是否相同），而equals()判断引用的对象是否等价

```java
Integer x = new Integer(1);
Integer y = new Integer(1);
System.out.println(x.euqals(y));    //true
System.out.println(x == y);         //false
```

**实现：**

1. 检查是否为同一个对象的引用，如果是直接返回true；
2. 检查是否是同一个类型，如果不是，直接返回false;
3. 将Object对象进行转型；
4. 判断每个关键域是否相等

```java
public class EqualExample{
    private int x;
    private int y;
    private int z;

    public EqualExample(int x, int y, int z){
        this.x = x;
        this.y = y;
        this.z = z;
    }

    @Override
    public boolean equals(Object o){
        if(this == o) return true;
        if(o == null || getClass() != o.getClass()) return false;

        EqualExample that = (EqualExample) o;

        if(x != that.x) return false;
        if(x != that.y) return false;
        return z == that.z;
    }
}
```

### hashCode()

`如果两个对象equals()相等，hashCode()也要相等`。因为HashMap等容器用的是hashCode() 判断两个对象是否相等。所以，我们在重写equals()方法时，hashCode()应该也要重写。

但是有一点需要注意，hashCode()相等，不一定equal()相等。有可能两个对象的值不相同，但是计算出来的散列值是相同的。

理想的哈希函数应当具有均匀性，即不相等的对象应该均匀分不到所有可能的哈希值上。这就要求哈希函数要把所有域的值都考虑进来。

### toString()

默认返回ToStringExample@4554617c这种形式，其中@后面的数值为散列码的无符号十六进制标识。

### clone()

Effective Java书上讲到，最好不要去使用clone(),可以使用拷贝构造函数或者拷贝工厂拷贝一个对象，因为clone()方法来拷贝一个对象既复杂又有风险，它会抛出异常，并且还需要类型转换。

## 六、继承

### 访问权限

Java中有三个访问权限修饰符：private、protected以及public，如果不加访问修饰符表示包级可见。

|     /     | 本类其他方法使用 | 本包(package)其他类使用 | 本包(package)其他类使用，并且非本包的子类也能使用 | 所有类可以使用 |
| :-------: | :--------------: | :---------------------: | :-----------------------------------------------: | :------------: |
|  public   |        √         |            √            |                         √                         |       √        |
| protected |        √         |            √            |                         √                         |       ×        |
|  default  |        √         |            √            |                         ×                         |       ×        |
|  private  |        √         |            ×            |                         ×                         |       ×        |

protected用于修饰成员，表示在继承体系中成员对于子类可见，但是这个访问修饰符对于类没有意义。

良好的设计的模块会隐藏所有的细节实现，把它的API与它的实现清晰的隔离开。模块之间只通过它们的API进行通信，一个模块不需要知道其他模块的内部工作情况，这个概念称为信息隐藏或封装。因此访问权限应当尽可能使每个类或成员不被外界访问。

如果子类的方法重写了父类的方法，那么子类中该方法的访问级别不允许低于父类的访问级别。这是为了确保可以使用父类实例的地方都可以使用子类实例去替代，也就是确保满足`里氏替换原则`。

`字段决不能是公有的`，因为这么做的话就失去了对这个字段修改行为的控制，客户端可以对其随意修改。例如下面的例子中，AccessExample 拥有 id 公有字段，如果在某个时刻，我们想要使用 int 存储 id 字段，那么就需要修改所有的客户端代码。

### 抽象类与接口

#### 1. 抽象类

抽象类和抽象方法都使用abstract关键字进行声明。如果一个类中包含抽象方法，那么这个类必须声明为抽象类。

`抽象类和抽象方法的最大区别是：抽象类不能被实例化，只能被继承。`

```java
public abstract class AbstractClassExample {

    protected int x;
    private int y;

    public abstract void func1();

    public void func2() {
        System.out.println("func2");
    }
}
```

#### 2. 接口

接口是类的延伸，在JDK 8之前，它可以看成是一个完全抽象的类，也就是说不能有任何的方法实现。

`从JDK 8开始，接口也可以有默认的方法实现`，这是因为不支持默认的接口维护成本太高了。在JDK 8之前，如果一个接口想要添加新的方法，那么要修改所有实现了该接口的类，让它们都实现新增的方法。

- `接口的默认属性是：public static final`
- `接口的默认方法是：public 且 函数体不能有内容`
- `接口默认方法的实现：default (实例创建时才有) / static (类加载时就有了)`
- 且不允许定义成其他，如private，protected等

```java
public interface InterfaceExample {

    void func1();               //默认是public，且不能有函数体

    default void func2(){       //要实现函数体，必须设置为default（对象实例的方法），或者是static（类方法）
        System.out.println("func2");
    }

    static void staticFunc3(){  //static方法
        System.out.println("staticFunc3");
    }

    int INTERFACE_X = 123;            //默认为public static final,且不能为其他
    // int y;               // Variable 'y' might not have been initialized
    public int INTERFACE_Z = 0;       // Modifier 'public' is redundant(多余的) for interface fields
    // private int k = 0;   // Modifier 'private' not allowed here
    // protected iXnt l = 0; // Modifier 'protected' not allowed here
    // private void fun3(); // Modifier 'private' not allowed here
}
```

### super()

- 访问父类的构造函数：可以使用super()函数访问父类的构造函数，从而委托完成一些初始化的工作。应该注意到，子类一定会调用父类的构造函数来完成初始化工作，一般是调用父类的默认构造函数，如果子类需要调用父类其他的构造函数，那么就可以使用super()函数。
- 访问父类的成员：如果子类重写了父类的某个方法，可以通过super关键字来引用父类的方法实现。

### 重写与重载

#### 1. 重写（Override）

`存在于继承体系中`，指子类实现了一个与父类在方法上声明完成相同的一个方法。

为了满足里氏替换原则,重写有以下三个权限：

- 子类方法的访问权限必须大于等于父类方法；
- 子类方法的返回类型必须是父类方法返回类型或为子类型；
- 子类方法抛出的异常必须是父类抛出异常或为子类型。

使用@Override注解，可以让编译器帮忙检查是否满足上面三个限制条件。

在调用一个方法时，先从本类中查找看是否有对应的方法，如果没有再到父类中查看，看是否从父类继承来。否则就要对参数进行转型，转成父类之后看是否有对应的方法。总的来说，方法调用的优先级为：

- this.func(this)
- super.func(this)
- this.func(super)
- super.func(super)

```java
class A {

    public void show(A obj) {
        System.out.println("A.show(A)");
    }

    public void show(C obj) {
        System.out.println("A.show(C)");
    }
}

class B extends A {

    @Override
    public void show(A obj) {
        System.out.println("B.show(A)");
    }

    public void showSelf(){
        System.out.println("Show Self");
    }
}

class C extends B {
}

class D extends C {
}
```

```java
public static void main(String[] args) {

    A a = new A();
    B b = new B();
    C c = new C();
    D d = new D();

    // 在 A 中存在 show(A obj)，直接调用
    a.show(a); // A.show(A)
    // 在 A 中不存在 show(B obj)，将 B 转型成其父类 A
    a.show(b); // A.show(A)
    // 在 B 中存在从 A 继承来的 show(C obj)，直接调用
    b.show(c); // A.show(C)
    // 在 B 中不存在 show(D obj)，但是存在从 A 继承来的 show(C obj)，将 D 转型成其父类 C
    b.show(d); // A.show(C)

    // 引用的还是 B 对象，所以 ba 和 b 的调用结果一样
    A ba = new B();
    ba.show(c); // A.show(C)
    ba.show(d); // A.show(C)
    // ba.showSelf();   A ba = new B();只能调用A和B共有的方法。B私有的方法不能调用。但是调用的共有的方法却以B的实现为主
}
```

### 重载

存在于`同一个类中`，指一个方法与已经存在的方法名称上相同，但是`参数类型、个数、顺序`至少有一个不同。

应该注意的是，`返回值不同，其它都相同不算是重载`。因为从编译器的角度来看，它会将一个函数签名（`包括它的函数名&参数列表`）以前缀或后缀的方式组织成一个名称。这里不包括它的返回值，所以从编译器的角度来来看，这两个函数是同一个，区分不开。或者简单来说，`编译器不知道你要返回什么值`。

## 七、反射

每个类都有一个Class对象，包含了与类有关的信息。当编译一个新类时，会产生一个同名的.class文件，该文件内容保存着Class对象。

类加载相当于Class对象的加载，类在第一次使用时才动态加载到JVM中。也可以使用Class.forName("com.mysql.jdbc.Driver")这种方式来控制类的加载，该方法返回一个Class对象。

反射可以提供`运行时的类信息`，并且这个类可以在运行时才加载进来，甚至在编译期间该类的.class不存在也可以加载进来。

Class和java.lang.reflect一起对反射提供了支持，java.lang.reflect类库包含了以下三个类：

- Filed：可以使用get()和set()方法读取和修改Field对象关联的字段；
- Method：可以使用invoke()方法调用与Method对象关联的方法；
- Constructor：可以用Constructor的newinstance()创建新的对象。

**反射的优点：**

- **可拓展性：**应用程序可以利用全限定类名创建可拓展对象的实例，来使用来自外部的用户自定义类。
- **类浏览器和可视化开发环境：**一个类浏览器需要可以枚举类的成员。可视化开发环境（如IDE）可以从利用反射中可用的类型信息中受益，以帮助程序员编写正确的代码。
- **调试器和测试工具：**调试器需要能够检查一个类里的私有成员。测试工具可用利用反射来自动地调用类里定义地可被发现地API定义，以确保一组测试中有较高地代码覆盖率。

**反射的缺点：**

尽管反射非常强大，但也不能滥用。如果一个功能可以不用反射完成，那么最好就不用。在我们使用反射技术时，下面几条内容应该牢记于心。

- **性能开销：**反射涉及了动态类型的解析，所以JVM无法对这些代码进行优化。因此，反射操作的效率比那些非反射操作低得多。我们应该避免在经常被执行的代码或对性能要求很高的程序中使用反射。
- **安全限制：**使用反射技术要求程序必须在一个没有安全限制的环境中运行。如果一个程序必须在安全限制的环境中运行，如Applet，那么这就是个问题了。
- **内容暴露：**由于反射允许代码执行一些在正常情况下不被允许的操作（比如访问私有属性和方法），所以使用反射可能导致意料之外的副作用，这可能导致代码功能失调并破坏可移植性。反射代码破坏了抽象性，因此当平台发生改变时，代码的行为可以随之变化。

## 八、异常

Throwable可以用来表示任何可以作为异常抛出的类，分为两种：Error 和 Exception。Error用来表示JVM无法处理的错误，Exception分为两种：

- `受检异常`：需要用try..catch..语句捕获并进行处理，并且可以从异常中恢复；
- 非受检异常：是程序运行时错误，例如除0会引发Arithmetic Exception,此时程序崩溃并且无法恢复。

![java基础02.png](../../_img/java基础02.png)

## 九、泛型

```java
public class Box<T> {
    // T stands for "Type"
    private T t;
    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```