# MyBatis介绍

## 是什么
> 是一个基于Java的持久层框架

## 为什么使用
无论是Mybatis、Hibernate都是ORM的一种实现框架，都是对JDBC的一种封装；

常见的持久层框架：
* Hibernate
* jdbc
* SpringDAO
* MyBatis
## 解决了什么问题

Hibernate是一个比较老旧的框架，用起来十分舒服…啥sql代码都不用写...但是，它也是有的缺点：处理复杂业务时，灵活度差, 复杂的HQL难写难理解，例如多表查询的HQL语句

而JDBC很容易理解，就那么几个固定的步骤，就是开发起来太麻烦了，因为什么都要我们自己干...

而SpringDAO其实就是JDBC的一层封装，就类似于dbutils一样，没有特别出彩的地方...

我们可以认为，Mybatis就是jdbc和Hibernate之间的一个平衡...现在业界都是用这个框架，必须学

## 工作流程
1. 通过Reader对象读取Mybatis配置文件
2. 通过SqlSessionFactoryBuilder对象创建SqlSessionFactory对象
3. 获取当前线程的SQLSession
4. 事务默认开启
5. 通过SQLSession读取映射文件中的操作编号，从而读取SQL语句
6. 提交事务
7. 关闭资源

# 快速上手
## 导入开发包
引用MyBatis相关jar包
## 准备测试工作
MySQL创建表
创建实体类
## 创建MyBatis配置文件
配置数据库相关信息
## 编写工具类测试是否获取到连接
编写常用工具类操作数据库连接
## 创建实体与映射关系文件
创建mapper.xml文件配置实体类和表的关系
## 编写DAO
dao层编写代码
# 映射文件

### 占位符

*   \#{}解析传递进来的参数数据
*   ${}对传递进来的参数**原样**拼接在SQL中
    *   表名、order by的排序字段作为变量时，使用${}。

### 主键生成策略

UUID

主键返回

select LAST_INSERT_ID()

### 字段属性映射

##### resultMap

```xml
    <resultMap id="userListResultMap" type="user" >
         <!-- 列名
         id_,username_,birthday_
         id：要映射结果集的唯 一标识 ，称为主键
         column：结果集的列名
         property：type指定的哪个属性中
          -->
          <id column="id_" property="id"/>
          <!-- result就是普通列的映射配置 -->
          <result column="username_" property="username"/>
          <result column="birthday_" property="birthday"/>

     </resultMap>
```

当我们数据表的字段和JavaBean的属性名称不是相同时，我们就需要使用resultMap

###### **association**

-   作用：

-   -   **将关联查询信息映射到一个pojo类中。**

-   场合：

-   -   为了方便获取关联信息可以使用association将关联订单映射为pojo，比如：查询订单及关联用户信息。

###### collection

-   作用：

-   -   **将关联查询信息映射到一个list集合中。**

-   场合：

-   -   为了方便获取关联信息可以使用collection将关联信息映射到list集合中，比如：查询用户权限范围模块和功能，可使用collection将模块和功能列表映射到list中。

### 别名

_或大小写自动转换

### Mapper加载

```xml
   <mappers>
        <!-- 通过resource引用mapper的映射文件 -->
        <mapper resource="sqlmap/User.xml" />
        <!-- <mapper resource="mapper/UserMapper.xml" /> -->
        <!-- 通过class引用mapper接口 
        class：配置mapper接口全限定名
        要求：需要mapper.xml和mapper.java同名并且在一个目录 中
        -->
        <!-- <mapper class="cn.itcast.mybatis.mapper.UserMapper"/> -->
        <!-- 批量mapper配置 
        通过package进行自动扫描包下边的mapper接口，
        要求：需要mapper.xml和mapper.java同名并且在一个目录 中

        -->
        <package name="cn.itcast.mybatis.mapper"/>
    </mappers>
```

### 延迟加载

在进行数据查询时，**为了提高数据库查询性能，尽量使用单表查询**，因为单表查询比多表关联查询速度要快。

如果查询单表就可以满足需求，一开始先查询单表，当需要关联信息时，再关联查询，**当需要关联信息再查询这个叫延迟加载**。

**在Mybatis中延迟加载就是在resultMap中配置具体的延迟加载**..

![这里写图片描述](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3W3VSvKKR7ialreweHt7HP0LrG2icuJUO2O6LC23QSaB1LZW56iaRqdzKHUUX0yYsyobfcH6La8iag5A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)这里写图片描述

在Mybatis的文件中配置全局延迟加载

```xml
  <!-- 全局配置参数 -->
    <settings>
        <!-- 延迟加载总开关 -->
        <setting name="lazyLoadingEnabled" value="true" />  
        <!-- 设置按需加载 -->
        <setting name="aggressiveLazyLoading" value="false" />
    </settings>
```



# 执行器

Mybatis有三种基本的Executor执行器，**SimpleExecutor、ReuseExecutor、BatchExecutor**。

-   SimpleExecutor：每执行一次update或select，就开启一个Statement对象，**用完立刻关闭Statement对象**。

-   ReuseExecutor：执行update或select，以sql作为key查找Statement对象，存在就使用，不存在就创建，用完后，不关闭Statement对象，而是放置于Map

    内，供下一次使用。简言之，**就是重复使用Statement对象**。

-   BatchExecutor：执行update（没有select，JDBC批处理不支持select），将所有sql都添加到批处理中（addBatch()），等待统一执行（executeBatch()），**它缓存了多个Statement对象，每个Statement对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC批处理相同**。

# 缓存

**mybatis提供一级缓存和二级缓存**

-   **mybatis一级缓存是一个SqlSession级别，sqlsession只能访问自己的一级缓存的数据**
-   **二级缓存是跨sqlSession，是mapper级别的缓存，对于mapper级别的缓存不同的sqlsession是可以共享的。**

### 一级缓存

**第一次发出一个查询sql，sql查询结果写入sqlsession的一级缓存中，缓存使用的数据结构是一个map**

-   key：hashcode+sql+sql输入参数+输出参数（sql的唯一标识）
-   value：用户信息

**同一个sqlsession再次发出相同的sql，就从缓存中取不走数据库。如果两次中间出现commit操作（修改、添加、删除），本sqlsession中的一级缓存区域全部清空，下次再去缓存中查询不到所以要从数据库查询，从数据库查询到再写入缓存。**

Mybatis一级缓存值得注意的地方：

-   **Mybatis默认就是支持一级缓存的，并不需要我们配置.**
-   mybatis和spring整合后进行mapper代理开发，不支持一级缓存，mybatis和spring整合，**spring按照mapper的模板去生成mapper代理对象，模板中在最后统一关闭sqlsession。**

### 二级缓存

**二级缓存的范围是mapper级别（mapper同一个命名空间），mapper以命名空间为单位创建缓存数据结构，结构是map。**

每次查询先看是否开启二级缓存，如果开启从二级缓存的数据结构中取缓存数据，如果从二级缓存中没有取到，再从一级缓存中找，如果一级缓存中也没有，从数据库查询

##### 配置：

```xml
   <!-- 全局配置参数 -->
    <settings>
        <!-- 开启二级缓存 -->
        <setting name="cacheEnabled" value="true"/>
    </settings>
	
	<cache/>
```

##### 注意

###### 查询结果映射的pojo序列化：

**二级缓存可以将内存的数据写到磁盘，存在对象的序列化和反序列化**，所以要实现java.io.serializable接口。
如果结果映射的pojo中还包括了pojo，都要实现java.io.serializable接口。

###### 禁用二级缓存

对于变化频率较高的sql，需要禁用二级缓存：

**在statement中设置useCache=false可以禁用当前select语句的二级缓存**，即每次查询都会发出sql去查询，**默认情况是true，**即该sql使用二级缓存



# 分页

Mybatis使用RowBounds对象进行分页，它是针对ResultSet结果集执行的内存分页，而非物理分页，可以在sql内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。

### 原理

分页插件的基本原理是使用Mybatis提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的sql，然后重写sql，根据dialect方言，添加对应的物理分页语句和物理分页参数。

# 动态SQL
> **以标签的形式编写动态sql，完成逻辑判断和动态拼接sql的功能**。

### 分类

trim|where|set|foreach|if|choose|when|otherwise|bind

### 执行原理

>   使用OGNL从sql参数对象中计算表达式的值，**根据表达式的值动态拼接sql，以此来完成动态sql的功能**

# 事务
> MyBatis事务是自动关机开启的，需要我们手动去提交事务





