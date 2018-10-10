

### Restart vs Reload 

Spring Boot提供的重启技术是通过使用两个类加载器实现的。没有变化的类（比如那些第三方jars）会加载进一个基础（basic）classloader，正在开发的类会加载进一个重启（restart）classloader。当应用重启时，restart类加载器会被丢弃，并创建一个新的。这种方式意味着应用重启通常比冷启动（cold starts）快很多，因为基础类加载器已经可用


### Application属性文件
SpringApplication将从以下位置加载application.properties文件，并把它们添加到Spring Environment中：
+ 当前目录下的/config子目录
+ 当前目录
+ classpath下的/config包
+ classpath根路径（root）

上面的优先级高于下面的优先级；properties文件高于yaml文件

- - -

### 注解
**元注解**
>元注解是一种标注在别的注解之上的注解。如果一个注解可以标注在别的注解上，那么这个注解就是元注解。

**Stereotype注解（模式化注解、角色类注解）**
>Stereotype注解是一种在应用中，常被用于声明要扮演某种职责或者角色的注解。

例如：
+ @Repository   常用与Data Access Object或者DAO
+ @Component    被Spring管理的组件的对应注解。任何标注了@Component的组件都会在Spring组件扫描时被扫描到。同样的，任何标注了 被元注解@Component标注过的注解 的组件，也会在Spring组件扫描时被扫描到。例如，@Service就是一种被元注解@Component标注过的注解。

**组合注解**
>组合注解是一种被一个或者多个元注解标注过的注解，用以撮合多个元注解的特性到新的注解


### @AliasFor


**在同一个注解内使用**
互为别名，值必须一样，并且可以一个值获取另一个值
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



### @Configuration和@Bean

`@Configuration`是一个类级别的注解，用于表明此对象是一个bean定义的资源。Spring的容器会根据它来生成IOC容器去装配Bean；`@Configuration`类通过public的`@Bean`注解的方法来声明beans。将返回的POJO装配到IOC容器中。调用`@Configuration`类的`@Bean`方法也可以被用于定义inter-bean依赖。

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

### @Import
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

~~~

~~~



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
}
~~~



#### MyBatis

MyBatis是一个基于SqlSessionFactory构建的框架。对于SqlSessionFactory而言，它的作用是生成SqlSession接口对象，这个接口对象是MyBatis操作的核心。

![](/image/MyBatis配置内容结构图.png)



#### 事务

通过@Transactional进行标注启用数据库事务功能。配置内容是在Spring IOC容器在加载时就会将这些配置信息解析出来，然后把这些信息存到事务定义器（TransactionDefinition接口的实现类）里，并且记录哪些类或者方法需要启动事务功能，采取什么策略去执行事务。

![](/image/Spring数据库事务约定.png)



### Spring MVC

![](/image/SpringMVC架构设计图.png)



流程和组件是Spring MVC的核心，Spring MVC的流程是围绕DispatcherServlet而工作的。

首先，在Web服务器启动的过程中，如果在Spring Boot机制下启用Spring MVC，它就开始初始化一些重要的组件，如DispatcherServlet、HandlerAdapter的实现类RequestMappingHandlerAddapter等组件对象。属性文件DispatcherServlet.properties中定义的对象都是在Spring MVC开始时就初始化，并且存放在Spring IOC容器中。

![](/image/SpringMVC全流程.png)



实例在SpringMVC全流程

![](/image/实例在SpringMVC全流程.png)

如果Web工程使用了Spring MVC，那么它在启动阶段就会将注解@RequestMapping所配置的内容保存到处理器映射（HandlerMapping）机制中去，然后等待请求的到来，通过拦截请求信息与HandlerMapping进行匹配，找到对应的处理器，并将处理器及其拦截器保存到HandlerExecutionChain对象中，返回给DispatcherServlet，这样DispatcherServlet就可以运行它们。



#### 获取控制器参数

处理器是对控制器的包装，在处理器运行的过程中会调度控制器的方法，只是它在进入控制器方法之前会对HTTP的参数和上下文进行解析，将它们转换为控制器所需的参数。





