### 概念和作用
待续
### 基础
常用支持事务的引擎InnoDB  
开启事务：start transaction  
提交事务：commit  
常见的CURD语句，受autocommit影响，默认为ON，表示每个语句都当成一个事务  
但是，当开启事务后，不受autocommit影响，不会自动提交，commit提交事务，rollback可以回滚事务  
查询autocommit状态，只在当前session有效  
show variables like 'autocommit';
### ACID
原子性：不可再分的事务，要么提交，要么回滚，其通过Innodb引擎的redo log和undo log实现  
持久性：通过redo log记录各个事务，避免MySQL挂掉  
一致性：主键存在且唯一，列的类型，大小，长度符合要求，外键及用户自定义一致性等  
隔离性：各个事务之间不能互相干扰  
### 隔离性
分为写操作对写操作的影响和写操作对读操作的影响，前者通过锁机制实现，后者通过MVCC多版本控制实现  
Innodb支持行锁和表锁，Myisam仅支持表锁  
锁分为读锁（lock in share mode），也叫共享锁，写锁（for update），也叫排他锁  
读锁的作用：其他事务也可以获得读锁，但获取不到写锁，即阻止其他事务修改数据  
写锁的作用：获得写锁的事务可以修改数据，其他事务可以查询，但是获取不到读锁，更获取不到写锁  
**只有条件是索引的查询才使用行锁，一般条件只能使用表锁**  
**查询条件所有字段都有索引，并不代表查询计划就使用了索引**  
间隙锁：没有符合条件的记录时，也加的一种锁，此外还有意向排他锁和意向共享锁，这些时MySQL自动处理的  

脏读：读取到未提交的数据  
不可重复读：读取到已提交的数据，但是两次读取数据不一样  
幻读：读取到已提交数据，但两次读取的行数不一样  
事务隔离级别：  
- 读未提交，READ UNCOMMITTED，上述三种问题都可能出现  
- 读已提交，READ COMMITTED，可能出现不可重复读和幻读  
- 可重复读，REPEATABLE READ，可能出现幻读  
- 可串行化，SERIALIZABLE，上述三种问题都可避免，但性能特别差  

默认隔离级别是可重复读，修改隔离级别：  
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;  
查询默认隔离级别：select @@tx_isolation,@@global.tx_isolation;  
