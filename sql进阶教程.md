> SQL，面向集合



所有包含NULL的计算，结果肯定是NULL

最好使用<>，这是被标准SQL所承认

表中存储的是实际数据，而视图中保存的是从表中取出数据所使用的SELECT语句



FROM子句不是必要的

~~~
SELECT (100 + 200) * 3 AS calculation;
~~~



SQL中真值除了真假之外，还有第三种值——不确定（UNKNOW）

设置NOT NULL约束的原因：原本只有4行的真值表，如果考虑NULL的化就会增加为9行，条件判断会变得异常



COUNT函数的结果根据参数的不同而不同。COUNT(*)会得到包含NULL的数据行数，而COUNT(<列名>)会得到NULL之外的数据行数

> 聚合函数会将NULL排除在外。但COUNT(*)例外，并不会排除NULL

AVG函数会事先删除NULL再计算

> 只有SELECT子句和HAVING子句（以及ORDER BY子句）中能够使用聚合函数



> 聚合键中（group by）包含NULL时，在结果中会以“不确定”行（空行）的形式表现出来



FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY

WHERE子句 = 指定行所对应的条件

HAVING子句 = 指定组所对应的条件



排序键中包含NULL时，会在开头或末尾进行汇总



EXIST判断是否存在满足某种条件的记录



----

当IN的参数是子查询时，数据库首先会执行子查询，然后将结果存储在一张临时的工作表里（内联视图）。使用EXISTS的话，数据库不会生成临时的工作表

但是从代码的可读性上来看，IN要比EXISTS号。使用IN时的代码看起来更加一目了然，易于理解。因此，如果确信使用IN也能快速获取结果，就没有必要非得改成EXISTS了

~~~
SELECT *
  FROM Class_A
 WHERE id IN (SELECT id
                FROM Class_B);

SELECT *
  FROM Class_A A
 WHERE EXISTS
        (SELECT *
           FROM Class_B B
          WHERE A.id = B.id);
~~~



+ 如果连接列上建立了索引，那么查询class_b时不用查实际的表，只需查索引就可以了
+ 如果使用EXISTS，那么只要查到一行数据满足条件就会终止查询，不用像使用IN时一样扫描全表

































