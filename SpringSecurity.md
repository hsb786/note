#### SecurityContextHolder

> 存储安全上下文信息：当前操作用户是谁、该用户是否被认证、他拥有哪些角色权限

SecurityContextHolder默认使用`ThreadLocalSecurityContextHolderStrategy`来存储认证信息。SpringSecurity在用户登录时自动绑定认证信息到当前线程，在用户退出时，自动清除当前线程的认证信息 （前提是在web应用）



#### Authentication

Authentiaction在spring security中是最高级别的身份/认证的抽象

~~~
public interface Authentication extends Principal,Serializable{
    // 权限信息列表，通常是代表权限信息的一系列字符串
    Collection<? extends GrantedAuthority> getAuthorities();
    
    // 密码信息，用户输入的密码字符串，在认证过后通常会被移除，用于保障安全
    Object getCredentials();
    
    // 细节信息，web应用中的实现接口通常为WebAuthenticationDetails，它记录了访问者的ip地址和sessionId的值
    Object getDetails();
    
    // 身份信息，大部分情况下放回的是UserDetails接口的实现类
    Object getPrincipal();
    
    boolean isAuthenticated();
    
    void setAuthenticated();
    
}
~~~





**身份认证过程**

1. 用户名和密码被过滤器获取到，封装成Authentication
2. AuthenticationManager身份管理器负责验证这个Authentication
3. 认证成后，AuthenticationManager身份管理器返回一个被填充满了信息的（权限信息，身份信息，细节信息，但密码通常会被移除）Authentication实例
4. SecurityContextHolder填充Authentication



#### AuthenticationManager

~~~
public class ProviderManager implements AuthenticationManager,MessageSourceAware,InitializingBean{
    
    private List<AuthenticationProvider> providers=Collections.emptyList(); 
}
~~~



认证管理接口，常用实现类ProviderManager内部会维护一个List<AuthenticationProvider>列表，存放多种认证方式；依照次序去认证，认证成功则立即返回，若认证失败，下一个继续尝试认证，所有认证器都无法认证成功，则ProviderManager抛出异常



#### DaoAuthenticationProvider

AuthenticationProvider常用的一个实现



1. 提交的用户名和密码，被封装成了UsernamePasswordAuthenticationToken
2. DaoAuthenticationProvider中的retrieveUser方法，通过调用UserDetailsService根据用户名加载用户，返回UserDetails
3. additionalAuthenticationChecks完成UsernamePasswordAuthenticationToken和UserDetails密码的对比



UserDetails代表了最详细的用户信息

UserDetailsService负责从特定的地方加载用户信息，常见的实现类有jdbcDaoImpl、InMemoryUserDetailsManager，前者从数据库加载用户，后者从内存中加载用户



![](D:\note\image\SpringSecurity架构图.png)



UserDetailsService接口作为桥梁，是DaoAuthenticationProvider与特定用户信息来源解耦的地方，UserDetailsService由UserDetails和UserDetailsManager所构成；UserDetails和UserDetailsManager各司其职，一个是对基本用户信息进行封装，一个是对基本用户信息进行管理



java assist idea添加

jsonpath  github

controller  url参数 正则表达式  

jsonview

@valid    BindingResult   hibernate validator    自定义validator注解

BasicErrorController   请求头  accept  ControllerAdvice





filter -> interceptor -> controllerAdvice -> aspect -> controller

过滤器 http请求和响应对象    

 拦截器  http请求和响应对象  处理请求方法信息

切面 参数值



ExceptionTranslationFilter

FilterSecurityInterceptor



 TokenEnhancer