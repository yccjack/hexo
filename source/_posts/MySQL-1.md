---

title: MySQL-1
categories: 数据库 
tags: [MySQL]
date: 2018-10-11 
cover: https://mysticalyu.gitee.io/pic/img/bada-kim-1.jpg

---

![](https://mysticalyu.gitee.io/pic/img/bada-kim-1.jpg)

 

<!-- more -->



# 逻辑架构

 mysql的逻辑架构分为3层；

![img](/images/storage/838913-fb7f263a0d00afe7.png)

- 第一层：服务层(为客户端服务):为请求做连接处理，授权认证，安全等。
- 第二层：Mysql核心服务层：主要提供，查询解析、分析、优化、缓存以及内置函数，跨存储引擎功能（存储过程、视图、触发器）。
- 第三层：存储引擎层，负责数据的存储和提取。


## 连接管理与安全性

这里有第一层处理，每个客户端的连接都会在服务器进程中拥有一个线程，连接的查询在这个线程中单独进行。



## 优化与执行

此处由第二层，中间层处理，该处的优化器负责创建内部数据结构，然后优化，包括重写查询，决定表的读取顺序，以及选择合适的索引。



# 并发控制

## 读写锁

通常也称为共享锁和排他锁；

- 读锁是共享的，多个客户在同一时间可以同时读取同一个资源，而互不干扰。
- 写锁则是排他的，也就是说一个写锁会阻塞其它的写锁和读锁。

## 锁力度

一种优化的策略，对于不同的锁提供不同的力度，让锁定对象更有选择性。当然加锁的操作也增加系统的开销。包括（获得锁，检查锁是否解除，是否锁）。下面介绍两种最重要的锁力度

- 表锁（table lock）
  顾名思义就是将整张表锁定，Mysql中最基本的锁策略，并且是开销最小的策略，加锁之后，整个表数据受到影响，不利于并发，写锁优先级高于读锁，因此一个写锁请求可能会被插入到读锁的队列前面。服务器也会使用ALTER TABLE之类的语句使用表锁。
- 行级锁（row lock）
  支持高并发，同事带来最大的锁开销。只有存储引擎实现，第二层不会实现。

# 事务

事务是一个独立的工作单元，可以用**START TRANSACTION**语句开始一个事务，然后要么使用**COMMIT**提交，或者使用**ROLLBACK**撤销所有的修改。

事务有四大特性；

- **A(Atomicity-原子性)**：一事务必须被视为一个不可分割的最小工作单元，整个事务的操作要么全部提交成功，要么全部失败会滚。
- **C(Consistency-一致性)**：一事务必须被视为一个不可分割的最小工作单元，整个事务的操作要么全部提交成功，要么全部失败会滚。
- **I(Isolation-隔离性)**：通常来说，一个事务所做的修改在最终提交以前，对其它事务是不可见的。
- **D(Durability-持久性)**：一旦事务提交，则其所做的修改就会永久保存到数据库中。

不是所有的数据库都支持事务，像现在大火的nosql大多都不支持事务，而mysql不同的引擎对事务的支持也不一样，像默认的InnoDB是支持事务的，而Myisam这一的引擎就不支持事务。这个需要根据业务去做相应的选择。

## 隔离级别

数据库提供了四种事务隔离级别, 不同的隔离级别采用不同的锁类开来实现。

| **隔离级别**     | **脏读可能性** | **不可重复读可能性** | **幻读可能性** | **加锁读** |
| ---------------- | -------------- | -------------------- | -------------- | ---------- |
| READ UNCOMMITTED | YES            | YES                  | YES            | NO         |
| READ COMMITED    | NO             | YES                  | YES            | NO         |
| REPEATABLE READ  | NO             | NO                   | YES            | NO         |
| SERIALIZABLE     | NO             | NO                   | NO             | YES        |

**相关概念**

-  `脏读`：事务中的修改，即使未提交，对其他事务也是课件的。事务可以读取未提交的数据。
-  `不可重复读`：在同一个事务中，再次读取数据时，所读取的数据，和第1次读取的数据，不一样了
-  `幻读`：幻读的重点在于新增或者删除，同样的条件, 第1次和第2次读出来的记录数不一样。
   幻读是指当一个事务在读取某个范围内的数据时，另一个事务在这个范围内插入了一行记录并提交，于是当前一个事务再次读取该范围内的数据时，发现多出了一行，即幻行。

```xml
脏读、不可重复读、幻读的级别高低是：  
脏读 < 不可重复读 < 幻读
所以，设置了最高级别的SERIALIZABLE_READ就不用在设置REPEATABLE_READ和READ_COMMITTED了
```

- READ UNCOMMITTED(未提交读)：数据库中几乎不用这种事务。
- READ COMMITED(已提交读)：大多数数据库的默认隔离级别(MySQL不是)。这个级别代表事务开始后，只能读到其他事务提交后的修改，未提交的修改是不可见的。显然这样就解决了脏读的问题。但是还是会遇到不可重复读的问题。
- REPEATABLE READ(可重复读)：Mysql的默认隔离级别，该级别保证了同一个事务中多次读取同样记录的结果是一致的。
- SERIALIZABLE(可序列化读)：最高的隔离级别，通过强制事务串行执行，避免了前面说的幻读问题。该级别会在读取的每一行上都加锁，所以可能导致大量的超时和锁争用问题。

## 死锁

两个活多个事务在同一资源上相互占用，并请求锁定对方占用的资源，从而导致恶性循环的现象。
 为解决这种问题，数据库都是些了各种死锁检测和死锁超时机制。

- 如InnoDB若检测到死锁循环依赖，就立即返回一个错误。
- 当查询时间到锁等待超时的设定后放弃锁清秋。
   InnoDB的处理方式是，将持有最少行级排他锁的事务进行回滚。

## 事务日志

事务日志可以提高事务的效率，使用事务日志，存储引擎在修改表的数据时只要需要修改内存拷贝，再把修改行为记录到硬盘上的事务日志中，而不用每次都将修改的数据持久到磁盘。事务日志持久以后，内存中被修改的数据在后台可以慢慢的刷回到磁盘。目前大多数存储引擎都是这样实现的，我们通常称之为预写式日志。

MySQL中的事务。

MySQL中提供了两种事务型的存储引擎：InnoDB和NDB Cluster。

MySQL默认采用自动提交模式。也就是说，如果不是显式地开始一个事务，则每个查询都被当作一个事务执行提交操作。可以通过设置**AUTOCOMMIT**变量来启用或者禁用自动提交模式。

```sql
SHOW VARIABLES LIKE 'AUTOCOMMIT';SET AUTOCOMMIT = 1;
```

 

MySQL也可以通过执行**SET TRANSACTION ISOLATION LEVEL**命令来设置隔离级别。

也可以只改变当前会话的隔离级别：

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITED;
```

 

不要在事务中混合使用存储引擎，例如InnoDB和MyISAM，在正常提交的情况下不会有什么问题。但如果该事务需要回滚，非事务型的表上的变更就无法撤销。

InnoDB采用的是两阶段锁定协议，锁只有在**COMMIT**或者**ROLLBACK**的时候才会释放，并且所有的锁是在同一时刻被释放。

另外，InnoDB也支持通过特定的语句进行显式锁定，这些语句不属于SQL规范。

```sql
SELECT ... LOCK IN SHARE MODE
SELECT ... FOR UPDATE
```

 

MySQL也支持**LOCK TABLES**和**UNLOCK TABLES**语句，这是在服务层实现的与存储引擎无关，他们有自己的用途，但并不能代替事务处理。

# 多版本并发控制（MVCC）

不同的存储引擎MVCC实现不同，典型的有乐观并发和悲观并发控制。下面结合InnoDB说明下。
 InnoDB的MVCC，通过在每行记录后面保存两个隐藏列实现，这两个列一个保存行的创建时间，一个保存过期（删除时间），当然这里存储的创建时间不是真正的时间，而是系统版本号。

- REPEATABLE READ级别下，MVCC操作如下： 
  - select ：InnoDB只查找版本早于当前事务版本的数据行（即创建版本号《=当前事务版本号），这样保证事务读取的行是早于事务开始前就已经存在的。删除版本号》当前事务版本号，保证事情读取到的行在事务开始前未被删除。
  - insert：插入新行的时候，将事务分配到的版本号赋给创建版本号那个列属性。
  - delete：为删除的每一行保存当前系统版本号为行删除标识，即将该版本号存入删除版本号的那个列属性
  - update：实际上是新插入一条记录，然后将事务分配到的版本号赋给旧记录的删除版本号列以及新记录的创建版本号列。

MVCC只在REPEATABLE READ和READ COMMITTED两个隔离级别下工作。



# MySQL存储引擎

查询表相关信息，命令如下

```sql
mysql> show table status like 'city' \G
*************************** 1. row ***************************
           Name: city //表名
         Engine: InnoDB//引擎名
        Version: 10
     Row_format: Compact//行的格式
           Rows: 600//行数，对于InnoDB，该行是估计值，其他引擎为准确值
 Avg_row_length: 81//平均每行字节数
    Data_length: 49152//表数据大小，byte为单位
Max_data_length: 0//
   Index_length: 16384
      Data_free: 0
 Auto_increment: 601
    Create_time: 2017-10-09 19:59:23
    Update_time: NULL
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options: 
        Comment: 
1 row in set (0.00 sec)
```

修改表的引擎采用如下语句：

```sql
ALTER TABLE city ENGINE = InnoDB;
```

下面简单介绍下相关存储引擎的优缺点。

**InnoDB**

优点：Mysql当前的默认引擎，事务型引擎，用MVCC支持高并发，通过间隙锁在REPEATABLE READ级别下就能防止幻读。支持热备份。
 缺点：非常复杂，性能较一些简单的引擎要差一点儿。空间占用比较多。

 **MyISAM**:

优点：Mysql 5.1之前版本的默认引擎。全文索引、压缩、空间函数(GIS)、并发插入、某些场景下性能很好
 缺点：非事务型、不支持行锁、崩溃后数据不容易修复

** Archive**：

优点：支持高并发插入，解决不可重复读，针对高速插入和压缩做了优化的简单引擎
 缺点：只支持查询和插入操作，非事务型，仅适合日志和数据采集的应用场景

** CSV引擎**：

优点：有效支持CSV格式文件的导入导出。
 缺点：作者没说

** Memory引擎**：

优点：用来快速地访问数据的，比MyISAM快一个数量级。支持Hash索引，因此查询操作非常快。
 缺点：所有数据保存在内存里，重启只留下表结构。只支持表级锁，并发能力低下。不支持BLOB或TEXT类型的列。且行长度固定，容易导致内存浪费。

**NDB集群引擎**：

作者没有细讲，后文会细讲的吧，总之就是支持建立集群。

** XtraDB和PBXT等OLTP类引擎**：

优点：可完全替代InnoDB,或者高度相似。还额外提供了一些性能优化、可测量性、操作灵活性
 缺点：第三方的引擎，社区支持的存储引擎，可能不能保证质量
 其实最常用的是InnoDB和MyISAM。

- 相关参数对比

  | 特性                                                   | InnoDB | MyISAM | MEMORY | ARCHIVE |
  | ------------------------------------------------------ | ------ | ------ | ------ | ------- |
  | 存储限制(Storage limits)                               | 64TB   | No     | YES    | No      |
  | 支持事物(Transactions)                                 | Yes    | No     | No     | No      |
  | 锁机制(Locking granularity)                            | 行锁   | 表锁   | 表锁   | 行锁    |
  | B树索引(B-tree indexes)                                | Yes    | Yes    | Yes    | No      |
  | T树索引(T-tree indexes)                                | No     | No     | No     | No      |
  | 哈希索引(Hash indexes)                                 | Yes    | No     | Yes    | No      |
  | 全文索引(Full-text indexes)                            | Yes    | Yes    | No     | No      |
  | 集群索引(Clustered indexes)                            | Yes    | No     | No     | No      |
  | 数据缓存(Data caches)                                  | Yes    | No     | N/A    | No      |
  | 索引缓存(Index caches)                                 | Yes    | Yes    | N/A    | No      |
  | 数据可压缩(Compressed data)                            | Yes    | Yes    | No     | Yes     |
  | 加密传输(Encrypted data[1])                            | Yes    | Yes    | Yes    | Yes     |
  | 集群数据库支持(Cluster databases support)              | No     | No     | No     | No      |
  | 复制支持(Replication support[2])                       | Yes    | No     | No     | Yes     |
  | 外键支持(Foreign key support)                          | Yes    | No     | No     | No      |
  | 存储空间消耗(Storage Cost)                             | 高     | 低     | N/A    | 非常低  |
  | 内存消耗(Memory Cost)                                  | 高     | 低     | N/A    | 低      |
  | 数据字典更新(Update statistics for data dictionary)    | Yes    | Yes    | Yes    | Yes     |
  | 备份/时间点恢复(backup/point-in-time recovery[3])      | Yes    | Yes    | Yes    | Yes     |
  | 多版本并发控制(Multi-Version Concurrency Control/MVCC) | Yes    | No     | No     | No      |
  | 批量数据写入效率(Bulk insert speed)                    | 慢     | 快     | 快     | 非常快  |
  | 地理信息数据类型(Geospatial datatype support)          | Yes    | Yes    | No     | Yes     |
  | 地理信息索引(Geospatial indexing support[4])           | Yes    | Yes    | No     | Yes     |

  

> 1. 在服务器中实现（通过加密功能）。在其他表空间加密数据在MySQL 5.7或更高版本兼容。
> 2. 在服务中实现的，而不是在存储引擎中实现的。
> 3. 在服务中实现的，而不是在存储引擎中实现的。
> 4. 地理位置索引，InnoDB支持可mysql5.7.5或更高版本兼容