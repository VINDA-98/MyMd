# 05Mysql总结

[TOC]



## 1、什么是存储过程

存储过程就像是编程语言中的函数一样，封装了我们的代码（PLSQL,T-SQL）



```java
-------------创建名为GetUserAccount的存储过程----------------
create Procedure GetUserAccount
as
select * from UserAccount
go

-------------执行上面的存储过程----------------
exec GetUserAccount
```



存储过程的优点：

- **能够将代码封装起来**
- **保存在数据库之中**
- **让编程语言进行调用**
- **存储过程是一个预编译的代码块，执行效率比较高**
- **一个存储过程替代大量T_SQL语句 ，可以降低网络通信量，提高通信速率**

存储过程的缺点：

- **每个数据库的存储过程语法几乎都不一样，十分难以维护（不通用）**
- **业务逻辑放在数据库上，难以迭代**



## 2、三大范式

​        我们现在需要建立一个描述学校教务的数据库，该数据库涉及的对象包括学生的学号（Sno）、所在系（Sdept）、系主任姓名（Mname）、课程号（Cno）和成绩（Grade），假设我们使用单一的关系模式 Student 来表示，那么根据现实世界已知的信息，会描述成以下这个样子：

[![img](https://upload-images.jianshu.io/upload_images/7896890-344feccbb3cb1fad.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)](https://upload-images.jianshu.io/upload_images/7896890-344feccbb3cb1fad.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

但是，这个关系模式存在以下问题：

**（1） 数据冗余**
比如，每一个系的系主任姓名重复出现，重复次数与该系所有学生的所有课程成绩出现次数相同，这将浪费大量的存储空间。
**（2）更新异常（update anomalies）**
由于数据冗余，当更新数据库中的数据时，系统要付出很大的代价来维护数据库的完整性，否则会面临数据不一致的危险。比如，某系更换系主任后，必须修改与该系学生有关的每一个元组。
**（3）插入异常（insertion anomalies）**
如果一个系刚成立，尚无学生，则无法把这个系及其系主任的信息存入数据库。
**（4）删除异常（deletion anomalies）**
如果某个系的学生全部毕业了，则在删除该系学生信息的同时，这个系及其系主任的信息也丢失了。

- **总结：** 所以，我们在设计数据库的时候，就需要满足一定的规范要求，而满足不同程度要求的就是不同的范式。

> - **第一范式：** 列不可分

1NF（第一范式）是对属性具有**原子性**的要求，不可再分，例如：

[![img](https://upload-images.jianshu.io/upload_images/7896890-7a54e7d5d8fbf659.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)](https://upload-images.jianshu.io/upload_images/7896890-7a54e7d5d8fbf659.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

如果认为最后一列还可以再分成出生年，出生月，出生日，则它就不满足第一范式的要求。

> - **第二范式：** 消除非主属性对码的部分函数依赖

2NF（第二范式）是对记录有**唯一性**的要求，即实体的唯一性，不存在部分依赖，每一列与主键都相关，例如：

[![img](https://upload-images.jianshu.io/upload_images/7896890-a10fdbaf10a7e6b8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)](https://upload-images.jianshu.io/upload_images/7896890-a10fdbaf10a7e6b8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

该表明显说明了两个事物：学生信息和课程信息；正常的依赖应该是：学分依赖课程号，姓名依赖学号，但这里存在非主键字段对码的部分依赖，即与主键不相关，不满足第二范式的要求。

**可能存在的问题：**

- **数据冗余**：每条记录都含有相同信息；
- **删除异常**：删除所有学生成绩，就把课程信息全删除了；
- **插入异常**：学生未选课，无法记录进数据库；
- **更新异常**：调整课程学分，所有行都调整。

**正确的做法：**

[![img](https://upload-images.jianshu.io/upload_images/7896890-fba36ca283ffd5fc.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)](https://upload-images.jianshu.io/upload_images/7896890-fba36ca283ffd5fc.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

> - **第三范式：** 消除非主属性对码的传递函数依赖

3NF（第三范式）对字段有**冗余性**的要求，任何字段不能由其他字段派生出来，它要求字段没有冗余，即不存在依赖传递，例如：

[![img](https://upload-images.jianshu.io/upload_images/7896890-8d7548eb839bc8db.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)](https://upload-images.jianshu.io/upload_images/7896890-8d7548eb839bc8db.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

很明显，学院电话是一个冗余字段，因为存在依赖传递：（学号）→（学生）→（学院）→（学院电话）

**可能会存在的问题：**

- **数据冗余**：有重复值；
- **更新异常**：有重复的冗余信息，修改时需要同时修改多条记录，否则会出现数据不一致的情况 。

**正确的做法：**

[![img](https://upload-images.jianshu.io/upload_images/7896890-83c0288ea9150976.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)](https://upload-images.jianshu.io/upload_images/7896890-83c0288ea9150976.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)



## 3、数据库索引

​        索引是对数据库表中一个或多个列的值进行排序的数据结构，以协助快速查询、更新数据库表中数据。索引其实就是加快检索表中数据的方法。	

​		MySQL官方对索引的定义为：索引(Index)是帮助MySQL高效获取数据的数据结构。我们可以简单理解为：**快速查找排好序的一种数据结构**

​		使用B+树的原因：

​		查找速度快、效率高，在查找的过程中，每次都能抛弃掉一部分节点，减少遍历个数。

### 	  什么是B+树？

​		答：为了解决数据量大的时候存储在外存储器时候，查找效率低的问题，B+树的特点：

1. 中间元素不存数据，只是当索引用，所有数据都保存在叶子结点中。
2. 所有的中间节点在子节点中==要么是最大的元素要么是最小的元素== 。
3. 叶子结点包含所有的数据，和指向这些元素的指针，而且叶子结点的元素形成了==自小向大==这样子的链表。

![image-20201011111712349](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201011111712349.png)

B+树查找：

​	单元素：从根节点到叶子节点，中间有这个元素也得往下查，记住中间节点也只是索引。比如要查3

![image-20201011112558962](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201011112558962.png)

​	范围查找：直接从链表查找，查询3-8

​	![image-20201011112628050](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201011112628050.png)





红黑树：

![image-20201011112236903](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201011112236903.png)





### 索引的分类

唯一索引：唯一索引不允许两行具有相同的索引值

主键索引：为表定义一个主键自动创建主键索引，主键索引是唯一索引的特殊类型。主键索引要求主键中每个值都是唯一的，并且不能为空

聚集索引：表中各行的物理顺序与键值的逻辑（索引）顺序相同，每个表只能一个

非聚集索引：非聚集索引指定表的逻辑顺序，数据存储在一个位置，索引存储在另一个位置，索引中包含指向数据存储位置的指针。可以有多个，小于249个。

**（1）优点：**

- **大大加快数据的检索速度**，这也是创建索引的最主要的原因；
- 加速表和表之间的连接；
- 在使用分组和排序子句进行数据检索时，同样可以显著减少查询中分组和排序的时间；
- 通过创建唯一性索引，可以保证数据库表中每一行数据的唯一性；

**（2）缺点：**

- 时间方面：创建索引和维护索引要耗费时间，具体地，当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，这样就降低了数据的维护速度；
- 空间方面：索引需要占物理空间。



###   什么样的字段适合创建索引？

- 经常作查询选择的字段

- 经常作表连接的字段

- 经常出现在order by, group by, distinct 后面的字段

  

- ### 创建索引时需要注意什么？

- **非空字段**：应该指定列为NOT NULL，除非你想存储NULL。在mysql中，含有空值的列很难进行查询优化，因为它们使得索引、索引的统计信息以及比较运算更加复杂。你应该用0、一个特殊的值或者一个空串代替空值；
- **取值离散大的字段**：（变量各个取值之间的差异程度）的列放到联合索引的前面，可以通过count()函数查看字段的差异值，返回值越大说明字段的唯一值越多字段的离散程度高；
- **索引字段越小越好**：数据库的数据存储以页为单位一页存储的数据越多一次IO操作获取的数据越大效率越高。





## 4、事务（重点）

**事务简单来说：一个 Session 中所进行所有的操作，要么同时成功，要么同时失败**；作为单个逻辑工作单元执行的一系列操作，满足四大特性：

1. 原子性（Atomicity）：事务作为一个整体被执行 ，要么全部执行，要么全部不执行
2. 一致性（Consistency）：保证数据库状态从一个一致状态转变为另一个一致状态
3. 隔离性（Isolation）：多个事务并发执行时，一个事务的执行不应影响其他事务的执行
4. 持久性（Durability）：一个事务一旦提交，对数据库的修改应该永久保存

**事务的并发问题有哪几种？**

1. 丢失更新：一个事务的更新覆盖了另一个事务的更新；
2. 脏读：一个事务读取了另一个事务未提交的数据；
3. 不可重复读：不可重复读的重点是修改，同样条件下两次读取结果不同，也就是说，被读取的数据可以被其它事务修改；
4. 幻读：幻读的重点在于新增或者删除，同样条件下两次读出来的记录数不一样。

------

**事务的隔离级别有哪几种？**

**MySQL默认的隔离级别是可重复读（REPEATABLE READ）**

隔离级别决定了一个session中的事务可能对另一个session中的事务的影响。ANSI标准定义了4个隔离级别，MySQL的InnoDB都支持，分别是：

1. 读未提交（READ UNCOMMITTED）：最低级别的隔离，通常又称为dirty read，它允许一个事务读取另一个事务还没 commit 的数据，这样可能会提高性能，但是会导致脏读问题；
2. 读已提交（READ COMMITTED）：在一个事务中只允许对其它事务已经 commit 的记录可见，该隔离级别不能避免不可重复读问题；
3. 可重复读（REPEATABLE READ）：在一个事务开始后，其他事务对数据库的修改在本事务中不可见，直到本事务 commit 或 rollback。但是，其他事务的 insert/delete 操作对该事务是可见的，也就是说，该隔离级别并不能避免幻读问题。在一个事务中重复 select 的结果一样，除非本事务中 update 数据库。
4. 序列化（SERIALIZABLE）：最高级别的隔离，只允许事务串行执行。



**mysql的事务支持**

mysql的事务支持不是绑定在mysql服务器本身，而是与存储引擎相关：

​	MyISAM：不支持事务，用于只读程序提高性能；

​	InnoDB：支持ACID事务、行级锁、并发；

​	Beakeley DB ： 支持事务



## 5、drop,delete与truncate的区别？

​	drop table: 属于DDL 数据库定义语言，不可回滚，不带where，表内容和结构删除，速度快

​	truncate table ：属于DDL 数据库定义语言，不可回滚，不带where，表内容删除，删除速度快

​	delete from 属于DML  数据库操纵语言，可以回滚，可以带where，表内指定内容删除，数据量较大时候删除速度慢



## 6、数据库的乐观锁和悲观锁

​	数据库管理系统（DBMS）中的并发控制的任务是确保在多个事务同时存取数据库中同一数据时不破坏事务的隔离性和统一性一级数据库的统一性。

​	乐观并发控制（乐观锁）和悲观并发控制（悲观锁）是并发控制主要采用的技术手段。

####    悲观锁

​	悲观锁是一种利用数据库内部机制提供的锁的方式，也就是对更新的数据加锁，这样在并发期间==一旦有一个事务持有了数据库记录的锁，其他的线程将不能再对数据进行更新==了，这就是悲观锁的实现方式。

**MySQL InnoDB中使用悲观锁：**

要使用悲观锁，我们必须关闭mysql数据库的自动提交属性，因为MySQL默认使用autocommit模式，也就是说，当你执行一个更新操作后，MySQL会立刻将结果进行提交。 set autocommit=0;



```java
//0.开始事务
begin;/begin work;/start transaction; (三者选一就可以)
//1.查询出商品信息
select status from t_goods where id=1 for update;
//2.根据商品信息生成订单
insert into t_orders (id,goods_id) values (null,1);
//3.修改商品status为2
update t_goods set status=2;
//4.提交事务
commit;/commit wor
```



**优点与不足：**

悲观并发控制实际上是“先取锁再访问”的保守策略，为数据处理的安全提供了保证。但是在效率方面，处理加锁的机制会让数据库产生额外的开销，还有增加产生死锁的机会；另外，在只读型事务处理中由于不会产生冲突，也没必要使用锁，这样做只能增加系统负载；还有会降低了并行性，一个事务如果锁定了某行数据，其他事务就必须等待该事务处理完才可以处理那行数



### 乐观锁

​	假设不会发生并发冲突，只在提交操作时检查是否违反数据完整性。

​	乐观锁是一种不会阻塞其他线程并发的控制，它不会使用数据库的锁进行实现，它的设计里面由于不阻塞其他线程，所以并不会引起线程频繁挂起和恢复，这样便能够提高并发能力，所以也有人把它称为非阻塞锁。一般的实现乐观锁的方式就是记录数据版本。

数据版本,为数据增加的一个版本标识。当读取数据时，将版本标识的值一同读出，数据每更新一次，同时对版本标识进行更新。当我们提交更新的时候，判断数据库表对应记录的当前版本信息与第一次取出来的版本标识进行比对，如果数据库表当前版本号与第一次取出来的版本标识值相等，则予以更新，否则认为是过期数据。

实现数据版本有两种方式，第一种是使用版本号，第二种是使用时间戳。

**使用版本号实现乐观锁：**

使用版本号时，可以在数据初始化时指定一个版本号，每次对数据的更新操作都对版本号执行+1操作。并判断当前版本号是不是该数据的最新的版本号。



```
1.查询出商品信息
select (status,status,version) from t_goods where id=#{id}
2.根据商品信息生成订单
3.修改商品status为2
update t_goods 
set status=2,version=version+1
where id=#{id} and version=#{version};
```

**优点与不足：**

乐观并发控制相信事务之间的数据竞争(data race)的概率是比较小的，因此尽可能直接做下去，直到提交的时候才去锁定，所以不会产生任何锁和死锁。但如果直接简单这么做，还是有可能会遇到不可预期的结果，例如两个事务都读取了数据库的某一行，经过修改以后写回数据库，这时就遇到了问题。



## 7、sql约束

not null ： 用于控制字段得内容一定不能为空

unique：控件字段内容不能重复，允许有多个Unique约束

primary key ：主键，也是用于控件内容不能重复，一个表只能允许出现一个

foreign key ：外键，用于预防破坏表之间的连接动作，也能防止非法数据插入外键列，因为他必须是它指向的那个表中的值之一。

check 用于控制字段的值范围



## 8、sql优化

答：

![image-20201011162826275](https://gitee.com/Vinda_Boy/myphoto/raw/master/img/image-20201011162826275.png)

### （1）sql语句优化

​	**1.优化insert语句：一次插入多个值**

​	INSERT INTO table_name (column1,column2,column3,...)
​	VALUES (value1,value2,value3,...)，(value1,value2,value3,...)；

​	**2.避免在where子句中使用！=或者<>操作符，避免对字段进行null判断，否则将导致引擎放弃使用索引而进行全表扫描**

​	**3.优化嵌套查询：子查询可以被更有效率的连接（join）替代；**

​	**4.很多时候用exists替代in是较好选择**

  ```sql
SELECT *
FROM class_a
WHERE id IN ( SELECT id FROM class_b);

SELECT *
FROM class_a A
WHERE EXISTS  ( 
    SELECT * 
    FROM class_b B
    WHERE A.id = B.id
);

	如果连接列id 上有索引，那么查询CLASS_B时，无需查询实际表，仅需要查索引就可以了。
	使用exists ，那么只有查到一行数据满足条件就会终止查询，不会产生临时表。
	使用in查询时，数据库首先会执行子查询，然后将结果保存在临时表中，然后扫描整个临时表，很多情况下非常耗费资源。

  ```

**⒍选择最有效率的表名顺序：数据库的解析器按照从右到左的顺序处理FROM子句中的表名，FROM子句中写在最后的表将被最先处理**

在FROM子句中包含多个表的情况下：

- 如果三个表是完全无关系的话，将记录和列名最少的表，写在最后，然后依次类推
- 也就是说：选择记录条数最少的表放在最后

如果有3个以上的表连接查询：

- 如果三个表是有关系的话，将引用最多的表，放在最后，然后依次类推。
- 也就是说：被其他表所引用的表放在最后

**⒎用IN代替OR：**



```
select * from emp where sal = 1500 or sal = 3000 or sal = 800;
select * from emp where sal in (1500,3000,800);
```

**⒏SELECT子句中避免使用\*号：**

我们最开始接触 SQL 的时候，“`*`” 号是可以获取表中全部的字段数据的，**但是它要通过查询数据字典完成，这意味着将消耗更多的时间**，而且使用 “`*`” 号写出来的 SQL 语句也不够直观。

### （2）索引优化

建议在经常作查询选择的字段、经常作表连接的字段以及经常出现在 order by、group by、distinct 后面的字段中建立索引。但必须注意以下几种可能会引起索引失效的情形：

- 以 “%(表示任意0个或多个字符)” 开头的 LIKE 语句，模糊匹配；
- OR语句前后没有同时使用索引；
- 数据类型出现隐式转化（如varchar不加单引号的话可能会自动转换为int型）；
- 对于多列索引，必须满足最左匹配原则(eg,多列索引col1、col2和col3，则 索引生效的情形包括col1或col1，col2或col1，col2，col3)。



### （3）数据表结构的优化

**① 选择合适数据类型：**

- 使用较小的数据类型解决问题；
- 使用简单的数据类型(mysql处理int要比varchar容易)；
- 尽可能的使用not null 定义字段；
- 尽量避免使用text类型，非用不可时最好考虑分表；

**② 表的范式的优化：**

一般情况下，表的设计应该遵循三大范式。

**③ 表的垂直拆分：**

把含有多个列的表拆分成多个表，解决表宽度问题，具体包括以下几种拆分手段：

- 把不常用的字段单独放在同一个表中；
- 把大字段独立放入一个表中；
- 把经常使用的字段放在一起；

这样做的好处是非常明显的，具体包括：拆分后业务清晰，拆分规则明确、系统之间整合或扩展容易、数据维护简单

**④ 表的水平拆分：**

表的水平拆分用于解决数据表中数据过大的问题，水平拆分每一个表的结构都是完全一致的。一般地，将数据平分到N张表中的常用方法包括以下两种：

- 对ID进行hash运算，如果要拆分成5个表，mod(id,5)取出0~4个值；
- 针对不同的hashID将数据存入不同的表中；

表的水平拆分会带来一些问题和挑战，包括跨分区表的数据查询、统计及后台报表的操作等问题，但也带来了一些切实的好处：

- 表分割后可以降低在查询时需要读的数据和索引的页数，同时也降低了索引的层数，提高查询速度；
- 表中的数据本来就有独立性，例如表中分别记录各个地区的数据或不同时期的数据，特别是有些数据常用，而另外一些数据不常用。
- 需要把数据存放到多个数据库中，提高系统的总体可用性(分库，鸡蛋不能放在同一个篮子里)。

### （4）系统配置的优化和硬件



- 操作系统配置的优化：增加TCP支持的队列数
- mysql配置文件优化：Innodb缓存池设置(innodb_buffer_pool_size，推荐总内存的75%)和缓存池的个数（innodb_buffer_pool_instances）



### （5）硬件的优化

- CPU：核心数多并且主频高的
- 内存：增大内存
- 磁盘配置和选择：磁盘性能



### （6）我的回答

​	 关于数据库优化可以从一下几个方面表述：

（一）sql语句优化

​	1、避免在where语句中使用!= 、<= 、>=，避免使用null判断，这样会导致搜索引擎进行全表扫描

​	2、使用insert语句，尽量使用一次性插入，避免多条语句插入。

​	3、一般情况下使用exists替换in,先查询主查询的表，根据主查询的表执行 exists ( select stuid from course c where c.stuid = s.stuid )，根据where返回true，false删除改行。（内表比较小的时候，in的速度较快。）

```sql
 select *form students  where exists ( select stuid from course c where c.stuid = s.stuid )
```

​	4、尽量避免使用使用*查询，指定查询字段



（二）索引优化

​	1、为搜索字段简历索引

```sql
alter table 'users' add index ('username');
select count(*) from 'user' where username like '%da' ;
```

​    2、避免使用"%"开头的like查询语句



（三）数据库表结构优化

​	1、永远为每张表设置一个ID

​	我们应该为数据库里的每张表都设置一个 ID 做为其主键，而且最好的是一 个 INT 型的(推荐使用 UNSIGNED)，并设置上自动增加的 AUTO_INCREMENT 标 志。

​	2、某些知道固定长度的字段，推荐使用ENUM类型，例如性别、民族、状态等，而不是用varchar，enum显示也是为字符串的。

​	3、“Empty”和“NULL”有多大的区别(如果是 INT，那就 是 0 和 NULL)。那么就不要使用 NULL。 在 Oracle 里，NULL 和 Empty 的字符串是一样的。

​	4、选择合适的存储引擎

​	MyISAM适合于一些大型的查询应用，对于大量的写操作并不是很友好，有时候我们的一个update操作都能引起整张表的锁，但是对于select count(*)这种计算是非常快的。

​	InnoDb是一个比较复杂的存储引擎，对于小的应用，它会比MyISAM还慢，但是它支持“行锁”，对于写操作的时候，效率会更好一些，还支持一些高级应用，比如：事务.

​	

