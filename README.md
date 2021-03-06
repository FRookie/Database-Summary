# Database-Summary

## 推荐一个可视化数据结构学习网站：https://www.cs.usfca.edu/~galles/visualization

## 数据库引擎：
### MyISAM(非聚集索引：索引（.myi文件）和数据(.myd文件)分开存储)：
 
    1)基于ISAM存储引擎，并在其基础上进行扩展，不支持事务。
    2)不支持外键（<kbd>Foreign_Key</kbd>）。
    3)读写操作属于串行模式，如果对同一张表进行读写操作时，写操作的进程会优先获取锁，因此不适合执行有大量读取和更新操作的数据，适合读操作多的少量数据。
    4)采用表级锁（共享锁和排它锁），这类锁的特点是：资源开销少，加锁的速度快，不会出现死锁，并发的程度低，加锁的粒度大，冲突概率比较高。
    5)不保存表记录数，执行'select count(1) from xxx'时会进行全表扫描。
    6)创建数据库将会产生3个文件，.frm存储表结构，.MYD存储表数据，.MYI存储表索引，索引被压缩过。(首先从.frm中加载判断表中字段是否添加索引等，有索引便从.myi中定位到索引的地址,根据地址从.myd文件中将数据加载进内存 )
### InnoDB(聚集索引：索引和数据存储在同一个文件中(.ibd文件)):
  
    1)为Mysql在事务提交，回滚，灾难恢复等安全方面提供了支持。
    2)支持外键(<kbd>Foreign_Key</kbd>)。
    3)与Mysql完全整合，是为了处理巨大量数据而设计。
    4)支持表级锁与行级锁，默认为行级锁。行级锁开销大，加锁慢，锁粒度小，冲突概率低，并发度高，会出现死锁。表级锁一般由数据库内部进行，因此不需要特别的关注。
    5)保留表记录数，当执行'select count(1) from xxx'时不会进行全表扫描。
    6)索引与数据紧密捆绑，索引没有压缩。在索引方面的内存使用率，不如MyISAM。

### 比较：

  |          |     MyISAM     |     InnoDB     |
  | :---- | :----: | :----: |
  | 存储限制 | 无限制 | 64TB |
  | 锁机制 | 表锁 | 表锁·行锁 |
  | 死锁 | 不会 | 会 |
  | B树索引 | 是 | 是 |
  | 全文索引 | 支持 |  |
  | 集群索引 | 不支持 | 支持 |
  | 数据可压缩 | 支持 | 不支持 |
  | 空间使用率 | 低 | 高 |
  | 内存使用率 | 低 | 高 |
  | 批量插入速度 | 高 | 低 |
  | 外键 | 不支持 | 支持 |
  | 事务 | 不支持 | 支持 |

### 事务：
> 1、什么是事务
   >> 事务是一条或多条数据库操作语句的组合，具备ACID，4个特点。 
   
   >>> 原子性：要不全部成功，要不全部撤销
   
   >>> 隔离性：事务之间相互独立，互不干扰
   
   >>> 一致性：数据库正确地改变状态后，数据库的一致性约束没有被破坏
   
   >>> 持久性：事务的提交结果，将持久保存在数据库中

> 2、事务并发会产生什么问题
 >> 1. 第一类丢失更新：在没有事务隔离的情况下，两个事务都同时更新一行数据，但是第二个事务却中途失败退出， 导致对数据的两个修改都失效了。
 >> 2. 脏读：脏读就是指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。
 >> 3. 不可重复读：是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。
 >> 4. 第二类丢失更新：不可重复读的特例。有两个并发事务同时读取同一行数据，然后其中一个对它进行修改提交，而另一个也进行了修改提交。这就会造成第一次写操作失效。 
 >> 5. 幻读：是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样。

#### 提醒：
不可重复读的重点是修改，同样的条件，你读取过的数据，再次读取出来发现值不一样了。<br/>
幻读的重点在于新增或者删除，同样的条件，第 1 次和第 2 次读出来的记录数不一样。<br/>

> 3、事务隔离级别，解决什么并发问题，以及存在什么并发问题

>> 1. READ UNCOMMITTED

   >>> 这是事务最低的隔离级别，它充许另外一个事务可以看到这个事务未提交的数据。

   >>> 解决第一类丢失更新的问题，但是会出现脏读、不可重复读、第二类丢失更新的问题，幻读 。
  
>> 2. READ COMMITTED

   >>> 保证一个事务修改的数据提交后才能被另外一个事务读取，即另外一个事务不能读取该事务未提交的数据。

   >>> 解决第一类丢失更新和脏读的问题，但会出现不可重复读、第二类丢失更新的问题，幻读问题。
  
>> 3. REPEATABLE READ

   >>> 保证一个事务相同条件下前后两次获取的数据是一致的。 
  
   >>> 解决第一类丢失更新，脏读、不可重复读、第二类丢失更新的问题，但会出幻读。
    
>> 4. SERIALIZABLE
   
   >>> 事务被处理为顺序执行。
   
   >>> 解决所有问题。

#### 提醒：

Mysql默认的事务隔离级别为repeatable read,Oracle为read commited。

> 4、MyISAM引擎的锁机制

>> 共享锁：可同时读、读的同时不能写

>> 独占锁：写的同时不能读和写


如何加锁：

select前，对所有涉及的表自动加共享锁

更新数据时，加独占写锁


MyISAM支持并发插入：读的时候插入，但是插入的数据当前事务无法读取

设置参数concurrent_Insert

0:不允许并发插入，1：无空洞，允许并发插入，2：有无都允许并发插入

空洞：删除记录造成

> 5、InnoDB引擎的锁机制

（之所以以InnoDB为主介绍锁，是因为InnoDB支持事务，支持行锁和表锁用的比较多，Myisam不支持事务，只支持表锁）
共享锁（S）：允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。<br/>
排他锁（X)：允许获得排他锁的事务更新数据，阻止其他事务取得相同数据集的共享读锁和排他写锁。<br/>
意向共享锁（IS）：事务打算给数据行加行共享锁，事务在给一个数据行加共享锁前必须先取得该表的IS锁。<br/>
意向排他锁（IX）：事务打算给数据行加行排他锁，事务在给一个数据行加排他锁前必须先取得该表的IX锁。<br/>

说明：

1）共享锁和排他锁都是行锁，意向锁都是表锁，应用中我们只会使用到共享锁和排他锁，意向锁是mysql内部使用的，不需要用户干预。

2）对于UPDATE、DELETE和INSERT语句，InnoDB会自动给涉及数据集加排他锁（X)；对于普通SELECT语句，InnoDB不会加任何锁，事务可以通过以下语句显示给记录集加共享锁或排他锁。

     共享锁（S）：SELECT * FROM table_name WHERE ... LOCK IN SHARE MODE。
     排他锁（X)：SELECT * FROM table_name WHERE ... FOR UPDATE。

3）InnoDB行锁是通过给索引上的索引项加锁来实现的，因此InnoDB这种行锁实现特点意味着：只有通过索引条件检索数据，InnoDB才使用行级锁，否则，InnoDB将使用表锁！。

## drop、truncate和delete的区别

> （1）DELETE语句执行删除的过程是每次从表中删除一行，并且同时将该行的删除操作作为事务记录在日志中保存以便进行进行回滚操作。

>    TRUNCATE TABLE 则一次性地从表中删除所有的数据并不把单独的删除操作记录记入日志保存，删除行是不能恢复的。并且在删除的过程中不会激活与表有关的删除触发器。执行速度快。

> （2）表和索引所占空间。

  >>  当表被TRUNCATE 后，这个表和索引所占用的空间会恢复到初始大小，

   >> DELETE操作不会减少表或索引所占用的空间。

   >> drop语句将表所占用的空间全释放掉。

> （3）一般而言，drop > truncate > delete

> （4）应用范围。

    TRUNCATE 只能对TABLE；         DELETE可以是table和view

> （5）TRUNCATE 和DELETE只删除数据， DROP则删除整个表（结构和数据）。

> （6）truncate与不带where的delete ：只删除数据，而不删除表的结构（定义）drop语句将删除表的结构被依赖的约束（constrain),触发器（trigger)索引（index);依赖于该表的存储过程/函数将被保留，但其状态会变为：invalid。

> （7）delete语句为DML（data maintain Language),这个操作会被放到 rollback segment中,事务提交后才生效。如果有相应的 tigger,执行的时候将被触发。

> （8）truncate、drop是DLL（data define language),操作立即生效，原数据不放到 rollback segment中，不能回滚

> （9）在没有备份情况下，谨慎使用 drop 与 truncate。要删除部分数据行采用delete且注意结合where来约束影响范围。回滚段要足够大。要删除表用drop;若想保留表而将表中数据删除，如果于事务无关，用truncate即可实现。如果和事务有关，或老师想触发trigger,还是用delete。

> （10） Truncate table 表名 速度快,而且效率高,因为: 
>> truncate table 在功能上与不带 WHERE 子句的 DELETE 语句相同：二者均删除表中的全部行。但 TRUNCATE TABLE 比 DELETE 速度快，且使用的系统和事务日志资源少。DELETE 语句每次删除一行，并在事务日志中为所删除的每行记录一项。TRUNCATE TABLE 通过释放存储表数据所用的数据页来删除数据，并且只在事务日志中记录页的释放。 

> （11） TRUNCATE TABLE 删除表中的所有行，但表结构及其列、约束、索引等保持不变。新行标识所用的计数值重置为该列的种子。如果想保留标识计数值，请改用 DELETE。如果要删除表定义及其数据，请使用 DROP TABLE 语句。 

> （12） 对于由 FOREIGN KEY 约束引用的表，不能使用 TRUNCATE TABLE，而应使用不带 WHERE 子句的 DELETE 语句。由于 TRUNCATE TABLE 不记录在日志中，所以它不能激活触发器。

## 数据库SQL语句执行顺序(应用于索引查询优化)

> + 完整语句

    SELECT 
    DISTINCT <select_list>
    FROM <left_table>
    <join_type> JOIN <right_table>
    ON <join_condition>
    WHERE <where_condition>
    GROUP BY <group_by_list>
    HAVING <having_condition>
    ORDER BY <order_by_condition>
    LIMIT <limit_number>
 
> + 执行顺序

    from →join →on →where →group by→having→select→order by→limit
    
> + 各个关键词的作用

    from:需要从哪个数据表检索数据
    join：联合多表查询返回记录时，并生成一张临时表
    on：在生成临时表时使用的条件
    where:过滤表中数据的条件
    group by:如何将上面过滤出的数据分组
    having:对上面已经分组的数据进行过滤的条件
    select:查看结果集中的哪个列，或列的计算结果
    order by :按照什么样的顺序来查看返回的数据
    limit：限制查询结果返回的数量

> + on与where的用法区别

    on后面的筛选条件主要是针对的是关联表【而对于主表刷选条件不适用】。
    如果是想再连接完毕后才筛选就应把条件放置于where后面。对于关联表我们要区分对待。如果是要条件查询后才连接应该把查询件放置于on后。
    对于主表的筛选条件应放在where后面，不应该放在on后面

> + having和where的用法区别

    having只能用在group by之后，对分组后的结果进行筛选(即使用having的前提条件是分组)。
    where肯定在group by 之前，即也在having之前。
    where后的条件表达式里不允许使用聚合函数，而having可以。

> + count用法

    使用count（列名）当某列出现null值的时候，count（*）仍然会计算，但是count(列名)不会。

## Explain查看SQL执行计划

| id | select_type | table | possible_keys | key | key_len | ref | rows | Extra |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |

其中重要的字段为：id、type、key、rows、Extra

### 各字段介绍

#### id

select 查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序：

> id

>> id相同，执行顺序由上至下

>> id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

>> id相同又不同（两种情况同时出现），id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行

#### select_type

查询的类型，主要是用于区分普通查询、联合查询、子查询等复杂的查询

    1、SIMPLE：简单的select查询，查询中不包含子查询或者union 
    2、PRIMARY：查询中包含任何复杂的子部分，最外层查询则被标记为primary 
    3、SUBQUERY：在select 或 where列表中包含了子查询 
    4、DERIVED：在from列表中包含的子查询被标记为derived（衍生），mysql或递归执行这些子查询，把结果放在零时表里 
    5、UNION：若第二个select出现在union之后，则被标记为union；若union包含在from子句的子查询中，外层select将被标记为derived 
    6、UNION RESULT：从union表获取结果的select 
#### type

访问类型，sql查询优化中一个很重要的指标，结果值从好到坏依次是：

    system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

一般来说，好的sql查询至少达到range级别，最好能达到ref

>> 1、system：表只有一行记录（等于系统表），这是const类型的特例，平时不会出现，可以忽略不计

>> 2、const：表示通过索引一次就找到了，const用于比较primary key 或者 unique索引。因为只需匹配一行数据，所有很快。如果将主键置于where列表中，mysql就能将该查询转换为一个const 

>> 3、eq_ref：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键 或 唯一索引扫描。

>> 4、ref：非唯一性索引扫描，返回匹配某个单独值的所有行。本质是也是一种索引访问，它返回所有匹配某个单独值的行，然而他可能会找到多个符合条件的行，所以它应该属于查找和扫描的混合体

>> 5、range：只检索给定范围的行，使用一个索引来选择行。key列显示使用了那个索引。一般就是在where语句中出现了bettween、<、>、in等的查询。这种索引列上的范围扫描比全索引扫描要好。只需要开始于某个点，结束于另一个点，不用扫描全部索引 

>> 6、index：Full Index Scan，index与ALL区别为index类型只遍历索引树。这通常为ALL块，应为索引文件通常比数据文件小。（Index与ALL虽然都是读全表，但index是从索引中读取，而ALL是从硬盘读取）

>> 7、ALL：Full Table Scan，遍历全表以找到匹配的行 

#### possible_keys

查询涉及到的字段上存在索引，则该索引将被列出，但不一定被查询实际使用

#### key

实际使用的索引，如果为NULL，则没有使用索引。 <br/>
查询中如果使用了覆盖索引，则该索引仅出现在key列表中 

#### key_len

表示索引中使用的字节数，查询中使用的索引的长度（最大可能长度），并非实际使用长度，理论上长度越短越好。key_len是根据表定义计算而得的，不是通过表内检索出的

#### ref

显示索引的那一列被使用了，如果可能，是一个常量const。

#### rows

根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数

#### Extra

不适合在其他字段中显示，但是十分重要的额外信息

> Using filesort  

>> mysql对数据使用一个外部的索引排序，而不是按照表内的索引进行排序读取。也就是说mysql无法利用索引完成的排序操作成为“文件排序”

> Using temporary

>> 使用临时表保存中间结果，也就是说mysql在对查询结果排序时使用了临时表，常见于order by 和 group by 

> Using index

>> 表示相应的select操作中使用了覆盖索引（Covering Index），避免了访问表的数据行，效率高

>> 如果同时出现Using where，表明索引被用来执行索引键值的查找 

>> 如果没用同时出现Using where，表明索引用来读取数据而非执行查找动作 

覆盖索引（Covering Index）：也叫索引覆盖。就是select列表中的字段，只用从索引中就能获取，不必根据索引再次读取数据文件，换句话说查询列要被所建的索引覆盖。 

     注意： 
     a、如需使用覆盖索引，select列表中的字段只取出需要的列，不要使用select * 
     b、如果将所有字段都建索引会导致索引文件过大，反而降低crud性能

> Using where

>> 使用了where过滤

> Using join buffer

>> 使用了链接缓存

> Impossible WHERE

>>　where子句的值总是false，不能用来获取任何元祖 

> select tables optimized away

>> 在没有group by子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎优化COUNT（*  操作，不必等到执行阶段在进行计算，查询执行计划生成的阶段即可完成优化

> distinct

>> 优化distinct操作，在找到第一个匹配的元祖后即停止找同样值得动作
