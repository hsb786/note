端口表示与服务器硬件上运行的一个特定软件的逻辑连接

从0到1023的TCP端口已经保留，有一些众所周知的服务使用。不能使用3000一下的端口



| 应用 | 端口号 |
| :--: | :--: |
| HTTP |   80   |
|  HTTPS | 443|





如果你需要即时页面（动态创建的页面，在发出请求之前还不存在），而且希望能够把数据保存到数据库中，只依靠Web服务器是不够的

**动态内容**

Web服务器应用只提供静态页面，但是有一个“辅助”应用可以生成非静态的即时页面，而且这个辅助应用能与Web服务器通信。



容器为Web应用提供了通信支持、生命周期管理、多线程支持、声明方法安全、以及JSP支持，这样你就能全神贯注地专心开发你自己的业务逻辑



**多线程支持**

容器为自动地为它接收的每个servlet请求创建一个新的Java线程。针对客户的请求，如果servlet已经运行完相应的HTTP服务方法，这个线程就会结束。



**容器如何处理请求**

1. 用户点击一个链接，其URL指向一个servlet而不是静态页面

2. 容器“看出来”这个请求要的是一个servlet，所以容器创建两个对象：

   ​        (1) HttpServletResponse

   ​	(2) HttpServletRequest

3. 容器根据请求中的URL找到正确的servlet，为这个请求创建或分配一个线程，并把请求和响应对象传递给这个servlet线程

4. 容器调用servlet的service()方法。根据请求的不同类型，service()方法会调用doGet()或doPost()方法

5. servlet使用响应对象将响应写至客户，响应通过容器放回

6. service()方法结束，所以线程要么撤销，要么返回到容器管理的一个线程池。请求和响应对象引用已经出了作用域，所以这些对象已经没有意义（可以垃圾回收）

   ​	

一个servlet可以有3个名字

+ 客户知道的URL名
+ 部署人员知道的秘密的内部名
+ 开发人员知道的实际的文件名



将servlet部署到Web容器时，会创建一个相当简单的XML文档，这称为部署描述文件

可以使用两个XML元素把URL映射到servlet，其中一个将客户知道的公共URL名映射到你自己的内部名，另一个元素把你自己的内部名映射到一个完全限定类名

<servlet> 内部名映射到完全限定类名

<servlet-mapping> 内部名映射到公共URL名

```
<servlet>
   <servlet-name>uaa</servlet-name>
   <servlet-class>com.zuche.framework.extend.spring.DispatcherServletExtend</servlet-class>
</servlet>
<servlet-mapping>
   <servlet-name>uaa</servlet-name>
   <url-pattern>*.login</url-pattern>
   <url-pattern>*.logout</url-pattern>
   <url-pattern>*.do_</url-pattern>
   <url-pattern>*.action</url-pattern>
   <url-pattern>*.xls</url-pattern>
   <url-pattern>*.srv</url-pattern>
</servlet-mapping>
```



MVC的关键是，业务逻辑要与表示分离，而且要 在两者之间 放上别的东西，这样业务逻辑本身就能作为一个可重用的Java类存在，它根本不用对视图有任何了解



**ServletConfig**

+ 每个servlet都有一个ServletConfig对象
+ 用于向servlet传递部署时信息，而你不想把这个信息硬编码到servlet中
+ 用于访问servletContext
+ 参数在部署描述文件中配置



**ServletContext**

+ 每个Web应用有一个ServletContext（应该叫AppContext才对）
+ 用于访问Web应用参数
+ 相当于一种应用公告栏，可以在这里放置消息，应用的其他部分可以访问这些消息
+ 用于得到服务器消息，包括容器名和容器版本，以及所支持的API的版本等



-----



+ 容器要加载类，调用servlet的无参数构造函数，并调用servlet的init()方法，从而初始化servlet
+ init()方法在servlet一生中只调用一次，往往在servlet为客户请求提供服务之前调用
+ init()方法使servlet可以访问ServletConfig和ServletContext对象，servlet需要从这些对象得到有关servlet配置和Web应用的信息
+ 对servlet的每个请求都在一个单独的线程中运行！任何特定servlet类都只有一个实例
+ HttpServlet扩展了javax.servlet.GenericServlet，这是一个抽象类，实现了大多数基于servlet方法
+ 可以覆盖init()方法，而且必须覆盖一个服务方法（doGet()，doPost()等）
+ GET请求本质上讲（根据HTTP规范）是幂等的。它们应当能多次运行而不会对服务器产生任何副作用。GET请求不应修改服务器上的任何东西。但是你也可以写一个非幂等的doGet()方法（不过这是很糟糕的做法）
+ POST本质上讲不是幂等的，所以要由你来适当地设计和编写代码，如果客户错误地把一个请求发送了两次，你也能正确地加以处理
+ 可以用getParameter("paramname")方法从请求得到参数。返回值总是一个String
+ 如果对应一个给定的参数名有多个参数值，要使用getParameterValues("paramname")方法来返回一个String数组



**Servlet初始化参数只能读一次——就是在容器初始化servlet的时候**

1. 容器为这个servlet读取部署描述文件，包括servlet的初始化参数(<init-param>)
2. 容器为这个servlet创建一个性的ServletConfig实例
3. 容器为每个servlet初始化参数创建一个String 键值对
4. 容器向ServletConfig提供键值初始化参数的引用
5. 容器创建servlet类的一个新实例
6. 容器创建servlet类的一个新实例
7. 容器调用servlet的init()方法，传入ServletConfig引用



+ 每个servlet有一个ServletConfig
+ 每个Web应用有一个ServletContext
+ 如果应用是分布式应用，那么每个JVM有 一个ServletContext



如果修改XML来改变一个初始化参数的值(servlet初始化参数或上下文初始化参数)，servlet或web应用的其余部分只有当web应用重新部署时才会看到。servlet只会初始化一次，就是在它生命刚开始时初始化，也正是这个时候会为它提供ServletConfig和ServletContext。容器创建这两个对象时会从部署文件读取值，并设置对象的值



> 要把初始化参数认为是部署时常量



+ 初始化参数表示：servlet初始化参数
+ 上下文参数或者应用参数表示：上下文初始化参数



> servlet规范中定义，部署描述文件中的单个servlet声明会在运行时成为单个对象实例

servlet只有一个实例，但可以有多个线程



HttpSession对象可以保存跨同一个客户多个请求的会话状态

对于会话期间客户做的所有请求，从中得到的所有信息都可以用HttpSession对象保存

+ 相同的客户
+ 相同的servlet
+ 不同的请求
+ 不同的线程
+ 相同的会话



禁用cookie的客户会忽略“Set-Cookie”响应首部，如果客户不接受cookie，URL重写会自动发生，但是必须显示地对所有URL编码

没有办法对静态页面完成自动的URL重写，所以，如果你依赖于会话，就必须使用动态生成的页面

+ 在响应发回的HTML中，把会话ID增加到所有URL的最后
+ 会话ID放在请求URL的最后作为“额外”信息返回



当容器看到你调用了request.getSession()，而且认识到它要与这个客户建立一个新的会话，容器就会发回一个“双保险”响应，不仅针对会话ID有一个“Set-Cookie”首部，而且会向URL追加会话ID(假设使用了response.encodeURL())

假设同一个客户发出下一个请求，它把会话ID追加到请求URL，但是如果客户接受cookie，这个请求还会有一个会话ID cookie。servlet调用request.getSession()时，容器从请求读取会话ID，找到会话，并且这样考虑：“这个客户接受cookie”，所以我可以忽略response.encodeURL()调用。在响应中，我要发送一个cookie，因为我知道它能正常工作，而且没有必要完成任何URL重写

> URL编码由响应处理！encodeURL()方法是HttpServletResponse对象上调用的方法！





有些分布式应用使用租约，使服务器能知道客户什么时候离开。客户从服务器得到一个租约，然后必须按指定的间隔续租，告诉服务器这个客户仍存活。如果客户的租约到期，服务器就知道可以释放为这个客户保留的所有资源



WAR（Web archive）web应用结构的一个快照，采用了一种更可移植的压缩形式



servlet默认地会在每一个请求到来时初始化。这说明，第一个客户要承受类加载、实例化和初始化（建立一个ServletContext、调用监听者等）等一系列开销，然后容器才能正常工作：分配一个线程，并调用servlet的service()方法

如果你希望在部署时（或在服务器重启时）加载servlet，而不是等到第一个请求到来时才加载，可以在DD中使用<load-on-startup>元素。如果<load-on-startup>的值非负，就是在告诉容器要在应用部署时（或服务器重启时）初始化servlet，越小优先级越高



