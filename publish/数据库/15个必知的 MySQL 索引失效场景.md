

![mp_index_diss](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/mp_index_diss_853e8cb9396405bf01bc9ffccd1c7d5b.png)


> 哈喽，大家好🎉，我是**世杰**。
>
> 本文我为大家介绍的「**索引失效的12种常见场景**」


我们在日常开发过程中，时不时都会踩到 `Mysql` 数据库不走索引的坑。

常见的现象就是：明明在字段上**添加了索引，但却并未生效**。甚至于同一条SQL语句，在某些参数下生效，在**某些参数下不生效**，这是为什么呢?

这篇文件将常见的12种不走索引情况进行汇总，并以实例展示，帮助大家更好地避免踩坑。



# 1. 数据准备

## 1.1 创建表结构

为了逐项验证索引的使用情况，我们先准备一张表t_user：

```sql
CREATE TABLE `t_user` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `id_no` varchar(18) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin DEFAULT NULL COMMENT '身份编号',
  `username` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin DEFAULT NULL COMMENT '用户名',
  `age` int(11) DEFAULT NULL COMMENT '年龄',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`),
  KEY `union_idx` (`id_no`,`username`,`age`),
  KEY `create_time_idx` (`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
```

在上述表结构中有三个索引：

- `id`：为数据库主键;
- `union_idx`：为id_no、username、age构成的联合索引;
- `create_time_idx`：是由create_time构成的普通索引;



## 1.2 初始化数据

创建存储过程，方便批量插入数据，用来验证数据比较多的场景

```sql
-- 删除历史存储过程
DROP PROCEDURE IF EXISTS `insert_t_user`

-- 创建存储过程
delimiter $

CREATE PROCEDURE insert_t_user(IN limit_num int)
BEGIN
 DECLARE i INT DEFAULT 10;
    DECLARE id_no varchar(18) ;
    DECLARE username varchar(32) ;
    DECLARE age TINYINT DEFAULT 1;
    WHILE i < limit_num DO
        SET id_no = CONCAT("NO", i);
        SET username = CONCAT("Tom",i);
        SET age = FLOOR(10 + RAND()*2);
        INSERT INTO `t_user` VALUES (NULL, id_no, username, age, NOW());
        SET i = i + 1;
    END WHILE;

END $
-- 调用存储过程
call insert_t_user(100);
```



## 1.3 数据库版本

查看当前数据库的版本：

```
select version();

8.0.34
```

上述为本人测试的数据库版本：`8.0.34`。当然，以下的所有示例，大家可在其他版本进行执行验证。





# 2. 失效场景总结

1. 联合索引不满足**最左匹配原则**
2. 使用了 `select *`
3. 索引列参与**运算**
4. 索引列使用**函数**
5. 错误的 `like` 使用
6. 类型**隐式转换**
7. 使用 `OR` 操作
8. 两列做比较
9. `<>`, `!=`不等于比较
10. `is not null`, `not in`, `not exists` 条件
11. `order by` 普通索引
12. 优化器认为全表扫描 > 索引扫描





# 3. 失效场景详解

## 3.1 联合索引不满足最左匹配原则

联合索引遵从最左匹配原则，顾名思义，在联合索引中，最左侧的字段优先匹配。因此，在创建联合索引时，where子句中使用最频繁的字段放在组合索引的最左侧。

而在查询时，要想让查询条件走索引，则需满足：最左边的字段要出现在查询条件中。

实例中，union_idx联合索引组成：

```sql
KEY `union_idx` (`id_no`,`username`,`age`)
```

最左边的字段为id_no，一般情况下，只要保证id_no出现在查询条件中，则会走该联合索引。



『**示例 1**』

```sql
explain select * from t_user where id_no = '1002';
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/999_a5309a8e9cd43b692170cdcabb01c5f1.jpg)

通过explain执行结果可以看出，上述SQL语句走了`union_idx`这条索引。



『**示例 2**』

```sql
explain select * from t_user where id_no = '1002' and username = 'Tom2';
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/1_eb17e07355272d46d309f97286737104.jpg)

依然命中union_idx索引，根据上面key_len的分析，不仅使用了id_no列，还使用了username列。



『**示例 3**』

```sql
explain select * from t_user where id_no = '1002' and age = 12;
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/2_83b937051472c65d79333b50ae9c3855.jpg)

命中union_idx索引，但跟示例1一样，只用到了id_no列，满足最左匹配原则。



『**反向示例**』

```sql
explain select * from t_user where username = 'Tom2' and age = 12;
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/3_640742dc301bab5f8eed1f9ceb690aab.jpg)

此时，未命中任何索引，即索引失效。

同样的，下面只要没出现最左条件的组合，索引也是失效的：



```sql
explain select * from t_user where age = 12;
explain select * from t_user where username = 'Tom2';
```



**即在联合索引的场景下，查询条件不满足最左匹配原则，索引失效**。



## 3.2 使用了 select *

在《**阿里巴巴开发手册**》的ORM映射章节中有一条【强制】的规范：

> 【**强制**】在表查询中，一律**不要使用 * 作为查询**的字段列表，需要哪些字段必须明确写明。
>
> 说明:
>
> 1)增加查询分析器解析成本。
>
> 2)增减字段容易与 resultMap 配置不一致。
>
> 3)无用字段增加网络 消耗，尤其是 text 类型的字段。



虽然在规范手册中没有提到索引方面的问题，但禁止使用select * 语句可能会带来的附带好处就是：某些情况下可以走**覆盖索引**。

比如，在上面的联合索引中，如果查询条件是`age`或`username`，当使用了select * ，肯定是不会走索引的。

但如果希望根据`username`查询出`id_no`、`username`、`age`这三个结果(**均为索引字段**)，明确查询结果字段，是可以走覆盖索引的：



```sql
explain select id_no, username, age from t_user where username = 'Tom2';
explain select id_no, username, age from t_user where age = 12;
```



explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/4_10dde794cf0fd4fec800f49ebff3d023.jpg)



## 3.3 索引列参与运算

直接来看示例：

```sql
explain select * from t_user where id + 1 = 2 ;
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/5_7e32f3011d6ae33b4d681f79812c596c.jpg)



可以看到，即便id列有索引，由于进行了计算处理，导致无法正常走索引。

针对这种情况，其实不单单是索引的问题，还会增加**数据库的计算负担**。就以上述SQL语句为例，数据库需要**全表扫描**出所有的id字段值，然后对其计算，计算之后再与参数值进行比较。如果每次执行都经历上述步骤，性能损耗可想而知。

建议的使用方式是：

- 先在内存中进行计算好预期的值
- 在SQL语句条件的右侧进行参数值的计算。



## 3.4 索引列使用函数

```sql
explain select * from t_user where SUBSTR(id_no,1,3) = '100';
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/6_829f9f68276fe0b41efbdb295c8e6a13.jpg)

上述示例中，索引列使用了函数(`SUBSTR`，字符串截取)，导致索引失效。

此时，索引失效的原因与第三种情况一样，都是因为数据库要先进行全表扫描，获得数据之后再进行截取、计算，导致索引索引失效。同时，还伴随着性能问题。

示例中只列举了`SUBSTR`函数，像`CONCAT`等类似的函数，也都会出现类似的情况。

## 3.5 错误的 like 使用

示例：

```sql
explain select * from t_user where id_no like '%00%';
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/7_fc8c1fb092689b3096451de8b0059c34.jpg)



针对like的使用非常频繁，但使用不当往往会导致不走索引。常见的like使用方式有：

- **方式 1**：like '%abc';
- **方式 2**：like 'abc%';
- **方式 3**：like '%abc%';

其中方式一和方式三，由于**占位符出现在首部**，导致无法走索引。



## 3.6 类型隐式转换

示例：

```
explain select * from t_user where id_no = 1002;
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/8_ebd6abf2de7a4c926e1298e210c94402.jpg)



id_no字段类型为`varchar`，但在SQL语句中使用了`int`类型，导致全表扫描。

出现索引失效的原因是：varchar和int是两个种**不同的类型**。



## 3.7 使用 OR 操作

OR是日常非常高频的操作关键字，但使用不当，也会导致索引失效。

示例：

```sql
explain select * from t_user where id = 2 or username = 'Tom2';
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/9_da46905d71c1d32e56100f97598808e6.jpg)



看到上述执行结果是否是很惊奇啊，明明**id字段是有索引**的，由于使用or关键字，索引竟然失效了。

其实，换一个角度来想，如果单独使用username字段作为条件很显然是全表扫描，既然已经进行了全表扫描了，前面id的条件再走一次索引反而是浪费了。所以，在使用or关键字时，切记**两个条件都要添加索引**，否则会导致索引失效。



但如果or两边同时使用` > 和 < `，则索引也会失效：

```sql
explain select * from t_user where id  > 1 or id  < 80;
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/10_5fb12f03f01fe064840c04f3c90b178b.jpg)



## 3.8 两列做比较

如果两个列数据都有索引，但在查询条件中对**两列数据进行了对比操作**，则会导致索引失效。

这里举个不恰当的示例，比如age小于id这样的两列(真实场景可能是两列同维度的数据比较，这里迁就现有表结构)：

```
explain select * from t_user where id > age;
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/11_11c4b4464985a11ea1f953109045bf16.jpg)



两列数据做比较，**即便两列都创建了索引，索引也会失效**。



## 3.9 不等于比较

示例：

```sql
explain select * from t_user where id_no <> '1002';
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/12_7ede7358f3fe942211977c08e552cb49.jpg)



当查询条件为字符串时，使用”<>“或”!=“作为条件查询，有可能不走索引，但也不全是。

上述语句是id进行不等操作，则正常走索引。

```
explain select * from t_user where id != 2;
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/13_99f16fab27d567af0011203039ab7b7b.jpg)



查询条件使用不等进行比较时，需要慎重，**普通索引会查询结果集占比较大时索引会失效**。当查询结果集占比比较小时，会走索引，占比比较大时不会走索引。此处与结果集与总体的占比有关。



## 3.10 is not null

示例：

```sql
explain select * from t_user where id_no is not null;
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/14_b2ce9b8415f67a37b0674c46b37701ac.jpg)



查询条件使用`is null`时正常走索引，使用`is not null`时，不走索引。



## 3.11 not in和not exists

在日常中使用比较多的范围查询有`in、exists、not in、not exists、between and`等。

```sql
explain select * from t_user where id in (2,3);

explain select * from t_user where id_no in ('1001','1002');

explain select * from t_user u1 where exists (select 1 from t_user u2 where u2.id  = 2 and u2.id = u1.id);

explain select * from t_user where id_no between '1002' and '1003';
```

上述四种语句执行时都会正常走索引，具体的explain结果就不再展示。主要看不走索引的情况：

```sql
explain select * from t_user where id_no not in('1002' , '1003');
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/15_be6c3cabcce3c600b81454a2b863457b.jpg)



- 查询条件使用 `not in` 时，如果是**主键则走索引**，如果是普通索引，则索引失效。
- 查询条件使用 `not exists` 时，索引失效。



## 3.12 order by 导致索引失效

『**order by 普通索引**』

```
explain select * from t_user order by id_no ;
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/16_bb03cb310acccdddf2e2c69e047dd905.jpg)



其实这种情况的索引失效很容易理解，毕竟需要对**全表数据**进行排序处理。



那么，添加删`limit`关键字是否就走索引了呢?

```
explain select * from t_user order by id_no limit 10;
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/17_db8526d008e97f514dd11b22af3b920c.jpg)



如果联合索引，`id_no='xxx' order by username`是可以走索引的

```
explain select * from t_user where id_no = 'NO10' order by username desc;
```

explain结果：

![image-20240713151108381](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/888_55f8df31d1c0b8f6299f9f42785dba17.png)





『**order by 主键索引**』

这里还有一个特例，就是主键使用order by时，可以正常走索引。

```sql
explain select * from t_user order by id desc;
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/19_cd45d7fbe6cdbc0a23dc3bf334545c8a.jpg)



『**order by 索引覆盖**』

另外，笔者测试如下SQL语句：

```
explain select id from t_user order by age;
explain select id , username from t_user order by age;
explain select id_no from t_user order by id_no;
```

上述三条SQL语句都是走索引的，也就是说**覆盖索引**的场景也是可以正常走索引的。



当查询条件涉及到`order by`、`limit`等条件时，是否走索引情况比较复杂，而且与Mysql版本有关，通常普通索引，如果未使用limit，则不会走索引。order by 多个索引字段时，可能不会走索引。其他情况，建议在使用时进行expain验证。



## 3.13 全表扫描效率大于索引

执行如下SQL：

```
explain select * from t_user where create_time > '2024-07-14 12:37:48';
```

其中，时间是**未来的时间**

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/20_f684bf57e5c667708d103567872ab7b8.jpg)

可以看到，正常走索引。



随后，我们将查询条件的参数换个日期，换成2023年

```
explain select * from t_user where create_time > '2023-07-14 12:37:48';
```

explain结果：

![img](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/21_32a09797fbc843a7af25bb3682ad97de.jpg)

此时，进行了全表扫描。



> 为什么同样的查询语句，只是查询的参数值不同，却会出现一个走索引，一个不走索引的情况呢?

答案很简单：上述索引失效是因为DBMS发现全表扫描比走索引效率更高，因此就放弃了走索引。

也就是说，当Mysql发现通过索引扫描的行记录数**超过全表的10%-30%时**，优化器可能会放弃走索引，自动变成全表扫描。某些场景下即便强制SQL语句走索引，也同样会失效。

类似的问题，在进行范围查询(比如`>、< 、>=、<=、in`等条件)时往往会出现上述情况，而上面提到的临界值根据场景不同也会有所不同。

## 小结

本篇文章为大家总结了12个常见的索引失效的场景，**由于不同的Mysql版本，索引失效策略也有所不同**。大多数索引失效情况都是明确的，有少部分索引失效会因Mysql的版本不同而有所不同。

因此，建议收藏本文，当在实践的过程中进行对照，如果没办法准确把握，则可直接执行`explain`进行验证。

> 引自

[索引失效场景](https://www.51cto.com/article/702691.html)