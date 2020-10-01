## 三大特性
### 封装

>   封装是指把一个对象的状态信息（也就是属性）隐藏在对象内部，不允许外部对象直接访问对象的内部信息。但是可以提供一些可以被外界访问的方法来操作属性。就好像我们看不到挂在墙上的空调的内部的零件信息（也就是属性），但是可以通过遥控器（方法）来控制空调。如果属性不想被外界访问，我们大可不必提供方法给外界访问。但是如果一个类没有提供给外界访问的方法，那么这个类也没有什么意义了。就好像如果没有空调遥控器，那么我们就无法操控空凋制冷，空调本身就没有意义了

### 继承

>   不同类型的对象，相互之间经常有一定数量的共同点。例如，小明同学、小红同学、小李同学，都共享学生的特性（班级、学号等）。同时，每一个对象还定义了额外的特性使得他们与众不同。例如小明的数学比较好，小红的性格惹人喜爱；小李的力气比较大。继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承，可以快速地创建新的类，可以提高代码的重用，程序的可维护性，节省大量创建新类的时间 ，提高我们的开发效率。

注意：

*   子类的修饰符应>=父类
*   子类的异常<=父类

### 多态

>   多态，顾名思义，表示一个对象具有多种的状态。具体表现为父类的引用指向子类的实例。

注意：

*   对象类型和引用类型之间具有继承（类）/实现（接口）的关系；

多态的必要条件：
1. 要有继承
2. 要有重写
3. 父类引用指向子类对象



## 枚举

在Java中，我们可以通过static final来定义常量。例如，我们希望定义周一到周日这7个常量，可以用7个不同的int表示：
``` java
public class Weekday {
    public static final int SUN = 0;
    public static final int MON = 1;
    public static final int TUE = 2;
    public static final int WED = 3;
    public static final int THU = 4;
    public static final int FRI = 5;
    public static final int SAT = 6;
}
```
使用：类名.常量名
使用这些常量来表示一组枚举值时，有一个严重的问题：
编译器无法检查每个值的合理性。

为了让编译器能自动检查某个值在枚举的集合内，并且，不同用途的枚举需要不同的类型来标记，不能混用，我们可以使用enum来定义枚举类：
``` java
public class Main {
    public static void main(String[] args) {
        Weekday day = Weekday.SUN;
        if (day == Weekday.SAT || day == Weekday.SUN) {
            System.out.println("Work at home!");
        } else {
            System.out.println("Work at office!");
        }
    }
}

enum Weekday {
    SUN, MON, TUE, WED, THU, FRI, SAT;
}
```
使用enum定义枚举有如下好处：
首先，enum常量本身带有类型信息，即Weekday.SUN类型是Weekday，编译器会自动检查出类型错误。例如，下面的语句不可能编译通过:
``` java
int day = 1;
if (day == Weekday.SUN) { // Compile error: bad operand types for binary operator '=='
}
```
其次，不可能引用到非枚举的值，因为无法通过编译。

最后，不同类型的枚举不能互相比较或者赋值，因为类型不符。例如，不能给一个Weekday枚举类型的变量赋值为Color枚举类型的值：
``` java 
Weekday x = Weekday.SUN; // ok!
Weekday y = Color.RED; // Compile error: incompatible types
```
这就使得编译器可以在编译期自动检查出所有可能的潜在错误。    

enum的特点：
* 定义的enum类型总是继承自java.lang.Enum，且无法被继承；
* 只能定义出enum的实例，而无法通过new操作符创建enum的实例；
* 定义的每个实例都是引用类型的唯一实例；
* 可以将enum类型用于switch语句。

### 我对enum的理解
做demo时，使用enum做自定义异常类里的属性，自定义enum属性{状态码, 提示信息}
## 注解
### 使用注解
#### 注解是什么？
> 注解是放在Java源码的类、方法、字段、参数前的一种特殊“注释”；注释会被编译器直接忽略，注解则可以被编译器打包进入class文件，因此，注解是一种用作标注的“元数据”。

#### 注解的分类
##### 1.由编译器使用的注解
* @Override：让编译器检查该方法是否正确地实现了覆写；
* @SuppressWarnings：告诉编译器忽略此处代码产生的警告。
这类注解不会被编译进入.class文件，它们在编译后就被编译器扔掉了。
##### 2.由工具处理.class文件使用的注解，
比如有些工具会在加载class的时候，对class做动态修改，实现一些特殊的功能。这类注解会被编译进入.class文件，但加载结束后并不会存在于内存中。这类注解只被一些底层库使用，一般我们不必自己处理。
##### 3.在程序运行期能够读取的注解
它们在加载后一直存在于JVM中，这也是最常用的注解。例如，一个配置了@PostConstruct的方法会在调用构造方法后自动被调用（这是Java代码读取该注解实现的功能，JVM并不会识别该注解）

#### 注解的参数
定义一个注解时，还可以定义配置参数。配置参数可以包括:
* 所有基本类型；
* String；
* 枚举类型；
* 基本类型、String、Class以及枚举的数组。

因为配置参数必须是常量，所以，上述限制保证了注解在定义时就已经确定了每个参数的值。
注解的配置参数可以有默认值，缺少某个配置参数时将使用默认值。
此外，大部分注解会有一个名为value的配置参数，对此参数赋值，可以只写常量，相当于省略了value参数。
如果只写注解，相当于全部使用默认值。

举个栗子:
``` java 
public class Hello {
    @Check(min=0, max=100, value=55)
    public int n;

    @Check(value=99)
    public int p;

    @Check(99) // @Check(value=99)
    public int x;

    @Check
    public int y;
}
```
@Check就是一个注解。第一个@Check(min=0, max=100, value=55)明确定义了三个参数，第二个@Check(value=99)只定义了一个value参数，它实际上和@Check(99)是完全一样的。最后一个@Check表示所有参数都使用默认值。

### 定义注解
#### 格式
Java语言使用@interface语法来定义注解（Annotation），它的格式如下：
``` java
public @interface Report {
    int type() default 0;
    String level() default "info";
    String value() default "";
}
```
注解的参数类似无参数方法，可以用default设定一个默认值（强烈推荐）。最常用的参数应当命名为value。
#### 元注解
有一些注解可以修饰其他注解，这些注解就称为元注解（meta annotation）。Java标准库已经定义了一些元注解，我们只需要使用元注解，通常不需要自己去编写元注解。
##### @Target
最常用的元注解是@Target。使用@Target可以定义Annotation能够被应用于源码的哪些位置：
* 类或接口：ElementType.TYPE；
* 字段：ElementType.FIELD；
* 方法：ElementType.METHOD；
* 构造方法：ElementType.CONSTRUCTOR；
* 方法参数：ElementType.PARAMETER。
##### @Retention
另一个重要的元注解@Retention定义了Annotation的生命周期：
* 仅编译期：RetentionPolicy.SOURCE；
* 仅class文件：RetentionPolicy.CLASS；
* 运行期：RetentionPolicy.RUNTIME。

如果@Retention不存在，则该Annotation默认为CLASS。因为通常我们自定义的Annotation都是RUNTIME，所以，务必要加上@Retention(RetentionPolicy.RUNTIME)这个元注解：
``` java 
@Retention(RetentionPolicy.RUNTIME)
public @interface Report {
    int type() default 0;
    String level() default "info";
    String value() default "";
}
```
##### @Repeatable
使用@Repeatable这个元注解可以定义Annotation是否可重复。这个注解应用不是特别广泛。
``` java
@Repeatable(Reports.class)
@Target(ElementType.TYPE)
public @interface Report {
    int type() default 0;
    String level() default "info";
    String value() default "";
}

@Target(ElementType.TYPE)
public @interface Reports {
    Report[] value();
}
```
经过@Repeatable修饰后，在某个类型声明处，就可以添加多个@Report注解：
``` java
@Report(type=1, level="debug")
@Report(type=2, level="warning")
public class Hello {
}
```
#### @Inherited
使用@Inherited定义子类是否可继承父类定义的Annotation。@Inherited仅针对@Target(ElementType.TYPE)类型的annotation有效，并且仅针对class的继承，对interface的继承无效：
``` java
@Inherited
@Target(ElementType.TYPE)
public @interface Report {
    int type() default 0;
    String level() default "info";
    String value() default "";
}
```
在使用的时候，如果一个类用到了@Report：
``` java
@Report(type=1)
public class Person {
}
```
则它的子类默认也定义了该注解：
``` java
public class Student extends Person {
}
```
#### 如何定义Annotation
第一步，用@interface定义注解
第二步，添加参数、默认值        
第三步，用元注解配置注解
``` java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Report {
    int type() default 0;
    String level() default "info";
    String value() default "";
}
```
其中，必须设置@Target和@Retention，@Retention一般设置为RUNTIME，因为我们自定义的注解通常要求在运行期读取。一般情况下，不必写@Inherited和@Repeatable。
### 处理注解
Java的注解本身对代码逻辑没有任何影响。根据@Retention的配置：
* SOURCE类型的注解在编译期就被丢掉了；
* CLASS类型的注解仅保存在class文件中，它们不会被加载进JVM；
* RUNTIME类型的注解会被加载进JVM，并且在运行期可以被程序读取。

如何使用注解完全由工具决定：
* SOURCE类型的注解主要由编译器使用，因此我们一般只使用，不编写。
* CLASS类型的注解主要由底层工具库使用，涉及到class的加载，一般我们很少用到。
* 只有RUNTIME类型的注解不但要使用，还经常需要编写。

因此，我们只讨论如何读取RUNTIME类型的注解。

因为注解定义后也是一种class，所有的注解都继承自java.lang.annotation.Annotation，因此，读取注解，需要使用反射API。

Java提供的使用反射API读取Annotation的方法包括：
判断某个注解是否存在于Class、Field、Method或Constructor：
* Class.isAnnotationPresent(Class)
* Field.isAnnotationPresent(Class)
* Method.isAnnotationPresent(Class)
* Constructor.isAnnotationPresent(Class)

``` java
// 判断@Report是否存在于Person类:
Person.class.isAnnotationPresent(Report.class);
```

使用反射API读取Annotation：
Class.getAnnotation(Class)
Field.getAnnotation(Class)
Method.getAnnotation(Class)
Constructor.getAnnotation(Class)

举个栗子：
``` java
// 获取Person定义的@Report注解:
Report report = Person.class.getAnnotation(Report.class);
int type = report.type();
String level = report.level();
```
使用反射API读取Annotation有两种方法。方法一是先判断Annotation是否存在，如果存在，就直接读取：
``` java
Class cls = Person.class;
if (cls.isAnnotationPresent(Report.class)) {
    Report report = cls.getAnnotation(Report.class);
    ...
}
```

第二种方法是直接读取Annotation，如果Annotation不存在，将返回null：
``` java
Class cls = Person.class;
Report report = cls.getAnnotation(Report.class);
if (report != null) {
   ...
}
```
读取方法、字段和构造方法的Annotation和Class类似。但要读取方法参数的Annotation就比较麻烦一点，因为方法参数本身可以看成一个数组，而每个参数又可以定义多个注解，所以，一次获取方法参数的所有注解就必须用一个二维数组来表示。例如，对于以下方法定义的注解：
``` java
public void hello(@NotNull @Range(max=5) String name, @NotNull String prefix) {
}
```

要读取方法参数的注解，我们先用反射获取Method实例，然后读取方法参数的所有注解：
``` java
// 获取Method实例:
Method m = ...
// 获取所有参数的Annotation:
Annotation[][] annos = m.getParameterAnnotations();
// 第一个参数（索引为0）的所有Annotation:
Annotation[] annosOfName = annos[0];
for (Annotation anno : annosOfName) {
    if (anno instanceof Range) { // @Range注解
        Range r = (Range) anno;
    }
    if (anno instanceof NotNull) { // @NotNull注解
        NotNull n = (NotNull) anno;
    }
}
```
#### 使用注解
注解如何使用，完全由程序自己决定。例如，JUnit是一个测试框架，它会自动运行所有标记为@Test的方法。

我们来看一个@Range注解，我们希望用它来定义一个String字段的规则：字段长度满足@Range的参数定义：
``` java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Range {
    int min() default 0;
    int max() default 255;
}
```
在某个JavaBean中，我们可以使用该注解：
``` java
public class Person {
    @Range(min=1, max=20)
    public String name;

    @Range(max=10)
    public String city;
}
```

定义了注解，本身对程序逻辑没有任何影响。我们必须自己编写代码来使用注解。这里，我们编写一个Person实例的检查方法，它可以检查Person实例的String字段长度是否满足@Range的定义：
``` java
void check(Person person) throws IllegalArgumentException, ReflectiveOperationException {
    // 遍历所有Field:
    for (Field field : person.getClass().getFields()) {
        // 获取Field定义的@Range:
        Range range = field.getAnnotation(Range.class);
        // 如果@Range存在:
        if (range != null) {
            // 获取Field的值:
            Object value = field.get(person);
            // 如果值是String:
            if (value instanceof String) {
                String s = (String) value;
                // 判断值是否满足@Range的min/max:
                if (s.length() < range.min() || s.length() > range.max()) {
                    throw new IllegalArgumentException("Invalid field: " + field.getName());
                }
            }
        }
    }
}
```
## 泛型

## 异常

Java异常层次结构图

![](..\images\异常体系.png)

### Error（错误）

**是程序无法处理的错误**，表示运行应用程序中较严重问题。大多数错误与代码编写者执行的操作无关，而表示代码运行时 JVM（Java 虚拟机）出现的问题。例如，Java 虚拟机运行错误（Virtual MachineError），当 JVM 不再有继续执行操作所需的内存资源时，将出现 OutOfMemoryError。这些异常发生时，Java 虚拟机（JVM）一般会选择线程终止。

### Execptions（异常）

**是程序本身可以处理的异常**。Exception 类有一个重要的子类 **RuntimeException**。RuntimeException 异常由 Java 虚拟机抛出。**NullPointerException**（要访问的变量没有引用任何对象时，抛出该异常）、**ArithmeticException**（算术运算异常，一个整数除以 0 时，抛出该异常）和 **ArrayIndexOutOfBoundsException** （下标越界异常）。

### 区别

>    **异常能被程序本身处理，错误是无法处理。**

### try-catch-finally

-   **try 块：** 用于捕获异常。其后可接零个或多个 catch 块，如果没有 catch 块，则必须跟一个 finally 块。
-   **catch 块：** 用于处理 try 捕获到的异常。
-   **finally 块：** 无论是否捕获或处理异常，finally 块里的语句都会被执行。当在 try 块或 catch 块中遇到 return 语句时，finally 语句块将在方法返回之前被执行。

### 拓展

#### try-with-resources(JDK1.7)

>   面对必须要关闭的资源，我们总是应该优先使用try-with-resources而不是`try-finally`。随之产生的代码更简短，更清晰，产生的异常对我们也更有用。`try-with-resources`语句让我们更容易编写必须要关闭的资源的代码，若采用`try-finally`则几乎做不到这点。

Java 中类似于`InputStream`、`OutputStream` 、`Scanner` 、`PrintWriter`等的资源都需要我们调用`close()`方法来手动关闭，一般情况下我们都是通过`try-catch-finally`语句来实现这个需求

**多个资源需要关闭的时候，通过使用分号分隔**

### 面试题

当 try 语句和 finally 语句中都有 return 语句时，在方法返回之前，finally 语句的内容将被执行，并且 finally 语句的返回值将会覆盖原始的返回值。

## IO

### 分类

-   按照流的流向分，可以分为输入流和输出流；
-   按照操作单元划分，可以划分为字节流和字符流；
-   按照流的角色划分为节点流和处理流。

Java Io 流共涉及 40 多个类，这些类看上去很杂乱，但实际上很有规则，而且彼此之间存在非常紧密的联系， Java I0 流的 40 多个类都是从如下 4 个抽象类基类中派生出来的。

-   InputStream/Reader: 所有的输入流的基类，前者是字节输入流，后者是字符输入流。
-   OutputStream/Writer: 所有输出流的基类，前者是字节输出流，后者是字符输出流。

按操作方式分类结构图：

![](../images/IO结构.png)

按操作对象分类结构图：

![](../images/IO结构2.png)

### 拓展

#### BIO,NIO,AIO

-   **BIO (Blocking I/O):** 同步阻塞 I/O 模式，数据的读取写入必须阻塞在一个线程内等待其完成。在活动连接数不是特别高（小于单机 1000）的情况下，这种模型是比较不错的，可以让每一个连接专注于自己的 I/O 并且编程模型简单，也不用过多考虑系统的过载、限流等问题。线程池本身就是一个天然的漏斗，可以缓冲一些系统处理不了的连接或请求。但是，当面对十万甚至百万级连接的时候，传统的 BIO 模型是无能为力的。因此，我们需要一种更高效的 I/O 处理模型来应对更高的并发量。
-   **NIO (Non-blocking/New I/O):** NIO 是一种同步非阻塞的 I/O 模型，在 Java 1.4 中引入了 NIO 框架，对应 java.nio 包，提供了 Channel , Selector，Buffer 等抽象。NIO 中的 N 可以理解为 Non-blocking，不单纯是 New。它支持面向缓冲的，基于通道的 I/O 操作方法。 NIO 提供了与传统 BIO 模型中的 `Socket` 和 `ServerSocket` 相对应的 `SocketChannel` 和 `ServerSocketChannel` 两种不同的套接字通道实现,两种通道都支持阻塞和非阻塞两种模式。阻塞模式使用就像传统中的支持一样，比较简单，但是性能和可靠性都不好；非阻塞模式正好与之相反。对于低负载、低并发的应用程序，可以使用同步阻塞 I/O 来提升开发速率和更好的维护性；对于高负载、高并发的（网络）应用，应使用 NIO 的非阻塞模式来开发
-   **AIO (Asynchronous I/O):** AIO 也就是 NIO 2。在 Java 7 中引入了 NIO 的改进版 NIO 2,它是异步非阻塞的 IO 模型。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。AIO 是异步 IO 的缩写，虽然 NIO 在网络操作中，提供了非阻塞的方法，但是 NIO 的 IO 行为还是同步的。对于 NIO 来说，我们的业务线程是在 IO 操作准备好时，得到通知，接着就由这个线程自行进行 IO 操作，IO 操作本身是同步的。查阅网上相关资料，我发现就目前来说 AIO 的应用还不是很广泛，Netty 之前也尝试使用过 AIO，不过又放弃了。

### 面试题

#### 既然有了字节流,为什么还要有字符流?

问题本质想问：**不管是文件读写还是网络发送接收，信息的最小存储单元都是字节，那为什么 I/O 流操作要分为字节流操作和字符流操作呢？**

回答：字符流是由 Java 虚拟机将字节转换得到的，问题就出在这个过程还算是非常耗时，并且，如果我们不知道编码类型就很容易出现乱码问题。所以， I/O 流就干脆提供了一个直接操作字符的接口，方便我们平时对字符进行流操作。如果音频文件、图片等媒体文件用字节流比较好，如果涉及到字符的话使用字符流比较好。

## 类的加载

## 反射

### 为什么需要反射

1.  提高程序的灵活性
2.  屏蔽掉实现的细节，让使用者更加方便好用

### API

#### 获取Class对象的几种途径

1. Object类的getClass()方法
2. 数据类型的静态属性class
3. Class类中的静态方法：public static Class ForName(String className)

#### 通过Class对象创建出对象，获取出构造器，成员变量，方法

获取成员变量并使用
1. 获取Class对象
2. 通过Class对象获取Constructor对象
3. Object obj = Constructor.newInstance()创建对象
4. Field field = Class.getField("指定变量名")获取单个成员变量对象
5. field.set(obj,"") 为obj对象的field字段赋值
如果需要访问私有或者默认修饰的成员变量
1. Class.getDeclaredField()获取该成员变量对象
2. setAccessible() 暴力访问 

#### 通过反射的API修改成员变量的值，调用方法

通过反射调用成员方法
1. 获取Class对象
2. 通过Class对象获取Constructor对象
3. Constructor.newInstance()创建对象
4. 通过Class对象获取Method对象 ------getMethod("方法名");
5. Method对象调用invoke方法实现功能
如果调用的是私有方法那么需要暴力访问
1. getDeclaredMethod()
2. setAccessiable();   

## JDK1.8新特性

### 函数式编程

### stream

## 锁



>   -   从线程是否需要对资源加锁可以分为 `悲观锁` 和 `乐观锁`
>   -   从资源已被锁定，线程是否阻塞可以分为 `自旋锁`
>   -   从多个线程并发访问资源，也就是 Synchronized 可以分为 `无锁`、`偏向锁`、 `轻量级锁` 和 `重量级锁`
>   -   从锁的公平性进行区分，可以分为`公平锁` 和 `非公平锁`
>   -   从根据锁是否重复获取可以分为 `可重入锁` 和 `不可重入锁`
>   -   从那个多个线程能否获取同一把锁分为 `共享锁` 和 `排他锁`

### 线程是否需要对资源加锁

>   Java 按照是否对资源加锁分为`乐观锁`和`悲观锁`，乐观锁和悲观锁并不是一种真实存在的锁，而是一种设计思想，乐观锁和悲观锁对于理解 Java 多线程和数据库来说至关重要，下面就来探讨一下这两种实现方式的区别和优缺点

####  悲观锁

`悲观锁`是一种悲观思想，它总认为最坏的情况可能会出现，它认为数据很可能会被其他人所修改，所以悲观锁在持有数据的时候总会把`资源` 或者 `数据` 锁住，这样其他线程想要请求这个资源的时候就会阻塞，直到等到悲观锁把资源释放为止。传统的关系型数据库里边就用到了很多这种锁机制，**比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。**悲观锁的实现往往依靠数据库本身的锁功能实现。

Java 中的 `Synchronized` 和 `ReentrantLock` 等独占锁(排他锁)也是一种悲观锁思想的实现，因为 Synchronzied 和 ReetrantLock 不管是否持有资源，它都会尝试去加锁，生怕自己心爱的宝贝被别人拿走。

#### 乐观锁

乐观锁的思想与悲观锁的思想相反，它总认为资源和数据不会被别人所修改，所以读取不会上锁，但是乐观锁在进行写入操作的时候会判断当前数据是否被修改过(具体如何判断我们下面再说)。乐观锁的实现方案一般来说有两种：`版本号机制` 和 `CAS实现` 。乐观锁多适用于多读的应用类型，这样可以提高吞吐量。

在Java中`java.util.concurrent.atomic`包下面的原子变量类就是使用了乐观锁的一种实现方式 CAS 实现的。
* CAS

CAS 中涉及三个要素：

1.  需要读写的内存值 V
2.  进行比较的值 A
3.  拟写入的新值 B

>   当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。

* 版本号机制

版本号机制是在数据表中加上一个 `version` 字段来实现的，表示数据被修改的次数，当执行写操作并且写入成功后，version = version + 1，当线程A要更新数据时，在读取数据的同时也会读取 version 值，在提交更新时，若刚才读取到的 version 值为当前数据库中的version值相等时才更新，否则重试更新操作，直到更新成功。

##### 问题
* ABA问题
> 如果一个变量第一次读取的值是 A，准备好需要对 A 进行写操作的时候，发现值还是 A，那么这种情况下，能认为 A 的值没有被改变过吗？可以是由 A -> B -> A 的这种情况，但是 AtomicInteger 却不会这么认为，它只相信它看到的，它看到的是什么就是什么。
* 循环开销大
> 乐观锁在进行写操作的时候会判断是否能够写入成功，如果写入不成功将触发等待 -> 重试机制，这种情况是一个自旋锁，简单来说就是适用于短期内获取不到，进行等待重试的锁，它不适用于长期获取不到锁的情况，另外，自旋循环对于性能开销比较大。

#### 两种锁的使用场景

一般来说，悲观锁不仅会对写操作加锁还会对读操作加锁

### 资源已被锁定，线程是否阻塞

### 多个线程并发访问资源

### 锁的公平性

### 锁是否重复获取

### 多个线程能否获取同一把锁

# 常见的问题
## i++和++i
	 i++ 先赋值在运算,例如 a=i++,先赋值a=i,后运算i=i+1,所以结果是a==1
	 ++i 先运算在赋值,例如 a=++i,先运算i=i+1,后赋值a=i,所以结果是a==2
## 移位运算>>和<<
     >> 右移，除以2
     << 左移，乘以2