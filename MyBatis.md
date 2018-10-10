### 实体类中为什么不用基本类型定义属性

由于Java中的基本类型会有默认值，例如当某个类中存在private int age；创建这个类时，age会有默认值0.当使用age属性时，它总会有值。因此在某些情况下，便无法实现使age为null。并且在动态SQL的部分，如果使用age!=null进行判断，结果总会为true，因而会导致很多隐藏的问题。



### xml文件中为什么需要指定jdbcType

为了防止类型错误，对于一些特殊的数据类型，建议指定具体的jdbcType值。例如 headimg 指定 BLOB 类型， createTime 指定 TIMESTAMP 类型

> BLOB对应的类型是 ByteArrayinputStream ，就是二进制数据流
>
> 由于数据库区分 date time datetime 类型，但是 Java 中一般都使用java.util.Date
> 类型。因此为了保证数据类型的正确，需要手动指定日期类型，date、time、datetime
> 应的 JDBC 类型分别为 DATE、TIME、TIMESTAMP



### 自动将以下画线方式命名的数据库列映射到 Java 对象的驼峰式命名属性中

~~~
<settings>
	<！－－其他配置 -->
	<setting name= ” mapUnderscoreToCamelCase ” value=”true ” />
</sett ngs>
~~~



### useGeneratedKeys，keyProperty

使用主键自增（如 MySQ SQL Server 数据库）时，插入数据库后可能需要得到自增的主键值

useGeneratedKeys设置为true后，MyBatis会使用JDBC的getGeneratedKeys方法来取出由数据库内部生成的主键。获得主键值后将其赋值给keyProperty配置的id属性。

~~~
<insert i d=” insert2 ” useGeneratedKeys=” true ” keyProperty=” id” >
	insert into sys_user(
		user_ name, user_password, user_email ,
		user info, head img , create time)
	values(
		#{userName}, #{userPassword}, #{userEmail} ,
		#{u serinfo }, #{headimg , jdbcType=BLOB} ,
		#{ createTime , jdbcType= TIMESTAMP})
</insert> 
~~~



### 使用 selectKey 返回主键的值

上面这种回写主键的方法只适用于支持主键自增的数据库。有些数据库（如 Oracle ）不提供主键自增的功能，而是使用序列得到一个值，然后将这个值赋给 id ，再将数据插入数据库对于这种情况，可以采用另外一种方式：使用＜ sele ctKey＞标签来获取主键的值，这种方式不仅适用于不提供主键自增功能的数据库，也适用于提供主键自增功能的数据库

~~~
<insert id="insert3">
    insert into sys user(
        user name , user password, user email ,
        user_ info , head_img , create_time)
    values(
        #{us erName} , #{userPassword} , #{userEmail} ,
        #{u se rinfo} , #{headimg , jdbcType=BLOB} ,
        #{createTime , jdbcType= TIMESTAMP} )
    <selectKey keyColumn=” id” resultType=” long” keyProperty=” id” order=” AFTER” >
    	SELECT LAST INSERT ID ()
    </selec tKey>
</ insert> 
~~~

order属性的设置和使用的数据库有关。

+ 在MySql数据库中，order属性设置的值是AFTER，因为当前记录的主键值在insert语句执行成功后才能获取到
+ 在Oracle数据库中，order的值要设置为BEFORE，这是因为Oracle中需要先从序列获取值，然后将值作为主键插入到数据库中

~~~
<insert id=” insert3” >
    <selectKey keyColumn="id" resultType="long" keyProperty="id" order="BEFORE">
        SELECT SEQ ID.nextval from dual
    </selectKey>
    insert into sys_user(
   	 	id, user_name user_password, user_email ,
    	user_info, head_img , create_time)
    values (
   		 #{id} , # {userName} , #{userPassword} , #{userEmail} , 
   		 # {userInfo) , # { headlmg, jdbcType=BLOB) , # { createTime , jdbcType=TIMESTAMP))
</insert>	
~~~





可以发现， selectKey 元素放置的位置和之前 MySQL 例子中的不同，其实这个元素放置的位置不会影响 selectKey 中的方法在 insert 前面或者后面执行的顺序，影响执行顺序的是order 属性，这么写仅仅是为了符合实际的执行顺序，看起来更直观而己

> Oracle 方式的副SERT 语句中明确写出了 id 列和值＃｛ id ｝，因为执行 selectKey 中的语句后 id 就有值了，我们需要把这个序列值作为主键值插入到数据库中，所以必须指定 id列，如果不指定这一列，数据库就会因为主键不能为空而抛出异常



### 多个接口参数的用法

+ 将多个参数合并到一个JavaBean中
+ 使用Map
+ 使用@Param注解

通过Map的key-value方式传递参数值，需要自己手动创建Map以及对参数进行赋值，并不简洁。



给参数配置@Param注解后，MyBatis就会自动将参数封装成Map类型，@Param注解值会作为Map中的key，

~~~
List<SysRole> selectRo lesByUseridAndRoleEnabled(
@Param (”user Id”) Long user Id,
@Param (” enable d ”) Integer enabled) ; 
~~~





### Mapper接口动态代理实现原理

假设有 个如下的 Mapper 接口。

~~~
public interface UserMapper {
	List<SysUser> selectAll(); 
}
~~~

这里使用 Java 动态代理方式创建一个代理类，代码如下

~~~
public class MyMapperProxy<T ＞工mplements InvocationHandler {
    private Class<T> mapperinterface;
    private SqlSession sqlSession;
    
    public MyMapperProxy(Class<T> mapperinterface, SqlSession sqlSession ) {
        this . mapperinterface = mapperinterface ;
        this.sqlSession = sqlSession ;
    }    
    @Override
    public Object invoke(Object proxy , Method method , Object[] args ) throws Throwable {
        ／／针对不同的 sql 类型，需要调用 sqlSession 不同的方法
        ／／接口方法中的参数也有很多情况 ，这里只考虑没有有参数的情况
        List<T> list= sqlSession.selectList(
        mapperinterface.getCanonicalName() + ” . ” + method .getName());
        ／／返回位也有很多情况，这里不做处理直接返回
        return list; 		
    }
 }
~~~



测试代码如下。

~~~

~~~



> 当调用一个接口的方法时，会先通过接口的全限定名称和当前调用的方法名的组合得到一个方法id，这个id的值就是映射XML中的namespace和具体方法 id的组合。
>
> 所以可以在代理方法中使用 sqlSession 以命名空间的方式调用方法。通过这种方式可以将接口和 XML 文件中的方法关联起来。这种代理方式和常规代理的不同之处在于，这里没有对某个具体类进行代理，而是通过代理转化成了对其他代码的调用。



### 动态SQL

+ if
+ choose (when、otherwise)
+ trim （where、set）
+ foreach
+ bind

#### where

如果该标签包含的元素中有返回值，就插入一个where；如果where后面的字符串是以 AND 和 OR 开头的，就将它们剔除。这种情况下生成的 SQ 更干净、更贴切，不会在任何情况下都有 wher 1 = 1 这样的条件

#### set

如果该标签包含的元素中有返回值，就插入一个set ；如果 set 后面的字符串是 以逗号结尾的，就将这个逗号剔除



#### trim

where和set 标签 的功能都可以用 trim 标签来实现，并且在底层就是通过
TrimSqlNode 实现的

~~~
<trim prefix=”WHERE ” prefixOverrides=”AND IOR ” >
。。。。
</trim> 

<trim prefix=” SET” suffixOverrides=”, ” >
。。。。
</ trim> 
~~~



+ prefix ：当 trim 元素内包含内容时，会给内容增加 prefix 指定的前缀。

+ prefixOverrides ：当 trim 元素内包含内容时，会把内容中匹配的前缀字符串去掉。

+ suffix ：当 trim 元素内包含内容时，会给内容增加 suffix 指定的后缀。

+ suffixOverrides ：当 trim 元素内包含内容时，会把内容中匹配的后缀字符串去掉。



#### bind

bind 标签可以使用 OGNL 表达式创建一个变量井将其绑定到上下文中。

~~~
<if test=” userName != null and userName ! = ””>
	and user_name like concat ('%'， ＃｛ userName ｝，'%')
</if> 
~~~

使用 con cat 函数连接字符串，在MySQL 中，这个函数支持多个参数，但在Oracle 中只支持两个参数。由于不 同数据库之间的语法差异 ，如果更换数据库，有些 SQL 语句可能就需要重写。针对这种情况，可 以使用 bind 标签来避免由于更换数据库带来的一些麻烦。将上面的方法改为 bind 方式后，代码如下。

~~~
<if test=” userName != null and userName !=””>
	<bind name= "userNaMeLike" value="'%'＋ userName +'%'"/>
	and user_name like #{userNaMeLike}
</if> 
~~~

