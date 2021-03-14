### 使用缓存

1. 首先在主类中使用注解 @EnableCaching

```java
@EnableCaching
public class Springboot01CacheApplication {

	public static void main(String[] args) {
		SpringApplication.run(Springboot01CacheApplication.class, args);
	}
}

```



### Spring Boot 整合 Redis

1. 拉取镜像

   首先配置镜像加速，这里使用阿里云的镜像加速：<https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors>

   ```shell
   docker pull redis
   ```

2. 运行 redis 镜像

   需要添加什么参数呢？从系统角度考虑：内存 -d  --name, 文件 -v, 网络 -p , 

```shell
docker run -p 6379:6379 --name redis \
> -v /mydata/redis/data:/data \
> -v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
> -d redis redis-server /etc/redis/redis.conf
```

3. spring boot 导入相关依赖，并在配置文件中配置相关的属性

   上maven 仓库查找 redis 相关的依赖，在项目中进行导包。

   在配置文件中配置 Redis的端口。（在配置文件中直接打Redis，根据提示，看看需要什么配置。相关的配置都在 `RedisProperties` 这个类下）

   测试相关的代码：

   ```java
   
   @SpringBootTest
   class SpringbootLearningApplicationTests {
   
       @Autowired
       DataSource dataSource;
   
       @Autowired
       StringRedisTemplate stringRedisTemplate;
   
       @Autowired
       EmployeeMapper employeeMapper;
   
       @Test
       void testMysqlConnect() throws SQLException {
           Connection connection = dataSource.getConnection();
           System.out.println(connection);
       }
   
       @Test
       void testRedisConnect() {
           // stringRedisTemplate.opsForValue().set("hello", "helloWorld");
   
           // System.out.println(stringRedisTemplate.opsForValue().get("hello"));
   
           // Employee employee = employeeMapper.getEmployee(1);
           // stringRedisTemplate.opsForValue().set("obj", String.valueOf(employee));
   
           System.out.println(stringRedisTemplate.opsForValue().get("obj"));
       }
   
   }
   
   ```

   

4. 如果想要看原理，先看 `RedisAutoConfiguration` 的代码

5. Redis 和 @Cacheable搭配使用：

```java
// 在Service层开启缓存

@CacheConfig(cacheNames="emp"/*,cacheManager = "employeeCacheManager"*/) //抽取缓存的公共配置
@Service
public class EmployeeService {

    @Autowired
    EmployeeMapper employeeMapper;

    /**
     * 将方法的运行结果进行缓存；以后再要相同的数据，直接从缓存中获取，不用调用方法；
     * CacheManager管理多个Cache组件的，对缓存的真正CRUD操作在Cache组件中，每一个缓存组件有自己唯一一个名字；
     *

     *
     * 原理：
     *   1、自动配置类；CacheAutoConfiguration
     *   2、缓存的配置类
     *   org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration【默认】
     *   org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration
     *   3、哪个配置类默认生效：SimpleCacheConfiguration；
     *
     *   4、给容器中注册了一个CacheManager：ConcurrentMapCacheManager
     *   5、可以获取和创建ConcurrentMapCache类型的缓存组件；他的作用将数据保存在ConcurrentMap中；
     *
     *   运行流程：
     *   @Cacheable：
     *   1、方法运行之前，先去查询Cache（缓存组件），按照cacheNames指定的名字获取；
     *      （CacheManager先获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建。
     *   2、去Cache中查找缓存的内容，使用一个key，默认就是方法的参数；
     *      key是按照某种策略生成的；默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key；
     *          SimpleKeyGenerator生成key的默认策略；
     *                  如果没有参数；key=new SimpleKey()；
     *                  如果有一个参数：key=参数的值
     *                  如果有多个参数：key=new SimpleKey(params)；
     *   3、没有查到缓存就调用目标方法；
     *   4、将目标方法返回的结果，放进缓存中
     *
     *   @Cacheable标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，
     *   如果没有就运行方法并将结果放入缓存；以后再来调用就可以直接使用缓存中的数据；
     *
     *   核心：
     *      1）、使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurrentMapCache】组件
     *      2）、key使用keyGenerator生成的，默认是SimpleKeyGenerator
     *
     *
     *   几个属性：
     *      cacheNames/value：指定缓存组件的名字;将方法的返回结果放在哪个缓存中，是数组的方式，可以指定多个缓存；
     *
     *      key：缓存数据使用的key；可以用它来指定。默认是使用方法参数的值  1-方法的返回值
     *              编写SpEL； #id;参数id的值   #a0  #p0  #root.args[0]
     *              getEmp[2]
     *
     *      keyGenerator：key的生成器；可以自己指定key的生成器的组件id
     *              key/keyGenerator：二选一使用;
     *
     *
     *      cacheManager：指定缓存管理器；或者cacheResolver指定获取解析器
     *
     *      condition：指定符合条件的情况下才缓存；
     *              ,condition = "#id>0"
     *          condition = "#a0>1"：第一个参数的值》1的时候才进行缓存
     *
     *      unless:否定缓存；当unless指定的条件为true，方法的返回值就不会被缓存；可以获取到结果进行判断
     *              unless = "#result == null"
     *              unless = "#a0==2":如果第一个参数的值是2，结果不缓存；
     *      sync：是否使用异步模式
     * @param id
     * @return
     *
     */
    @Cacheable(value = {"emp"}/*,keyGenerator = "myKeyGenerator",condition = "#a0>1",unless = "#a0==2"*/)
    public Employee getEmp(Integer id){
        System.out.println("查询"+id+"号员工");
        Employee emp = employeeMapper.getEmpById(id);
        return emp;
    }

    /**
     * @CachePut：既调用方法，又更新缓存数据；同步更新缓存
     * 修改了数据库的某个数据，同时更新缓存；
     * 运行时机：
     *  1、先调用目标方法
     *  2、将目标方法的结果缓存起来
     *
     * 测试步骤：
     *  1、查询1号员工；查到的结果会放在缓存中；
     *          key：1  value：lastName：张三
     *  2、以后查询还是之前的结果
     *  3、更新1号员工；【lastName:zhangsan；gender:0】
     *          将方法的返回值也放进缓存了；
     *          key：传入的employee对象  值：返回的employee对象；
     *  4、查询1号员工？
     *      应该是更新后的员工；
     *          key = "#employee.id":使用传入的参数的员工id；
     *          key = "#result.id"：使用返回后的id
     *             @Cacheable的key是不能用#result
     *      为什么是没更新前的？【1号员工没有在缓存中更新】
     *
     */
    @CachePut(/*value = "emp",*/key = "#result.id")
    public Employee updateEmp(Employee employee){
        System.out.println("updateEmp:"+employee);
        employeeMapper.updateEmp(employee);
        return employee;
    }

    /**
     * @CacheEvict：缓存清除
     *  key：指定要清除的数据
     *  allEntries = true：指定清除这个缓存中所有的数据
     *  beforeInvocation = false：缓存的清除是否在方法之前执行
     *      默认代表缓存清除操作是在方法执行之后执行;如果出现异常缓存就不会清除
     *
     *  beforeInvocation = true：
     *      代表清除缓存操作是在方法运行之前执行，无论方法是否出现异常，缓存都清除
     *
     *
     */
    @CacheEvict(value="emp",beforeInvocation = true/*key = "#id",*/)
    public void deleteEmp(Integer id){
        System.out.println("deleteEmp:"+id);
        //employeeMapper.deleteEmpById(id);
        int i = 10/0;
    }

    // @Caching 定义复杂的缓存规则
    @Caching(
         cacheable = {
             @Cacheable(/*value="emp",*/key = "#lastName")
         },
         put = {
             @CachePut(/*value="emp",*/key = "#result.id"),
             @CachePut(/*value="emp",*/key = "#result.email")
         }
    )
    public Employee getEmpByLastName(String lastName){
        return employeeMapper.getEmpByLastName(lastName);
    }
}
```

```java
// Redis config

@Configuration
public class MyRedisConfig {

    // 自己自定义 RedisTemplate，使用自己定义的 json 数据
    @Bean
    public RedisTemplate<Object, Employee> empRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Employee> template = new RedisTemplate<Object, Employee>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Employee> ser = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(ser);
        return template;
    }
    @Bean
    public RedisTemplate<Object, Department> deptRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Department> template = new RedisTemplate<Object, Department>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Department> ser = new Jackson2JsonRedisSerializer<Department>(Department.class);
        template.setDefaultSerializer(ser);
        return template;
    }



    //CacheManagerCustomizers可以来定制缓存的一些规则
    @Primary  //将某个缓存管理器作为默认的
    @Bean
    public RedisCacheManager employeeCacheManager(RedisTemplate<Object, Employee> empRedisTemplate){
        RedisCacheManager cacheManager = new RedisCacheManager(empRedisTemplate);
        //key多了一个前缀

        //使用前缀，默认会将CacheName作为key的前缀
        cacheManager.setUsePrefix(true);

        return cacheManager;
    }

    @Bean
    public RedisCacheManager deptCacheManager(RedisTemplate<Object, Department> deptRedisTemplate){
        RedisCacheManager cacheManager = new RedisCacheManager(deptRedisTemplate);
        //key多了一个前缀

        //使用前缀，默认会将CacheName作为key的前缀
        cacheManager.setUsePrefix(true);

        return cacheManager;
    }
}
```



### Spring 整合 RabbitMQ



#### 运行RabbitMQ



1. 运行 docker 镜像

```shell
docker pull rabbitmq:3-management

# -management，是带有管理界面, 15672端口是后台管理界面的端口
docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq 87505dc99f21
```

2. 进入 RabbitMQ 的后台管理界面，管理界面的端口是 15672



3. 在 RabbitMQ 后台管理界面创建交换机 <http://192.168.56.2:15672/#/exchanges> 

![image-20201124164052319](C:\Users\25894\AppData\Roaming\Typora\typora-user-images\image-20201124164052319.png)



#### RabbitMQ管理页面创建交换机和队列



创建队列

![image-20201124164244952](C:\Users\25894\AppData\Roaming\Typora\typora-user-images\image-20201124164244952.png)



在 Exchange 中点击队列，将交换机和队列进行绑定

![image-20201124164609057](C:\Users\25894\AppData\Roaming\Typora\typora-user-images\image-20201124164609057.png)



#### RabbitMQ常用功能代码测试



```java
@SpringBootTest
class SpringbootLearningApplicationTests {

    @Autowired
    RabbitTemplate rabbitTemplate;

    @Autowired
    RabbitAdmin rabbitAdmin;

    @Test
    void testMysqlConnect() throws SQLException {
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
    }

    // 测试接受消息
    @Test
    void testSendRabbitMq() {
        // Map<String, Object> map = new HashMap<>();
        // map.put("msg", "这是第一条消息");
        // map.put("data", Arrays.asList("helloWorld", 1));
        // rabbitTemplate.convertAndSend("exchange.direct", "atguigu.news", map);

        Employee employee = new Employee(1, "zzl");
        rabbitTemplate.convertAndSend("exchange.fanout", "",employee);
    }

    // 测试发送消息
    @Test
    void  testReceiveRabbitMQ() {
        Object o = rabbitTemplate.receiveAndConvert("atguigu.news");

        System.out.println(o);
    }

    // 测试创建 交换机和队列
    @Test
    void createExchange() {
        rabbitAdmin.declareQueue(new Queue("admin.queue"));
        rabbitAdmin.declareExchange(new TopicExchange("admin.exchange"));
    }

    // 测试将 交换机和队列进行绑定
    @Test
    void bindExchange() {
        rabbitAdmin.declareBinding(new Binding("admin.queue", Binding.DestinationType.QUEUE,
                "admin.exchange", "admin.*", null));
    }
}
```



#### @RabbitListener 和 @EnableRabbit



使用@RabbitListener进行监听。要使@RabbitListener注解生效，还要在主配置文件中使用@EnableRabbit

```java
@Service
public class RabbitmqService {

    @RabbitListener(queues = "atguigu.news")
    public void rabbitmqListen(Message str) {
        System.out.println(str);
    }
}
```

```java
@SpringBootApplication
@EnableRabbit
@MapperScan("com.atguigu.springbootlearning.mapper")
public class SpringbootLearningApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootLearningApplication.class, args);
    }

}

```



### Spring Boot 整合 Elasticsearch



#### 运行 ES



```shell
docker pull elasticsearch

# -e 配置环境参数，ElasticSerch默认Java堆的大小为2G，测试环境内存不够，所以就设置少了一点
# 9300端口是不同的 es 通信的端口
docker run -e ES_JAVA_OPTS="-Xms32m -Xmx32m" -d -p 9200:9200 -p 9300:9300 --name es01 5acf0e8da90b
```



访问 `http://192.168.56.2:9200/`，如果能返回 json 数据，则说明安装成功



#### 入门



官方文档：<https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html>



使用 REST风格的数据同 es 交互

![image-20201125194424590](C:\Users\25894\AppData\Roaming\Typora\typora-user-images\image-20201125194424590.png)





### SpringBoot 使用任务



#### 异步任务



在主程序开启 `@EnableAsync`

```java
@SpringBootApplication
@EnableAsync
public class SpringbootLearningApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootLearningApplication.class, args);
    }

}
```



Controller

```java
@RestController
public class TastController {

    @Autowired
    AsyncService asyncService;

    @GetMapping("/async")
    String async() {
        try {
            asyncService.async();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        return "Async";
    }
}
```



Service中，使用异步注解`@Async`

```java
@Service
public class AsyncService {

    @Async
    public void async() throws InterruptedException {
        System.out.println(new Date().toString());
        Thread.sleep(5000);
        System.out.println(new Date().toString());

        System.out.println("Hello");
    }
}
```



#### 定时任务



```java
@SpringBootApplication
@EnableScheduling
public class SpringbootLearningApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootLearningApplication.class, args);
    }

}
```

```java
@Service
public class ScheduledService {

    // 每个月每天 10点到11点， 每一分钟，每隔3秒进行输出
    @Scheduled(cron = "*/3 * 10,11 * * *")
    public void hello() {
        System.out.println("定时任务" + new Date().toString());
    }
}
```



##### cron表达式

| **字段** | **允许值**             | **允许的特殊字符** |
| -------- | ---------------------- | ------------------ |
| 秒       | 0-59                   | , -  * /           |
| 分       | 0-59                   | , -  * /           |
| 小时     | 0-23                   | , -  * /           |
| 日期     | 1-31                   | , -  * ? / L W C   |
| 月份     | 1-12                   | , -  * /           |
| 星期     | 0-7或SUN-SAT  0,7是SUN | , -  * ? / L C #   |

| **特殊字符** | **代表含义**               |
| ------------ | -------------------------- |
| ,            | 枚举                       |
| -            | 区间                       |
| *            | 任意                       |
| /            | 步长                       |
| ?            | 日/星期冲突匹配            |
| L            | 最后                       |
| W            | 工作日                     |
| C            | 和calendar联系后计算过的值 |
| #            | 星期，4#2，第2个星期四     |



#### 邮件任务



### SpringBoot 整合 zk

 运行zookeeper

```shell
docker run --name zk01 -p 2181:2181 --restart always -d ea93faa92337
```

