### mysql-acid

事务原子性Atomic，一个事务是一系列sql语句，这些sql要么全都执行，要么都不执行。实现原理：每次对表/临时表的插入，删除，更新操作都会写日志（撤销日志/undo log），记录相反的操作。

事务持久性durable，更新，删除，插入一条记录，这个操作不会立即被同步到磁盘去，因为磁盘IO很慢，这个操作会被首先同步到内存缓冲区中，然后定时将缓冲区的内容刷盘。但是问题来了，如果mysql程序中途由于某些原因退出了，那么内存里的数据就没了（mysql服务器进程的数据）。所以，每次更新，删除，插入操作先写日志，再写内存缓冲。

事务隔离性isolation，两个事务的执行，不能互相干扰。

事务一致性，这是目标，上述三个性质是手段。



### mysql-脏读

举个例子，事务A修改了张三的年龄，但是还没commit，然后事务B读取了张三的年龄。



### mysql-不可重复读

举个例子，事务A读取了张三的年龄为25，然后事务B突然插进来修改了张三的年龄为20并提交了，最后事务A又读取张三的年龄发现是20，和第一次不一样。



### mysql-幻读

微软官网

A *phantom* is a row that matches the search criteria but is not initially seen. For example, suppose transaction 1 reads a set of rows that satisfy some search criteria. Transaction 2 generates a new row (through either an update or an insert) that matches the search criteria for transaction 1. If transaction 1 reexecutes the statement that reads the rows, it gets a different set of rows.

就比方说，事务A从一张员工表里筛选所有的男性，然后事务B插入了一条男性员工的记录，最后事务A再次筛选所有的男性时，发现多了一个人。



### mysql-隔离级别

可参考https://docs.microsoft.com/en-us/sql/odbc/reference/develop-app/transaction-isolation-levels?view=sql-server-ver15

读未提交-最低的等级-连脏读都无法避免

读已提交-可避免脏读

可重复读

可串行化



### msyql-mvcc

可以达到读已提交，可重复读。

这是一个大的话题，需要参考《Msql是怎样运行的》第24章，有详细说明。



### mysql-间隙锁-gap lock

给一条记录加锁，不允许其他事务往这条记录前边的间隙插入新记录。防止插入幻影记录。



### mysql-正经记录锁

参考《Msql是怎样运行的》第25章

### 数据库范式
1nf：表中的每一列都是不可再分的。\
2nf：表中的非主键列，必须严格依赖于主键，而不能只依赖于主键的一部分。(主键可能有多列)\
3nf：在2NF基础上，任何非主属性不依赖于其它非主属性（在2NF基础上消除传递依赖）

### 索引

mysql支持b+树索引，哈希索引。

只有memory引擎显示支持哈希索引。

 

### 哈希索引-缺点

1.无法用于排序

2.不支持部分索引列匹配查找，因为哈希索引是用索引列的全部内容来计算哈希值的。就比如说，我拿列A和列B键建立索引，然后我想查询列A，就用不上。

3.哈希索引只包含哈希值和行指针，所以不能避免读取行。

4.只支持等值查询

5.哈希冲突，维护索引的代价很高。就比如说我在某个哈希冲突很多的列上建立索引



### 聚簇索引

聚簇索引是一种数据存储格式，可以存储主键索引，也可以存储二级索引（非主键）。

主键索引的叶子节点保存了索引值和一整行的数据。

二级索引的叶子节点只保存了索引值和主键的值。



### 覆盖索引-回表查询

举个例子

```mysql
CREATE TABLE `tb2`
( 
    `id` int(11) NOT NULL DEFAULT '0', 
    `name` varchar(100) NOT NULL, 
    `nickname` varchar(100) NOT NULL, 
     PRIMARY KEY (`id`), KEY `index_name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8

INSERT INTO tb2 ( id, name, nickname )VALUES (1, 'chenyifei','cyf');
INSERT INTO tb2 ( id, name, nickname )VALUES (2, 'liujiaye','ljy');
INSERT INTO tb2 ( id, name, nickname )VALUES (3, 'chengang','ca');
INSERT INTO tb2 ( id, name, nickname )VALUES (4, 'lianchenglong','lcl');

explain select name from tb2 where name = 'chenyifei' 
-- 这条语句的执行结果的Extra会显示Using index，意思就是使用了覆盖索引

explain select name from tb2 where name = 'chenyifei' and nickname = 'cyf'
-- 这条语句的执行结果虽然type显示ref，代表使用了索引，但是Extra没有显示Using index，代表没有使用覆盖索引。

SET profiling = 1;
select name from tb2 where name = 'chenyifei';
select name from tb2 where name = 'chenyifei' and nickname = 'cyf';
SHOW PROFILES;
-- 测时间，差距很大。
```
### binlog

记录sql语句

主库把binlog同步给从库。

如果误删了一张表，那么可以通过binlog里的sql语句进行恢复（mysql自带一些工具可以解析binlog）。



### redolog

这里面记录的都是对还没刷入磁盘里的页的修改，如果对某个页的修改刷入了磁盘，那么就删除redolog的一部分内容，从尾巴删。

redolog是一个循环的缓冲区。

如果宕机，我们可以通过redolog来恢复数据。

但是，binlog没办法应对这个情况，比如说有两条插入语句，第一条修改刷入磁盘了，第二条没有，那么我们打开binlog看到的是两条插入语句，我们也不知道第二条插入语句到底有没有刷入磁盘。

