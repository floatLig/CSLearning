##### 不熟悉

1. finalize 垃圾回收的时候调用，一般用于资源的释放

2. finally  相当于一个新的函数

3. stream流

   >  https://www.jianshu.com/p/6fdc94ea577d

   ![image-20200817233243545](_img/image-20200817233243545.png)

   ```java
   // 携程 
   
   class WorkflowNode {
       String nodeId;
       int timeoutMillis;
       List<WorkflowNode> nextNodes;
       boolean initialised;
   
       public WorkflowNode(String nodeId) {
           this.nodeId = nodeId;
       }
   
       public static WorkflowNode load(String value) {
           // Create head node;
           Map<String, WorkflowNode> map = new HashMap<>();
           WorkflowNode head = new WorkflowNode("HEAD", 0, null);
           map.put(head.nodeId, head);
   
           for (String nodeValue : value.split("\\|")) {
               String[] properties = nodeValue.split("\\`");
               WorkflowNode node = map.get(properties[0]);
   
               node.timeoutMillis = Integer.parseInt(properties[1]);
               node.initialised = true;
   
               // Check next nodes
               if (properties[2].equals("END")) {
                   continue;
               }
               
               // nextNodes 结点
               // 利用stream流获取nextNodes结点
               node.nextNodes = Arrays.stream(properties[2].split(","))
                       .map(p -> map.containsKey(p) ? map.get(p) : new WorkflowNode(p, 0, null))
                       .collect(Collectors.toList());
   
               map.put(node.nodeId, node);
               //
           }
   
           return head;
       }
   
       public WorkflowNode(String nodeId, int timeoutMillis, List<WorkflowNode> nextNodes) {
           this.nodeId = nodeId;
           this.timeoutMillis = timeoutMillis;
           this.nextNodes = nextNodes;
       }
   }
   ```

   ```java
   // int[] 转 List<Integer>
   List<Integer> list = Arrays.stream(ints).boxed().collect(Collectors.toList());
   
   //int[] 转 Integer[]
   // Integer[]::new, 的new方法
   Integer[] integers = Arrays.stream(ints).boxed().toArray(Integer[]::new);
   
   // List<Integer> 转 int[]
   int[] ints = list.stream().mapToInt(Integer::valueOf).toArray();
   
   // Integer[] 转 int[]
   int[] ints = Arrays.stream(integers).mapToInt(Integer::valueOf).toArray();
   
   // Integer[] 转 List<Integer>
   List<Integer> list = Arrays.stream(integrs);
   
   // List<Integer> 转 Integer[]
   Integer[] integers = list.toArray(new Integer[list.size()]);
   ```

   



##### 自我介绍

1. 应届生，自学了Java和Java相关的知识，参照了网上资料 秒杀系统，springBoot，在前段时间的实习，做的项目主要是 持续集成，持续部署的项目，dockers kubernetes

---



##### 项目中用到的集合，源码

1. ArrayList从数据库取出多条数据，HashMap将这个数据放到HashMap中去，因为秒杀系统是热电数据，如果用户进行查找，直接从 HashMap中取出来，（手机名，iPhone 10）
2. ArrayList 底层是数组，add，扩容
3. HashMap，1.8，put，数组是否为空，初始化容量 16，key 扰动函数 hash, &(容量 n  - 1)， 判断有没有元素，没有元素，就插入，size++, threadhold比较。如果说有元素的话，判断 key相等，覆盖，不相等话，treeNode, 不是的话，遍历数组，8，64 转换成红黑树
4. resize , 初始化，size >= threadhold, 2倍的方式进行扩容

---



##### 详细说说你的项目业务逻辑和使用的技术栈

1. 登录，@valid，注解进行数据验证，redis判断有没有这个对象（有预热），MySQL，查不到这个对象，查到 密码验证，cookie，tooken
2. 秒杀，秒杀地址进行验证，秒杀结束标志，redis预减库存，更新标志，消息队列中去，标记（客户端轮询），redis中是否用用户Id的订单，是否重复秒杀，MySQL减库存生成订单，更新标志。
3. `public class GlobalException extends RuntimeException`, 拦截器`extends HandlerInterceptorAdapter`

---



##### 多线程怎么理解，怎么实现多线程

1. 程序调度基本单位，多线程，Java
2. 继承 Runnable， Callable,  继承Thread， Callable需要再封装成FutureTask，获取到运行的返回结果


##### 动态代理、cglib


###### 1. 代理机制的介绍

代理机制是JavaSE 1.3新增加的特性。利用代理机制可以`在运行时` 创建一个 `实现了一组给定接口的新类`。

spring的IoC是用了代理来实现`解耦【在编译期间减少类和类之间的依赖】`，AOP用代理来实现`方法的增强`，也可以在编译时无法确定（哪个实现类）需要实现某个接口时使用。

<!-- more -->

假设有一个`表示接口`的Class对象（有可能只包含一个接口 ）,它的`确切类型`在编译时无法知道。因为我不知道它的确切类型，所以要想构造一个实现这些接口的类（设为类A），不能简单的new出来。那能怎么做呢？利用反射！使用`newlnstance方法或反射找出这个类的构造器`。那要怎么执行类A的方法method()呢？因为Java`不能实例化一个接口`，需要在程序处于运行状态时`定义一个新类【即代理类】`，然后在这个新类【即代理类】的方法中执行A的方法method()。

代理类能够实现指定的接口。它具有下列方法：

- 指定接口所需要的全部方法。
- Object类中的全部方法，例如，toString, equals等

###### 2. 调用处理器

好，现在我们知道了代理类是为了什么而产生的了，那要怎么实现这个代理类呢？

要定义该代理类，`首先`要提供一个`调用处理器（invocation handler）`,调用处理器的作用是在代理类中调用类A的方法，以便能够在程序运行时，运行A的方法，当然，也可以对类A的方法进行增强。

`调用处理器`是实现了InvocationHandler接口的类对象。在这个接口中只有一个方法：

```java
Object invoke(Object proxy, Method method, Object[] args)
```

我们来解释一下invoke()方法怎么发挥代理的作用。调用代理对象任何的方法，调用处理器InvocationHandler的invoke方法会被调用执行。invoke方法表示：无论外界要执行什么方法，都必须经过invoke()方法，而外界的方法method保存在`Method对象`中，方法的参数保存在`Object args[]`中。

###### 3. 代理对象实例代码

我们先来看一下实现代理对象的实例代码，后面再解释代码。

```java
public class ProxyTest {
    public static void main(String[] args) {
        Object[] elements = new Object[1000];

        //每个代理对象的属性target赋值1~1000
        for (int i = 0; i < elements.length; i++) {
            Integer value = i + 1;
            //处理器，处理器你也可以写成匿名内部类
            InvocationHandler handler = new TraceHandler(value);
            Object proxy = Proxy.newProxyInstance(null, new Class[]{Comparable.class}, handler);
            //elements每个元素都是proxy
            elements[i] = proxy;
        }

        //随机函数获得一个key
        Integer key = new Random().nextInt(elements.length) + 1;

        //利用二分查找，打印出找key的过程
        int result = Arrays.binarySearch(elements, key);

        //打印出相匹配的结果
        if (result >= 0) {
            System.out.println(elements[result]);
        }
    }
}

/**
 * 调用处理器，先打印出方法和名字，然后再调用原来的方法
 */
class TraceHandler implements InvocationHandler {

    private Object target;

    public TraceHandler(Object t) {
        target = t;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //打印出查找的位置的target，二分查找，从中点开始查找，第一个数肯定是500
        System.out.print(target);
        //打印出调用方法的名字
        System.out.print(":" + method.getName() + "(");
        if (args != null) {
            for (int i = 0; i < args.length; i++) {
                //打印出参数，二分查找的参数是查找的“key”
                System.out.print(args[i]);
                if (i < args.length - 1) {
                    System.out.print(", ");
                }
            }
            System.out.println(")");
        }
        //调用原来的方法，即二分查找的compareTo()，对比方法
        return method.invoke(target, args);
    }
}
```

结果为：

```text
500:compareTo(708)
750:compareTo(708)
625:compareTo(708)
687:compareTo(708)
718:compareTo(708)
702:compareTo(708)
710:compareTo(708)
706:compareTo(708)
708:compareTo(708)
708:toString()
```

###### 3. 创建一个代理对象

创建一个代理对象,`调用处理器（invocation handler）`只是需要的一部分，现在我们来看看该`代理对象`是怎么实现的？

需要`使用Proxy类的newPmxylnstance方法`。这个方法冇三个参数：

- `类加载器(class loader)`。作为Java安全模型的一部分，对于系统类和从因特网上下载下来的类，可以使用不同的类加载器，后面我们再详细谈论类加载器。  
目前，用null表示使用默认的类加载器。  
- `Class对象数组`，每个Class对象都是类A需要实现的接口。
- 一个`调用处理器（invocation handler）`。

上面我们知道调用处理器的invoke()。那如何定义一个调用处理器(invocation handler)呢？能够用结果代理对象做些什么？当然， 这两个问题的答案取决于打算使用代理机制解决什么问题，使用代理可能出于很多原因，例如：

- 路由对远程服务器的方法调用
- 在程序运行期间，将用户接口事件与动作关联起来
- 为调试，跟踪方法调用

在下面的示例中，使用代理和调用处理器`跟踪方法调用`，并且定义了一个`TraceHander包装器类`存储包装的对象。其中的`invoke方法打印出被调用方法的名字和参`数【方法的增强】，随后用包装好的对象作为隐式参数调用这个方法.

```java
class TraceHandler implements InvocationHandler{
    private Object target;

    public TraceHandler(Object t){
        target t;
    }

    public Object invoke(Objact proxy, Method m, Object[] args) throws Thnotable
    {
        // print method name and parameters【方法的增强】
        ...
        // invoke actual method【调用原来的方法】
        return m.invoke{target, args);
    }

}
```

下面说明一下如何构造：用于`跟踪方法调用`的代理对象。

```java
Object value = ···;
// construct wrapper构造包装
InvocationHardler handler = new TraceHandler(value);
// cunstruct proxy for one or more interfaces
Class[] interfaces = new Class[] {Comparable.class};
//代理对象
Object proxy = Proxy.newProxyInstaiice(nul1, interfaces, handler);
```

现在，proxy成为代理对象。这时，无论何时用proxy调用哪个方法，这个方法的名字和参数就会打印出来，之后再用value调用它。

在下面的代码中，使用代理对象对二分査找进行跟踪。这里，首先将用1~1000整数的代理填充数组，然后调用Arrays类中的binarySearch方法在数组中査找一个随机整数e。最后，打印出与之匹配的元素。

```java
Object[] elements = new Object[1000];
// fill elements with proxies for the integers 1~1000  
for (int i =0; i < elownts.length; i++){
    Integtr value = i + 1；
    // proxy for value，elements成为代理对象，elements执行的任何方法都会首先执行invoke()方法;
    elenments[i] = Proxy.newlnstance(...); 
}

// construct a random integer
Integer key = new Rartdom().nextInt(elements.length) + 1;

// search for the key
int result = Arrays.birarySearch(elements, key);

// print natch if found
if (result >= 0) {
    System.out.prirvtln(elements[result]);
}
```

在上述代码中，Integer类实现了`Comparable接口`【数值比较】。代理对象属于在运行时定义的类（它有一个名字，如$Proxy0）。`这个代理类也实现了Compamble接口`。然而，它`的compareTo方法调用了代理对象处理器的invoke方法`。

> 注释：在Java SE 5.0中Integer类实际上实现了 Comparable[Integer]。然而，在运行时，所有的泛型类都被取消，代理将它们构造为原Comparable类的类对象。

binary Search方法按下面这种方式调用：

```java
if (elements[i].compareTo[key) < 0)...
```

由于数组中填充了代理对象，所以`compareTo调用了TraceHander类中的invoke方法`。这个方法打印出了方法名和参数，之后用包装好的Integer对象调用compareTo。

最后，在实例程序的结尾调用：

```java
System.out.println(elemenets[results]);
```

println方法调用代理对象的toString,这个调用也会被重定向到调用处理器上，下面是程序运行的全部跟踪结果：

```text
500:compareTo(708)
750:compareTo(708)
625:compareTo(708)
687:compareTo(708)
718:compareTo(708)
702:compareTo(708)
710:compareTo(708)
706:compareTo(708)
708:compareTo(708)
708:toString()
```

可以看出，二分査找算法査找关键字的过程，即每一步都将查找区间缩减一半。注意，即使不属于Comparable接口，toString方法也被代理。

###### 5. 代理类的特性

上面，我们看到了代理类的应用，接下来，我们来了解一下它们的一些特性。

需要记住，`代理类是在程序运行过程中创建的`。然后，`一旦被创建，就变成了常规类，与虚拟机中的任何其他类没有什么区别`。

所有的代理类都拓展于roxy类。`一个代理类只有一个实例域--调用处理器`，它定义在Proxy的超类中。为了履行代理对象的职责，`所需要的任何附加数据都必须存储在调用处理器中`。例如，在例6-7给出的程序中，代理Comparable对象时，`TranceHandler包装了实际的对象`。

所有的代理类都覆盖了Object类中的方法`toString、equals和hashCode`。如同所有的代理方法一样，这些方法仅仅`调用了处理器的invoke`。Object类中的其他方法（如`clone和getClass`）`没有被重新定义。`

没有定义代理类的名字，Sun虚拟机中的Proxy类将生成一个以字符串`$Proxy`开头的类名。

对于特定的类加载器和预设的一组接口来说，`只能有一个`代理`类`。也就是说，如果使用`同一个类加载器`和`接口`调用`两次newProxyInstance`方法的话，那么只能够得到`同一个类的两个对象`，也可以利用getProxyClass方法获得这个类。

```java
Class proxyClass = Proxy.getProxyClass(null,interfaces);
```

代理类一定是public和final。如果代理类实现的所有接口都是public，代理类就不属于某个特定的包；否则，所有非共有的接口都必须属于同一个包，同时，代理类也属于这个包。

可以通过调用Proxy类中的isProxyClass方法检测一个特定的Class对象是否代表一个代理类。

**代理对象的方法：**

```java
//返回指定接口的代理类
static Class getProxyClass(ClassLoader loader,Class[] interfaces)

//构造一个实现指定接口的代理类的实例。所有的方法都将调用给指定处理器对象的invoke方法
static object newProxyInstance(ClassLoader loader,Class[] interfaces, InvocationHandler handler)

//定义代理对象调用方法时希望执行的动作
Object invoke(Object proxy, Method method, Object[] args)

//如果c是一个代理类返回true
static boolean isProxyClass(Class c)
```



##### ping, tracert

1. ICMP，ICMP的回送请求和回送回答报文。ping 是应用层直接使用 ICMP的一个例子，没有经过运输层
2. 向目的主机发送4个ICMP回送请求报文，目的主机收到后，返回回送回答报文，因为有时间戳，所以很容易计算出往返时间
3. tracert，ttl从1开始设置，然后逐渐增大，这样子就可以得到报文经过的路由器。tracert中封装的是无法交付的UDP用户数据报

---



##### 对[用友](https://www.nowcoder.com/jump/super-jump/word?word=用友)的了解，对业务的了解，最大的挫折，自己的优缺点，将来的打算

---



##### 内部类有哪些分类，有什么特点？



##### 泛型和泛型擦除

##### 泛型标记

1. E, N, T, K, V, ?

##### 泛型限定

1. ? extends T, ? super T

---



向上转型+重载， 只有方法表现多态。 如果 `Parent parent = new Son();  parent.sum 是父类的`

---



son.join()

---



谈谈对synchronized的理解

---



对象头信息

---



monitor对象

---



单例模式，双重锁检测为什么要写 volatile

---



AQS

---



谈谈你对 ReentrantLock的理解

---



ConcurrentHashMap

---



创建线程的的方式，还有线程池

---



OOM

---



垃圾收集器

---



类的加载机制

双亲委派模型

---



