#### SecurityContextHolder

> 存储安全上下文信息：当前操作用户，该用户是否被认证，他拥有的权限



**身份认证过程**

1. 用户名和密码被过滤器获取到，封装成Authentication
2. AuthenticationManager身份管理器负责验证这个Authentication
3. 认证成后，AuthenticationManager身份管理器返回一个被填充满了信息的（权限信息，身份信息，细节信息，但密码通常会被移除）Authentication实例
4. SecurityContextHolder填充Authentication



#### AuthenticationManager

认证管理接口，常用实现类ProviderManager内部会维护一个List<AuthenticationProvider>列表，存放多种认证方式；依照次序去认证，认证成功则立即返回，若认证失败，下一个继续尝试认证，所有认证器都无法认证成功，则ProviderManager抛出异常



#### DaoAuthenticationProvider

AuthenticationProvider常用的一个实现



1. 提交的用户名和密码，被封装成了UsernamePasswordAuthenticationToken
2. 而根据用户名加载用户的任务则是交给了UserDetailsService，而根据用户名加载用户的任务则是交给了UserDetailsService，在DaoAuthenticationProvider中，对应的方法便是retrieveUser，返回一个UserDetails
3. additionalAuthenticationChecks完成UsernamePasswordAuthenticationToken和UserDetails密码的对比



UserDetailsService负责从特定的地方加载用户信息，常见的实现类有jdbcDaoImpl、InMemoryUserDetailsManager，前者从数据库加载用户，后者从内存中加载用户