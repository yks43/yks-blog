> 索引其实是一种数据结构，能够帮助我们快速的检索数据库中的数据

## 数据结构：
常见的MySQL主要有两种结构：Hash索引和B+ Tree索引，我们使用的是InnoDB引擎，默认的是B+树

### Hash

>   通过哈希字段值隐射到不同位置

特点：

>   **可以快速的精确查询，但是不支持范围查询**。

适用场景：

>   等值查询的场景，就只有KV（Key，Value）的情况

InnoDB 存储引擎有一个特殊的功能叫“自适应哈希索引”，当某个索引值被使用的非常频繁时，会在 B+Tree 索引之上再创建一个哈希索引，这样就让 B+Tree 索引具有哈希索引的一些优点，比如快速的哈希查找。

#### 有序的数据结构？

在等值查询的和范围查询的时候都很好，适合静态数据（去年的支付宝账单/去年购物记录），如果我们新增、删除、修改数据的时候就会改变他的结构，比如你新增一个，那在你新增的位置后面所有的节点都会后移，成本很高



### B+树

最开始的Hash不支持范围查询，二叉树树高很高，只有B树跟B+有的一比。

B树一个节点可以存储多个元素，相对于完全平衡二叉树整体的树高降低了，磁盘IO效率提高了。

而B+树是B树的升级版，只是把非叶子节点冗余一下，这么做的好处是**为了提高范围查找的效率**。（有指针指向下一个节点的叶子节点。）

#### 二叉树？

二叉树是有序的，所以是支持范围查询的。它的时间复杂度是O(log(N))，为了维持这个时间复杂度，更新的时间复杂度也得是O(log(N))，那就得保持这棵树是完全平衡二叉树了。

缺点：

>   如果数据多了，树高会很高，查询的成本就会随着树高的增加而增加。

#### B树？

同样的元素，B树的表示要比完全平衡二叉树要“矮”，原因在于B树中的一个节点可以存储多个元素。



#### B+树节点元素个数？

B+树中一个节点为一页或页的倍数最为合适

因为如果一个节点的大小小于1页，那么读取这个节点的时候其实也会读出1页，造成资源的浪费。

如果一个节点的大小大于1页，比如1.2页，那么读取这个节点的时候会读出2页，也会造成资源的浪费。

所以为了不造成浪费，所以最后把一个节点的大小控制在1页、2页、3页、4页等倍数页大小最为合适。

#### 各种树

**AVL 树**

平衡二叉树，一般是用平衡因子差值决定并通过旋转来实现，左右子树树高差不超过1，那么和红黑树比较它是严格的平衡二叉树，平衡条件非常严格（树高差只有1），只要插入或删除不满足上面的条件就要通过旋转来保持平衡。由于旋转是非常耗费时间的。所以 AVL 树适用于插入/删除次数比较少，但查找多的场景。

**红黑树**

通过对从根节点到叶子节点路径上各个节点的颜色进行约束，确保没有一条路径会比其他路径长2倍，因而是近似平衡的。所以相对于严格要求平衡的AVL树来说，它的旋转保持平衡次数较少。适合，查找少，插入/删除次数多的场景。（现在部分场景使用跳表来替换红黑树，可搜索“为啥 redis 使用跳表(skiplist)而不是使用 red-black？”）

**B/B+ 树**

多路查找树，出度高，磁盘IO低，一般用于数据库系统中。

##### B + 树与红黑树的比较

红黑树等平衡树也可以用来实现索引，但是文件系统及数据库系统普遍采用 B+ Tree 作为索引结构，主要有以下两个原因：

（一）磁盘 IO 次数

B+ 树一个节点可以存储多个元素，相对于红黑树的树高更低，磁盘 IO 次数更少。

（二）磁盘预读特性

为了减少磁盘 I/O 操作，磁盘往往不是严格按需读取，而是每次都会预读。预读过程中，磁盘进行顺序读取，顺序读取不需要进行磁盘寻道。每次会读取页的整数倍。

操作系统一般将内存和磁盘分割成固定大小的块，每一块称为一页，内存与磁盘以页为单位交换数据。数据库系统将索引的一个节点的大小设置为页的大小，使得一次 I/O 就能完全载入一个节点。



## 索引分类（逻辑）
### 聚簇索引：
主键索引，索引B+ Tree的叶子节点存储了整行数据

InnoDB 的表必定是有一个主键索引（聚簇索引）的，即使不指定某个字段为主键，表结构中也会一个row_id字段来充当聚簇索引

聚簇索引的叶子节点存储结构（primary key id）

| 页4   |       |       |
| ----- | ----- | ----- |
| 1     | 2     | 3     |
| name1 | name2 | name3 |
| pwd1  | pwd2  | pwd3  |




### 二级索引（非聚簇索引）：
非主键索引

叶子节点存储结构（name）：

| 页6   |       |       |
| ----- | ----- | ----- |
| name1 | name2 | name3 |
| 1     | 2     | 3     |

存储内容：

二级索引叶子节点存储的是索引字段和主键字段。

#### 回表

>   当使用二级索引无法得到所需的字段时，先拿到id的信息，然后到聚簇索引中根据id查找

使用覆盖索引

选择查询字段，少用select *

### 覆盖索引

```sql
select * from table where name = 'yks43'
```



select * ，查询所有的，我们如果只查询ID那，其实在Name字段的索引上就已经有了，那就不需要回表了。

覆盖索引可以减少树的搜索次数，提升性能

### 联合索引

多字段索引

创建联合索引时，根据业务需求，where字句中使用最频繁的一列放在最左边，因为MySQL索引查询会遵循最左前缀匹配的原则，即最左优先

#### 最左匹配原则

-   索引可以简单如一个列 (a)，也可以复杂如多个列 (a,b,c,d)，即联合索引。
-   如果是联合索引，那么key也由多个列组成，同时，索引只能用于查找key是否**存在（相等）**，遇到范围查询 (>、<、between、like左匹配)等就**不能进一步匹配**了，后续退化为线性查找。
-   因此，**列的排列顺序决定了可命中索引的列数**。

*   使用like模糊查询时，前缀模糊匹配会导致联合索引无法使用，后缀模糊匹配则可以使用

*   使用or时，有时会走索引，有时不会走索引，优化器会分析走了几次索引，有没有回表，并与全表扫描的效率对比，然后进行选择

#### 索引下推

```sql
select * from user where name like '喻%' and size=22 and age = 20;
```

这个语句在搜索索引树的时候，只能用 “喻”，找到第一个满足条件的记录ID1，当然，这还不错，总比全表扫描要好。然后判断其他条件是否满足，在MySQL 5.6之前，只能从ID1开始一个个回表，到主键索引上找出数据行，再对比字段值。而MySQL 5.6 引入的索引下推优化（index condition pushdown)， 可以在索引遍历过程中，对索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数。



##### 配置
索引下推优化是默认开启的。可以通过下面的脚本控制开关
```sql
SET optimizer_switch = 'index_condition_pushdown=off'; 
SET optimizer_switch = 'index_condition_pushdown=on';
```

##### 思考
索引下推优化技术其实就是充分利用了索引中的数据，尽量在查询出整行数据之前过滤掉无效的数据。

由于需要存储引擎将索引中的数据与条件进行判断，所以这个技术是基于存储引擎的，只有特定引擎可以使用。并且判断条件需要是在存储引擎这个层面可以进行的操作才可以，比如调用存储过程的条件就不可以，因为存储引擎没有调用存储过程的能力。



### 全文索引

MyISAM 存储引擎支持全文索引，用于查找文本中的关键词，而不是直接比较是否相等。

查找条件使用 MATCH AGAINST，而不是普通的 WHERE。

全文索引使用倒排索引实现，它记录着关键词到其所在文档的映射。

InnoDB 存储引擎在 MySQL 5.6.4 版本中也开始支持全文索引。

### 空间数据索引

MyISAM 存储引擎支持空间数据索引（R-Tree），可以用于地理数据存储。空间数据索引会从所有维度来索引数据，可以有效地使用任意维度来进行组合查询。

必须使用 GIS 相关的函数来维护数据。