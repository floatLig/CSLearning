
在实际中，很多业务不是一条SQL语句就能完成的。有些时候需要多条SQL语句一起使用，如`转账`，`即一个事务`。而这个事务`具有原子性：要么全做，要么全不做`。

但是很多时候有可能因为一些原因，出现异常等，使得一些语句提交到数据库了，但是一些语句没有提交到数据库。这个时候就要求我们进行`事务控制`。

我们可以用前面讲的`AOP进行事务控制`。

## 1. AOC实现转账事务控制

先来看代码【`AOP进行事务控制---转账`】：

（很多代码跟前面一样，只写出关键代码）

用到的数据库表是account_spring表，表结构如下：

```text
+-------+-------------+------+-----+---------+----------------+  
| Field | Type        | Null | Key | Default | Extra          |  
+-------+-------------+------+-----+---------+----------------+  
| id    | int(11)     | NO   | PRI | NULL    | auto_increment |  
| name  | varchar(40) | YES  |     | NULL    |                |  
| money | float       | YES  |     | NULL    |                |  
+-------+-------------+------+-----+---------+----------------+  
```

pom.xml

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.44</version>
        </dependency>

        <dependency>
            <groupId>commons-dbutils</groupId>
            <artifactId>commons-dbutils</artifactId>
            <version>1.4</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>c3p0</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.1.2</version>
        </dependency>

        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.7</version>
        </dependency>
    </dependencies>
```

AccountServiceImpl类：

```java
@Service("accountService")
public class AccountServiceImpl implements AccountService {

    @Resource(name = "accountDao")
    private AccountDao accountDao;

    public void transfer(String sourceName, String destinationName, int money) {
        accountDao.lessMoney(sourceName,money);
        //用10/0模拟出现异常，看看出现异常有没有rollback，没有出现异常能不能正常commit
//        int a = 10/0;
        accountDao.increaseMoney(destinationName,money);
    }
}
```

AccountDaoImpl类

```java
@Repository("accountDao")
public class AccountDaoImpl implements AccountDao {

    @Autowired
    @Qualifier("runner")
    private QueryRunner runner;

    @Autowired
    @Qualifier("connectionUtils")
    private ConnectionUtils connectionUtils;

    public void increaseMoney(String destinationName, int money) {
        try {
            runner.update(connectionUtils.getThreadConnection(),"update account_spring set money = money+? where name = ?", money,destinationName);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void lessMoney(String sourceName, int money) {
        try {
            runner.update(connectionUtils.getThreadConnection(),"update account_spring set money = money-? where name = ?", money,sourceName);
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }
}
```

这里用到的JDBC封装是：`dbutils`【需引入注解】

ConnectionUtils类

```java
@Component("connectionUtils")
public class ConnectionUtils {
    private ThreadLocal<Connection> threadLocal = new ThreadLocal<Connection>();

    @Resource(name = "dataSource")
    private DataSource dataSource;

    /**
     * 获取当前线程的连接
     * @return
     */
    public Connection getThreadConnection(){
        try {
            //1.先从ThreadLocal上获取
            Connection connection = threadLocal.get();
            //2.判断当前线程上是否有连接
            if (connection == null) {
                //3.从数据源中获取一个连接，并且存入ThreadLocal中
                connection = dataSource.getConnection();
                threadLocal.set(connection);
            }
            //4.返回当前线程上的连接
            return connection;
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    public void removeConnection(){
        threadLocal.remove();
    }
}
```

TransactionManager类

```java
@Component("transactionManager")
public class TransactionManager {

    @Resource(name = "connectionUtils")
    private ConnectionUtils connectionUtils;

    /**
     * 开启事务
     */
    public void beginTransaction(){
        try {
            System.out.println("连接");
            connectionUtils.getThreadConnection().setAutoCommit(false);
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    /**
     * 提交事务
     */
    public void commit(){
        try {
            connectionUtils.getThreadConnection().commit();
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    /**
     * 回滚事务
     */
    public void rollback(){
        try {
            connectionUtils.getThreadConnection().rollback();
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    /**
     * 释放事务
     */
    public void release(){
        try {
            connectionUtils.getThreadConnection().close();
            connectionUtils.removeConnection();
        }catch (Exception e){
            e.printStackTrace();
        }
    }

}
```

bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:component-scan base-package="com.itheima"/>

    <bean id="runner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype">
<!--        <constructor-arg name="ds" ref="dataSource"/>-->
    </bean>

    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/myproject_textdb?characterEncoding=utf8"/>
        <property name="user" value="root"/>
        <property name="password" value="uber"/>
    </bean>

    <aop:config>
        <aop:pointcut id="accountServicePointCut" expression="execution(* com.itheima.service.impl.*.*(..))"/>
        <aop:aspect id="transactionManager" ref="transactionManager">
            <aop:before method="beginTransaction" pointcut-ref="accountServicePointCut"/>
            <aop:after-returning method="commit" pointcut-ref="accountServicePointCut"/>
            <aop:after-throwing method="rollback" pointcut-ref="accountServicePointCut"/>
            <aop:after method="release" pointcut-ref="accountServicePointCut"/>
        </aop:aspect>
    </aop:config>

</beans>
```

可以看到，`transactionManager作为切面，accountServiceImpl作为切点`，将commit,rollback放在关键SQL之间。

test

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:bean.xml")
public class testAccount {

    //这里的AccountService是接口类，不能写成实例类。否则，则会出现问题。

    @Autowired
    @Qualifier("accountService")
    private AccountService accountService;

    @Test
    public void testTransfer(){
        accountService.transfer("aaa","bbb",100);
    }
}
```

## 2. Spring的事务控制--xml

当然，spring也能实现事务控制，来代替我们上面写的“ConnectionUtils”，“TransactionManager”。

bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd">

    <context:component-scan base-package="com.itheima"/>

    <!-- 配置账户的持久层-->
    <bean id="accountDao" class="com.itheima.dao.Impl.AccountDaoImpl">
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <bean id="runner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype">
    </bean>

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/myproject_textdb?characterEncoding=utf8"/>
        <property name="username" value="root"/>
        <property name="password" value="uber"/>
    </bean>

    <!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--配置事务的通知-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED" read-only="false"/>
            <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
        </tx:attributes>
    </tx:advice>

    <!--配置aop-->
    <aop:config>
        <!--配置切入点表达式-->
        <aop:pointcut id="pt1" expression="execution(* com.itheima.service.impl.*.*(..))"/>
        <!--建立切入点表达式和事务通知的对应关系-->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"/>
    </aop:config>

</beans>
```

上面的bean.xml，我们看到accountDao类下的属性DataSource也用了注入。

这里的属性DataSource是`extends JdbcDaoSupport来的`

```java
public class AccountDaoImpl extends JdbcDaoSupport implements AccountDao {

    public void increaseMoney(String destinationName, int money) {
        super.getJdbcTemplate().update("update account_spring set money = money+? where name = ?", money, destinationName);
    }

    public void lessMoney(String sourceName, int money) {
        super.getJdbcTemplate().update("update account_spring set money = money-? where name = ?", money, sourceName);
    }
}
```

这样，就能删掉上面的“ConnectionUtils”，“TransactionManager”了。

test类

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:bean.xml")
public class testAccount {

    //这里的AccountService是接口类，不能写成实例类。否则，则会出现问题。

    @Autowired
    @Qualifier("accountService")
    private AccountService accountService;

    @Test
    public void testTransfer(){
        accountService.transfer("aaa","bbb",100);
    }
}
```

## 3. Spring的事务控制--注解

`那么，AccountImpl就不能用extends了【因为不能修改源码】，改成私有属性`

```java
@Repository("accountDao")
public class AccountDaoImpl implements AccountDao {

    @Autowired
    private JdbcTemplate jdbcTemplate;
```

AccountServiceImpl类

```java
@Service("accountService")
@Transactional(propagation = Propagation.SUPPORTS,readOnly = true)//只读型事务配置
public class AccountServiceImpl implements AccountService {

    @Resource(name = "accountDao")
    private AccountDao accountDao;

    @Transactional(propagation = Propagation.REQUIRED,readOnly = false)//增删改事务配置
    public void transfer(String sourceName, String destinationName, int money) {
        accountDao.lessMoney(sourceName,money);
        int a = 10/0;
        accountDao.increaseMoney(destinationName,money);
    }
}
```

这里的两个事务配置要特别注意！

从上面可以看到，增删改的事务要单独写出来，如果有很多增删改的事务，那么要写很多这个配置，造成冗余，`这不如xml来的方便【xml利用通配符可以只写一次】`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd">

    <context:component-scan base-package="com.itheima"/>

    <!-- 配置账户的持久层-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <bean id="runner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype">
    </bean>

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/myproject_textdb?characterEncoding=utf8"/>
        <property name="username" value="root"/>
        <property name="password" value="uber"/>
    </bean>

    <!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <tx:annotation-driven transaction-manager="transactionManager"/>

</beans>
```

## 4. 全注解方式实现转账

原来类的配置不用改变，现在考虑的是如何将xml的内容变成Java类

SpringConfiguration配置类

```java
@Configuration
@ComponentScan(value = "com.itheima")
@Import({JdbcConfiguration.class,TransactionConfiguration.class})
@PropertySource("jdbcConfig.properties")
@EnableTransactionManagement
public class SpringConfiguration {

    @Bean("transactionManager")
    public DataSourceTransactionManager createTransaction(DataSource dataSource){
        return new DataSourceTransactionManager(dataSource);
    }
}
```

JdbcConfiguration配置类【被SpringConfiguration类导入】

```java
@Configuration
public class JdbcConfiguration {
    @Value("${jdbc.driver}")
    private String driver;

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String userName;

    @Value("${jdbc.password}")
    private String password;

    @Bean(name = "dataSource")
    public DataSource createDataSource(){
        DriverManagerDataSource driverManagerDataSource = new DriverManagerDataSource();
        driverManagerDataSource.setDriverClassName(driver);
        driverManagerDataSource.setUrl(url);
        driverManagerDataSource.setUsername(userName);
        driverManagerDataSource.setPassword(password);
        return driverManagerDataSource;
    }

    @Bean(name = "jdbcTemplate")
    public JdbcTemplate createJdbcTemplate(DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }
}
```

TransactionConfiguration配置类【被SpringConfiguration类导入】

```java
@Configuration
public class TransactionConfiguration {

    @Bean(name = "transactionManager")
   public PlatformTransactionManager createTransactionManager(DataSource dataSource){
        //参数会默认为autowired
       return new DataSourceTransactionManager(dataSource);
   }
}
```

jdbcConfiguration.properties【被SpringConfiguration类导入,JdbcConfiguration类调用】

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/myproject_textdb?characterEncoding=utf8
jdbc.username=root
jdbc.password=root
```

当然，test类，ContextConfiguration也要换成(classes = SpringConfiguration.class)

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfiguration.class)
public class testAccount {
    ****
}
```

从上面可以看到，全注解的方式，也是用一个主配置类（SpringConfiguration类），import其他配置类，然后在测试方法调用这个主配置类的class。

`可以看到，不如xml方便。`

---

配置事务的属性
            isolation:用于指定事务的隔离级别。默认值为DEFAULT，表示使用数据库的默认隔离级别
            propagation:用于指定事务的传播行为。默认值为REQUIRED,表示一定会有事务，增删改的选择。查询方法为SUPPORTS
            read-only：用于指定事务是否只读。只有查询方法才能设置为true，默认值为false，表示读写
            timeout:用于指定事务的超时时间，默认值为-1，表示永不超时，如果指定了数值，以秒为单位
            rollback-for：用于指定一个异常，当产生该异常时，事务回滚，产生其他异常时，事务不回滚。不设置的时候表示任何异常都回滚
            no-rollback-for：用于指定一个异常，当产生该异常时，事务不回滚，产生其他异常时事务回滚。不设置的时候表示任何异常都回滚

