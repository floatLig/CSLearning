## 一、数据类型

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

### String

> [GitHub - kangjianwei/OpenJDK-11: OpenJDK-11源码，可供阅读学习使用。](https://github.com/kangjianwei/LearningJDK)  
> [JDK源码阅读顺序](https://blog.csdn.net/qq_21033663/article/details/79571506)

## 参考资料

[cyc-Notes](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E5%9F%BA%E7%A1%80)