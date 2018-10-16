
#### Application属性文件
SpringApplication将从以下位置加载application.properties文件，并把它们添加到Spring Environment中：
+ 当前目录下的/config子目录
+ 当前目录
+ classpath下的/config包
+ classpath根路径（root）

上面的优先级高于下面的优先级；properties文件高于yaml文件

- - -

#### 注解
**元注解**

>元注解是一种标注在别的注解之上的注解。如果一个注解可以标注在别的注解上，那么这个注解就是元注解。

**Stereotype注解（模式化注解、角色类注解）**

>Stereotype注解是一种在应用中，常被用于声明要扮演某种职责或者角色的注解。

例如：
+ @Repository   常用与Data Access Object或者DAO
+ @Component    被Spring管理的组件的对应注解。任何标注了@Component的组件都会在Spring组件扫描时被扫描到。同样的，任何标注了 被元注解@Component标注过的注解 的组件，也会在Spring组件扫描时被扫描到。例如，@Service就是一种被元注解@Component标注过的注解。

**组合注解**

>组合注解是一种被一个或者多个元注解标注过的注解，用以撮合多个元注解的特性到新的注解


#### @AliasFor

**在同一个注解内使用**
互为别名，值必须一样

~~~
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
 
    @AliasFor("path")
    String[] value() default {};
 
    @AliasFor("value")
    String[] path() default {};
 
    //...
}
~~~

**覆盖元注解**
注解@One有成员A，注解@Two有也有成员A，如果@One本身是被元注解@Two标注的，那么按照命名约定，注解@One中的成员A实际会覆盖注解@Two中的成员A

~~~
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

	@AliasFor(annotation = EnableAutoConfiguration.class)
	Class<?>[] exclude() default {};

    .............
}
~~~



#### @Configuration和@Bean

`@Configuration`是一个类级别的注解，用于表明此对象是一个bean定义的资源。Spring的容器会根据它来生成IOC容器去装配Bean；`@Configuration`类通过public的`@Bean`注解的方法来声明bean。将返回的POJO装配到IOC容器中。调用`@Configuration`类的`@Bean`方法也可以被用于定义inter-bean依赖。

#### Injection inter-bean dependencies
~~~
@Configuration
public class AppConfig {

    @Bean
    public Foo foo() {
        return new Foo(bar());
    }

    @Bean
    public Bar bar() {
        return new Bar();
    }

}
~~~

>声明inter-bean依赖的方法只会在`@Bean`声明在`@Configuration`类下时才会生效。你不能使用`@Component`类声明inter-bean依赖。

#### @Import
`Import`注解允许从其它配置类中加载`@Bean`的配置：
~~~
@Configuration
public class ConfigA {

     @Bean
    public A a() {
        return new A();
    }

}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }

}
~~~
现在，当实例化上下文时，不需要指定ConfigA.class 和 ConfigB.class 了，仅仅 ConfigB 需要被显式提供：
~~~
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
~~~
这种方式简化了容器的实例化，仅仅是一个类需要被处理，而不是需要开发人员在构造时记住很多大量的@Configuration类。



#### @Autowired

@Autowired规则：首先它会根据类型找到对应的Bean，如果对应类型的Bean不是唯一的，那么它会根据其属性名称和Bean的名称进行匹配。如果匹配得上，就会使用该Bean；如果还无法匹配，就会抛出异常。

@Autowired是一个默认必须找到对应Bean的注解，如果不能确定其标注属性一定会存在并且允许这个被标注的属性为null，那么你可以配置@Autowired属性required为false

#### 消除歧义性--@Primary和@Quelifier

@Primary的含义：告诉SpringIOC容器，当发现有多个同样类型的Bean时，请优先使用我进行注入。

然而，有时候@Primary也可以使用在多个类上，那么可以使用@Quelifier。它的配置项value需要一个字符串去定义，它将于@Autowired组合在一起，通过类型和名称一起找到Bean。Bean名称在Spring IOC容器中是唯一的标识，通过这个就可以消除歧义性了。

~~~
@Autowired
@Qualifier("dog")
private Animal animal=null;
~~~



带有参数的构造方法

~~~
private Animal animal=null;
public BusinessPerson(@Autowired @Qualifier("dog") Animal animal){
    this.animal=animal;
}
~~~



#### 生命周期

+ Spring通过 我们的配置，如@ComponentScan定义的扫描路径去找到带有@Component的类，这个过程就是一个资源定位的过程。
+ 一旦找到了资源，那么它就开始解析，并且将定义的信息保存起来。注意，此时还没有初始化Bean，也就没有Bean的实例，它有的仅仅是Bean的定义。
+ 然后就会把Bean定义发布到Spring IOC容器中。此时，IOC容器也只有Bean的定义，还是没有Bean实例生成。
+ 创建Bean的实例对象
+ 通过@Autowired注入各种资源

> 资源定位 -> Bean定义 -> 发布Bean定义 -> 实例化 -> 依赖注入

ComponentScan中有一个配置项lazyInit，只可以配置Boolean值，且默认值为false，也就是默认不进行延迟初始化。因此在默认的情况下Spring会对Bean进行实例化和依赖注入对应的属性值。设置为true时，只有当我们取出Bean的时候才做初始化和依赖注入等操作

![Spring Bean的生命周期](/image/Bean生命周期.png)



#### AOP

![](/image/AOP流程图.png)

~~~
/**
* 取代原有事件方法
* @param invocation -- 回调参数，可以通过它的processed方法，回调原有事件
* @return 原有事件返回对象
public Object around(Invocation invocation){
    System.out.println("around before...");
    Object obj=invocation.processed();
    System.out.println("around after ....");
    return obj;
}
~~~

processed()方法会以反射的形式去调用原有的方法。



![](/image/SpringAOP流程约定.png)

> 织入（weaving）：它是一个通过动态代理技术，为原有服务对象生成代理对象，然后将与切点定义匹配的连接点拦截，并按约定将各类通知织入约定流程的过程。

#### 切点定义

我们会在@Before、@After......等注解定义一个正则表达式，这个正则表达式的作用就是定义什么时候启用AOP，Spring会通过这个正则表达式去匹配、去确定对应的方法（连接点）是否启用切面编程。但每个注解可能都重复写了同一个正则表达式，为了克服这个问题，Spring定义了切点的概念，切点的作用就是向Spring描述哪些类的哪些方法需要启用AOP编程。

~~~
@Aspect
public class MyAspect{
    @Pointcut(execution(* com........))
    public void pointCut(){
        
    }
    
    @Before("pointCut()")
    public void before(){
        
    }
    
    @DeclareParents(value="xxx.xxx.UserServiceImpl+"，defaultImpl=UserValidatorImpl.class)
    public UserValidator userValidator
}
~~~



value：指向你要增强功能的目标对象

defaultImpl：引入增强功能的类



~~~
UserValidator userValidator=(UserValidator)userService;
~~~

Proxy.newProxyInstance()中，Spring会把UserService和UserValidator两个接口 传递进去，让代理对象挂到这两个接口下，这样这个代理对象就能够相互转换并且使用它们的方法 



> 在自调用的过程中，是类自身的调用，而不是代理对象去调用，那么就不会产生AOP，这样Spring就不能把你的代码织入到约定的流程中



----



#### MyBatis

MyBatis是一个基于SqlSessionFactory构建的框架。对于SqlSessionFactory而言，它的作用是生成SqlSession接口对象，这个接口对象是MyBatis操作的核心。

![](/image/MyBatis配置内容结构图.png)



#### 事务

Spring利用其AOP为我们提供了一个数据库事务的约定流程。通过这个约定流程就可以减少大量的冗余代码和一些没有必要的try...catch...finally语句，让开发者能够更加集中与业务的开发，这样开发的代码可读性就更高，也更好维护。

通过@Transactional进行标注启用数据库事务功能。配置内容是在Spring IOC容器在加载时就会将这些配置信息解析出来，然后把这些信息存到事务定义器（TransactionDefinition接口的实现类）里，并且记录哪些类或者方法需要启动事务功能，采取什么策略去执行事务。

![](/image/Spring数据库事务约定.png)



#### Spring事务管理器

事务的打开、回滚和提交是由事务管理器来完成的。在Spring中，事务管理器的顶层接口为PlatformTransactionManager

![](D:\note\image\Spring事务管理器.png)



~~~
package org.springframework.transaction;

public interface PlatformTransactionManager {
	// 获取事务
    TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;
	// 提交事务
    void commit(TransactionStatus status) throws TransactionException;
	// 回滚事务
    void rollback(TransactionStatus status) throws TransactionException;

}
~~~



Spring在事务管理时，就是将这些方法按照约定 织入对应的流程中，其中getTransaction方法的参数是一个事务定义器（TransactionDefinition），它是依赖于我们配置的@Transacional的配置项生成的，于是通过它就能够设置事务的属性了。



在SpringBoot中，当你依赖于mybatis-spring-boot-starter之后，它会自动创建一个DataSource-TransactionManager对象，作为事务管理器。

#### 数据库事务基本特征

+ Atomic(原子性)：事务中包含的操作被看作一个整体的业务单元，这个业务单元中的操作要么全部成功，要么全部失败
+ Consistency(一致性)：事务在完成时，必须使所有的数据都保持一致状态
+ Isolation(隔离性)：多个应用程序线程同时访问同一数据，这样数据库同样的数据就会在各个不同的事务中被访问，这样会产生丢失更新。所以数据库定义了隔离级别的概念
+ Durabillity(持久性)：事务结束后，所有的数据会固化到一个地方

一般而言，存在两种类型的丢失更新

![](D:\note\image\第一类丢失更新.png)

T5时刻事务1回滚，导致原本库存为99的变为了100，显然事务2的结果就丢失了。

> 对于这样一个事务回滚，另外一个事务提交而引发的数据不一致的情况，称为`第一类丢失更新`（如今数据库系统不会出现这种情况）



![](D:\note\image\第二类丢失更新.png)

T5时刻事务1提交的事务，就会引发事务2提交结果的丢失

> 多个事务都提交引发的丢失更新称为`第二类丢失更新`



#### 隔离级别

为了压制丢失更新，数据库标准提出了4类隔离级别，在不同的程度上压制丢失更新

**未提交读（read uncommitted）**

允许一个事务读取另外一个事务没有提交的数据。    最大的坏处是出现脏读

![](D:\note\image\脏读.png)

在T3时刻，因为采用未提交读，所以事务2可以读取事务1未提交的库存数据为1，这里当它扣减库存后则数据为0，然后它提交了事务，库存就变为了0，而事务1在T5时刻回滚事务，因为第一类丢失更新已经被克服，所以它不会将库存回滚到2，那么最后的结果就变为了0



**读写提交（read committed）**

一个事务只能读取另外一个事务已经提交的数据

 ![](D:\note\image\克服脏读.png)

可以克服脏读，但会出现重复读的问题



![](D:\note\image\不可重复读.png)

在T5时刻，扣减库存的时候发现库存为0，于是就无法扣减库存了。这里的问题在于事务2之前认为可以扣减，而到扣减那一步却发现已经不可以扣减，于是库存对于事务2而言是一个可变话的值，这样的现象我们称为不可重复读。

**可重复读**

![](D:\note\image\克服不可重复读.png)

事务2在T3时刻尝试读取库存，但是此时这个库已经被事务1事先读取，所以这个时候数据库就阻塞它的读取，直至事务1提交，事务2才能读取库存的值



![](D:\note\image\幻读.png)

幻读不是针对一条数据库记录而言，而是多条记录。而可重复读是针对数据库的单一条记录，例如，商品的库存是以数据库里面的一条记录存储的，它可以产生可重复读，而不能产生幻读。



**串行化（Serializable）**

要求所有的SQL都会按照顺序执行，能够完全保证数据的一致性

**使用合理的隔离级别**

![](D:\note\image\隔离级别和可能发生的现象.png)

在现实中一般而言，选择隔离级别会以读写提交为主，它能够防止脏读，而不能避免不可重复读和幻读。

对于隔离级别，不同的数据库的支持也是不一样的。例如，Oracle只能支持读写提交和串行化，而MySQL则能够支持4种，对于Oracle默认的隔离级别为读写提交，MySQL则是可重复读。

**传播行为**

> 传播行为是方法之间调用事务采取的策略问题。

![](D:\note\image\事务的传播行为.png)

~~~
package org.springframework.transaction.annotation;

public enum Propagation {

	/**
	* 需要事务，它是默认传播行为，如果当前存在事务，就沿用当前事务，
	* 否则新建一个事务允许子方法
	*/
	REQUIRED(TransactionDefinition.PROPAGATION_REQUIRED),

	/**
	* 支持事务，如果当前存在事务，就沿用当前事务，
	* 如果不存在，则继续采用无事务的方式允许子方法
	*/
	SUPPORTS(TransactionDefinition.PROPAGATION_SUPPORTS),

	/**
	* 必须使用事务，如果当前没有事务，则会抛出异常，
	* 如果存在当前事务，就沿用当前事务
	*/
	MANDATORY(TransactionDefinition.PROPAGATION_MANDATORY),

	/**
	* 无论当前事务是否存在，都会创建新事务运行方法，
	* 这样新事务就可以拥有新的锁和隔离级别等特性，与当前事务湘湖独立
	*/
	REQUIRES_NEW(TransactionDefinition.PROPAGATION_REQUIRES_NEW),

	/**
	* 不支持事务，当前存在事务时，将挂起事务，运行方法
	*/
	NOT_SUPPORTED(TransactionDefinition.PROPAGATION_NOT_SUPPORTED),

	/**
	* 不支持事务，如果当前方法存在事务，则抛出异常，否则继续使用无事务机制运行
	*/
	NEVER(TransactionDefinition.PROPAGATION_NEVER),

	/**
	* 在当前方法调用子方法时，如果子方法发生异常，
	* 只回滚子方法执行过的SQL，而不回滚当前方法的事务
	*/
	NESTED(TransactionDefinition.PROPAGATION_NESTED);

}
~~~



常用的传播行为：REQUIRED、REQUIRES_NEW和NESTED

在大部分的数据库中，一段SQL语句中可以设置一个标志位，然后后面的代码执行时如果有异常，只是回滚到这个标志位的数据状态，而不会让这个标志位之前的代码也回滚。这个标志位，在数据库的概念中被称为保存点（save point）。

Spring也是使用保存点技术来完成让子事务回滚而不致使当前事务回滚的工作。注意，并不是所有的数据库都支持保存点技术，因此Spring内部有这样的规则：

> 当数据库支持保存点技术时，就启用保存点技术；如果不支持，就新建一个事务去运行你的代码，即等价于REQUIRES_NEW传播行为

NESTED传播行为和REQUIRES_NEW还是有区别的。NESTED传播行为会沿用当前事务的隔离级别和锁等特性，而REQUIRES_NEW则可以拥有自己独立的隔离级别和锁等特性



**@Transactional自调用失效问题**

Spring数据库事务的约定，其实现原理是AOP，而AOP的原理是动态代理。

> 在自调用的过程中，是类自身的调用，而不是代理对象去调用，那么就不会产生AOP



#### 悲观锁

在高并发中出现超发现象，根本在于贡献的数据被多个线程所修改，无法保证其执行的顺序。如果一个数据库事务读取到数据后，就将数据直接绑定，不允许别的线程进行读写操作，直至当前数据库事务完成后才释放这条数据的锁，则不会出现超发问题。

> 在SQL的最后加入for update   。   这样在数据库事务执行的过程中，就会锁定查询出来的数据，其它的事务将不能再对其进行读写，这样就避免了数据的不一致       

![](/image/悲观锁等待图示.png)

悲观锁是使用 数据库内部的锁对记录进行加锁，从而使得其他事务等待以保证数据的一致。但这样会造成过多的等待和事务上下文的切换导致缓慢。



**乐观锁**

~~~
<update id="decreaseProduct">
	update t_product set stock = stock - #{quantity},
	version = version +1
	where id = #{id}  and versioin=#{version}
</update>
~~~



使用多线程的概念CAS（Compare and Swap），并用版本号解决ABA问题

![](D:\note\image\使用版本号解决ABA问题.png)

但是，因为加入了版本号的判断，所以大量的请求得到了失败的结果，而且这个失败率有点高。

为了处理这个结果，乐观锁还可以引入重入机制，也就是一旦更新失败，就重新做一次，所以有时候 也可以称乐观锁为可重入的锁。其原理是一旦发现版本号被更新，不是结束请求，而是重新做一次乐观锁流程，直至成功为止。但是这个流程的重入会带来一个问题，那就是可能造成大量的SQL被执行。为了克服这个问题，一般会考虑使用限制时间或者重入次数的办法，以压制过多的SQL被执行

~~~
// 当前时间
long start = System.currentTimeMillis();
// 循环尝试直至成功
while (true) {
    // 循环时间
    long end = System.currentTimeMillis();
    // 如果循环时间大于100毫秒返回终止循环
    if (end - start > 100) {
    	return false;
    }
    // 执行一些操作
    ..............
}
//
~~~



**总结**

乐观锁是一种不使用数据库锁的机制，并且不会造成线程的阻塞，只是采用多版本号机制来实现。但是，因为版本的冲突造成了请求失败的概率剧增，所以这时往往需要通过重入的机制将请求失败的概率降低。但是，多次的重入会带来过多执行SQL问题。为了克服这个问题，可以考虑使用按时间戳或者限制重入次数的办法。



****



#### Redis

在Java中与Redis连接的驱动存在很多种，目前比较广泛使用的是Jedis

Spring提供了一个RedisConnectionFactory接口，通过它可以生成一个RedisConnection接口对象，而RedisConnection接口对象是对Redis底层接口的封装。例如Spring会提供RedisConnection接口的实现类JedisConnection去封装原有的Jedis（redis.client.jedis.Jedis）对象

![](D:\note\image\Spring对Redis的类设计.png)



#### RedisTemplate

RedisTemplate是一个强大的类，首先它会自动从RedisConnectionFactory工厂中获取连接，然后执行对应的Redis命令，在最后还会关闭Redis的连接。



 ![](D:\note\image\Spring关于Redis的序列化器设置.png)

JdkSerializationREdisSerializer是RedisTemplate默认的序列化器

![](D:\note\image\spring-data-redis序列化器实现原理.png)

![](D:\note\image\RedisTemplate中的序列化器属性.png)



~~~

~~~

 ![](D:\note\image\redis01.png)

RedisTemplate会默认使用JdkSerializationRedisSerializer进行序列化键值，Redis服务器存入的是一个经过序列化后的特殊字符串，导致上图得到的序列化后的二进制字符串



~~~
RedisSerializer<String> stringRedisSerializer = redisTemplate.getStringSerializer();
redisTemplate.setKeySerializer(stringRedisSerializer);
redisTemplate.setHashKeySerializer(stringRedisSerializer);
redisTemplate.setHashValueSerializer(stringRedisSerializer);
~~~

主动将Redis的键和散列结构的field和value均采用字符串序列化器

![](D:\note\image\redis02.png)



**获取Redis数据类型操作接口**

~~~
// 获取散列操作接口
redisTemplate.opsForHash()
// 获取字符串操作接口
redisTemplate.opsForValue()
// 获取列表接口
redisTemplate.opsForList()
// 获取集合接口
redisTemplate.opsForSet()
// 获取有序集合接口
redisTemplate.opsForZSet()
// 获取地理位置接口
redisTemplate.opsForGeo()
// 获取基数接口
redisTemplate.opsForHyperLogLog()
~~~



**SessionCallback和RedisCallback接口**

可以让RedisTemplate进行回调，通过它们可以在同一条连接上执行多个Redis命令

| 接口名          | 优点                                                         |
| --------------- | ------------------------------------------------------------ |
| SessionCallback | 高级接口，提供了良好的封装，比较友好，优先使用               |
| RedisCallback   | 比较底层，需要处理的内容也比较多，可读性较差，如果不考虑改写底层，尽量不使用它 |

~~~
// RedisCallback
redisTemplate.execute((RedisConnection rc) -> {
    rc.set("key1".getBytes(), "value1".getBytes());
    rc.hSet("hash".getBytes(), "field".getBytes(), "hvalue".getBytes());
    return null;
});

// SessionCallback
redisTemplate.execute((RedisOperations ro) -> {
    ro.opsForValue().set("key1", "value1");
    ro.opsForHash().put("hash", "field", "hvalue");
    return null;
});
~~~



**Redis数据类型**

常用的Redis数据类型（字符串、散列、列表、集合、有序集合）

RedisTemplate并不能支持底层所有的Redis命令，可以使用原始的Redis连接的Jedis对象

~~~
Jedis jedis = (Jedis) stringRedisTemplate.getConnectionFactory().getConnection().getNativeConnection();

~~~

在Redis中列表是一种链表结构



#### Redis事务

在Redis中使用事务，通常的命令组合是watch... multi ... exec，也就是要在一个Redis连接中执行多个命令，可以使用SessionCallback接口

![](D:\note\image\Redis事务执行过程.png)



~~~
 redisTemplate.opsForValue().set("key1", "value1");
 List list= (List) redisTemplate.execute((RedisOperations operations)->{
 	 // 设置要监控key1
     operations.watch("key1");
     // 开启事务，在exec命令执行前，全部都只是进入队列
     operations.multi();
     operations.opsForValue().set("key2", "value2");
     // 执行加1命令，因为key1对应的值为字符串，所以这里报错
     operations.opsForValue().increment("key1", 1);
     // 执行exec命令，判断key1是否在监控后被修改过
     return operations.exec();
 });
~~~



上面代码中，服务器抛出了异常，但key2和key3已经赋值成功了。

> Redis事务是先让命令进入队列，所以一开始它并没有检查这个加一命令是否能够成功，只有在exec命令执行的时候，才能发现错误，对于出错的命令Redis只是报出错误，而错误后面的命令依旧被执行



**Lua脚本**

优点：

+ 强大的运算功能
+ 具备原子性



允许Lua的方法：

+ 直接发生Lua到Redis服务器去执行
+ 先把Lua发送给Redis，Redis会对Lua脚本进行缓存，然后返回一个SHA1的32为编码回来，之后只需要发送ShA1和相关参数给Redis便可以执行了



**为什么通过32为编码执行**

如果Lua脚本很长，那么就需要通过网络传递脚本给Redis去执行，而现实的情况是网络的传输速度往往跟不上Redis的执行速度，所以网络就成为Redis执行的瓶颈。如果 只是传递32位编码和参数，那么需要传递的消息就少了许多，这样就可以极大地减少网络传输的内容，从而提供系统的性能



为了支持Redis的Lua脚本，Spring提供了RedisScript接口，与此同时也有一个DefaultRedisScript实现类

~~~
public interface RedisScript<T>{
    // 获取脚本的Sha1
    String getSha1();
    // 获取脚本的返回值
    Class<T> getResultType();
    // 获取脚本的字符串
    String getScriptAsString();
}
~~~



**缓存管理器**

Spring在使用缓存注解前，需要配置缓存管理器，缓存管理器将提供一些重要的信息，如缓存类型、超过时间等。Spring可以支持多种缓存的使用，因此它存在多种缓存处理器，并提供了缓存处理器的接口CacheManager和与之相关的类

+ @CachePut 表示将方法结果返回存放到缓存中
+ @Cacheable 表示先从缓存中通过定义的键查询，如果可以查询到数据，则返回，否则执行该方法，返回数据，并且将返回结果保存到缓存中
+ @CacheEvict 通过定义的键移除缓存，它有一个Boolean类型的配置项beoreInvocation，表示在方法之前或者之后移除缓存。因为其默认值为false，所以默认为方法之后将缓存移除

对于命中率很低的场景，使用缓存并不能有效提高系统性能，一般不采用缓存机制

Redis缓存机制会使用#{cacheName}:#{key}的形式作为键保存数据

RedisCacheManager采用永不超时的机制

> 对于数据的写操作，一般会认为缓存不可信，所以会考虑从数据库中先读取最新数据，然后再更新数据，以避免将缓存的脏数据写入数据库中，导致出现业务问题





****



#### Spring MVC

![](/image/SpringMVC架构设计图.png)



流程和组件是Spring MVC的核心，Spring MVC的流程是围绕DispatcherServlet而工作的。

首先，在Web服务器启动的过程中，如果在Spring Boot机制下启用Spring MVC，它就开始初始化一些重要的组件，如DispatcherServlet、HandlerAdapter的实现类RequestMappingHandlerAddapter等组件对象。属性文件DispatcherServlet.properties中定义的对象都是在Spring MVC开始时就初始化，并且存放在Spring IOC容器中。

![](/image/SpringMVC全流程.png)



实例在SpringMVC全流程

![](/image/实例在SpringMVC全流程.png)



#### 处理器映射

> 如果Web工程使用了Spring MVC，那么它在启动阶段就会将注解@RequestMapping所配置的内容保存到处理器映射（HandlerMapping）机制中去，然后等待请求的到来，通过拦截请求信息与HandlerMapping进行匹配，找到对应的处理器，并将处理器及其拦截器保存到HandlerExecutionChain对象中，返回给DispatcherServlet，这样DispatcherServlet就可以运行它们。

~~~
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
	
	// 配置请求映射名称
	String name() default "";
	
	// 通过路径映射
	@AliasFor("path")
	String[] value() default {};
	
	// 通过路径映射会path配置项
	@AliasFor("value")
	String[] path() default {};
	
	// 限定只响应HTTP请求类型，如GET、POST、HEAD.....
	// 默认的情况下，可以响应所有的请求类型
	RequestMethod[] method() default {};
	
	// 当存在对应的HTTP参数时才响应
	String[] params() default {};
	
	// 限定请求头存在对应的参数时才响应
	String[] headers() default {};

	// 限定HTTP请求体提交类型，如"application/json"、"text/html"
	String[] consumes() default {};
	
	// 限定返回的内容类型，仅当HTTP请求头的（Accept）类型中包含该指定类型时才返回
	String[] produces() default {};
}
~~~





#### 获取控制器参数

处理器是对控制器的包装，在处理器运行的过程中会调度控制器的方法，只是它在进入控制器方法之前会对HTTP的参数和上下文进行解析，将它们转换为控制器所需的参数。

##### 在无注解下获取参数

在没有注解的情况下，Spring MVC也可以获取参数，且参数允许为空，唯一的要求是参数名称和HTTP请求的参数名称保持一致

##### 使用@RequestParam获取参数

@Requestparam定义了前后端参数名称的映射关系，在默认情况下@RequestParam标注的参数是不能为空的，为了让它能够为空，可以配置其属性required为false

~~~
@RequestParam(value="str_val",required=false) String strVal
~~~

##### 传递数组

SpringMVC内部已经能够支持用逗号分隔的数组参数

~~~
public Map<String,Object> requestArray(int[] intArr,Long[] longArr)
http://localhost:8080/my/requestArray?intArr=1,2,3&longArr=4,5,6
~~~

##### 传递JSON

~~~
public User insert(@RequestBody User user)
~~~

@RequestBody意味着它将接受前端提交的JSON请求，而在JSON请求体与User类之间的属性名称是保持一致的，这样Spring MVC就会通过这层映射关系将JSON请求体转换为User对象

##### 通过URL传递参数

通过处理器映射和注解@PathVariable的组合获取URL参数。

~~~
@GetMapping{"/{id}"}
public User get(@PathVariable("id") Long id)
~~~

##### 获取格式化参数

在一些应用中，往往需要格式化数据，例如日期约定为yyyy-MM-dd，金额约定为$1,000.00；SpringMVC提供了@DateTimeFormat和@NumberFormat对其格式化

~~~
public Map<String,Object> format(@DateTimeFormat(iso=ISO.DATE) Date date,
							@NumberFormat(pattern="#,###.##") Double number)
~~~

在SpringBoot中，日期参数的格式化也可以不使用@DateTimeFormat，而只在配置文件application.properties中加入如下配置项即可：

~~~
spring.mvc.date-format=yyyy-MM-dd
~~~





####  参数转换

当一个请求来到时，在处理器执行的过程中，它首先从HTTP请求和上下文环境来得到参数。如果是简易的参数它会以简单的转换器进行转换，而这些简单的转换器是Spring MVC自身已经提供了的。但是如果是转换HTTP请求体（Body），它就会调用HttpMessageConverter接口的方法对请求体的信息进行转换，首先它会先判断能否对请求体进行转换，如果可以就会将其转换为Java类型。

![](/image/SpringMVC处理器HTTP请求体转换流程图.png)

> 在Spring MVC中，是通过WebDataBinder机制来获取参数的，它的作用是解析HTTP请求的上下文，然后在控制器的调用之前转换参数并且提供验证的功能，为调用控制器方法做准备。

处理器会从HTTP请求中读取数据，然后通过三种接口来进行各类参数转换，这三种接口是Converter、Formatter、GenericConverter。



+ Converter：一对一转换器，也就是从一种类型转换为另外一种类型。例如，有 一个Integer类型的控制器参数，而从HTTP对应的为字符串，对应的Converter就会将字符串转换为Integer类型
+ Formatter：格式化转换器，类似那些日期字符串就是通过它按照约定的格式转换为日期的
+ GenericConverter：将HTTP参数转换为数组



> 对于数据类型转换，SpringMVC提供了一个服务机制去管理，它就是ConversionService接口。在默认的情况下，会使用这个接口的子类DefaultFormattingConversionService对象来管理这些转换类。

Converter、Formatter和GenericConverter可以通过注册机接口进行注册

![](/image/ConversionService转化机制设计.png)



> 在SpringBoot中提供了特殊的机制来管理这些转换器。
>
> SpringBoot的自动配置类WebMvcAutoConfiguration定义了一个内部类WebMvcAutoConfigurationAddapter。

~~~
// SpringBoog的自动注册机制
// 注册各类转换器，registry实际为DefaultFormattingConversionService对象
@Override
public void addFormatters(FormatterRegistry registry) {
	// 遍历IOC容器，找到Converter类型的Bean注册到服务器类中
    for (Converter<?, ?> converter : getBeansOfType(Converter.class)) {
   		 registry.addConverter(converter);
    }
    for (GenericConverter converter : getBeansOfType(GenericConverter.class)) {
   		 registry.addConverter(converter);
    }
    for (Formatter<?> formatter : getBeansOfType(Formatter.class)) {
    	 registry.addFormatter(formatter);
    }
}
~~~



#### 一对一转换器（Converter）

~~~
public interface Converter<S, T> {
	// 转换方法，S代表原类型，T代表目标类型
	T convert(S source);
}
~~~

例如，HTTP的类型为字符串（String），而控制器参数为Long，那么就可以通过Spring内部提供的StringToNumber<T Extends Number>进行转换，

假设前端要传递一个用户的信息，这个用户信息的格式是{id}-{userName}-{note}，而控制器的参数是User类对象，就需要自定义一个从String转换为User的转换器。

~~~
@Component
public class StringToUserConverter implements Converter<String, User> {
	@Override
	public User convert(String userStr){
        User user=new User();
        String[] strArr=userStr.split("-");
        Long id=Long.parseLong(strArr[0]);
        user.setId(id);
        ......
        return user;
	}
}
~~~



#### 参数验证机制

在WebDataBinder中除了可以注册转换器外，还运行注册验证器（Validator）。

可以在Spring控制器中，使用注解@InitBinder，这个注解的作用是运行在进入控制器方法前修改WebDataBinder机制。在SpringMVC中，定义了一个接口Validator

~~~
public interface Validator{
	/**
	* 判定当前验证器是否支持该Class类型的验证
	*/
    boolean supports(Class<?> clazz);
    
    /**
    * 如果supports返回true，则这个方法执行验证逻辑
    * @param target 被验证POJO对象
    * @param errors 错误对象   发现错误，保存包erors对象中
    */
    void validate(Object target，Errors errors)；
}
~~~



自定义用户验证器

~~~
pbulic class UserValidator implements Validator{
    @Override
    public boolean supports(Class<?>clazz){
        return clazz.equals(User.class)；
    }
    
    @Override
    public void validate(Object target,Errors errors){
        if(target==null){
        	//直接在参数处报错，这样就不能进入控制器的方法
            errors.rejectValue("",null,"用户不能为空");
            return;
        }
        User user=(User)target;
        if(StringUtils.isEmpty(user.getUserName)){
        	//增加错误，可以进入控制器方法
            errors.rejectValue("userName",null,"用户名不能为空");
        }
        
    }
}
~~~

有了这个验证器，Spring还不会自动启用它，因为还没有绑定给WebDataBinder机制。在Spring MVC中提供了一个注解@InitBinder，它的作用是在执行控制器方法前，处理器会执行@InitBinder标注的方法。这时可以将WebDataBinder对象作为参数传递到方法中，通过这层关系得到WebDataBinder对象，这个对象有一个setValidator方法，它可以绑定自定义的监听器。

~~~
 @InitBinder
 public void initBinder(WebDataBinder binder) {
     // 绑定验证器
     binder.setValidator(new UserValidator());
 }

~~~





#### 拦截器

> 当请求来到DispatcherServlet时，它会根据HandlerMapping的机制找到处理器，这样就会返回一个HandlerExecutionChain对象，这个对象包含处理器和拦截器。这里的拦截器会对处理器进行拦截，这样通过拦截器就可以增强处理器的功能

所有的拦截器都需要实现HandlerInterceptor接口

![](D:\note\image\拦截器执行过程.png)

有多个拦截器时，处理器前（preHandle）方法采用先注册先执行，而处理器后方法和完成方法则是先注册后执行的规则。处理器前（preHandle）一旦返回false，后续的拦截器、处理器和所有拦截器的处理器后（postHandle）方法都不会被执行。完成方法afterCompletion则不一样，它只会执行返回true的拦截器的完成方法，而且时先注册后执行





#### 给控制器增加通知

+ @ControllerAdvice：定义一个控制器的通知类，允许定义一些关于增强控制器的各类通知和限定增强哪些控制器功能等
+ @InitBinder：定义控制器参数绑定规则，如转换规则、格式化等，它会在参数转换之前执行
+ @ExceptionHandler：定义控制器发生异常后的操作
+ @ModelAttrivute：在控制器方法执行之前，对数据模型进行操作



----



#### Spring Boot 自动装配

+ @ConditionalOnClass：存在哪些类，才去装配当前类
+ @EnableConfigurationProperties：使得哪个类可以通过配置文件装配（在配置文件中可以配置的原因）
+ @ConditionalOnMissingBean：在缺失某些类型的Bean的时候，才将方法返回的Bean装配到IOC容器中（使用开发者自己定制的Bean）
+ @ConditionalOnProperty：检测属性配置的注解，满足条件才会启动这个类作为配置文件
+ @import：加载其他的类到当前的环境中来
+ @AutoConfigureAfter：在完成指定类的装配后才执行