# 事务

## 原理
> Spring事务的本质就是数据库对事务的支持，使用Spring管理事务后，coding时可以省略对事务的操作，通过AOP将操作在代理类中加上，关键切片类->TransactionAspectSupport

## 什么时候回滚

>   所拦截的方法有指定异常抛出，事务自动回滚；默认情况下，事务只对Error与RuntimeException及其子类有效，如果一般的Exception想回滚，需要配置rollbackFor=Exception.Class

## 什么时候失效

##### 发生自调用

*   同一个Service中，未被事务管理的函数调用被事务管理的函数，本质是本类this调用而不是代理类，无法通过AOP增强

##### 方法修饰符不是public

*   private不符合逻辑，被事务管理的函数不被调用毫无意义
*   其他修饰符不会被事务管理

##### 发生了错误的异常

*   被捕获的异常
*   非RuntimeExecption未加上rollbackFor

##### 数据库不支持事务

>   Spring事务用的是数据库的事务，如果Spring不支持事务，Spring事务肯定不会生效。

## 事务隔离

>   Spring事务隔离在MySQL事务隔离的基础上抽象了一种默认隔离级别，表示数据库默认的隔离级别

*   MySQL配置的隔离级别与Spring配置的隔离级别冲突时，以Spring配置为准
*   Spring配置的隔离级别MySQL不支持时，以MySQL为准

## 事务传播机制

> 在当前含有事务方法内部调用其他的方法(无论该方法是否含有事务)

##### PROPAGATION_REQUIRED

>   支持当前事务，如果当前没有事务，就新建一个事务

比如说，ServiceB.methodB的事务级别定义为PROPAGATION_REQUIRED, 那么由于执行ServiceA.methodA的时候，
ServiceA.methodA已经起了事务，这时调用ServiceB.methodB，ServiceB.methodB看到自己已经运行在ServiceA.methodA
的事务内部，就不再起新的事务。而假如ServiceA.methodA运行的时候发现自己没有在事务中，他就会为自己分配一个事务。
这样，在ServiceA.methodA或者在ServiceB.methodB内的任何地方出现异常，事务都会被回滚。即使ServiceB.methodB的事务已经被
提交，但是ServiceA.methodA在接下来fail要回滚，ServiceB.methodB也要回滚



##### PROPAGATION_REQUIRES_NEW

>   新建事务，如果当前存在事务，把当前事务挂起，执行当前新建事务完成以后，上下文事务恢复再执行。







## 可能出现的情况
### 第一种
在Service层抛出Exception，在Controller层捕获，那如果在Service中有异常，那会事务回滚吗？

如果是编译时异常不会自动回滚，如果是运行时异常，那会自动回滚！
### 第二种
带有@Transactional注解所包围的方法就能被Spring事务管理起来，那如果我在当前类下使用一个没有事务的方法去调用一个有事务的方法，那我们这次调用会怎么样？是否会有事务呢？

![Spring会自动生成代理对象](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3icByoHrJW0micXhLLaMqUZ6VDo6CZg2cfBDIIv0ic6xDRT8ulbM6zjbE1UsmHU101LyZHFVEY3dHUQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)Spring会自动生成代理对象

显然地，我们拿到的是代理(Proxy)对象，调用`addEmployee2Controller()`方法，而`addEmployee2Controller()`方法的逻辑是`target.addEmployee()`，调用回原始对象(target)的`addEmployee()`。所以这次的调用**压根就没有事务存在**，更谈不上说Spring事务传播机制了。

