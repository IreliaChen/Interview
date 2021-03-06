# 综述

其实是关系数据库，主要是mysql



# 存储引擎

## innodb

- 支持事务的存储引擎，合于插入和更新操作比较多的应用，支持行级锁（最大区别就在锁的级别上），适合大数据，大并发。
- Innodb引擎的默认**索引结构**是B+Tree，支持 B-tree、Full-text 等索引，不支持 Hash 索引；
- Innodb的索引文件本身就是数据文件，即B+Tree的数据域存储的就是实际的数据，这种索引就是聚集索引。这个索引的key就是数据表的主键，因此InnoDB表数据文件本身就是主索引。



## myisam

- 非事务的存储引擎，适合用于频繁查询的应用。支持表级别锁定，不会出现死锁，适合小数据，小并发。
- 没有提供对数据库事务的支持，也不支持行级锁和外键，因此当INSERT(插入)或UPDATE(更新)数据时即写操作需要锁定整个表，效率便会低一些。
- 不过和Innodb不同，MyIASM中存储了表的行数，于是SELECT COUNT(*) FROM TABLE时只需要直接读取已经保存好的值而不需要进行全表扫描。
- MyISAM引擎的默认**索引结构为B+Tree**，其中B+Tree的数据域存储的内容为实际数据的地址，也就是说它的索引和实际的数据是分开的，只不过是用索引指向了实际的数据，这种索引就是所谓的非聚集索引。
- 支持 B-tree、Full-text 等索引，不支持 Hash 索引



区别：

- InnoDB支持事务，而MyISAM不支持事务
- InnoDB支持行级锁，而MyISAM支持表级锁
- InnoDB支持MVCC, 而MyISAM不支持
- InnoDB支持外键，而MyISAM不支持
- InnoDB不支持全文索引，而MyISAM支持



# 索引（mysql）

## 类型

MySQL目前主要有以下几种索引类型：

1. **普通索引**（index ）

   是最基本的索引，它没有任何限制。

2. **唯一索引**

   - **主键索引**（primary key）

     是一种特殊的唯一索引，一个表只能有一个主键，不允许有空值。一般是在建表的时候同时创建主键索引

   - **唯一索引**（unique）

     与前面的普通索引类似，不同的就是：索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一

3. **组合索引** 

   指多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。使用组合索引时遵循**最左前缀集合**

   - primary key(id,name):联合主键索引
   - unique(id,name):联合唯一索引
   - index(id,name):联合普通索引



5. **全文索引**

   主要用来查找文本中的关键字，而不是直接与索引中的值相比较。fulltext索引跟其它索引大不相同，它更像是一个搜索引擎，而不是简单的where语句的参数匹配。fulltext索引配合match
   against操作使用，而不是一般的where语句加like。它可以在create table，alter table ，create 
   index使用，不过目前只有char、varchar，text 
   列上可以创建全文索引。值得一提的是，在数据量较大时候，现将数据放入一个没有全局索引的表中，然后再用CREATE 
   index创建fulltext索引，要比先为一张表建立fulltext然后再将数据写入的速度快很多。

6. **空间索引**

   几乎不用







## 名词解释

> ## 主键索引和非主键索引有什么区别？

![表](截图/Mysql/表.png)

![主键索引](截图/Mysql/主键索引.png)

从图中不难看出，主键索引和非主键索引的区别是：

- 非主键索引的叶子节点存放的是**主键的值**，而主键索引的叶子节点存放的是**整行数据**，其中非主键索引也被称为**二级索引**，而主键索引也被称为**聚簇索引**。
- 非主键的查询方式，则先搜索k索引树，得到ID=100,再到ID索引树搜索一次，这个过程也被称为回表。



[面试小知识：MySQL索引相关](https://mp.weixin.qq.com/s/yP25TAqHCN4umwDSN53qPQ)



> ## 聚簇索引 & 非聚簇索引

这两个名字虽然都叫做索引，但这并不是一种单独的索引类型，而是一种数据存储方式。对于聚簇索引存储来说，行数据和主键B+树存储在一起，辅助键B+树只存储辅助键和主键，主键和非主键B+树几乎是两种类型的树。对于非聚簇索引存储来说，主键B+树在叶子节点存储指向真正数据行的指针，而非主键。

- 所谓聚簇索引，就是指主**索引文件和数据文件为同一份文件**，聚簇索引主要用在Innodb存储引擎中。在该索引实现方式中B+Tree的叶子节点上的data就是数据本身，key为主键，如果是一般索引的话，data便会指向对应的主索引。
- 非聚簇索引就是指B+Tree的叶子节点上的data，并不是数据本身，而是数据存放的地址。主索引和辅助索引没啥区别，只是主索引中的key一定得是唯一的。



[B-tree/b+tree 原理以及聚簇索引和非聚簇索引](https://blog.csdn.net/u010727189/article/details/79399384)



> ## 覆盖索引

如果一个索引包含(或覆盖)所有需要查询的字段的值，称为‘覆盖索引’。即只需扫描索引而无须回表。

只扫描索引而无需回表的优点：

1. 索引条目通常远小于数据行大小，只需要读取索引，则mysql会极大地减少数据访问量。
2. 因为索引是按照列值顺序存储的，所以对于IO密集的范围查找会比随机从磁盘读取每一行数据的IO少很多。
3. 一些存储引擎如myisam在内存中只缓存索引，数据则依赖于操作系统来缓存，因此要访问数据需要一次系统调用
4. innodb的聚簇索引，覆盖索引对innodb表特别有用。(innodb的二级索引在叶子节点中保存了行的主键值，所以如果二级主键能够覆盖查询，则可以避免对主键索引的二次查询)



## 索引的缺点

1. 虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行insert、update和delete。因为更新表时，不仅要保存数据，还要保存一下索引文件。
2. 建立索引会占用磁盘空间的索引文件。一般情况这个问题不太严重，但如果你在一个大表上创建了多种组合索引，索引文件的会增长很快。索引只是提高效率的一个因素，如果有大数据量的表，就需要花时间研究建立最优秀的索引，或优化查询语句。



## 索引结构

> ## B+Tree索引

- B+Tree索引是最为常见的MySQL索引类型，一般谈论MySQL索引时，如果没有特别说明，就是指B+Tree索引。
- B+Tree索引使用B+Tree作为其存储数据的数据结构。一般来说，B+Tree索引适用于全键值、键值范围和键前缀查找，其中键前缀查找只适用于根据最左前缀查找。
- B+Tree索引支持的查询原则如下所示：
  - 全值匹配：全值匹配指的是和索引中的所有列进行匹配，
  - 匹配最左前缀：前边提到的索引可以用于查找所有姓Allen的人，即只使用索引中的第一列。
  - 匹配列前缀：也可以只匹配某一列的值的开头部分。例如前面提到的索引可用于查找所有以J开头的姓的人。这里也只用到了索引的第一列。
  - 匹配范围值：例如前边提到的索引可用于查找姓在Allen和Barrymore之间的人。这里也只使用了索引的第一列。
  - 精确匹配某一列并范围匹配另外一列：前边提到的索引也可用于查找所有姓为Allen，并且名字是字母K开头(比如Kim,Karl等)的人。即第一列last_name全匹配，第二列first_name范围匹配。



![b+树索引数据结构](截图/Mysql/b+树索引数据结构.png)



​	如上图，是一颗b+树，浅蓝色的块为一个磁盘块，可以看到每个磁盘块包含几个数据项（深蓝色所示）和指针（黄色所示），如磁盘块1包含数据项17和35，包含指针P1、P2、P3，P1表示小于17的磁盘块，P2表示在17和35之间的磁盘块，P3表示大于35的磁盘块。真实的数据存在于叶子节点即3、5、9、10、13、15、28、29、36、60、75、79、90、99。**非叶子节点只不存储真实的数据**，只存储指引搜索方向的数据项，如17、35并不真实存在于数据表中。



### b+ 树的查找过程

​	如图所示，如果要查找数据项29，那么首先会把磁盘块1由磁盘加载到内存，此时发生一次IO，在内存中用二分查找确定29在17和35之间，锁定磁盘块1的P2指针，内存时间因为非常短（相比磁盘的IO）可以忽略不计，通过磁盘块1的P2指针的磁盘地址把磁盘块3由磁盘加载到内存，发生第二次IO，29在26和30之间，锁定磁盘块3的P2指针，通过指针加载磁盘块8到内存，发生第三次IO，同时内存中做二分查找找到29，结束查询，总计三次IO。真实的情况是，3层的b+树可以表示上百万的数据，如果上百万的数据查找只需要三次IO，性能提高将是巨大的，如果没有索引，每个数据项都要发生一次IO，那么总共需要百万次的IO，显然成本非常非常高。



### b+ 树性质

1. **索引字段要尽量的小**：通过上面的分析，我们知道IO次数取决于b+树的高度h，假设当前数据表的数据为N，每个磁盘块的数据项的数量是m，则有h=㏒(m+1)N，当数据量N一定的情况下，m越大，h越小；而m
   = 磁盘块的大小 / 数据项的大小，磁盘块的大小也就是一个数据页的大小，是固定的，如果数据项占的空间越小，数据项的数量越多，树的高度越低。这就是为什么每个数据项，即索引字段要尽量的小，比如int占4字节，要比bigint8字节少一半。这也是为什么b+树要求把真实的数据放到叶子节点而不是内层节点，一旦放到内层节点，磁盘块的数据项会大幅度下降，导致树增高。当数据项等于1时将会退化成线性表。
2. **索引的最左匹配特性（即从左往右匹配）**：当b+树的数据项是复合的数据结构，比如(name,age,sex)的时候，b+数是按照从左到右的顺序来建立搜索树的，比如当(张三,20,F)这样的数据来检索的时候，b+树会优先比较name来确定下一步的所搜方向，如果name相同再依次比较age和sex，最后得到检索的数据；但当(20,F)这样的没有name的数据来的时候，b+树就不知道下一步该查哪个节点，因为建立搜索树的时候name就是第一个比较因子，必须要先根据name来搜索才能知道下一步去哪里查询。比如当(张三,F)这样的数据来检索时，b+树可以用name来指定搜索方向，但下一个字段age的缺失，所以只能把名字等于张三的数据都找到，然后再匹配性别是F的数据了，
   这个是非常重要的性质，即索引的最左匹配特性



> ## hash索引

//todo



[mysql索引最左匹配原则的理解](https://blog.csdn.net/u013164931/article/details/82386555)

## 注意事项

1. **索引不会包含有null值的列。**只要列中包含有null值都将不会被包含在索引中，复合索引中只要有一列含有null值，那么这一列对于此复合索引就是无效的。所以我们在数据库设计时不要让字段的默认值为null。
2. **使用短索引**。对串列进行索引，如果可能应该指定一个前缀长度。例如，如果有一个char(255)的列，如果在前10个或20个字符内，多数值是惟一的，那么就不要对整个列进行索引。短索引不仅可以提高查询速度而且可以节省磁盘空间和I/O操作。
3. **索引列排序**。查询**只使用一个**索引，因此如果where子句中已经使用了索引的话，那么order by中的列是不会使用索引的。因此数据库默认排序可以符合要求的情况下不要使用排序操作；尽量不要包含多个列的排序，如果需要最好给这些列创建复合索引。
4. **like语句操作**。一般情况下不推荐使用like操作，如果非使用不可，如何使用也是一个问题。like “%aaa%” 不会使用索引而like “aaa%”可以使用索引。
5. **不要在列上进行运算**。这将导致索引失效而进行全表扫描，例如

```sql
SELECT * FROM table_name WHERE YEAR(column_name) < 2017;
```

6. **不使用not in和<>操作**



# 日志

错误日志：记录出错信息，也记录一些警告信息或者正确的信息。
查询日志：记录所有对数据库请求的信息，不论这些请求是否得到了正确的执行。
慢查询日志：设置一个阈值，将运行时间超过该值的所有SQL语句都记录到慢查询的日志文件中。
二进制日志：记录对数据库执行更改的所有操作。
中继日志：
事务日志：数据库的事务功能基于此日志实现



> #### 慢查询日志

MySQL的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超过阀值的语句，具体指运行时间超过long_query_time值的SQL，则会被记录到慢查询日志中。long_query_time的**默认值为10**，意思是运行10S以上的语句。默认情况下，Mysql数据库并不启动慢查询日志，需要我们手动来设置这个参数，当然，如果不是调优需要的话，一般不建议启动该参数，因为开启慢查询日志会或多或少带来一定的性能影响。慢查询日志支持将日志记录写入文件，也支持将日志记录写入数据库表。



查看是否开启，slow_query_log 为 on则为开启。

```
mysql> show variables like 'slow_query%';
+---------------------------+----------------------------------+
| Variable_name             | Value                            |
+---------------------------+----------------------------------+
| slow_query_log            | OFF                              |
| slow_query_log_file       | /mysql/data/localhost-slow.log   |
+---------------------------+----------------------------------+

mysql> show variables like 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
```



开启方法

1. 方法一：全局变量设置
   将 slow_query_log 全局变量设置为“ON”状态

   ```
   mysql> set global slow_query_log='ON'; 
   ```

   设置慢查询日志存放的位置

   ```
   mysql> set global slow_query_log_file='/usr/local/mysql/data/slow.log';
   ```

   查询超过1秒就记录

   ```
   mysql> set global long_query_time=1;
   ```



2. 配置文件设置
   修改配置文件my.cnf，在[mysqld]下的下方加入

   ```
   [mysqld]
   slow_query_log = ON
   slow_query_log_file = /usr/local/mysql/data/slow.log
   long_query_time = 1
   ```

   重启MySQL服务

   ```
   service mysqld restart
   ```



![慢查询结果图](截图/Mysql/慢查询结果图.png)



# 事务

事务是通过日志来实现的。事务日志是通过redo和innodb的存储引擎日志缓冲（Innodb log buffer）来实现的，当开始一个事务的时候，会记录该事务的lsn(log sequence number)号; 当事务执行时，会往InnoDB存储引擎日志缓存里面插入事务日志；当事务提交时，必须将存储引擎的日志缓冲写入磁盘（通过innodb_flush_log_at_trx_commit来控制），也就是写数据前，需要先写日志。这种方式称为“预写日志方式”



## 隔离级别

| 事务隔离级别                   | 脏读 | 不可重复读 | 幻读 |
| ------------------------------ | ---- | ---------- | ---- |
| 读未提交（read-uncommitted）   | 是   | 是         | 是   |
| 读取提交内容（read-committed） | 否   | 是         | 是   |
| 可重复读（repeatable-read）    | 否   | 否         | 是   |
| 串行化（serializable）         | 否   | 否         | 否   |

Mysql 默认:可重复读 
Oracle默认:读已提交



1. 脏读：事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数，一个事务读到了另一个事务的未提交的数据 。
2. 不可重复读：事务 A 多次读取同一数据，事务 B 在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果不一致。一个事务读到了另一个事务已经提交的 update 的数据导致多次查询结果不一致. 
3. 幻读：系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级，但是系统管理员B就在这个时候插入了一条具体分数的记录，当系统管理员A改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样，这就叫幻读。一个事务读到了另一个事务已经提交的 insert 的数据导致多次查询结果不一致.

**小结：不可重复读的和幻读很容易混淆，不可重复读侧重于修改，幻读侧重于新增或删除。解决不可重复读的问题只需锁住满足条件的行，解决幻读需要锁表**





# 锁 

## 共享锁

1. 共享锁（Shared Lock，也叫S锁）又称**读锁**，是读取操作创建的锁。因此多个事务可以同时为一个对象加共享锁。
2. 用户可以并发读取数据，但任何事务都不能对数据进行修改（获取数据上的排他锁），直到已释放所有共享锁。
3. 如果事务T对数据A加上共享锁后，则其他事务只能对A再加共享锁，不能加排他锁。获准共享锁的事务只能读数据，不能修改数据。



  　　SELECT ... LOCK IN SHARE MODE走的是IS锁(意向共享锁)，即在符合条件的rows上都加了共享锁，这样的话，其他人**可以读取**这些记录，也可以继续添加IS锁，但是**无法修改**这些记录直到你这个加锁的过程执行完成(完成的情况有：事务的提交，事务的回滚，否则直接锁等待超时)。



**共享锁的使用场景**

  　　SELECT ... LOCK IN SHARE  MODE的应用场景适合于两张表存在关系时的写操作，拿mysql官方文档的例子来说，一个表是child表，一个是parent表，假设child表的某一列child_id映射到parent表的c_child_id列，那么从业务角度讲，此时我直接insert一条child_id=100记录到child表是存在风险的，因为刚insert的时候可能在parent表里删除了这条c_child_id=100的记录，那么业务数据就存在不一致的风险。正确的方法是再插入时执行

`select * from parent where c_child_id=100 lock in share mode`,锁定了parent表的这条记录，然后执行insert into child(child_id) values (100)就不会存在这种问题了。



## 排他锁

1. 排他锁(Exclusive Lock，也叫X锁)也叫**写锁**(X)。

2. 如果事务T对数据A加上排他锁后，则其他事务不能再对A加任何类型的封锁。获准排他锁的事务既能读数据，又能修改数据。

    

**排他锁的使用场景：**

**使用场景一：订单的商品数量**

​	但是如果是同一张表的应用场景，举个例子，电商系统中计算一种商品的剩余数量，在产生订单之前需要确认商品数量>=1,产生订单之后应该将商品数量减1。

​	显然做法是是有问题，因为如果1查询出amount为1，但是这时正好其他session也买了该商品并产生了订单，那么amount就变成了0，那么这时第二步再执行就有问题。那么采用lock in share mode可行吗，也是不合理的，因为两个session同时锁定该行记录时，这时两个session再update时必然会产生死锁导致事务回滚。

**使用场景二：数据表的状态**



## 意向锁

InnoDB还有两个表锁：

- 意向共享锁（IS）：表示事务准备给数据行加入共享锁，也就是说一个数据行加共享锁前必须先取得该表的IS锁
- 意向排他锁（IX）：类似上面，表示事务准备给数据行加入排他锁，说明事务在一个数据行加排他锁前必须先取得该表的IX锁。

意向锁是InnoDB自动加的，不需要用户干预。

对于insert、update、delete，InnoDB会自动给涉及的数据加排他锁（X）；对于一般的Select语句，InnoDB不会加任何锁，事务可以通过以下语句给显示加共享锁或排他锁。

共享锁：`SELECT ... LOCK IN SHARE MODE;`

排他锁：`SELECT ... FOR UPDATE;`



**InnoDB行锁实现方式**

InnoDB行锁是通过索引上的索引项来实现的，这一点ＭySQL与Oracle不同，后者是通过在数据中对相应数据行加锁来实现的。InnoDB这种行锁实现特点意味者：只有通过索引条件检索数据，InnoDB才会使用行级锁，否则，InnoDB将使用表锁！

在实际应用中，要特别注意InnoDB行锁的这一特性，不然的话，可能导致大量的锁冲突，从而影响并发性能。

例: select * from tab_with_index where id = 1 for update;
for update 可以根据条件来完成行锁锁定,并且 id 是有索引键的列,
如果 **id 不是索引键那么InnoDB将完成表锁**，并发将无从谈起



## 乐观锁

​	乐观锁**不是数据库自带的**，需要我们自己去实现。乐观锁是指操作数据库时(更新操作)，想法很乐观，认为这次的操作不会导致冲突，在操作数据时，并不进行任何其他的特殊处理（也就是不加锁），而在进行更新后，再去判断是否有冲突了。

通常实现是这样的：在表中的数据进行操作时(更新)，先给数据表加一个**版本(version)**字段，每操作一次，将那条记录的版本号加1。也就是先查询出那条记录，获取出version字段,如果要对那条记录进行操作(更新),则先判断此刻version的值是否与刚刚查询出来时的version的值相等，如果相等，则说明这段期间，没有其他程序对其进行操作，则可以执行更新，将version字段的值加1；如果更新时发现此刻的version值与刚刚获取出来的version的值不相等，则说明这段期间已经有其他程序对其进行操作了，则不进行更新操作。

**时间戳（使用数据库服务器的时间戳）**、**待更新字段**、**所有字段** 



悲观锁

​	与乐观锁相对应的就是悲观锁了。悲观锁就是在操作数据时，认为此操作会出现数据冲突，所以在进行每次操作时都要通过获取锁才能进行对相同数据的操作，这点跟java中的synchronized很相似，所以悲观锁需要耗费较多的时间。另外与乐观锁相对应的，悲观锁是由数据库自己实现了的，要用的时候，我们直接调用数据库的相关语句就可以了。

说到这里，由悲观锁涉及到的另外两个锁概念就出来了，它们就是共享锁与排它锁。**共享锁和排它锁是悲观锁的不同的实现，它俩都属于悲观锁的范畴**。





# 优化

> #### explain

通过 Explain 命令来分析低效SQL的执行计划。

示例 explain select * from adminlog

执行结果:

| id   | select_type | table    | partitjons | type | possible_keys | key  | key_len | ref  | row  | filtered | Extra |
| ---- | ----------- | -------- | ---------- | ---- | ------------- | ---- | ------- | ---- | ---- | -------- | ----- |
| 1    | SIMPLE      | adminlog |            | ALL  |               |      |         |      | 2    | 100      |       |

mysql查看是否使用索引，简单的看type类型就可以。如果它是all，那说明这条查询语句遍历了所有的行，并没有使用到索引。



执行结果每一列的说明:

1. select_type : 查询类型，常见的值:

   - SIMPLE：简单表，不使用表连接或子查询。
   - PRIMARY : 主查询，外层的查询。UNION 第二个或者后面的查询语句。
   - SUBQUERY : 子查询中的第一个select

2. table : 输出结果的表

3. type : 表示MySql在表中找到所需行的方式，或者叫访问类型。常见的类型：

   | ALL  | index | range | ref  | eq_ref | const | system | NULL |
   | ---- | ----- | ----- | ---- | ------ | ----- | ------ | ---- |
   |      |       |       |      |        |       |        |      |

   从左到右，性能由最差到最好。

   - type＝ALL全表扫描

   - type＝index 索引全扫描，遍历整个索引来查询匹配的行

   - type = range 索引范围扫描，常见于　<,<=,>,>=,between,in等操作符。

     ```
     　例: 
     　	  explain select * from adminlog where id>0 , 
     　　　　explain select * from adminlog where id>0 and id<=100
     　　　　explain select * from adminlog where id in (1,2) 
     ```

   - type=ref　使用非唯一索引或唯一索引的前缀扫描，返回匹配某个单独值的记录行。ref还经常出现在JOIN操作中

   - type=eq_ref 类似于ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中有一条记录匹配；简单来说，说是多表连接中使用　主建或唯一健作为关联条件

   - type=const/system 单表中最多有一个匹配行。主要用于比较primary key [主键索引]或者unique[唯一]索引,因为数据都是唯一的，所以性能最优。条件使用=。 

   - type=NULL　不用访问表或者索引，直接就能够得到结果　



5. possible_keys
   查询可能使用到的索引都会在这里列出来

6. key
   查询真正使用到的索引，select_type为index_merge时，这里可能出现两个以上的索引，其他的select_type这里只会出现一个。

7. key_len
   用于处理查询的索引长度，如果是单列索引，那就整个索引长度算进去，如果是多列索引，那么查询不一定都能使用到所有的列，具体使用到了多少个列的索引，这里就会计算进去，没有使用到的列，这里不会计算进去。留意下这个列的值，算一下你的多列索引总长度就知道有没有使用到所有的列了。要注意，mysql的ICP特性使用到的索引不会计入其中。另外，key_len只计算where条件用到的索引长度，而排序和分组就算用到了索引，也不会计算到key_len中。

8. ref
   如果是使用的常数等值查询，这里会显示const，如果是连接查询，被驱动表的执行计划这里会显示驱动表的关联字段，如果是条件使用了表达式或者函数，或者条件列发生了内部隐式转换，这里可能显示为func

9. rows
   这里是执行计划中估算的扫描行数，不是精确值

10. Extra

    该列包含MySQL解决查询的详细信息。



[MySql优化-你的SQL命中索引了吗](https://www.cnblogs.com/stevenchen2016/p/5770214.html)

[MySQL的Explain关键字查看是否使用索引](https://www.cnblogs.com/acm-bingzi/p/mysqlExplain.html)

[mysql explain用法和结果的含义](https://www.cnblogs.com/yycc/p/7338894.html)



> #### profile

Show profile 是mysql 提供可以用来分析当前会话中语句执行的资源消耗情况。



查看当前的mysql版本是否开启，默认关闭，使用前需要开启；

```
show variables like ‘profiling%’;

set profiling=on
```



**使用**：

```
select goods_name from ecs_goods where goods_id <5000;
show  profiles;
show profile cpu, block io for query1 
```



**show profile 的格式如下：**

SHOW PROFILE [type [, type] ... ]

​    [FOR QUERY n]

​    [LIMIT row_count [OFFSET offset]]

 

type:

​    ALL

| BLOCK IO

| CONTEXT SWITCHES

| CPU

| IPC

| MEMORY

| PAGE FAULTS

| SOURCE

| SWAPS 



> #### 查询优化几个方向

1. 尽量避免全文扫描，给相应字段增加索引，应用索引来查询
2. 删除不用或者重复的索引
3. 查询重写，等价转换（谓词、子查询、连接查询）
4. 删除内容重复不必要的语句，精简语句
5. 整合重复执行的语句
6. 缓存查询结果



> #### SQL语句优化

1. 连接效率大于子查询

   查询优化器对子查询一般采用嵌套执行的方式，即对父查询中的每一行，都执行一次子查询，这样子查询会执行很多次。这种执行方式效率很低。

2. 别用*

3. explain、profile分析一下

4. join小标驱动大表

5. 千万级分页用limit

6. BETWEEEN AND改写为 >= 、<=之类的。实测：十万条数据，重写前后时间，1.45s、0.06s

7. in转换多个or。字段为索引时，两个都能用到索引，or效率相对in好一点

8. name like ‘abc%’改写成name>=’abc’ and name<’abd’;



> #### 索引优化

对查询进行优化，应尽量避免全表扫描，首先应考虑在 where 及 order by涉及的列上建立索引

1. 字段尽可能not null

2. 避免在 where 子句中使用 !=或<> 操作符，否则将引擎放弃使用索引而进行全表扫描。

3. 应尽量避免在 where 子句中对字段进行 null 值 判断，否则将导致引擎放弃使用索引而进行全表扫描，可以设置0。

4. 应尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描

   ```
      select id from t where num=10 or num=20
   　　可以这样查询：
   　　select id from t where num=10
   　　union all
   　　select id from t where num=20
   ```

5. %abc 也会导致全表扫描

6. in 和 not in 也要慎用，否则会导致全表扫描，对于连续的数值，能用 between 就不要用 in 了。

7. 如果在 where 子句中使用参数，也会导致全表扫描。因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时;它必须在编译时进行选择。然而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。

8. 应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。

9. 在使用索引字段作为条件时，如果该索引是【复合索引】，那么必须使用到该索引中的【第一个字段】作为条件时才能保证系统使用该索引，否则该索引将不会被使用。

10. 数据高度重复时，加了索引但是也不起什么作用。

11. 查询取出数据大于20%，将采用全文扫描，不用索引



> #### 表优化

1. 字段长度固定。
2. 字段长度能小就小。
3. 尽量避免使用游标，因为游标的效率较差，如果游标操作的数据超过1万行，那么就应该考虑改写。
4. 尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理。
5. 尽量避免大事务操作，提高系统并发能力。



[MySQL查询优化](https://www.cnblogs.com/phpstudy2015-6/p/6509331.html)



# 面试问题

1. mysql底层引擎数据存储结构，索引实现的存储结构。

2. MySQL 分页查询语句

   `select * from table limit (startPage-1)*limit,limit`

3. MySQL 事务特性和隔离级别

4. 可重复读会出现在什么场景？

   在A事务进行读取的时候，B事务更改了这一条数据。

5. sql  having 的使用场景

   - “Having”是一个过滤声明，是在查询返回结果集以后对查询结果进行的过滤操作，在Having中可以使用聚合函数。
   - “Where” 是一个约束声明，使用Where来约束来之数据库的数据，Where是在结果返回之前起作用的，且Where中不能使用聚合函数。

6. mysql的存储过程

7. sql 语句

8. sql底层些东西 最好了解



- ## MySQL的复制原理以及流程

  基本原理流程，3个线程以及之间的关联；

  1. 主：binlog线程——记录下所有改变了数据库数据的语句，放进master上的binlog中；
  2. 从：io线程——在使用start slave 之后，负责从master上拉取 binlog 内容，放进 自己的relay log中；
  3. 从：sql执行线程——执行relay log中的语句；

- ## innodb引擎的4大特性

  插入缓冲（insert buffer),二次写(double write),自适应哈希索引(ahi),预读(read ahead)



> ## 文件索引和数据库索引为什么使用B+树?

- 文件与数据库都是需要较大的存储，也就是说，它们都不可能全部存储在内存中，故需要存储到磁盘上。而所谓索引，则为了数据的快速定位与查找，那么索引的结构组织要尽量减少查找过程中磁盘I/O的存取次数，因此B+树相比B树更为合适。数据库系统巧妙利用了局部性原理与磁盘预读原理，将一个节点的大小设为等于一个页，这样每个节点只需要一次I/O就可以完全载入
- 而红黑树这种结构，高度明显要深的多，并且由于逻辑上很近的节点(父子)物理上可能很远，无法利用局部性。最重要的是，B+树还有一个最大的好处：方便扫库。B树必须用中序遍历的方法按序扫库，而B+树直接从叶子结点挨个扫一遍就完了，B+树支持range-query非常方便，而B树不支持，这是数据库选用B+树的最主要原因。



> ## 为什么Mysql用B+树做索引而不用B-树

B-树和B+树最重要的一个区别就是B+树只有叶节点存放数据，其余节点用来索引，而B-树是每个索引节点都会有Data域。这就决定了B+树更适合用来存储外部数据，也就是所谓的磁盘数据。

1. 从Mysql的角度来看，B+树是用来充当索引的，一般来说索引非常大，尤其是关系性数据库这种数据量大的索引能达到亿级别，所以为了减少内存的占用，索引也会被存储在磁盘上。
   那么Mysql如何衡量查询效率呢？磁盘IO次数，B-树（B类树）的特定就是每层节点数目非常多，层数很少，目的就是为了就少磁盘IO次数，当查询数据的时候，最好的情况就是很快找到目标索引，然后读取数据，使用B+树就能很好的完成这个目的，但是B-树的每个节点都有data域（指针），这无疑增大了节点大小，增加了磁盘IO次数（磁盘IO一次读出的数据量大小是固定的，单个数据变大，每次读出的就少，IO次数增多），而B+树除了叶子节点其它节点并不存储数据，节点小，磁盘IO次数就少。这是优点之一。
2. B+树所有的Data域在叶子节点，一般来说都会进行一个优化，就是将所有的叶子节点用指针串起来。这样遍历叶子节点就能获得全部数据，这样就能进行区间访问啦。



> ## 为什么MongoDB采用B树索引，而Mysql用B+树做索引

先从数据结构的角度来答。

题主应该知道B树和B+树最重要的一个区别就是B+树只有叶节点存放数据，其余节点用来索引，而B树是每个索引节点都会有Data域。

Mysql用B+的理由见上题。

至于MongoDB为什么使用B-树而不是B+树，可以从它的设计角度来考虑，它并不是传统的关系性数据库，而是以Json格式作为存储的nosql，目的就是高性能，高可用，易扩展。首先它摆脱了关系模型，上面所述的优点2需求就没那么强烈了，其次Mysql由于使用B+树，数据都在叶节点上，每次查询都需要访问到叶节点，而MongoDB使用B-树，所有节点都有Data域，只要找到指定索引就可以进行访问，无疑单次查询平均快于Mysql（但侧面来看Mysql至少平均查询耗时差不多）。





> ## 为什么说B+tree比B 树更适合实际应用中操作系统的文件索引和数据库索引？



- B+tree的磁盘读写代价更低：B+tree的内部结点并没有指向关键字具体信息的指针(红色部分)，因此其内部结点相对B 树更小。如果把所有同一内部结点的关键字存放在同一盘块中，那么盘块所能容纳的关键字数量也越多。一次性读入内存中的需要查找的关键字也就越多，相对来说IO读写次数也就降低了；
- B+tree的查询效率更加稳定：由于内部结点并不是最终指向文件内容的结点，而只是叶子结点中关键字的索引，所以，任何关键字的查找必须走一条从根结点到叶子结点的路。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当；
- **数据库索引采用B+树而不是B树的主要原因：**B+树只要遍历叶子节点就可以实现整棵树的遍历，而且在数据库中基于范围的查询是非常频繁的，而B树只能中序遍历所有节点，效率太低。



> ## char，varchar，text，blob的关系和区别

- **char**:
  - 定长格式的，但是长度范围是0~255. 当你想要储存一个长度不足255的**字符**时，MySQL会用空格来填充剩下的字符。因此在读取数据时，char类型的数据要进行处理，把后面的空格去除。
- **varchar**: 
  - 关于varchar，有的说最大长度是255，也有的说是65535，查阅很多资料后发现是这样的：varchar类型在5.0.3以下的版本中的最大长度限制为255，而在5.0.3及以上的版本中，varchar数据类型的长度支持到了65535，也就是说可以存放65532个字节（注意是字节而不是字符！！！）的数据（起始位和结束位占去了3个字节），也就是说，在5.0.3以下版本中需要使用固定的TEXT或BLOB格式存放的数据可以在高版本中使用可变长的varchar来存放，这样就能有效的减少数据库文件的大小。
  - varchar(n)，如果字符数小于n，则只会占用字符加上1到2字节的空间，加上的几个字节用来存储数据大小。没有空间浪费。
- text:
  - 变长，有字符集的大对象，并根据字符集进行排序和校验，大小写不敏感
  - 与char和varchar不同的是，text不可以有默认值，其最大长度是2的16次方-1。
  - 按照字符数量来占用空间，用2字节记录存储数据大小，这2字节不占用text数据的空间。没有空间浪费。速度慢
- blob
  - 变长，无字符集的二进制大对象，大小写敏感
- double
  - 精度高，有效数字16位
  - double数值类型用于表示双精度浮点数值
- float
  - 精度7位
  - float数值类型用于表示单精度浮点数值



[mysql中char，varchar与text类型的区别和选用](https://blog.csdn.net/geniussnail/article/details/7753256)

[mysql的char，varchar，text类型的区别总结](https://blog.csdn.net/lkforce/article/details/79006838)



[数据库面试问题集锦](https://blog.csdn.net/si444555666777/article/details/82111355)

[mysql面试题的一些记录](https://www.jianshu.com/p/08618d256a97)





> ### oracle与mysql的区别

1. Oracle是大型数据库而Mysql是中小型数据库，Oracle市场占有率达40%，Mysql只有20%左右，同时Mysql是开源的而Oracle价格非常高。

2. Oracle支持大并发，大访问量，是OLTP最好的工具。

3. 安装所用的空间差别也是很大的，Mysql安装完后才152M而Oracle有3G左右，且使用的时候Oracle占用特别大的内存空间和其他机器性能。

4. 锁

   1. mysql以表级锁为主，对资源锁定的粒度很大，如果一个session对一个表加锁时间过长，会让其他session无法更新此表中的数据。虽然InnoDB引擎的表可以用行级锁，但这个行级锁的机制依赖于表的索引，如果表没有索引，或者sql语句没有使用索引，那么仍然使用表级锁。
   2. oracle使用行级锁，对资源锁定的粒度要小很多，只是锁定sql需要的资源，并且加锁是在数据库中的数据行上，不依赖与索引。所以oracle对并发性的支持要好很多。

5. 一致性

   1. oracle:
      oracle支持serializable的隔离级别，可以实现最高级别的读一致性。每个session提交后其他session才能看到提交的更改。oracle通过在undo表空间中构造多版本数据块来实现读一致性，
      每个session查询时，如果对应的数据块发生变化，oracle会在undo表空间中为这个session构造它查询时的旧的数据块。
   2. mysql:
      mysql没有类似oracle的构造多版本数据块的机制，只支持read commited的隔离级别。一个session读取数据时，其他session不能更改数据，但可以在表最后插入数据。
      session更新数据时，要加上排它锁，其他session无法访问数据。

6. 逻辑备份

   1. oracle逻辑备份时不锁定数据，且备份的数据是一致的。
   2. mysql逻辑备份时要锁定数据，才能保证备份的数据是一致的，影响业务正常的dml使用。

7. 热备份

   1. oracle有成熟的热备工具rman，热备时，不影响用户使用数据库。即使备份的数据库不一致，也可以在恢复时通过归档日志和联机重做日志进行一致的回复。
   2. mysql:
      myisam的引擎，用mysql自带的mysqlhostcopy热备时，需要给表加读锁，影响dml操作。
      innodb的引擎，它会备份innodb的表和索引，但是不会备份.frm文件。用ibbackup备份时，会有一个日志文件记录备份期间的数据变化，因此可以不用锁表，不影响其他用户使用数据库。但此工具是收费的。
      innobackup是结合ibbackup使用的一个脚本，他会协助对.frm文件的备份。

8. Oracle也Mysql操作上的区别

   - 主键

     Mysql一般使用自动增长类型，在创建表时只要指定表的主键为auto increment，插入记录时，不需要再指定该记录的主键值，Mysql将自动增长；

     Oracle没有自动增长类型，主 键一般使用序列，插入记录时将序列号的下一个值付给该字段即可；

   - 单引号的处理

     MYSQL里可以用双引号包起字符串；

     ORACLE里只可以用单引号包起字符串。在插入和修改字符串前必须做单引号的替换：把所有出现的一个单引号替换成两个单引号。

   - 翻页的SQL语句的处理

     MYSQL处理翻页的SQL语句比较简单，用LIMIT 开始位置, 记录个数；

     ORACLE处理翻页的SQL语句就比较繁琐了。每个结果集只有一个ROWNUM字段标明它的位置, 并且只能用ROWNUM<100, 不能用ROWNUM>80

   - 长字符串的处理

     长字符串的处理ORACLE也有它特殊的地方。INSERT和UPDATE时最大可操作的字符串长度小于等于4000个单字节, 如果要插入更长的字符串, 请考虑字段用CLOB类型，方法借用ORACLE里自带的DBMS_LOB程序包。插入修改记录前一定要做进行非空和长度判断，不能为空的字段值和超出长度字段值都应该提出警告,返回上次操作。



[mysql与Oracle的区别](https://blog.csdn.net/baidu_37107022/article/details/77043959)











插入一条记录时，聚集索引和非聚集索引是如何修改的

建立索引的标准是什么

SQL 索引的顺序，字段的顺序

MySQL 分页查询语句,mysql分页有什么优化