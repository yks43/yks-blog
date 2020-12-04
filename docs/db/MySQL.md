# 基础


# 存储引擎
MySQL架构图：
![](../images/存储引擎架构图.jpg)
## 连接器
我们要进行查询，第一步就是先去链接数据库，那这个时候就是连接器跟我们对接。

他负责跟客户端建立链接、获取权限、维持和管理连接。

链接的时候会经过TCP握手，然后身份验证，然后我们输入用户名密码就好了。

验证ok后，我们就连上了这个MySQL服务了，但是这个时候我们处于空闲状态。
### 空闲连接
show processlist：Command列显示为Sleep的这一行，就表示现在系统里面有一个空闲连接。
> 数据库的客户端太久没响应，连接器就会自动断开了，这个时间参数是wait_timeout控制住的，默认时长为8小时。断开后重连的时候会报错，如果你想再继续操作，你就需要重连了。

解决方案：
* 重新连接
* 使用长连接

长连接的问题：
* 使用长连接之后，内存会飙得很快，MySQL在执行过程中临时使用的内存是管理在连接对象里面的。只有在连接断开的时候才能得到释放，那如果一直使用长连接，那就会导致OOM（Out Of Memory），会导致MySQL重启，在JVM里面就会导致频繁的Full GC。

解决方案：
* 定期断开长连接，使用一段时间后，或者程序里面判断执行过一个占用内存比较大的查询后就断开连接，需要的时候重连就好了。
* 执行比较大的一个查询后，执行mysql_reset_connection可以重新初始化连接资源。这个过程相比上面一种会好点，不需要重连，但是会初始化连接的状态。

## 查询缓存
MySQL拿到一个查询请求后，会先到查询缓存看看，之前是不是执行过这条语句。

同一条语句在MySQL执行两次，第一次和后面的时间是不一样的，后者明显快一些，这就是因为缓存的存在。

跟Redis一样，只要是你之前执行过的语句，都会在内存里面用key-value形式存储着。

查询的时候就会拿着语句先去缓存中查询，如果能够命中就返回缓存的value，如果不命中就执行后面的阶段。

### 缓存利大于弊
缓存的失效很容易，只要对表有任何的更新，这个表的所有查询缓存就会全部被清空，就会出现缓存还没使用，就直接被清空了，或者积累了很多缓存准备用来着，但是一个更新打回原形。

这就导致查询的命中率低的可怕，只有那种只查询不更新的表适用缓存，但是这样的表往往很少存在，一般都是什么配置表之类的。

### 不用缓存的操作
显示调用，把query_cache_type设置成为DEMAND，这样SQL默认不适用缓存，想用缓存就用SQL_CACHE。

看sql执行时间，但是可能是有缓存的，一般我们就在sql前面使用SQL_NO_CACHE就可以知道真正的查询时间了。
```sql
select SQL_NO_CACHE * from A
```

## 分析器
在缓存没有命中的情况下，就开始执行语句了，检查SQL语句是否有问题，先做词法分析（语句有这么多单词、空格，MySQL就需要识别每个字符串所代表的是什么，是关键字，还是表名，还是列名等等。），然后语法分析（根据词法分析的结果，语法分析会判断你sql的对错，错了会提醒你的，并且会提示你哪里错了。）

## 优化器
分析无误后就进入优化器，优化的东西主要是：确认使用哪一个索引，使用你的主键索引，联合索引还是什么索引更好。；对执行顺序优化，条件那么多，先查哪个表，还是先关联，会出现很多方案，最后由优化器决定选用哪种方案。

## 执行器
优化后就执行了，第一步做权限的判断；执行的时候，就一行一行的去判断是否满足条件，有索引的执行起来可能就好点，一行行的判断就像是接口都提前在引擎定义好了，所以比较快。

## InnoDB
### InnoDB内存架构
InnoDB内存架构图：
![](../images/InnoDB内存架构.jpg)
两大块：
* InnoDB In-Memory Structures（内存）
* InnoDB On-Disk Structures（磁盘）

#### 内存
##### Buffer Pool
> The buffer pool is an area in main memory where InnoDB caches table and index data as it is accessed.

MySQL 不会直接去修改磁盘的数据，因为这样做太慢了，MySQL 会先改内存，然后记录 redo log，等有空了再刷磁盘，如果内存里没有数据，就去磁盘 load。

这些数据存放的地方，就是 Buffer Pool。

平时开发时，会用 redis 来做缓存，缓解数据库压力，其实 MySQL 自己也做了一层类似缓存的东西。

MySQL 是以「页」（page）为单位从磁盘读取数据的，Buffer Pool 里的数据也是如此，实际上，Buffer Pool 是a linked list of pages，一个以页为元素的链表。

为什么是链表？因为和缓存一样，它也需要一套淘汰算法来管理数据。

Buffer Pool 采用基于 LRU（least recently used） 的算法来管理内存

##### Change Buffer
如果内存里没有对应「页」的数据，MySQL 就会去把数据从磁盘里 load 出来，如果每次需要的「页」都不同，或者不是相邻的「页」，那么每次 MySQL 都要去 load，这样就很慢了。

于是如果 MySQL 发现你要修改的页，不在内存里，就把你要对页的修改，先记到一个叫 Change Buffer 的地方，同时记录 redo log，然后再慢慢把数据 load 到内存，load 过来后，再把 Change Buffer 里记录的修改，应用到内存（Buffer Pool）中，这个动作叫做 merge；而把内存数据刷到磁盘的动作，叫 purge：

merge：Change Buffer -> Buffer Pool
purge：Buffer Pool -> Disk



InnoDB 是 MySQL 默认的事务型存储引擎，只要在需要它不支持的特性时，才考虑使用其他存储引擎。

InnoDB 采用 MVCC 来支持高并发，并且实现了四个标准隔离级别(未提交读、提交读、可重复读、可串行化)。其默认级别时可重复读（REPEATABLE READ），在可重复读级别下，通过 MVCC + Next-Key Locking 防止幻读。

主索引时聚簇索引，在索引中保存了数据，从而避免直接读取磁盘，因此对主键查询有很高的性能。

InnoDB 内部做了很多优化，包括从磁盘读取数据时采用的可预测性读，能够自动在内存中创建 hash 索引以加速读操作的自适应哈希索引，以及能够加速插入操作的插入缓冲区等。

InnoDB 支持真正的在线热备份，MySQL 其他的存储引擎不支持在线热备份，要获取一致性视图需要停止对所有表的写入，而在读写混合的场景中，停止写入可能也意味着停止读取。

## MyISAM

设计简单，数据以紧密格式存储。对于只读数据，或者表比较小、可以容忍修复操作，则依然可以使用它。

提供了大量的特性，包括压缩表、空间数据索引等。

不支持事务。

不支持行级锁，只能对整张表加锁，读取时会对需要读到的所有表加共享锁，写入时则对表加排它锁。但在表有读取操作的同时，也可以往表中插入新的记录，这被称为并发插入（CONCURRENT INSERT）。

可以手工或者自动执行检查和修复操作，但是和事务恢复以及崩溃恢复不同，可能导致一些数据丢失，而且修复操作是非常慢的。

如果指定了 DELAY_KEY_WRITE 选项，在每次修改执行完成时，不会立即将修改的索引数据写入磁盘，而是会写到内存中的键缓冲区，只有在清理键缓冲区或者关闭表的时候才会将对应的索引块写入磁盘。这种方式可以极大的提升写入性能，但是在数据库或者主机崩溃时会造成索引损坏，需要执行修复操作。

## InnoDB 和 MyISAM 的比较

*   事务：InnoDB 是事务型的，可以使用 Commit 和 Rollback 语句。
*   并发：MyISAM 只支持表级锁，而 InnoDB 还支持行级锁。
*   外键：InnoDB 支持外键。
*   备份：InnoDB 支持在线热备份。
*   崩溃恢复：MyISAM 崩溃后发生损坏的概率比 InnoDB 高很多，而且恢复的速度也更慢。
*   其它特性：MyISAM 支持压缩表和空间数据索引。



## 基础知识

##### 页

储存结构：页（记录都存在页里面）：

![](../images/页.jpg)

* 各个数据页可以组成一个双向链表
* 而每个数据页中的记录又可以组成一个单向链表
    * 每个数据页都会为存储在它里边儿的记录生成一个页目录，在通过主键查找某条记录的时候可以在页目录中使用二分法快速定位到对应的槽，然后再遍历该槽对应分组中的记录即可快速找到指定的记录
    * 以其他列(非主键)作为搜索条件：只能从最小记录开始依次遍历单链表中的每条记录。

所以说，如果我们写 `select * from user where username='yks43'`这样没有进行任何优化的sql语句，默认会这样做：
* 定位到记录所在的页
    * 需要遍历双向链表，找到所在的页
* 从所在的页内中查找相应的记录
    * 由于不是根据主键查询，只能遍历所在页的单链表了

很明显，在数据量很大的情况下这样查找会很慢！



# explain

使用 explain 分析 select 查询语句

>   explain 用来分析 SELECT 查询语句，可以通过分析 Explain 结果来优化查询语句。

### select_type

常用的有 SIMPLE 简单查询，UNION 联合查询，SUBQUERY 子查询等。

### table

要查询的表

### possible_keys

>   The possible indexes to choose

可选择的索引

### key

>   The index actually chosen

实际使用的索引

### rows

>   Estimate of rows to be examined

扫描的行数

### type

索引查询类型，经常用到的索引查询类型：

*   const：使用主键或者唯一索引进行查询的时候只有一行匹配 

*   ref：使用非唯一索引 

*   range：使用主键、单个字段的辅助索引、多个字段的辅助索引的最后一个字段进行范围查询 

*   index：和all的区别是扫描的是索引树 

*   all：扫描全表：

#### system

触发条件：表只有一行，这是一个 const type 的特殊情况

#### const

触发条件：在使用主键或者唯一索引进行查询的时候只有一行匹配。

#### eq_ref

触发条件：在进行联接查询的，使用主键或者唯一索引并且只匹配到一行记录的时候

#### ref

触发条件：使用非唯一索引

#### range

触发条件：只有在使用主键、单个字段的辅助索引、多个字段的辅助索引的最后一个字段进行范围查询才是 range

#### index

触发条件：

只扫描索引树

1.  查询的字段是索引的一部分，覆盖索引。
2.  使用主键进行排序

#### all

触发条件：全表扫描，不走索引

## 优化数据访问

### 减少请求的数据量

*   只返回必要的列：最好不要使用 SELECT * 语句。
*   只返回必要的行：使用 LIMIT 语句来限制返回的数据。
*   缓存重复查询的数据：使用缓存可以避免在数据库中进行查询，特别在要查询的数据经常被重复查询时，缓存带来的查询性能提升将会是非常明显的。

### 减少服务器端扫描的行数

最有效的方式是使用索引来覆盖查询。

# 事务

>   事务是指满足 ACID 特性的一组操作，可以通过 Commit 提交一个事务，也可以使用 Rollback 进行回滚。

### ACID

事务最基本的莫过于 ACID 四个特性了，这四个特性分别是：

-   Atomicity：原子性
-   Consistency：一致性
-   Isolation：隔离性
-   Durability：持久性

**原子性**

事务被视为不可分割的最小单元，事务的所有操作要么全部成功，要么全部失败回滚。

**一致性**

数据库在事务执行前后都保持一致性状态，在一致性状态下，所有事务对一个数据的读取结果都是相同的。

**隔离性**

一个事务所做的修改在最终提交以前，对其他事务是不可见的。

**持久性**

一旦事务提交，则其所做的修改将会永远保存到数据库中。即使系统发生崩溃，事务执行的结果也不能丢。

#### ACID之间的关系

事务的 ACID 特性概念很简单，但不好理解，主要是因为这几个特性不是一种平级关系：

-   只有满足一致性，事务的结果才是正确的。
-   在无并发的情况下，事务串行执行，隔离性一定能够满足。此时只要能满足原子性，就一定能满足一致性。在并发的情况下，多个事务并行执行，事务不仅要满足原子性，还需要满足隔离性，才能满足一致性。
-   事务满足持久化是为了能应对数据库崩溃的情况。

### 隔离级别

**未提交读（READ UNCOMMITTED）**

事务中的修改，即使没有提交，对其他事务也是可见的。

**提交读（READ COMMITTED）**

一个事务只能读取已经提交的事务所做的修改。换句话说，一个事务所做的修改在提交之前对其他事务是不可见的。

**可重复读（REPEATABLE READ）**

保证在同一个事务中多次读取同样数据的结果是一样的。

**可串行化（SERIALIZABLE）**

强制事务串行执行。

需要加锁实现，而其它隔离级别通常不需要。

| 隔离级别 | 脏读 | 不可重复读 | 幻影读 |
| :------: | :--: | :--------: | :----: |
| 未提交读 |  √   |     √      |   √    |
|  提交读  |  ×   |     √      |   √    |
| 可重复读 |  ×   |     ×      |   √    |
| 可串行化 |  ×   |     ×      |   ×    |




# 分库分表

## 水平切分

水平切分又称为 Sharding，它是将同一个表中的记录拆分到多个结构相同的表中。

当一个表的数据不断增多时，Sharding 是必然的选择，它可以将数据分布到集群的不同节点上，从而缓存单个数据库的压力。

### Sharding 策略

*   哈希取模：hash(key)%N
*   范围：可以是 ID 范围也可以是时间范围
*   映射表：使用单独的一个数据库来存储映射关系

### Sharding 存在的问题

**事务问题**

使用分布式事务来解决，比如 XA 接口

**连接**

可以将原来的连接分解成多个单表查询，然后在用户程序中进行连接。

**唯一性**

-   使用全局唯一 ID （GUID）
-   为每个分片指定一个 ID 范围
-   分布式 ID 生成器（如 Twitter 的 Snowflake 算法）

## 垂直切分

垂直切分是将一张表按列分成多个表，通常是按照列的关系密集程度进行切分，也可以利用垂直气氛将经常被使用的列喝不经常被使用的列切分到不同的表中。

在数据库的层面使用垂直切分将按数据库中表的密集程度部署到不通的库中，例如将原来电商数据部署库垂直切分称商品数据库、用户数据库等。

# 主从复制
> 将主数据库的DDL和DML操作通过二进制日志传到从数据库上，然后对这些日志进行重新执行，从而使从数据库和主数据库的数据保持一致

## 原理
* MySql主库在事务提交时会把数据变更作为事件记录在二进制日志Binlog中；
* 主库推送二进制日志文件Binlog中的事件到从库的中继日志Relay Log中，之后从库根据中继日志重做数据变更操作，通过逻辑复制来达到主库和从库的数据一致性；
* MySql通过三个线程来完成主从库间的数据复制，其中Binlog Dump线程跑在主库上，I/O线程和SQL线程跑着从库上；
* 当在从库上启动复制时，首先创建I/O线程连接主库，主库随后创建Binlog Dump线程读取数据库事件并发送给I/O线程，I/O线程获取到事件数据后更新到从库的中继日志Relay Log中去，之后从库上的SQL线程读取中继日志Relay Log中更新的数据库事件并应用

## Docker搭建测试
### 主实例搭建
运行MySQL主实例：
```shell
docker run -p 3307:3306 --name mysql-master \
-v /mydata/mysql-master/log:/var/log/mysql \
-v /mydata/mysql-master/data:/var/lib/mysql \
-v /mydata/mysql-master/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:5.7
```

在mysql的配置文件夹/mydata/mysql-master/conf中创建一个配置文件my.cnf：
```shell
touch my.cnf
```
修改配置文件my.cnf，配置信息如下：
```cnf
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=101
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql
## 开启二进制日志功能
log-bin=mall-mysql-bin
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
```
修改完配置后重启实例，并进入到master容器中，连接到客户端：
```shell
docker restart mysql-master
docker exec -it mysql-master /bin/bash
mysql -uroot -proot
```
创建数据同步用户
```shell
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
```

### 从实例搭建
运行MySQL从实例
```shell
docker run -p 3308:3306 --name mysql-slave \
-v /mydata/mysql-slave/log:/var/log/mysql \
-v /mydata/mysql-slave/data:/var/lib/mysql \
-v /mydata/mysql-slave/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:5.7
```
在mysql的配置文件夹/mydata/mysql-slave/conf中创建一个配置文件my.cnf，并修改：
```shell
touch my.cnf
```
```cnf
mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=102
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql
## 开启二进制日志功能，以备Slave作为其它数据库实例的Master时使用
log-bin=mall-mysql-slave1-bin
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M
## 设置使用的二进制日志格式（mixed,statement,row）
binlog_format=mixed
## 二进制日志过期清理时间。默认值为0，表示不自动清理。
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
## relay_log配置中继日志
relay_log=mall-mysql-relay-bin
## log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1
## slave设置为只读（具有super权限的用户除外）
read_only=1
```
修改完配置后重启实例：
```shell
docker restart mysql-slave
```

### 主从数据库进行连接
连接到主数据库的mysql客户端，查看主数据库状态：
```shell
mysql> show master status;
```
进入mysql-slave容器中，连接到客户端，在数据库中配置主从复制
```shell
change master to master_host='192.168.6.132', master_user='slave', master_password='123456', master_port=3307, master_log_file='mall-mysql-bin.000001', master_log_pos=617, master_connect_retry=30;
```
查看主从同步状态
```
show slave status \G;
```
开启主从同步
```
start slave;
```
参数说明
```
master_host：主数据库的IP地址；
master_port：主数据库的运行端口；
master_user：在主数据库创建的用于同步数据的用户账号；
master_password：在主数据库创建的用于同步数据的用户密码；
master_log_file：指定从数据库要复制数据的日志文件，通过查看主数据的状态，获取File参数；
master_log_pos：指定从数据库从哪个位置开始复制数据，通过查看主数据的状态，获取Position参数；
master_connect_retry：连接失败重试的时间间隔，单位为秒。
```

### 测试
* 在主实例中创建一个数据库yks
* 在从实例中查看数据库，发现也有一个yks数据库，可以判断主从复制已经搭建成功。

# 读写分离

>   主服务器处理写操作以及实时性要求比较高的读操作，而从服务器处理读操作。以提高系统性能

读写分离能提高性能的原因在于：

-   主从服务器负责各自的读和写，极大程度缓解了锁的争用；
-   从服务器可以使用 MyISAM，提升查询性能以及节约系统开销；
-   增加冗余，提高可用性。

读写分离常用代理方式来实现，代理服务器接收应用层传来的读写请求，然后决定转发到哪个服务器。

![](../images/读写分离.jpg)

