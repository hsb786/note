实体引用 | 符号
--- | ---
\&lt; | <
\&gt; | >
\&amp; | &
\&apos; | '
\&quot; | "

- - -

### CDATA
术语 CDATA 指的是不应由 XML 解析器进行解析的文本数据（Unparsed Character Data）。

CDATA 部分由 "<![CDATA[" 开始，由 "]]>" 结束：


!=  <>  筛选结果不包含null的数据


### 函数
IF(true,a,b)    第一个参数为bool类型，true返回a，false返回b

IFNULL(a,b)     第一个参数不为null，则返回第一个参数，否则返回第二个参数

CONCAT(a,'xxx')     拼接xxx到a的后面

LTRIM(a)        除左边的空格
RTRIM(a)        除右边的空格
TRIM(a)         除两端的空格


\<selectKey keyProperty="id" resultClass="int">    返回插入数据后生成的id

\<isNotEmpty property="authorityCode" prepend="and">

\<iterate property="deleteId" open="(" close=")" conjunction=",">#deleteId[]#</iterate>


insert into xxx(aa,bb) select aa,bb from zzz


### LAST_INSERT_ID
基于Connection的，只要每个线程都使用独立的Connection对象，LAST_INSERT_ID函数将返回该Connection对Auto_INCREMENT列最新的insert or update操作生成的第一个record的ID。这个值不能被其它客户端(Connection)影响

\<selectKey resultClass="int" keyProperty="id">
	SELECT last_insert_id()  AS ID
\</selectKey>



select count(*)     统计查询结果记录数
不要使用count(列名)或count(常量)来替代count(*)，count(*)是SQL92定义的标准统计行数的语法



