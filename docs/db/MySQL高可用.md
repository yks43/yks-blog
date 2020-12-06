# 是什么
我们在考虑MySQL数据库的高可用的架构时，主要要考虑如下几方面：
* 如果数据库发生了宕机或者意外中断等故障，能尽快恢复数据库的可用性，尽可能的减少停机时间，保证业务不会因为数据库的故障而中断。
* 用作备份、只读副本等功能的非主节点的数据应该和主节点的数据实时或者最终保持一致。
* 当业务发生数据库切换时，切换前后的数据库内容应当一致，不会因为数据缺失或者数据不一致而影响业务。

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
