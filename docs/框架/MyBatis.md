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
# 分页

# 动态SQL
> 自动拼接SQL语句

# 事务
> MyBatis事务是自动关机开启的，需要我们手动去提交事务

# 缓存

# 映射

