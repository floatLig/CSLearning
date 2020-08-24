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

##### 反问

1. 部门的具体业务
2. 自己值得改进的地方

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

> https://apppukyptrl1086.pc.xiaoe-tech.com/detail/v_5e0cba7b5f987_rEHvwPy7/3?from=p_5dd3ccd673073_9LnpmMju&type=6

其实就是动态的创建一个代理类出来，创建这个代理类的实例对象，在这个里面引用你真正自己写的类，所有的方法的调用，都是先走代理类的对象，他负责做一些代码上的增强，再去调用你写的那个类

 

spring里使用aop，比如说你对一批类和他们的方法做了一个切面，定义好了要在这些类的方法里增强的代码，spring必然要对那些类生成动态代理，在动态代理中去执行你定义的一些增强代码

 

如果你的类是实现了某个接口的，spring aop会使用jdk动态代理，生成一个跟你实现同样接口的一个代理类，构造一个实例对象出来，jdk动态代理，他其实是在你的类有接口的时候，就会来使用

 

很多时候我们可能某个类是没有实现接口的，spring aop会改用cglib来生成动态代理，他是生成你的类的一个子类，他可以动态生成字节码，覆盖你的一些方法，在方法里加入增强的代码

 



---


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



创建线程的的方式，还有线程池（`Executors.newFixedThreadPool`）

---



OOM

---



垃圾收集器

ZGC：多重映射 染色指针，如果标志位更新了，那么就直接从重分配集中去查找。自愈

---



类的加载机制

双亲委派模型

---



DNS端口

HTTP端口

HTTPS端口

---



DNS原理和步骤？

顶级域名服务器、根据名服务器、权限域名服务器

---



消息队列的优点？

![图片](_img/图片.png)

![图片](_img/图片-1597901478289.png)

![图片](_img/图片-1597901503519.png)

---



HTTP 1.1: 长连接、流水线

HTTP 2.0: 二进制分层、首部压缩、服务端推送

---



Hash的原理：链表、开放地址法

---



虚拟内存，页面置换算法



```java
class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    
    public static Singleton() {
        if (instance == null) {
            synchroniezd(Singleton.class) {
                if (instance == null) {
                    intance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

---



BIO：同步阻塞

NIO：同步非阻塞

AIO：异步非阻塞

- A顾客去吃海底捞，就这样干坐着等了一小时，然后才开始吃火锅。(BIO)

- B顾客去吃海底捞，他一看要等挺久，于是去逛商场，每次逛一会就跑回来看有没有排到他。于是他最后既购了物，又吃上海底捞了。（NIO）

- C顾客去吃海底捞，由于他是高级会员，所以店长说，你去商场随便玩吧，等下有位置，我立马打电话给你。于是C顾客不用干坐着等，也不用每过一会儿就跑回来看有没有等到，最后也吃上了海底捞（AIO）

---



大多数Java虚拟机采用的 1:1 线程模型（映射到核心级线程）

---



du -ah 查看当前磁盘 （Disk Used）

df -h (Disk Free)

| -a   | 显示目录中所有文件大小 |
| ---- | ---------------------- |
| -k   | 以KB为单位显示文件大小 |
| -m   | 以MB为单位显示文件大小 |
| -g   | 以GB为单位显示文件大小 |
| -h   | 以易读方式显示文件大小 |
| -s   | 仅显示总计             |

---



io多路复用：

1. 通过一种机制，让一个进程监听多个套接字(非阻塞异步)
2. 系统调用

---



> 记录防止忘记： 服务器的带宽很重要， 1Mbps 和 20Mbs  QPS差距很大
>
> ![image-20200822193955159](_img/image-20200822193955159.png)



nginx:

nginx面试题：https://juejin.im/post/6844904125784653837

nginx高性能的原因：https://blog.csdn.net/yin__ren/article/details/93619025 1. io多路复用， 2. master-worker进程模型 3. 协程机制



![image-20200822193817334](_img/image-20200822193817334.png)

nginx做静态资源服务器

server模块下的location

![image-20200822193455585](_img/image-20200822193455585.png)

nginx负载均衡

配置`upstream`并选择负载均衡策略

![image-20200822193805440](_img/image-20200822193805440.png)

分布式session

**解决方案：**

- 使用cookie来完成（很明显这种不安全的操作并不可靠）
- 使用Nginx中的ip绑定策略，同一个ip只能在指定的同一个机器访问（不支持负载均衡）
- 利用数据库同步session（效率不高）
- 使用tomcat内置的session同步（同步可能会产生延迟）
- 使用token代替session
- **我们使用spring-session以及集成好的解决方案，存放在redis中**

作者：java高级架构41397
链接：https://juejin.im/post/6844903740479160334
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

![08115315_KJKw.jpg](https://static.oschina.net/uploads/img/201805/08115315_KJKw.jpg)

##### 架构

![image-20200824165846212](_img/image-20200824165846212.png)

秒杀系统：

请求量从 1w/s 到 10 w/s，架构改造：

1. 将秒杀系统模块独立出来，对这个系统做针对性的优化
2. 服务器集群也独立出来，这样大流量不会影响到其他商品
3. 添加redis
4. 秒杀答题

请求量从10w/s 到 100w/s，架构改造：

1. 动静分离，使用CDN，把页面刷新页面传送的数据减少到最小



##### 怎么保证缓存的数据和Mysql的数据的一致性。CAP理论

1. 在分布式系统中，C 一致性 A 可用性 P 分区容忍性（多个服务器的时候，通信延迟是存在）
2. P是通信延迟，一般来讲，这个是存在的，所以我们只能从 C 和 A中做出权衡
3. 选择了 C 一致性的话，修改的时候，多个服务器就要同时被锁定，失去了 A
4. 选择了 A 可用性的锁，多个服务器肯定不能被锁定，所以失去了 C

##### BASE理论：

1. BA 基本可用性， S 软状态 E 最终一致性
2. BASE理论是 一致性 和 可用性做出的权衡，即，不强制存在强一致性，我们只保证最终一致性



秒杀系统就是有大量的读请求和写请求

1. 写请求的瓶颈一般是存储层，利用CAP利用，做出权衡
2. 读请求

##### 热点数据：分为静态热点数据、动态热点数据。

1. 静态热点数据就是能够提前预测的数据，可以通过买家报名的方式，提前对热点数据打上标签。`做预热`
2. 动态热点数据就不能提前预测，比如抖音上突然某种产品火了，这里就只能做限流处理，进行保护。也需要发现动态热点数据。可以分析各个环节中间件的热点key，比如 Nginx的热点URL(https://www.jianshu.com/p/537a0bddda94)



##### 削峰的本质是延迟对请求的处理，让服务器处理的更加平滑。有几种方式：

1. 无损操作（不会丢弃用户的请求）：
   1. 排队： 消息队列，线程池加锁等待，先进先出的内存排队算法，把请求序列话到文件中，然后再顺序的读取
   2. 答题：答题的图片也可以做成CDN
   3. 分层过滤：对读数据不做强一致校验，根据CAP，因为这样子会造成性能瓶颈。限流保护
2. 有损操作：限流



尽量将不影响性能的检查条件提前，如用户是否具有秒杀资格、商品状态是否正常、用户答题是否正确、秒杀是否已经结束、是否非法请求、营销等价物是否充足等；

##### 检查的条件包括：

1. 用户是否存在
2. 秒杀答题对不对
3. 是否拿到了令牌
4. 秒杀是否已经结束
5. 秒杀的商品是否是否在内存中



服务器端的性能一般和QPS相关。

总 QPS =（1000ms / `响应时间`）× `线程数量`

对大部分的Web系统来说，响应时间一般是由CPU执行时间和等待时间（RPC、IO、Sleep、Wait）组成的。

在实际情况下，等待时间对QPS的影响不大，因为等待的时间，其他线程也会有CPU，这点就可以弥补。对QPS影响真正大的是，CPU的执行时间。

如果`减少CPU的执行时间`，就可以增加一倍的QPS。

在多线程的场景下，`线程数 = 2 * CPU核数 + 1`。当然，最好的办法是通过性能测试来发现最佳的线程数。



CPU的利用率不高，是否有太多的锁？



##### 如何发现瓶颈， 并提升

1. 对服务器而言，出现瓶颈会有很多地方，IO、内存、CPU。在秒杀场景中，瓶颈更多是在CPU上。
2. 如何解决CPU瓶颈呢？ 利用CPU诊断工具发现CPU的消耗，最常用的有JProfiler 和 Yourkit 这两个工具。看看哪个函数执行时间最长，然后做定制优化。

![image-20200824171135887](_img/image-20200824171135887.png)

还可从考虑从以下方面进行调整：【垂直扩展】

1. 提升硬件条件：CPU核数、主频、内存、磁盘I/O、SSD、网卡等
2.  JVM性能调优
3. 缓存



##### 如果用户买后不付款，怎么确保不少买

1. 在Redis中，将订单id放到一个Set中
2. 30分钟后流量没有那么多了，每隔1分钟查一下Set的容量，根据Set容量重新设置库存



##### 为什么要设置令牌的数量？

1. 是库存的3倍，或者5倍。因为太多后面也抢不到
2. 而且，如果说秒杀的商品有几万件，比如牛奶，那太多的令牌会占用服务端资源



预扣库存方案中如何确保十分钟后库存自动解冻？定时任务还是会有延迟吧？

自动解冻可以在应用程序中设置一个定时器来定时扫描数据库的下单时间，来比较是否已经超时延时一点问题也不大，因为超时时间也是人为主观设置的

```java
	@Override
    public String generateSecondKillToken(Integer promoId,Integer itemId,Integer userId) {

        //判断是否库存已售罄，若对应的售罄key存在，则直接返回下单失败
        if(redisTemplate.hasKey("promo_item_stock_invalid_"+itemId)){
            return null;
        }
        PromoDO promoDO = promoDOMapper.selectByPrimaryKey(promoId);

        //dataobject->model
        PromoModel promoModel = convertFromDataObject(promoDO);
        if(promoModel == null){
            return null;
        }

        //判断当前时间是否秒杀活动即将开始或正在进行
        if(promoModel.getStartDate().isAfterNow()){
            promoModel.setStatus(1);
        }else if(promoModel.getEndDate().isBeforeNow()){
            promoModel.setStatus(3);
        }else{
            promoModel.setStatus(2);
        }
        //判断活动是否正在进行
        if(promoModel.getStatus().intValue() != 2){
            return null;
        }
        //判断item信息是否存在
        ItemModel itemModel = itemService.getItemByIdInCache(itemId);
        if(itemModel == null){
            return null;
        }
        //判断用户信息是否存在
        UserModel userModel = userService.getUserByIdInCache(userId);
        if(userModel == null){
            return null;
        }

        //获取秒杀大闸的count数量
        long result = redisTemplate.opsForValue().increment("promo_door_count_"+promoId,-1);
        if(result < 0){
            return null;
        }
        //生成token并且存入redis内并给一个5分钟的有效期
        String token = UUID.randomUUID().toString().replace("-","");

        redisTemplate.opsForValue().set("promo_token_"+promoId+"_userid_"+userId+"_itemid_"+itemId,token);
        redisTemplate.expire("promo_token_"+promoId+"_userid_"+userId+"_itemid_"+itemId,5, TimeUnit.MINUTES);

        return token;
    }
```

```java
	@RequestMapping(value = "/createorder",method = {RequestMethod.POST},consumes={CONTENT_TYPE_FORMED})
    @ResponseBody
    public CommonReturnType createOrder(@RequestParam(name="itemId")Integer itemId,
                                        @RequestParam(name="amount")Integer amount,
                                        @RequestParam(name="promoId",required = false)Integer promoId,
                                        @RequestParam(name="promoToken",required = false)String promoToken) throws BusinessException {

        if(!orderCreateRateLimiter.tryAcquire()){
            throw new BusinessException(EmBusinessError.RATELIMIT);
        }

        String token = httpServletRequest.getParameterMap().get("token")[0];
        if(StringUtils.isEmpty(token)){
            throw new BusinessException(EmBusinessError.USER_NOT_LOGIN,"用户还未登陆，不能下单");
        }
        //获取用户的登陆信息
        UserModel userModel = (UserModel) redisTemplate.opsForValue().get(token);
        if(userModel == null){
            throw new BusinessException(EmBusinessError.USER_NOT_LOGIN,"用户还未登陆，不能下单");
        }
        //校验秒杀令牌是否正确
        if(promoId != null){
            String inRedisPromoToken = (String) redisTemplate.opsForValue().get("promo_token_"+promoId+"_userid_"+userModel.getId()+"_itemid_"+itemId);
            if(inRedisPromoToken == null){
                throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"秒杀令牌校验失败");
            }
            if(!org.apache.commons.lang3.StringUtils.equals(promoToken,inRedisPromoToken)){
                throw new BusinessException(EmBusinessError.PARAMETER_VALIDATION_ERROR,"秒杀令牌校验失败");
            }
        }

        //同步调用线程池的submit方法
        //拥塞窗口为20的等待队列，用来队列化泄洪
        Future<Object> future = executorService.submit(new Callable<Object>() {

            @Override
            public Object call() throws Exception {
                //加入库存流水init状态
                String stockLogId = itemService.initStockLog(itemId,amount);


                //再去完成对应的下单事务型消息机制
                if(!mqProducer.transactionAsyncReduceStock(userModel.getId(),itemId,promoId,amount,stockLogId)){
                    throw new BusinessException(EmBusinessError.UNKNOWN_ERROR,"下单失败");
                }
                return null;
            }
        });

        try {
            future.get();
        } catch (InterruptedException e) {
            throw new BusinessException(EmBusinessError.UNKNOWN_ERROR);
        } catch (ExecutionException e) {
            throw new BusinessException(EmBusinessError.UNKNOWN_ERROR);
        }

        return CommonReturnType.create(null);
    }
```



```java
	private RateLimiter orderCreateRateLimiter;

    @PostConstruct
    public void init(){
        executorService = Executors.newFixedThreadPool(20);

        orderCreateRateLimiter = RateLimiter.create(300);

    }

		Future<Object> future = executorService.submit(new Callable<Object>() {

            @Override
            public Object call() throws Exception {
                //加入库存流水init状态
                String stockLogId = itemService.initStockLog(itemId,amount);


                //再去完成对应的下单事务型消息机制
                if(!mqProducer.transactionAsyncReduceStock(userModel.getId(),itemId,promoId,amount,stockLogId)){
                    throw new BusinessException(EmBusinessError.UNKNOWN_ERROR,"下单失败");
                }
                return null;
            }
        });
```

