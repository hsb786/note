

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
`@Configuration`是一个类级别的注解，用于表明此对象是一个bean定义的资源。`@Configuration`类通过public的`@Bean`注解的方法来声明beans。调用`@Configuration`类的`@Bean`方法也可以被用于定义inter-bean依赖。

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

>申明inter-bean依赖的方法只会在`@Bean`声明在`@Configuration`类下时才会生效。你不能使用`@Component`类申明inter-bean依赖。

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

