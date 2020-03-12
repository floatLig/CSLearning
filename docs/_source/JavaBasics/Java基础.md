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

## 参考资料

[cyc-Notes](https://cyc2018.github.io/CS-Notes/#/notes/Java%20%E5%9F%BA%E7%A1%80)