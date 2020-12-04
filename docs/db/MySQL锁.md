# 锁

>   锁机制用于管理对共享资源的并发访问。



## 为什么要学习锁
即使我们不会这些锁知识，我们的程序在一般情况下还是可以跑得好好的。因为这些锁数据库隐式帮我们加了
* 对于 UPDATE、DELETE、INSERT语句，InnoDB会自动给涉及数据集加排他锁（X)
* MyISAM在执行查询语句 SELECT前，会自动给涉及的所有表加读锁，在执行更新操作（ UPDATE、DELETE、INSERT等）前，会自动给涉及的表加写锁，这个过程并不需要用户干预

只会在某些特定的场景下才需要手动加锁，学习数据库锁知识就是为了:
* 能让我们在特定的场景下派得上用场
* 更好把控自己写的程序
* 构建自己的知识库体系！在面试的时候不虚



## 锁分类

### 从锁的粒度角度：

-   **表锁**

-   -   开销小，加锁快；不会出现死锁；锁定力度大，发生锁冲突概率高，并发度最低

-   **行锁**

-   -   开销大，加锁慢；会出现死锁；锁定粒度小，发生锁冲突的概率低，并发度高

不同的存储引擎支持的锁粒度是不一样的：

-   **InnoDB行锁和表锁都支持**！
-   **MyISAM只支持表锁**！

InnoDB只有通过**索引条件**检索数据**才使用行级锁**，否则，InnoDB将使用**表锁**

-   也就是说，**InnoDB的行锁是基于索引的**！

**表锁下又分为两种模式**：

-   表读锁（Table Read Lock）

-   表写锁（Table Write Lock）

-   从下图可以清晰看到，在表读锁和表写锁的环境下：**读读不阻塞，读写阻塞，写写阻塞**！

-   -   读读不阻塞：当前用户在读数据，其他的用户也在读数据，不会加锁
    -   读写阻塞：当前用户在读数据，其他的用户**不能修改当前用户读的数据**，会加锁！
    -   写写阻塞：当前用户在修改数据，其他的用户**不能修改当前用户正在修改的数据**，会加锁！

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib18m2ejoyMtU86WQLnpnxCLEnccx6AelfqK8ic3MAlV6lYBNFKkQ4ACL3TzX44QoQhOiaEXjPn4ShXg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

从上面已经看到了：**读锁和写锁是互斥的，读写操作是串行**。

-   如果某个进程想要获取读锁，**同时**另外一个进程想要获取写锁。在mysql里边，**写锁是优先于读锁的**！
-   写锁和读锁优先级的问题是可以通过参数调节的： `max_write_lock_count`和 `low-priority-updates`

值得注意的是：

>   The LOCAL modifier enables nonconflicting INSERT statements (concurrent inserts) by other sessions to execute while the lock is held. (See Section 8.11.3, “Concurrent Inserts”.) However, READ LOCAL cannot be used if you are going to manipulate the database using processes external to the server while you hold the lock. **For InnoDB tables, READ LOCAL is the same as READ**

-   **MyISAM可以**支持查询和插入操作的**并发**进行。可以通过系统变量 `concurrent_insert`来指定哪种模式，在**MyISAM**中它默认是：如果MyISAM表中没有空洞（即表的中间没有被删除的行），MyISAM允许在一个进程读表的同时，另一个进程从**表尾**插入记录。
-   但是**InnoDB存储引擎是不支持的**！

#### 行锁细讲

上边简单讲解了表锁的相关知识，我们使用Mysql一般是使用InnoDB存储引擎的。InnoDB和MyISAM有两个本质的区别：

-   InnoDB支持行锁
-   InnoDB支持事务

从上面也说了：我们是**很少手动加表锁**的。表锁对我们程序员来说几乎是透明的，即使InnoDB不走索引，加的表锁也是自动的！

我们应该**更加关注行锁的内容**，因为InnoDB一大特性就是支持行锁！

InnoDB实现了以下**两种**类型的行锁。

-   共享锁（S锁）：允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。

-   -   也叫做**读锁**：读锁是**共享**的，多个客户可以**同时读取同一个**资源，但**不允许其他客户修改**。

-   排他锁（X锁)：允许获得排他锁的事务更新数据，阻止其他事务取得相同数据集的共享读锁和排他写锁。

-   -   也叫做**写锁**：写锁是排他的，**写锁会阻塞其他的写锁和读锁**。

看完上面的有没有发现，在一开始所说的：X锁，S锁，读锁，写锁，共享锁，排它锁其实**总共就两个锁**，只不过它们**有多个名字罢了**~~~

>   Intention locks do not block anything except full table requests (for example, LOCK TABLES ... WRITE). The main purpose of intention locks **is to show that someone is locking a row, or going to lock a row in the table**.

另外，**为了允许行锁和表锁共存，实现多粒度锁机制**，InnoDB还有两种内部使用的意向锁（Intention Locks），这两种意向锁都是**表锁**：

-   意向共享锁（IS）：事务打算给数据行加行共享锁，事务在给一个数据行加共享锁前必须先取得该表的IS锁。
-   意向排他锁（IX）：事务打算给数据行加行排他锁，事务在给一个数据行加排他锁前必须先取得该表的IX锁。
-   意向锁也是数据库隐式帮我们做了，**不需要程序员操心**！

##### MVCC和事务的隔离级别

数据库事务有不同的隔离级别，不同的隔离级别对锁的使用是不同的，**锁的应用最终导致不同事务的隔离级别**

MVCC(Multi-Version Concurrency Control)多版本并发控制，可以简单地认为：**MVCC就是行级锁的一个变种(升级版)**。

-   事务的隔离级别就是**通过锁的机制来实现**，只不过**隐藏了加锁细节**

在**表锁中我们读写是阻塞**的，基于提升并发性能的考虑，**MVCC一般读写是不阻塞的**(所以说MVCC很多情况下避免了加锁的操作)

-   MVCC实现的**读写不阻塞**正如其名：**多版本**并发控制--->通过一定机制生成一个数据请求**时间点的一致性数据快照（Snapshot)**，并用这个快照来提供一定级别（**语句级或事务级**）的**一致性读取**。从用户的角度来看，好像是**数据库可以提供同一数据的多个版本**。

快照有**两个级别**：

-   语句级

-   -   针对于 `Readcommitted`隔离级别

-   事务级别

-   -   针对于 `Repeatableread`隔离级别

我们在初学的时候已经知道，事务的隔离级别有**4种**：

-   Read uncommitted

-   -   会出现脏读，不可重复读，幻读

-   Read committed

-   -   会出现不可重复读，幻读

-   Repeatable read

-   -   会出现幻读(但在Mysql实现的Repeatable read配合gap锁不会出现幻读！)

-   Serializable

-   -   串行，避免以上的情况！

------

`Readuncommitted`会出现的现象--->脏读：**一个事务读取到另外一个事务未提交的数据**

-   例子：A向B转账，**A执行了转账语句，但A还没有提交事务，B读取数据，发现自己账户钱变多了**！B跟A说，我已经收到钱了。A回滚事务【rollback】，等B再查看账户的钱时，发现钱并没有多。
-   出现脏读的原因是因为在读的时候**没有加读锁**，导致可以**读取出还没释放锁的记录**。

`Readuncommitted`过程：

-   事务A读取记录(没有加任何的锁)
-   事务B修改记录(此时加了写锁，并且还没有commit-->也就没有释放掉写锁)
-   事务A再次读取记录(此时因为事务A在读取时没有加任何锁，所以可以读取到事务B还没提交的(没释放掉写锁)的记录

------

`Readcommitted`**避免脏读**的做法其实很简单：

-   **在读取的时候生成一个版本号，直到事务其他commit被修改了之后，才会有新的版本号**

`Readcommitted`过程：

-   事务A读取了记录(生成版本号)
-   事务B修改了记录(此时加了写锁)
-   事务A再读取的时候，**是依据最新的版本号来读取的**(当事务B执行commit了之后，会生成一个新的版本号)，如果事务B还没有commit，那事务A读取的还是之前版本号的数据。

但 `Readcommitted`出现的现象--->不可重复读：**一个事务读取到另外一个事务已经提交的数据，也就是说一个事务可以看到其他事务所做的修改**

-   注：**A查询数据库得到数据，B去修改数据库的数据，导致A多次查询数据库的结果都不一样【危害：A每次查询的结果都是受B的影响的，那么A查询出来的信息就没有意思了】**

------

上面也说了， `Readcommitted`是**语句级别**的快照！**每次读取的都是当前最新的版本**！

`Repeatableread`避免不可重复读是**事务级别**的快照！每次读取的都是当前事务的版本，即使被修改了，也只会读取当前事务版本的数据。

呃...如果还是不太清楚，我们来看看InnoDB的MVCC是怎么样的吧(摘抄《高性能MySQL》)

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib18m2ejoyMtU86WQLnpnxCL4glM4CRacSaClNXAZSxypvSxXVce0UehrxYQs8vHOchF5PPxcicgHwA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib18m2ejoyMtU86WQLnpnxCLWGshQ5CCQvRlPL8uJibcbiatUmBAuBiaZFOx3x0GHxCLNib3l1EISibicgDw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

至于虚读(幻读)：**是指在一个事务内读取到了别的事务插入的数据，导致前后读取不一致。**

-   注：**和不可重复读类似，但虚读(幻读)会读到其他事务的插入的数据，导致前后读取不一致**

-   ### MySQL的 `Repeatableread`隔离级别加上GAP间隙锁**已经处理了幻读了**。

### 乐观锁和悲观锁

无论是 `Readcommitted`还是 `Repeatableread`隔离级别，都是为了解决**读写冲突**的问题。

单纯在 `Repeatableread`隔离级别下我们来考虑一个问题：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/2BGWl1qPxib18m2ejoyMtU86WQLnpnxCLrgNYk2QdcSAzzEC9orlovpW120klTtKUIjb1MKVIllpXWunhz3yicaA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

此时，用户李四的操作就丢失掉了：

-   **丢失更新**：一个事务的更新**覆盖了其它事务的更新结果**。

(ps:暂时没有想到比较好的例子来说明更新丢失的问题，虽然上面的例子也是更新丢失，但**一定程度上是可接受的**..不知道有没有人能想到不可接受的更新丢失例子呢...)

解决的方法：

-   使用Serializable隔离级别，事务是串行执行的！
-   乐观锁
-   悲观锁

>   1.  乐观锁是一种思想，具体实现是，表中有一个版本字段，第一次读的时候，获取到这个字段。处理完业务逻辑开始更新的时候，需要再次查看该字段的值是否和第一次的一样。如果一样更新，反之拒绝。之所以叫乐观，因为这个模式没有从数据库加锁，等到更新的时候再判断是否可以更新。
>   2.  悲观锁是数据库层面加锁，都会阻塞去等待锁。



##### 悲观锁

所以，按照上面的例子。我们使用悲观锁的话其实很简单(手动加行锁就行了)：

-   `select*fromxxxxforupdate`

在select 语句后边加了 `forupdate`相当于加了排它锁(写锁)，加了写锁以后，其他的事务就不能对它修改了！需要等待当前事务修改完之后才可以修改.

-   也就是说，如果张三使用 `select...forupdate`，李四就无法对该条记录修改了~

##### 乐观锁

乐观锁不是数据库层面上的锁，是需要自己手动去加的锁。一般我们添加一个版本字段来实现：

具体过程是这样的：

张三 `select*fromtable` --->会查询出记录出来，同时会有一个version字段

![img](https://mmbiz.qpic.cn/mmbiz_jpg/2BGWl1qPxib18m2ejoyMtU86WQLnpnxCLDhVAG3OGC3SbXCQVPD82BZ7SoN4KW6qu6l9NzCQxshaoG6DOekFhhA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

李四 `select*fromtable` --->会查询出记录出来，同时会有一个version字段

![img](https://mmbiz.qpic.cn/mmbiz_jpg/2BGWl1qPxib18m2ejoyMtU86WQLnpnxCLDhVAG3OGC3SbXCQVPD82BZ7SoN4KW6qu6l9NzCQxshaoG6DOekFhhA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

李四对这条记录做修改： `update AsetName=lisi,version=version+1whereID=#{id}andversion=#{version}`，判断之前查询到的version与现在的数据的version进行比较，**同时会更新version字段**

此时数据库记录如下：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/2BGWl1qPxib18m2ejoyMtU86WQLnpnxCLTJHMJjxlicMRr9VZ87Ky7ryB72DZf7pQkiaHExtHiazQ3GSibR7ricjwMOA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

张三也对这条记录修改： `update AsetName=lisi,version=version+1whereID=#{id}andversion=#{version}`，但失败了！因为**当前数据库中的版本跟查询出来的版本不一致**！

![img](https://mmbiz.qpic.cn/mmbiz_jpg/2BGWl1qPxib18m2ejoyMtU86WQLnpnxCLNZP7V7AbwA62Bx6jIfchRavHxm1mmlsUW6yL4yeOfdMjUibBpRNYFKQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 间隙锁GAP

当我们**用范围条件检索数据**而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给**符合范围条件的已有数据记录的索引项加锁**；对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP)”。InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁。

值得注意的是：间隙锁只会在 `Repeatableread`隔离级别下使用~

例子：假如emp表中只有101条记录，其empid的值分别是1,2,...,100,101

```
Select * from  emp where empid > 100 for update;
```

上面是一个范围查询，InnoDB**不仅**会对符合条件的empid值为101的记录加锁，也会对**empid大于101（这些记录并不存在）的“间隙”加锁**。

InnoDB使用间隙锁的目的有两个：

-   **为了防止幻读**(上面也说了， `Repeatableread`隔离级别下再通过GAP锁即可避免了幻读)

-   **满足恢复和复制的需要**

-   -   MySQL的恢复机制要求：**在一个事务未提交前，其他并发事务不能插入满足其锁定条件的任何记录，也就是不允许出现幻读**

### 死锁

并发的问题就少不了死锁，在MySQL中同样会存在死锁的问题。

但一般来说MySQL通过回滚帮我们解决了不少死锁的问题了，但死锁是无法完全避免的，可以通过以下的经验参考，来尽可能少遇到死锁：

-   1）以**固定的顺序**访问表和行。比如对两个job批量更新的情形，简单方法是对id列表先排序，后执行，这样就避免了交叉等待锁的情形；将两个事务的sql顺序调整为一致，也能避免死锁。
-   2）**大事务拆小**。大事务更倾向于死锁，如果业务允许，将大事务拆小。
-   3）在同一个事务中，尽可能做到**一次锁定**所需要的所有资源，减少死锁概率。
-   4）**降低隔离级别**。如果业务允许，将隔离级别调低也是较好的选择，比如将隔离级别从RR调整为RC，可以避免掉很多因为gap锁造成的死锁。
-   5）**为表添加合理的索引**。可以看到如果不走索引将会为表的每一行记录添加上锁，死锁的概率大大增大。

## 总结

表锁其实我们程序员是很少关心它的：

-   在MyISAM存储引擎中，当执行SQL语句的时候是自动加的。
-   在InnoDB存储引擎中，如果没有使用索引，表锁也是自动加的。

现在我们大多数使用MySQL都是使用InnoDB，InnoDB支持行锁：

-   共享锁--读锁--S锁
-   排它锁--写锁--X锁

在默认的情况下， `select`是不加任何行锁的~事务可以通过以下语句显示给记录集加共享锁或排他锁。

-   共享锁（S）： `SELECT*FROM table_name WHERE...LOCK IN SHARE MODE`。
-   排他锁（X)： `SELECT*FROM table_name WHERE...FOR UPDATE`。

InnoDB**基于行锁**还实现了MVCC多版本并发控制，MVCC在隔离级别下的 `Readcommitted`和 `Repeatableread`下工作。MVCC能够实现**读写不阻塞**！

InnoDB实现的 `Repeatableread`隔离级别配合GAP间隙锁已经避免了幻读！

-   乐观锁其实是一种思想，正如其名：认为不会锁定的情况下去更新数据，如果发现不对劲，才不更新(回滚)。在数据库中往往添加一个version字段来实现。
-   悲观锁用的就是数据库的行锁，认为数据库会发生并发冲突，直接上来就把数据锁住，其他事务不能修改，直至提交了当前事务# 锁

>   锁机制用于管理对共享资源的并发访问。



## 为什么要学习锁
即使我们不会这些锁知识，我们的程序在一般情况下还是可以跑得好好的。因为这些锁数据库隐式帮我们加了
* 对于 UPDATE、DELETE、INSERT语句，InnoDB会自动给涉及数据集加排他锁（X)
* MyISAM在执行查询语句 SELECT前，会自动给涉及的所有表加读锁，在执行更新操作（ UPDATE、DELETE、INSERT等）前，会自动给涉及的表加写锁，这个过程并不需要用户干预

只会在某些特定的场景下才需要手动加锁，学习数据库锁知识就是为了:
* 能让我们在特定的场景下派得上用场
* 更好把控自己写的程序
* 构建自己的知识库体系！在面试的时候不虚



## 锁分类

### 从锁的粒度角度：

-   **表锁**

-   -   开销小，加锁快；不会出现死锁；锁定力度大，发生锁冲突概率高，并发度最低

-   **行锁**

-   -   开销大，加锁慢；会出现死锁；锁定粒度小，发生锁冲突的概率低，并发度高

不同的存储引擎支持的锁粒度是不一样的：

-   **InnoDB行锁和表锁都支持**！
-   **MyISAM只支持表锁**！

InnoDB只有通过**索引条件**检索数据**才使用行级锁**，否则，InnoDB将使用**表锁**

-   也就是说，**InnoDB的行锁是基于索引的**！

**表锁下又分为两种模式**：

-   表读锁（Table Read Lock）

-   表写锁（Table Write Lock）

-   从下图可以清晰看到，在表读锁和表写锁的环境下：**读读不阻塞，读写阻塞，写写阻塞**！

-   -   读读不阻塞：当前用户在读数据，其他的用户也在读数据，不会加锁
    -   读写阻塞：当前用户在读数据，其他的用户**不能修改当前用户读的数据**，会加锁！
    -   写写阻塞：当前用户在修改数据，其他的用户**不能修改当前用户正在修改的数据**，会加锁！

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib18m2ejoyMtU86WQLnpnxCLEnccx6AelfqK8ic3MAlV6lYBNFKkQ4ACL3TzX44QoQhOiaEXjPn4ShXg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

从上面已经看到了：**读锁和写锁是互斥的，读写操作是串行**。

-   如果某个进程想要获取读锁，**同时**另外一个进程想要获取写锁。在mysql里边，**写锁是优先于读锁的**！
-   写锁和读锁优先级的问题是可以通过参数调节的： `max_write_lock_count`和 `low-priority-updates`

值得注意的是：

>   The LOCAL modifier enables nonconflicting INSERT statements (concurrent inserts) by other sessions to execute while the lock is held. (See Section 8.11.3, “Concurrent Inserts”.) However, READ LOCAL cannot be used if you are going to manipulate the database using processes external to the server while you hold the lock. **For InnoDB tables, READ LOCAL is the same as READ**

-   **MyISAM可以**支持查询和插入操作的**并发**进行。可以通过系统变量 `concurrent_insert`来指定哪种模式，在**MyISAM**中它默认是：如果MyISAM表中没有空洞（即表的中间没有被删除的行），MyISAM允许在一个进程读表的同时，另一个进程从**表尾**插入记录。
-   但是**InnoDB存储引擎是不支持的**！

#### 行锁细讲

上边简单讲解了表锁的相关知识，我们使用Mysql一般是使用InnoDB存储引擎的。InnoDB和MyISAM有两个本质的区别：

-   InnoDB支持行锁
-   InnoDB支持事务

从上面也说了：我们是**很少手动加表锁**的。表锁对我们程序员来说几乎是透明的，即使InnoDB不走索引，加的表锁也是自动的！

我们应该**更加关注行锁的内容**，因为InnoDB一大特性就是支持行锁！

InnoDB实现了以下**两种**类型的行锁。

-   共享锁（S锁）：允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。

-   -   也叫做**读锁**：读锁是**共享**的，多个客户可以**同时读取同一个**资源，但**不允许其他客户修改**。

-   排他锁（X锁)：允许获得排他锁的事务更新数据，阻止其他事务取得相同数据集的共享读锁和排他写锁。

-   -   也叫做**写锁**：写锁是排他的，**写锁会阻塞其他的写锁和读锁**。

看完上面的有没有发现，在一开始所说的：X锁，S锁，读锁，写锁，共享锁，排它锁其实**总共就两个锁**，只不过它们**有多个名字罢了**~~~

>   Intention locks do not block anything except full table requests (for example, LOCK TABLES ... WRITE). The main purpose of intention locks **is to show that someone is locking a row, or going to lock a row in the table**.

另外，**为了允许行锁和表锁共存，实现多粒度锁机制**，InnoDB还有两种内部使用的意向锁（Intention Locks），这两种意向锁都是**表锁**：

-   意向共享锁（IS）：事务打算给数据行加行共享锁，事务在给一个数据行加共享锁前必须先取得该表的IS锁。
-   意向排他锁（IX）：事务打算给数据行加行排他锁，事务在给一个数据行加排他锁前必须先取得该表的IX锁。
-   意向锁也是数据库隐式帮我们做了，**不需要程序员操心**！

##### MVCC和事务的隔离级别

数据库事务有不同的隔离级别，不同的隔离级别对锁的使用是不同的，**锁的应用最终导致不同事务的隔离级别**

MVCC(Multi-Version Concurrency Control)多版本并发控制，可以简单地认为：**MVCC就是行级锁的一个变种(升级版)**。

-   事务的隔离级别就是**通过锁的机制来实现**，只不过**隐藏了加锁细节**

在**表锁中我们读写是阻塞**的，基于提升并发性能的考虑，**MVCC一般读写是不阻塞的**(所以说MVCC很多情况下避免了加锁的操作)

-   MVCC实现的**读写不阻塞**正如其名：**多版本**并发控制--->通过一定机制生成一个数据请求**时间点的一致性数据快照（Snapshot)**，并用这个快照来提供一定级别（**语句级或事务级**）的**一致性读取**。从用户的角度来看，好像是**数据库可以提供同一数据的多个版本**。

快照有**两个级别**：

-   语句级

-   -   针对于 `Readcommitted`隔离级别

-   事务级别

-   -   针对于 `Repeatableread`隔离级别

我们在初学的时候已经知道，事务的隔离级别有**4种**：

-   Read uncommitted

-   -   会出现脏读，不可重复读，幻读

-   Read committed

-   -   会出现不可重复读，幻读

-   Repeatable read

-   -   会出现幻读(但在Mysql实现的Repeatable read配合gap锁不会出现幻读！)

-   Serializable

-   -   串行，避免以上的情况！

------

`Readuncommitted`会出现的现象--->脏读：**一个事务读取到另外一个事务未提交的数据**

-   例子：A向B转账，**A执行了转账语句，但A还没有提交事务，B读取数据，发现自己账户钱变多了**！B跟A说，我已经收到钱了。A回滚事务【rollback】，等B再查看账户的钱时，发现钱并没有多。
-   出现脏读的原因是因为在读的时候**没有加读锁**，导致可以**读取出还没释放锁的记录**。

`Readuncommitted`过程：

-   事务A读取记录(没有加任何的锁)
-   事务B修改记录(此时加了写锁，并且还没有commit-->也就没有释放掉写锁)
-   事务A再次读取记录(此时因为事务A在读取时没有加任何锁，所以可以读取到事务B还没提交的(没释放掉写锁)的记录

------

`Readcommitted`**避免脏读**的做法其实很简单：

-   **在读取的时候生成一个版本号，直到事务其他commit被修改了之后，才会有新的版本号**

`Readcommitted`过程：

-   事务A读取了记录(生成版本号)
-   事务B修改了记录(此时加了写锁)
-   事务A再读取的时候，**是依据最新的版本号来读取的**(当事务B执行commit了之后，会生成一个新的版本号)，如果事务B还没有commit，那事务A读取的还是之前版本号的数据。

但 `Readcommitted`出现的现象--->不可重复读：**一个事务读取到另外一个事务已经提交的数据，也就是说一个事务可以看到其他事务所做的修改**

-   注：**A查询数据库得到数据，B去修改数据库的数据，导致A多次查询数据库的结果都不一样【危害：A每次查询的结果都是受B的影响的，那么A查询出来的信息就没有意思了】**

------

上面也说了， `Readcommitted`是**语句级别**的快照！**每次读取的都是当前最新的版本**！

`Repeatableread`避免不可重复读是**事务级别**的快照！每次读取的都是当前事务的版本，即使被修改了，也只会读取当前事务版本的数据。

呃...如果还是不太清楚，我们来看看InnoDB的MVCC是怎么样的吧(摘抄《高性能MySQL》)

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib18m2ejoyMtU86WQLnpnxCL4glM4CRacSaClNXAZSxypvSxXVce0UehrxYQs8vHOchF5PPxcicgHwA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib18m2ejoyMtU86WQLnpnxCLWGshQ5CCQvRlPL8uJibcbiatUmBAuBiaZFOx3x0GHxCLNib3l1EISibicgDw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

至于虚读(幻读)：**是指在一个事务内读取到了别的事务插入的数据，导致前后读取不一致。**

-   注：**和不可重复读类似，但虚读(幻读)会读到其他事务的插入的数据，导致前后读取不一致**

-   ### MySQL的 `Repeatableread`隔离级别加上GAP间隙锁**已经处理了幻读了**。

### 乐观锁和悲观锁

无论是 `Readcommitted`还是 `Repeatableread`隔离级别，都是为了解决**读写冲突**的问题。

单纯在 `Repeatableread`隔离级别下我们来考虑一个问题：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/2BGWl1qPxib18m2ejoyMtU86WQLnpnxCLrgNYk2QdcSAzzEC9orlovpW120klTtKUIjb1MKVIllpXWunhz3yicaA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

此时，用户李四的操作就丢失掉了：

-   **丢失更新**：一个事务的更新**覆盖了其它事务的更新结果**。

(ps:暂时没有想到比较好的例子来说明更新丢失的问题，虽然上面的例子也是更新丢失，但**一定程度上是可接受的**..不知道有没有人能想到不可接受的更新丢失例子呢...)

解决的方法：

-   使用Serializable隔离级别，事务是串行执行的！
-   乐观锁
-   悲观锁

>   1.  乐观锁是一种思想，具体实现是，表中有一个版本字段，第一次读的时候，获取到这个字段。处理完业务逻辑开始更新的时候，需要再次查看该字段的值是否和第一次的一样。如果一样更新，反之拒绝。之所以叫乐观，因为这个模式没有从数据库加锁，等到更新的时候再判断是否可以更新。
>   2.  悲观锁是数据库层面加锁，都会阻塞去等待锁。



##### 悲观锁

所以，按照上面的例子。我们使用悲观锁的话其实很简单(手动加行锁就行了)：

-   `select*fromxxxxforupdate`

在select 语句后边加了 `forupdate`相当于加了排它锁(写锁)，加了写锁以后，其他的事务就不能对它修改了！需要等待当前事务修改完之后才可以修改.

-   也就是说，如果张三使用 `select...forupdate`，李四就无法对该条记录修改了~

##### 乐观锁

乐观锁不是数据库层面上的锁，是需要自己手动去加的锁。一般我们添加一个版本字段来实现：

具体过程是这样的：

张三 `select*fromtable` --->会查询出记录出来，同时会有一个version字段

![img](https://mmbiz.qpic.cn/mmbiz_jpg/2BGWl1qPxib18m2ejoyMtU86WQLnpnxCLDhVAG3OGC3SbXCQVPD82BZ7SoN4KW6qu6l9NzCQxshaoG6DOekFhhA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

李四 `select*fromtable` --->会查询出记录出来，同时会有一个version字段

![img](https://mmbiz.qpic.cn/mmbiz_jpg/2BGWl1qPxib18m2ejoyMtU86WQLnpnxCLDhVAG3OGC3SbXCQVPD82BZ7SoN4KW6qu6l9NzCQxshaoG6DOekFhhA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

李四对这条记录做修改： `update AsetName=lisi,version=version+1whereID=#{id}andversion=#{version}`，判断之前查询到的version与现在的数据的version进行比较，**同时会更新version字段**

此时数据库记录如下：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/2BGWl1qPxib18m2ejoyMtU86WQLnpnxCLTJHMJjxlicMRr9VZ87Ky7ryB72DZf7pQkiaHExtHiazQ3GSibR7ricjwMOA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

张三也对这条记录修改： `update AsetName=lisi,version=version+1whereID=#{id}andversion=#{version}`，但失败了！因为**当前数据库中的版本跟查询出来的版本不一致**！

![img](https://mmbiz.qpic.cn/mmbiz_jpg/2BGWl1qPxib18m2ejoyMtU86WQLnpnxCLNZP7V7AbwA62Bx6jIfchRavHxm1mmlsUW6yL4yeOfdMjUibBpRNYFKQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 间隙锁GAP

当我们**用范围条件检索数据**而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给**符合范围条件的已有数据记录的索引项加锁**；对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP)”。InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁。

值得注意的是：间隙锁只会在 `Repeatableread`隔离级别下使用~

例子：假如emp表中只有101条记录，其empid的值分别是1,2,...,100,101

```
Select * from  emp where empid > 100 for update;
```

上面是一个范围查询，InnoDB**不仅**会对符合条件的empid值为101的记录加锁，也会对**empid大于101（这些记录并不存在）的“间隙”加锁**。

InnoDB使用间隙锁的目的有两个：

-   **为了防止幻读**(上面也说了， `Repeatableread`隔离级别下再通过GAP锁即可避免了幻读)

-   **满足恢复和复制的需要**

-   -   MySQL的恢复机制要求：**在一个事务未提交前，其他并发事务不能插入满足其锁定条件的任何记录，也就是不允许出现幻读**

### 死锁

并发的问题就少不了死锁，在MySQL中同样会存在死锁的问题。

但一般来说MySQL通过回滚帮我们解决了不少死锁的问题了，但死锁是无法完全避免的，可以通过以下的经验参考，来尽可能少遇到死锁：

-   1）以**固定的顺序**访问表和行。比如对两个job批量更新的情形，简单方法是对id列表先排序，后执行，这样就避免了交叉等待锁的情形；将两个事务的sql顺序调整为一致，也能避免死锁。
-   2）**大事务拆小**。大事务更倾向于死锁，如果业务允许，将大事务拆小。
-   3）在同一个事务中，尽可能做到**一次锁定**所需要的所有资源，减少死锁概率。
-   4）**降低隔离级别**。如果业务允许，将隔离级别调低也是较好的选择，比如将隔离级别从RR调整为RC，可以避免掉很多因为gap锁造成的死锁。
-   5）**为表添加合理的索引**。可以看到如果不走索引将会为表的每一行记录添加上锁，死锁的概率大大增大。

## 总结

表锁其实我们程序员是很少关心它的：

-   在MyISAM存储引擎中，当执行SQL语句的时候是自动加的。
-   在InnoDB存储引擎中，如果没有使用索引，表锁也是自动加的。

现在我们大多数使用MySQL都是使用InnoDB，InnoDB支持行锁：

-   共享锁--读锁--S锁
-   排它锁--写锁--X锁

在默认的情况下， `select`是不加任何行锁的~事务可以通过以下语句显示给记录集加共享锁或排他锁。

-   共享锁（S）： `SELECT*FROM table_name WHERE...LOCK IN SHARE MODE`。
-   排他锁（X)： `SELECT*FROM table_name WHERE...FOR UPDATE`。

InnoDB**基于行锁**还实现了MVCC多版本并发控制，MVCC在隔离级别下的 `Readcommitted`和 `Repeatableread`下工作。MVCC能够实现**读写不阻塞**！

InnoDB实现的 `Repeatableread`隔离级别配合GAP间隙锁已经避免了幻读！

-   乐观锁其实是一种思想，正如其名：认为不会锁定的情况下去更新数据，如果发现不对劲，才不更新(回滚)。在数据库中往往添加一个version字段来实现。
-   悲观锁用的就是数据库的行锁，认为数据库会发生并发冲突，直接上来就把数据锁住，其他事务不能修改，直至提交了当前事务