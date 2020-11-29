# Spring介绍
## Spring是什么

## 解决了什么问题

# IOC
> 依赖注入，控制反转；Spring IOC 解决的是对象管理和对象依赖的问题。本来我们的对象都是new出来的，而我们如果使用Spring 则把对象交给「IOC容器」来管理。

IOC容器：

> 是一个「工厂」，我们把对象都交由这个「工厂」来管理，包括对象的创建和对象之间的依赖关系等等。等我们要用到对象的时候，就从这个「工厂」里边取出来。

控制反转IOC

> 本来是「由我们自己」new出来的对象，现在交给了IOC容器。把这个对象的「控制权」给「他方」了。「控制反转」更多的是一种思想或者说是设计模式，把原有由自己掌控的事交给「别人」来处理。

依赖注入DI

> 「依赖注入」更多指的是「控制反转」这个思想的实现方式：对象无需自行创建或管理它们的依赖关系，依赖关系将被「自动注入」到需要它们的对象当中去。

## IOC&ID

### 将对象交给IOC容器

Spring提供了四种方式：

* 注解
* XML
* JavaConfig
* 基于Groovy DSL配置

以XML配置+注解来装配Bean比较多，其中注解这种方式占大部分。

理解：本来我们的对象都是「由我们自己」new出来的，现在我们把这个对象的创建权限和对象之间的依赖关系交由「IOC容器」来管理。

Spring容器(Bean工厂)可简单分成两种：

-   BeanFactory

-   -   这是最基础、面向Spring的

-   ApplicationContext

-   -   这是在BeanFactory基础之上，面向使用Spring框架的开发者。提供了一系列的功能！

几乎所有的应用场合**都是**使用ApplicationContext！

### 告诉IOC容器bean之间的依赖关系（依赖注入）

依赖关系：日常开发中其实很多时候就是A对象里边有B对象的属性而已。

**方式：**

* 构造器注入
* **属性（setting()方法）注入**
* 工厂方法注入

## 好处

1.  不用自己组装，拿来就用。
2.  享受单例的好处，效率高，不浪费空间。
3.  便于单元测试，方便切换mock组件。
4.  便于进行AOP操作，对于使用者是透明的。
5.  统一配置，便于修改。

## 原理

IOC容器其实就是一个大工厂，它用来管理我们所有的对象以及依赖关系。

-   原理就是通过Java的**反射技术**来实现的！通过反射我们可以获取类的所有信息(成员变量、类名等等等)！
-   再通过配置文件(xml)或者注解来**描述**类与类之间的关系
-   我们就可以通过这些配置信息和反射技术来**构建**出对应的对象和依赖关系了！

Spring IOC容器是怎么实现对象的创建和依赖的：

1.  根据Bean配置信息在容器内部创建Bean定义注册表
2.  根据注册表加载、实例化bean、建立Bean与Bean之间的依赖关系
3.  将这些准备就绪的Bean放到Map缓存池中，等待应用程序调用

# Bean

##### 作用域

-   单例Singleton

-   多例prototype

-   与Web应用环境相关的Bean作用域（需要手动配置）

-   -   reqeust
    -   session

##### 生命周期

![](../images/bean的生命周期.jpg)

* Bean 容器找到配置文件中 Spring Bean 的定义。
* Bean 容器利用 Java Reflection API 创建一个Bean的实例。
* 如果涉及到一些属性值 利用 set()方法设置一些属性值。
* 如果 Bean 实现了 BeanNameAware 接口，调用 setBeanName()方法，传入Bean的名字。
* 如果 Bean 实现了 BeanClassLoaderAware 接口，调用 setBeanClassLoader()方法，传入 ClassLoader对象的实例。
* 与上面的类似，如果实现了其他 *.Aware接口，就调用相应的方法。
* 如果有和加载这个 Bean 的 Spring 容器相关的 BeanPostProcessor 对象，执行postProcessBeforeInitialization() 方法
* 如果Bean实现了InitializingBean接口，执行afterPropertiesSet()方法。
* 如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
* 如果有和加载这个 Bean的 Spring 容器相关的 BeanPostProcessor 对象，执行postProcessAfterInitialization() 方法
* 当要销毁 Bean 的时候，如果 Bean 实现了 DisposableBean 接口，执行 destroy() 方法。
* 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。

##### 实际应用
日常开发中，我们很多时候用@Component注解标识将对象放到「IOC容器」中，用@Autowired注解将对象注入

下面这张图就很好总结了以各种方式来对Bean的定义和注入。
![](../images/bean的定义和注入.jpg)

##### 多线程问题

>    Spring用的ThreadLocal

我们知道在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域。就是因为Spring对一些Bean（如RequestContextHolder、**TransactionSynchronizationManager**、LocaleContextHolder等）中非线程安全状态的“状态性对象”采用ThreadLocal封装，让它们也成为线程安全的“状态性对象”，因此，有状态的Bean就能够以singleton的方式在多线程中工作。

# AOP

### 是什么
AOP：Aspect Object Programming  「面向切面编程」
Spring AOP主要做的事情就是：「把重复的代码抽取，在运行的时候往业务方法上动态植入“切面类代码”」
**底层技术：动态代理**

Spring事务基于Spring AOP，Spring AOP底层用的动态代理，动态代理有两种方式：

* 基于接口代理(JDK代理)
    * 基于接口代理，凡是类的方法非public修饰，或者用了static关键字修饰，那这些方法都不能被Spring AOP增强
    * 推荐多例（JDK在创建代理对象时的性能要高于CGLib代理）
* 基于CGLib代理(子类代理)
    * 基于子类代理，凡是类的方法使用了private、static、final修饰，那这些方法都不能被Spring AOP增强
    * 推荐单例（JDK生成代理对象的运行性能却比CGLib的低。）



### 术语

**连接点**(Join point)：

-   **能够被拦截的地方**：Spring AOP是基于动态代理的，所以是方法拦截的。每个成员方法都可以称之为连接点~

**切点**(Poincut)：

-   **具体定位的连接点**：上面也说了，每个方法都可以称之为连接点，我们**具体定位到某一个方法就成为切点**。

**增强/通知**(Advice)：

-   表示添加到切点的一段**逻辑代码**，并定位连接点的**方位信息**。

-   -   简单来说就定义了是干什么的，具体是在哪干
    -   Spring AOP提供了5种Advice类型给我们：前置、后置、返回、异常、环绕给我们使用！

**织入**(Weaving)：

-   将`增强/通知`添加到目标类的具体连接点上的过程。

**引入/引介**(Introduction)：

-   `引入/引介`允许我们**向现有的类添加新方法或属性**。是一种**特殊**的增强！

**切面**(Aspect)：

-   切面由切点和`增强/通知`组成，它既包括了横切逻辑的定义、也包括了连接点的定义。

### JDK动态代理实现AOP

![img](https://mmbiz.qpic.cn/sz_mmbiz_jpg/2BGWl1qPxib1Id9lfLbDPG8Qbc5RVwMpMdO23WUe6GtVxEAJtlia4gOQpoZNZfTboK2PIO72P7lgP9OQ6pv4w0jw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### Spring AOP

![img](https://mmbiz.qpic.cn/sz_mmbiz_jpg/2BGWl1qPxib1Id9lfLbDPG8Qbc5RVwMpMByIzzXeoKUhxkKrotm58YallaD0uJS5XzaJNfcCUm7jzky3SicU5H4Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





